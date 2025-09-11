Of course. Here is the detailed documentation for Phases 2 and 3, including the integration of the Asynchronous (Token-Based) communication model, which is the critical foundation for all scaling efforts.

---

# evalX Optimization Plan: Phase 2 - Core Optimizations

## Overview
Phase 2 focuses on replacing core components to achieve **Judge0-level performance and beyond**. The goal is to reduce latency to **<2s for 1000 requests × 20 test cases** and lay the groundwork for horizontal scaling. This phase introduces the **Isolate sandbox** for near-instant execution and a **proper distributed task queue** to replace the Redis lists from Phase 1.

**Key Goals for Phase 2:**
- Achieve sub-100ms code execution via Isolate sandboxes.
- Implement a robust, distributed task queue for fault-tolerant processing.
- Introduce the Asynchronous (Token-Based) API model.
- Maintain full backward compatibility with Docker sandboxes as a fallback.

## Phase 2 Changes Summary

### 1. Integrate Isolate Sandboxes (Primary Execution Engine)
**Current State (Post-Phase 1):** Reliance on Docker containers, which incur ~100-500ms startup overhead per execution.

**Changes Needed:**
- **Install Isolate:** Clone, build, and install the `isolate` binary system-wide.
- **Create Sandbox Module:** Create new directory `src/sandbox/` with:
  - `mod.rs`: Module declaration.
  - `isolate.rs`: A Rust struct (`IsolateSandbox`) with methods `compile()` and `run()`. This struct will use `std::process::Command` to call the `isolate` binary with appropriate flags (e.g., `--init`, `--run`, `--time`, `--mem`).
- **Modify Executor:** In `src/executor.rs`, add a feature flag (e.g., `USE_ISOLATE` from `.env`). When enabled, route compilation and execution calls to the new `IsolateSandbox` instead of the Docker-based `container_management` module.
- **Implement Fallback:** If Isolate fails (e.g., unsupported language, permission error), automatically fall back to the existing Docker system.
- **Security Hardening:** Configure Isolate with strict resource limits (time, memory, processes) and use `--enable-seccomp` for syscall filtering.

**Benefits:**
- **10x-50x Faster Execution:** Reduces sandbox initialization from ~100ms to **<1ms**.
- **Higher Density:** Run 1000+ concurrent Isolate sandboxes on a single machine vs. ~100 Docker containers.
- **Enhanced Security:** Fine-grained control over resource limits and syscall access.

**Files to Modify/Create:**
- **New:** `src/sandbox/mod.rs`
- **New:** `src/sandbox/isolate.rs`
- `src/executor.rs` (Major changes to integrate Isolate as primary)
- `src/compilers/compilers.rs` (Route compilation to Isolate)
- `.env` (Add `USE_ISOLATE=true`, `ISOLATE_PATH=/usr/local/bin/isolate`)

### 2. Implement Asynchronous (Token-Based) API
**Current State:** Synchronous API where the HTTP connection is held open until the execution is complete. This is fragile and does not scale.

**Changes Needed:**
This is a fundamental shift in API design. The new flow is:
1.  **POST /submissions**:
    - Client sends a batch of requests (code + array of test cases).
    - Server **immediately** validates the request.
    - Generates a unique `token` (UUIDv4).
    - Serializes the entire job and enqueues it into the task queue.
    - **Immediately** responds with HTTP `202 Accepted` and a JSON body: `{ "token": "<uuid>", "status": "In Queue" }`.
    - The HTTP connection closes, freeing the web server to handle the next request.
2.  **Worker Process**:
    - Workers pull jobs from the queue.
    - They process the job (compile once, run all test cases in parallel via Isolate).
    - They store the final `EvaluationResult` in **Redis** under the key `result:<token>`, with a TTL of e.g., 5 minutes.
3.  **GET /submissions/<token>**:
    - The client polls this endpoint to check for the result.
    - The server checks Redis for `result:<token>`.
    - If found: Returns the result with HTTP `200 OK`.
    - If not found: Returns `{ "status": "Processing" }` with HTTP `202 Accepted`.

**Benefits:**
- **Massive Scalability:** The web server only handles short-lived requests, enabling it to process 10,000+ requests per minute.
- **Resilience:** Jobs are persisted in the queue and result store. Server crashes or restarts do not cause job loss.
- **Predictable Performance:** Clients get an immediate acknowledgment and can poll at their own rate.
- **Industry Standard:** This is how all major code execution APIs (Judge0, AWS Lambda) operate.

**Files to Modify:**
- `src/controllers/executionControllers.rs` (Complete rewrite of endpoint logic)
- `src/models/request.rs` (Add `webhook_url` field to request model, optional)
- `src/models/response.rs` (Add `token` and `status` fields to response model)
- `src/routes.rs` (Add new `/submissions` and `/submissions/{token}` routes)
- `src/queue_management/task.rs` (Update task struct to hold entire batch)

### 3. Upgrade to a Robust Task Queue (Broccoli)
**Current State (Post-Phase 1):** Using Redis lists directly, which lacks features like retries, priorities, and durability.

**Changes Needed:**
- **Add Dependency:** Add `broccoli = "0.5"` to `Cargo.toml`. Broccoli is a Rust-native, Redis-backed distributed task queue.
- **Refactor Queue Management:** Replace the direct Redis list operations in `src/queue_management/manager.rs` with Broccoli's client.
- **Define Jobs:** Define a Broccoli job (e.g., `execute_batch_job`) that takes the serialized batch data and token.
- **Worker Service:** Refactor the worker startup in `src/setup.rs` to run Broccoli workers that consume the `execute_batch_job` queue.
- **Retries & DLQ:** Configure Broccoli to automatically retry failed jobs and move permanently failed jobs to a Dead Letter Queue for inspection.

**Benefits:**
- **Production-Ready:** Built-in support for retries, metrics, and job management.
- **Performance:** More efficient than manually managing Redis lists.
- **Reliability:** Guaranteed at-least-once delivery of tasks.

**Files to Modify:**
- `Cargo.toml` (Add Broccoli dependency)
- `src/queue_management/manager.rs` (Complete refactor to use Broccoli client)
- `src/setup.rs` (Initialize Broccoli and start workers)
- `src/main.rs` (Potentially split into `api` and `worker` binaries)
