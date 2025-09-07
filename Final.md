# README.md: Building evalX on Ubuntu 22.04 LTS to Outperform Judge0

Welcome to the **evalX** project! This README provides a complete, step-by-step guide to set up and develop your Rust-based online code evaluation system on Ubuntu 22.04 LTS (dual-boot with Windows). The goal is to optimize for faster compilation, handle 1000+ requests (each with 20 test cases) in ~2s, and scale to 10000 requests without latency spikes, outperforming Judge0 in speed, scalability, and features like AI-powered caching and webhooks.

This plan is **fully revised** based on your current tech stack (Rust with Axum, Bollard for Docker, Tokio async, Redis for caching/queue, custom in-memory queue, Docker sandboxes, and dependencies like anyhow, serde, uuid, etc.). We won't change the core stack or implementation methods—instead, we'll enhance it incrementally: switch to lighter Isolate sandboxes (while keeping Docker as fallback), optimize Redis for queuing/caching, add dynamic pre-warming, batch handling for 20 test cases, and local orchestration with Docker Compose/Minikube. All development is cost-free locally (no AWS until production testing).

**Key Principles**:
- **No Code Snippets Here**: Follow steps by editing files in VS Code (or terminal with nano/vim for quick fixes). Use your existing project files as reference.
- **Phased Approach**: Divided into Setup (Ubuntu + Tools), Phase 1 (Quick Wins for immediate speed), Phase 2 (Core Optimizations for Judge0-level performance), Phase 3 (Advanced Scaling & Features to outperform Judge0).
- **Testing Focus**: After each step, test with tools like `wrk` (load testing) and `redis-cli` (queue monitoring) to verify ~2s latency for batches.
- **Why This Outperforms Judge0**: Isolate for <10ms sandbox starts (vs. Docker's 100ms+), Rust-native queues (lighter than Celery), pre-warmed pools for 1000+ concurrent execs, and extras like webhooks/AI hints (Judge0 lacks native Rust efficiency and advanced caching).
- **Timeline**: 2 weeks total (1-2 days Phase 1, 3-5 days Phase 2, 1 week Phase 3). Assume basic Rust/Docker knowledge; test small (1 language first, e.g., Python).
- **Current Date Assumption**: September 07, 2025—use latest stable tools (e.g., Rust 1.80+, Docker 27+).

**Project Structure Overview** (Your Current + Additions):
- Keep existing: `src/` (caching, compilers, container_management, controllers, models, queue_management, types), `Docker/` (language images), `.env`, `Cargo.toml`, `build-images.ps1` (convert to `.sh`).
- Additions: `src/sandbox/` (Isolate wrapper), `src/monitoring/` (Prometheus metrics), `docker-compose.yml` (local orchestration), `minikube-config.yaml` (local K8s testing, optional).
- No major restructures—enhance existing modules (e.g., update `executor.rs` for Isolate fallback).

**Prerequisites**:
- Windows machine with 16GB+ RAM, 100GB+ free space.
- Backup your project files (e.g., to GitHub or USB).
- Internet for downloads.

---

## Section 1: Initial Setup - Install Ubuntu 22.04 LTS (Dual-Boot)

This phase sets up Ubuntu alongside Windows for native Linux performance (essential for Docker/Isolate speed; avoids WSL2/VM overhead). Dual-boot minimizes disruption—boot into Ubuntu for dev, Windows for other tasks.

### Step 1.1: Backup and Prepare Windows
- **What to Do**: Back up all evalX files (e.g., `src/`, `Docker/`, `Cargo.toml`) and important data to an external drive, GitHub repo, or cloud storage. This prevents data loss during partitioning.
- **How to Do It**:
  1. Copy your entire project folder to a USB drive or push to GitHub (`git add .; git commit -m "Backup before Ubuntu"; git push`).
  2. Verify the backup by cloning/pulling it on another machine or folder.
- **What Changes/Why**: No changes to project files yet—this ensures safe migration. Ubuntu's native environment will make Docker builds 20-50% faster than Windows/WSL2, critical for testing 1000+ requests.
- **What to Include**: Include all files from your `Files.txt` or project root (e.g., `.env`, PowerShell scripts). Exclude large Docker images (rebuild later).
- **Time**: 30 minutes. **Test**: Confirm backup by opening a Rust file (e.g., `main.rs`) in a text editor.

### Step 1.2: Download and Create Bootable USB
- **What to Do**: Download the Ubuntu ISO and make a bootable USB drive.
- **How to Do It**:
  1. Go to ubuntu.com/download/desktop, select Ubuntu 22.04 LTS (or 24.04 if available in 2025 for better hardware support).
  2. Download the ISO file (~4GB) to your Downloads folder.
  3. Download Rufus (free tool) from rufus.ie.
  4. Insert a USB drive (8GB+), open Rufus, select the ISO, choose the USB, and click Start (use DD Image mode for safety).
- **What Changes/Why**: Prepares for installation. Ubuntu 22.04 is LTS (long-term support) for stability with your stack (Rust, Docker 20.10+ compatible).
- **What to Include**: Only the ISO—no project files here. Eject USB safely after creation.
- **Time**: 20 minutes. **Test**: Restart PC, enter BIOS (F2/Del/Esc), ensure USB boot option is visible (but don't boot yet).

### Step 1.3: Partition Disk and Install Ubuntu
- **What to Do**: Shrink Windows partition and install Ubuntu in the free space.
- **How to Do It**:
  1. In Windows, right-click Start > Disk Management. Select C: drive > Shrink Volume > Enter 100GB+ (for Ubuntu root + swap).
  2. Leave the new unallocated space untouched.
  3. Restart, enter BIOS, set USB as first boot device, save/exit.
  4. Boot from USB: Select "Try Ubuntu" first to test, then "Install Ubuntu."
  5. In installer: Choose language (English), keyboard, updates/third-party software (enable for Docker/WiFi).
  6. Installation type: "Install Ubuntu alongside Windows Boot Manager" (auto-detects partitions).
  7. Allocate ~90GB to root (/), 4-8GB swap (if RAM <16GB), rest to /home. Enable LVM for flexibility.
  8. Create user/password (e.g., username: evalx-dev), set location/timezone.
  9. Let it install (~10-20 minutes), remove USB, reboot.
  10. At GRUB menu, select Ubuntu (Windows option remains for dual-boot).
- **What Changes/Why**: Creates dedicated space for evalX (Docker images can take 10GB+). Dual-boot via GRUB allows seamless switching. Native Ubuntu eliminates Windows I/O latency for Redis/Docker, enabling true ~2s benchmarks.
- **What to Include**: During install, enable full-disk encryption if security matters (uses your password). No project files yet—transfer later.
- **Time**: 1 hour. **Test**: Boot into Ubuntu desktop. Run `uname -a` in terminal (Ctrl+Alt+T) to confirm "Linux" and Ubuntu version. Reboot to GRUB, select Windows to verify dual-boot.

### Step 1.4: Post-Install Updates
- **What to Do**: Update system and install basics.
- **How to Do It**:
  1. Open terminal (Ctrl+Alt+T).
  2. Run `sudo apt update && sudo apt upgrade -y` (enter password when prompted).
  3. Reboot if kernel updated: `sudo reboot`.
- **What Changes/Why**: Ensures latest security/patches for tools like Docker (avoids compatibility issues with Rust 1.80+).
- **What to Include**: Nothing project-specific yet.
- **Time**: 15 minutes. **Test**: Run `lsb_release -a` to confirm Ubuntu 22.04.

---

## Section 2: Development Environment Setup

Set up VS Code as primary editor (familiar from Windows) + terminal for Linux tasks. This enables efficient editing of Rust files, Dockerfiles, and configs while running builds/tests.

### Step 2.1: Install Core Tools
- **What to Do**: Install Rust, Docker, Redis, Git, and build essentials.
- **How to Do It**:
  1. In terminal: `sudo apt install build-essential libseccomp-dev git curl -y` (for compiling Isolate and basics).
  2. Install Rust: `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh` (follow prompts, choose default install). Then `source ~/.cargo/env; rustup update`.
  3. Install Docker: `sudo apt install docker.io -y; sudo usermod -aG docker $USER` (log out/in to apply group).
  4. Install Redis: `sudo apt install redis-server -y; sudo systemctl enable --now redis-server`.
  5. Install wrk for load testing: `sudo apt install wrk -y`.
  6. Verify: `rustc --version` (1.80+), `docker --version`, `redis-cli ping` (PONG), `git --version`.
- **What Changes/Why**: Your stack relies on these (Rust for app, Docker for sandboxes, Redis for cache/queue). libseccomp-dev is for Isolate security. This mimics Judge0's Linux env but faster with Rust.
- **What to Include**: No project changes—global tools. Enable Docker service: `sudo systemctl enable --now docker`.
- **Time**: 30 minutes. **Test**: Run `docker run hello-world` (pulls image, prints message). `redis-cli set test 1; redis-cli get test` (returns "1").

### Step 2.2: Clone/Transfer Project and Convert Scripts
- **What to Do**: Bring evalX to Ubuntu and adapt Windows-specific files.
- **How to Do It**:
  1. Create project dir: `mkdir ~/evalx; cd ~/evalx`.
  2. Clone from GitHub: `git clone <your-repo-url> .` (or copy from USB: `cp -r /media/usb/evalx/* .`).
  3. Convert `build-images.ps1` to `build-images.sh`: Open in nano (`nano build-images.sh`), rewrite as a bash script that builds each Docker image sequentially (e.g., for Python: docker build -t python:3.9 -f Docker/python/Dockerfile.python3.9 .). Make executable: `chmod +x build-images.sh`.
  4. Edit `.env` if needed (e.g., add Redis URL: REDIS_URL=redis://localhost:6379).
  5. Run `cargo check` to verify Rust setup (fixes any path issues).
- **What Changes/Why**: PowerShell won't run on Linux—bash is native. This keeps your Docker images intact (pre-built for langs like Python 3.9, Java 21). Ensures project runs out-of-box.
- **What to Include**: All existing files (src/, Docker/, Cargo.toml). Add `.sh` script with error checking (e.g., [ $? -ne 0 ] && exit 1 after each build). Ignore large images—rebuild.
- **Time**: 20 minutes. **Test**: `./build-images.sh` (builds images without errors). `cargo build` (compiles without issues).

### Step 2.3: Install and Configure VS Code
- **What to Do**: Install VS Code and extensions for Rust/Docker editing.
- **How to Do It**:
  1. Install: `sudo snap install --classic code` (or download .deb from code.visualstudio.com and `sudo apt install ./code_*.deb`).
  2. Open project: `cd ~/evalx; code .` (launches VS Code with folder open).
  3. Install extensions: In VS Code (Ctrl+Shift+X), search/install: rust-analyzer (Rust support), Docker (container management), YAML (for compose files), CodeLLDB (debugging), GitLens (Git).
  4. Configure: In settings (Ctrl+,), set integrated terminal default to bash. Update rust-analyzer: Run `rustup component add rust-analyzer` in terminal.
  5. Open integrated terminal (Ctrl+` ) and test `cargo build`.
- **What Changes/Why**: VS Code provides autocompletion/error checking for Rust edits (e.g., in executor.rs), syntax for Dockerfiles/YAML. Integrated terminal runs cargo/docker without switching windows. Better than terminal-only (nano/vim lack features for multi-file Rust work).
- **What to Include**: Extensions only—no project files. Create .vscode/settings.json if needed (e.g., {"rust-analyzer.server.extraEnv": {"RUSTUP_TOOLCHAIN": "stable"}}).
- **Time**: 20 minutes. **Test**: Open src/main.rs, type a Tokio import—autocompletion should suggest. Debug: Set breakpoint, F5 to run.

### Step 2.4: Set Up Terminal Editing (Fallback)
- **What to Do**: Learn basic nano/vim for quick terminal edits (e.g., .env changes).
- **How to Do It**:
  1. For nano: Run `nano .env` to practice—edit, Ctrl+O (save), Ctrl+X (exit).
  2. For vim: Run `vim build-images.sh`—press i (insert), edit, Esc, :wq (save/exit).
  3. Use sed for bulk: e.g., `sed -i 's/old/new/g' file.rs` (replace in place).
- **What Changes/Why**: Backup for SSH/remote or quick fixes. Nano is simple for .env; vim for advanced. But prioritize VS Code for main edits.
- **What to Include**: No new files. Practice on a copy: `cp .env .env.bak; nano .env.bak`.
- **Time**: 10 minutes. **Test**: Edit .env with nano, add a dummy line, save, cat .env to verify.

---

## Section 3: Phase 1 - Quick Wins (Gunshot Changes for Immediate Speed)

Focus: Reduce overhead for 100-1000 requests x20 tests to <5s avg. Enhance existing Docker/Redis without new deps. Changes: Bump pools, parallelize batches, Redis queue, basic monitoring. Test locally with Docker Compose.

### Step 3.1: Increase Pre-Warmed Container Pools
- **What to Do**: Expand Docker pool size and add dynamic spawning to handle more concurrent tasks.
- **How to Do It**:
  1. In VS Code, open .env: Increase CONTAINER_POOL_SIZE to 50 (per language) and add MAX_CONCURRENT_CONTAINERS=500.
  2. Open src/container_management/init.rs: Modify pool initialization to use the env var for size (parse as usize). Limit to num_cpus * 2 for safety (import num_cpus crate if not already—add to Cargo.toml if missing, but assume it's there).
  3. In src/container_management/container_pool.rs: Add logic to check pool usage—if below 10% free, spawn idle containers in background using Tokio (e.g., spawn a task to create and add to pool).
  4. Run `./build-images.sh` to ensure images are ready.
  5. In terminal: `docker-compose up -d` (create basic compose if not—see Step 3.5).
- **What Changes/Why**: Your current pool (likely small) causes startup delays for 1000+ reqs. This pre-warms 50+ containers/language, reducing per-task overhead to <100ms. Dynamic spawn prevents exhaustion without over-provisioning. Handles 20-test batches by reusing pools.
- **What to Include**: Env vars for all languages (e.g., POOL_PYTHON=50). Log pool status in tracing (add debug logs). Fallback: If pool full, queue in Redis.
- **Time**: 2 hours. **Test**: Run cargo run, send 100 batch reqs via curl (20 tests each). Check docker ps (50+ containers idle). Latency <5s total.

### Step 3.2: Optimize Batch Execution for 20 Test Cases
- **What to Do**: Compile once per request, parallelize 20 runs using existing Tokio.
- **How to Do It**:
  1. Open src/executor.rs: In execute_batch function, ensure single compile (use artifact_handlers.rs), then spawn Tokio tasks for each test case (e.g., join_all on futures for runs, sharing the compiled artifact/container).
  2. Open src/controllers/executionControllers.rs: Add validation for batch (same code/language across 20 tests). Enqueue as one task but fan-out runs.
  3. Use Redis to aggregate results (store partials by token, fetch at end).
  4. For interpreted langs (Python/JS): Cache full script in Redis to skip container writes.
  5. Cargo build --release; cargo run.
- **What Changes/Why**: Current setup may re-compile per test— this cuts time by 80% (compile ~500ms, runs ~50ms each parallel). Tokio handles concurrency without blocking, scaling to 20+ on one machine. Redis aggregation ensures ~2s for small loads.
- **What to Include**: Error handling (if one run fails, continue others). Timeout per run (1s). Log fan-out (e.g., "Running 20 tests in parallel").
- **Time**: 3 hours. **Test**: Curl a batch JSON (20 stdins), time response (<2s for 20 tests). wrk -t10 -c50 -d10s http://localhost:3000/execute-batch (check avg latency).

### Step 3.3: Switch Queue to Redis (Ditch Custom In-Memory)
- **What to Do**: Use Redis lists/sorted sets for distributed queuing.
- **How to Do It**:
  1. Open src/queue_management/manager.rs: Replace custom queue with Redis ops (lpush for enqueue tasks as JSON, rpop/blpop for dequeue with blocking). Use sorted sets for priority (zadd with score for urgent).
  2. Open src/setup.rs: Initialize Redis conn pool (use mobc-redis), pass to queue manager and workers.
  3. Update enqueue in controllers: Serialize request to JSON, push to key like "queue:<lang>".
  4. Edit redis.conf (in Docker/redis/): Set maxmemory 512mb, policy allkeys-lru for auto-expiry.
  5. Restart Redis: sudo systemctl restart redis-server.
- **What Changes/Why**: In-memory queue doesn't scale beyond one instance—Redis enables multi-worker (for 1000+ reqs). Blocking pop reduces polling overhead. Preps for Phase 2 distribution.
- **What to Include**: Keys like queue:python:3.9, results:<token>. Expiry 1h for queues. Handle empty queue gracefully (return immediately).
- **Time**: 2 hours. **Test**: Enqueue 100 tasks via API, monitor redis-cli llen queue:python (grows/shrinks). Dequeue in worker, check no lost tasks.

### Step 3.4: Add Basic Monitoring
- **What to Do**: Track queue depth and container usage with Prometheus.
- **How to Do It**:
  1. Add prometheus = "0.13" to Cargo.toml (if not there), cargo update.
  2. Create src/monitoring/metrics.rs: Define counters/gauges (e.g., queue_length, container_usage) using prometheus crate. Update in manager.rs (inc on enqueue) and container_pool.rs (set on spawn).
  3. Open src/routes.rs: Add /metrics endpoint to serve prometheus data (use axum route).
  4. Install Grafana locally: sudo apt install grafana -y; sudo systemctl enable --now grafana-server. Access http://localhost:3000, add Prometheus datasource (http://localhost:9090).
  5. Cargo run; visit /metrics.
- **What Changes/Why**: No visibility into bottlenecks (e.g., queue >50 spikes latency). This adds metrics for Phase 3 scaling triggers. Free/local, unlike Judge0's basic logs.
- **What to Include**: Metrics: queue_depth (per lang), pool_free, latency_histogram. Expose only /metrics (secure).
- **Time**: 2 hours. **Test**: Run app, enqueue tasks, curl /metrics (see counters increase). Grafana dashboard for queue depth.

### Step 3.5: Local Orchestration with Docker Compose
- **What to Do**: Orchestrate app + Redis for multi-container testing.
- **How to Do It**:
  1. Create docker-compose.yml at root: Define services (app: build ., ports 3000, depends redis; redis: image redis:7, ports 6379; worker: build ., command cargo run -- worker, replicas 4, depends redis).
  2. In VS Code, validate YAML syntax.
  3. Run docker-compose up -d --scale worker=4 (starts 1 app + 4 workers + redis).
  4. Test scaling: docker-compose up --scale worker=8.
- **What Changes/Why**: Monolithic run limits to 100 reqs—compose enables worker scaling for 1000+. Simulates AWS ECS locally, free.
- **What to Include**: Volumes for Redis persistence (/data). Env_file: .env. Healthchecks for services.
- **Time**: 1 hour. **Test**: docker-compose ps (all up), send batches—workers dequeue via Redis.

**Phase 1 End Goal**: Handle 100-1000 reqs x20 in <5s. wrk benchmark: avg <2s for 50 concurrent.

---

## Section 4: Phase 2 - Core Optimizations (Perfect Changes for Judge0 Speed)

Focus: Integrate Isolate for <10ms sandboxes, upgrade queuing, dynamic pools. Changes: Add sandbox module, fallback to Docker, Broccoli for queues. Outperforms Judge0 with Rust efficiency (sub-100ms execs).

### Step 4.1: Integrate Isolate for Sandboxes
- **What to Do**: Add Isolate as primary (Docker fallback) for fast compiles/runs.
- **How to Do It**:
  1. In terminal: git clone https://github.com/ioi/isolate; cd isolate; make; sudo make install (builds binary).
  2. Create src/sandbox/mod.rs and isolate.rs: Define IsolateSandbox struct with compile (run gcc/g++ etc. in isolate --init box) and run (isolate --run with stdin, timeout). Handle output (stdout/stderr/exit), cache binary to /tmp or Redis.
  3. Open src/compilers/artifact_handlers.rs and compilers.rs: Route compiles to Isolate if USE_ISOLATE=true in .env (else Docker via bollard).
  4. Update src/executor.rs: For supported langs (C/C++/Java/Python), use IsolateSandbox::compile then ::run for tests. Fallback to Docker for unsupported.
  5. Add .env: USE_ISOLATE=true, ISOLATE_PATH=/usr/local/bin/isolate.
  6. Test single: isolate --init=-u /tmp/sandbox --run --time=1 python3 -c "print('Hello')".
- **What Changes/Why**: Docker startup (100-500ms) bottlenecks 1000+ reqs—Isolate (chroot/cgroup) forks in <10ms, handles 5000+ concurrent on 16-core. Compile once/cache binary in Redis, parallel runs for 20 tests. Safer than Judge0's Ruby with seccomp filters (add --enable-seccomp).
- **What to Include**: Supported langs: C (gcc), C++ (g++), Java (javac), Python (direct run). Limits: --time=2s, --mem=512000 (512MB). Error: Parse stderr for compile fails. Cache key: sha2 of code+lang.
- **Time**: 1 day. **Test**: Cargo run, single compile/run (<50ms). Batch 20 tests: parallel in Isolate boxes, aggregate in Redis.

### Step 4.2: Upgrade Queuing to Broccoli/Hatchet
- **What to Do**: Replace Redis queue with distributed Broccoli (Redis-backed).
- **How to Do It**:
  1. Add broccoli = "0.5" to Cargo.toml (or hatchet if preferred—Rust task queue), cargo update.
  2. Refactor src/queue_management/manager.rs: Create QueueManager with Broccoli Client (new from redis://localhost:6379). Add/enqueue: client.enqueue("evalx_worker", (requests vec, exec_type)). Workers consume via client.consume.
  3. Open src/setup.rs: Init client, spawn worker threads (tokio::spawn for consume loop, call executor).
  4. Update controllers/executionControllers.rs: Enqueue batches as jobs, return token. Add priority (high for small batches).
  5. In docker-compose.yml: Scale workers to consume from Broccoli.
- **What Changes/Why**: Basic Redis queue lacks retries/distribution—Broccoli adds reliability (dead-letter queues), lighter than Judge0's Celery. Handles 1000+ by distributing across workers/instances.
- **What to Include**: Job payload: Serialize ExecutionRequest vec. Retries: 3x on fail. Priority queues: Separate for urgent. Integrate with Redis cache (store results by job ID).
- **Time**: 1 day. **Test**: Enqueue 100 batches, workers process (check Broccoli queues via redis-cli). No lost jobs, <2s for 1000 reqs x20.

### Step 4.3: Dynamic Pre-Warmed Sandboxes
- **What to Do**: Auto-spawn Isolate/Docker based on queue.
- **How to Do It**:
  1. Rename src/container_management to sandbox_management (find/replace in VS Code, update mod.rs imports).
  2. In sandbox_management/container_pool.rs (now for both): If USE_ISOLATE, pre-spawn 100+ Isolate boxes (processes via Command). Check queue depth (Redis llen) >20, spawn more (Tokio background).
  3. For Docker fallback: Use bollard to pull/create idle if low.
  4. Limit: num_cpus * 10 for Isolate (cheap processes).
  5. Update executor.rs: Get from pool (Isolate or Docker).
- **What Changes/Why**: Static pools exhaust at scale—dynamic ensures always-ready for 10000 reqs (e.g., 200k execs parallel). Isolate multiplexing: 1000+ per machine vs. Docker's 100.
- **What to Include**: Metrics: Update monitoring for sandbox_free. Cleanup: Kill idle after 5min. Fallback switch if Isolate fails.
- **Time**: 1 day. **Test**: Load test wrk -c200 (queue builds, spawns auto). docker ps or ps aux | grep isolate (100+ running).

**Phase 2 End Goal**: <2s for 1000 reqs x20. Single exec <100ms. wrk: 500 concurrent, no spikes.

---

## Section 5: Phase 3 - Advanced Features (To Beat Judge0)

Focus: Scaling, monitoring, extras like webhooks/hints. Changes: Minikube for local cluster, Prometheus full, optional DB. Outperforms with global features (multi-region prep, AI caching).

### Step 5.1: Dynamic Scaling Setup
- **What to Do**: Use Minikube for local K8s-like scaling (mimic AWS ECS).
- **How to Do It**:
  1. Install Minikube: curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64; sudo install minikube-linux-amd64 /usr/local/bin/minikube; minikube start (uses Docker driver).
  2. Create minikube-config.yaml: Deployment for app (replicas 1, image build), HPA (horizontal pod autoscaler: scale on CPU>50% or custom queue metric via Redis exporter).
  3. Build image: eval Dockerfile (add to root: FROM rust:1.80, COPY . /app, RUN cargo build --release).
  4. Deploy: kubectl apply -f minikube-config.yaml; kubectl scale deployment app --replicas=4.
  5. Expose: minikube service app (port 3000).
  6. In sandbox_management: Query queue depth to trigger scale (via env var for kubectl).
- **What Changes/Why**: Local compose limits to one machine—Minikube simulates cluster for 10000 reqs (auto-scale pods/workers). Preps AWS migration (spot instances). Beats Judge0's single-node limits.
- **What to Include**: Redis in Minikube (statefulset). Metrics adapter for HPA (queue>100 scale up). Spot-like: Use taints for cost sim.
- **Time**: 2 days. **Test**: kubectl get pods (scales on load). wrk via minikube ip, 1000 reqs—pods increase.

### Step 5.2: Enhanced Monitoring & Webhooks
- **What to Do**: Full Prometheus/Grafana, add async webhooks.
- **How to Do It**:
  1. Update src/monitoring/metrics.rs: Add latency histograms, error counters. Scrape config for Prometheus (add prometheus.yml: scrape /metrics every 15s).
  2. Install Prometheus: sudo apt install prometheus -y; edit /etc/prometheus/prometheus.yml to include job for app:3000.
  3. In Grafana (already installed): Add dashboards for queue, latency, sandbox usage. Alerts: Email on queue>50.
  4. Open src/controllers/notificationControllers.rs: Add webhook support—on batch complete, POST results to client URL (from request JSON). Store token in Redis for polling fallback.
  5. Update routes.rs: /submit-batch returns token/webhook_url.
  6. Restart services.
- **What Changes/Why**: Basic metrics insufficient—full setup alerts on issues, visualizes for optimization. Webhooks reduce polling (Judge0 uses polling; this is async, faster for clients).
- **What to Include**: Dashboards: Throughput (reqs/s), error rate. Webhook: JSON payload with results vec. Retry: 3x on fail. Security: Validate URLs.
- **Time**: 2 days. **Test**: Submit with webhook_url, check POST received. Grafana: Load test, see metrics spike/scale.

### Step 5.3: AI-Powered Caching and Extras
- **What to Do**: Enhance caching, add hints, optional history DB.
- **How to Do It**:
  1. Open src/caching/redis_client.rs: Add preemptive cache—log common codes (from requests), predict/store outputs (e.g., if code SHA matches, return cached). Expire LRU.
  2. In models/response.rs: Add hints field to EvaluationResult (e.g., parse stderr for "syntax error", suggest fix—use simple string match or future AI).
  3. Optional DB: Add sea-orm = "0.12", sqlx to Cargo.toml. Create src/db/mod.rs: SQLite for submissions (store token, code hash, results). Init in setup.rs, query for history.
  4. Update compilers.rs: On compile fail, generate hint.
  5. For multi-region: Add .env MULTI_REGION=false (prep for AWS later).
- **What Changes/Why**: Basic cache misses on repeats—AI predicts common (e.g., hello world), cuts time 90%. Hints add value >Judge0. DB optional for history (beats token-less design).
- **What to Include**: Cache: Keys like cache:<sha>:<stdin>. Hints: 5-10 common messages. DB: Schema for requests (id, token, lang, results json). Migrate: diesel or sea-orm CLI.
- **Time**: 2 days. **Test**: Submit common code—hit cache (<10ms). Check hints in response. DB: Query history via new /history endpoint.

### Step 5.4: Final Testing and Prod Prep
- **What to Do**: Benchmark full stack, prep AWS.
- **How to Do It**:
  1. Run full benchmarks: wrk -t20 -c1000 -d60s http://minikube ip:3000/execute-batch (simulate 10000 reqs x20 over time).
  2. Monitor: Grafana for <2s p95 latency, no >100 queue.
  3. AWS Prep: Sign up free tier, create ECS cluster template (from compose). Add CloudWatch for queue alarms.
  4. Security: Add seccomp in Isolate (--enable-seccomp), rate-limit in axum (tower).
  5. More langs: Add Dockerfiles for new (e.g., Go), Isolate support.
- **What Changes/Why**: Validates outperformance (200k execs in <10s cluster). Prod: Auto-scale on metrics, spot for cost.
- **What to Include**: Load scripts: JSON batches. Alerts: Slack/email. Plugins: Modular for langs.
- **Time**: 1 day. **Test**: 10000 simulated—scale to 10 pods, <2s avg.

**Phase 3 End Goal**: Outperform Judge0—<2s for 10000 reqs x20, features like webhooks/hints. Ready for AWS prod.

---

## Troubleshooting & Next Steps
- **Common Issues**: Docker permission? sudo usermod -aG docker $USER, relog. Rust errors? rustup update. High latency? Check pool/queue metrics.
- **Resources**: Rust Book, Docker Docs, Isolate GitHub, Minikube Guide.
- **Next**: After Phase 1, commit/push. Migrate to AWS for final (use free credits). Contact if stuck on step.

Build iteratively—your evalX will be robust, fast, and superior!
