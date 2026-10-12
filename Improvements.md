## EvalX: Production Hardening & Architectural Fixes

### 1. Distributed Idempotency & The Race Condition

**The Flaw:** SQS "at-least-once" delivery causes duplicate executions. The current state overwrite (`conn.set(status_key, "processing")`) allows multiple workers to process the same payload simultaneously.
**The Fix (Small Code Change):**

* **The Lock:** Replace the blind status update with a conditional Redis lock: `SET lock:job_<id> <worker_uuid> NX PX 35000`.
* **The Check:** If `SETNX` returns 0, immediately drop the message and ACK SQS.
* **The Release:** Use a Lua script to verify the `<worker_uuid>` matches before executing `DEL` on completion, preventing workers from deleting each other's locks during GC pauses.

### 2. I/O Decoupling & Storage Resilience

**The Flaw:** Transient Redis network failures at the end of a successful Isolate compute cause the worker to bubble up an error, fail to ACK SQS, and force a complete 10-second CPU re-run of the exact same code.
**The Fix (Medium Code Change):**

* **Retry Loop:** Wrap the final `conn.hset` Redis write in an exponential backoff loop (e.g., 50ms, 100ms, 200ms) to survive connection pool exhaustion.
* **SQS Heartbeat:** If the Redis retry loop stalls, issue a `ChangeMessageVisibility` API call to SQS to extend the timeout and protect the compute result.
* **Spooling:** Never throw away finished compute. If Redis is completely dead, serialize the `EvaluationResult` to a local disk spool and ACK SQS immediately.

### 3. Hardware Affinity & Concurrency (Dev & Prod)

**The Flaw:** The Rust parent thread is hard-pinned to a single physical core (`core_affinity`), forcing 36 Tokio workers and 36 Isolate sandboxes to cooperatively choke on one CPU, leaving the rest of the host idle.
**The Fix (Small Code Change):**

* **Unleash Tokio (`src/setup.rs`):** Remove `core_affinity::set_for_current`. Change the runtime builder from `new_current_thread()` to `new_multi_thread().worker_threads(total_cores - 2)`. Let the Rust OS threads float.
* **Pin the Sandboxes (`src/sandbox/isolate.rs`):** Enforce hardware isolation at the child process level. Dynamically inject the `--cg-cpus=<core_id>` flag into your `Command::new("isolate")` builder so the Linux cgroup assigns each untrusted sandbox to its own distinct physical core.
