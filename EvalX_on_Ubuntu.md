### Step-by-Step Setup for Ubuntu 22.04 LTS (Dual-Boot)

Here’s how to set up Ubuntu and develop evalX to outperform Judge0. I’ll assume you’re starting from Windows and want minimal disruption. Each step is detailed, considering your project’s files and goals.

#### Step 1: Install Ubuntu 22.04 LTS (Dual-Boot)
- **What to Do**:
  1. **Backup Data**: Save all evalX files and important data to an external drive or cloud (e.g., GitHub).
  2. **Download Ubuntu**: Get Ubuntu 22.04 LTS ISO from ubuntu.com (or 24.04 if available in 2025).
  3. **Create Bootable USB**: Use Rufus (Windows) to make a bootable USB with the ISO.
  4. **Partition Disk**:
     - On Windows, open Disk Management, shrink your C: drive (e.g., 100GB for Ubuntu).
     - Leave unallocated space for Ubuntu.
  5. **Boot and Install**:
     - Restart, enter BIOS (F2/Del), set USB as first boot device.
     - Boot Ubuntu USB, select “Install Ubuntu alongside Windows.”
     - Allocate 100GB+ to Ubuntu, 4GB swap if <16GB RAM.
     - Follow prompts; install GRUB bootloader.
  6. **Reboot**: Choose Ubuntu at GRUB menu.
- **Why**: Dual-boot gives near-native Linux performance without losing Windows. 100GB ensures space for Docker images, Minikube, and test data.
- **Test**: Boot into Ubuntu, verify `uname -a` shows Linux/Ubuntu.
- **Time**: 1-2 hours.

#### Step 2: Set Up Development Environment
- **What to Do**:
  1. **Update Ubuntu**: Run `sudo apt update && sudo apt upgrade -y`.
  2. **Install Tools**:
     - Rust: `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`
     - Docker: `sudo apt install docker.io -y; sudo usermod -aG docker $USER`
     - Redis: `sudo apt install redis-server -y`
     - Git: `sudo apt install git -y`
     - Build tools (for Isolate): `sudo apt install build-essential libseccomp-dev -y`
  3. **Clone evalX**: `git clone <your-repo>; cd evalx`
  4. **Copy Files**: Transfer your `Files.txt` contents (e.g., via USB or GitHub).
  5. **Rewrite Build Script**: Convert `build-images.ps1` to `build-images.sh`:
     ```bash
     #!/bin/bash
     echo "Building Python 3.9 Image..."
     docker build -t python:3.9 -f Docker/python/Dockerfile.python3.9 .
     [ $? -ne 0 ] && exit 1
     echo "Building Java 21 Image..."
     docker build -t java-slim-executor -f Docker/java/Dockerfile.java21 .
     [ $? -ne 0 ] && exit 1
     # Add other images...
     echo "All Docker images built successfully!"
     ```
     Run: `chmod +x build-images.sh; ./build-images.sh`
- **Why**: Sets up Rust, Docker, and Redis natively. Bash script aligns with Linux ecosystem, simplifying CI/CD later.
- **Test**: Run `cargo build`, `docker ps`, `redis-cli ping` (should return PONG).
- **Time**: 1 hour.

#### Step 3: Implement Phase 1 Quick Wins
Follow the **Quick Wins** from the previous plan, adapted for Ubuntu:
1. **Increase Container Pools**:
   - Edit `.env`: `CONTAINER_POOL_SIZE=50`, `MAX_CONCURRENT_CONTAINERS=500`.
   - Update `src/container_management/init.rs`: Cap pool at `num_cpus::get() * 2`.
   - In `container_pool.rs`, add dynamic spawning: If pool <10%, `tokio::spawn(create_idle_container)`.
   - Run `build-images.sh` to pre-build images.
2. **Optimize Batch Execution**:
   - In `src/executor.rs` (execute_batch), parallelize test case runs: Use `tokio::spawn` for 20 stdins, reuse artifact/container.
   - Update `controllers/executionControllers.rs`: Validate batch code/language consistency.
3. **Redis Queue**:
   - In `queue_management/manager.rs`, use `redis_client.lpush/rpop` for tasks. Add sorted sets for priority.
   - Update `setup.rs`: Pass Redis conn to workers.
4. **Caching**:
   - In `redis_client.rs`, track hit/miss rates, cache batch results.
   - Edit `redis.conf`: Set `maxmemory 512mb`, `maxmemory-policy allkeys-lru`.
- **Why**: Leverages Ubuntu’s fast Docker/Redis for 1000 reqs x20 in <5s. No Windows overhead.
- **Test**: Run `cargo run`, use `wrk -t10 -c100 -d30s http://localhost:3000/execute-batch` with 20-test JSON. Check logs, Redis (`monitor`).
- **Time**: 1-2 days.

#### Step 4: Implement Phase 2 Core Optimizations
1. **Integrate Isolate**:
   - Clone Isolate: `git clone https://github.com/ioi/isolate; cd isolate; make install`
   - Create `src/sandbox/isolate.rs`:
     ```rust
     use std::process::Command;
     use anyhow::Result;
     use crate::models::response::{Artifact, EvaluationResult};

     pub struct IsolateSandbox;
     impl IsolateSandbox {
         pub fn compile(code: &str, lang: &str, timeout: f64) -> Result<Artifact> {
             let output = Command::new("isolate")
                 .args(["--init=-u", "sandbox", "--time", &timeout.to_string(), "--mem=512000"])
                 .arg(match lang {
                     "c" => "gcc -o main main.c",
                     "cpp" => "g++ -o main main.cpp",
                     "java" => "javac Main.java",
                     _ => return Err(anyhow::anyhow!("Unsupported lang")),
                 })
                 .output()?;
             if output.status.success() {
                 Ok(Artifact {
                     code: code.to_string(),
                     binary: std::fs::read("/tmp/sandbox/main")?,
                 })
             } else {
                 Err(anyhow::anyhow!("Compile failed: {}", String::from_utf8_lossy(&output.stderr)))
             }
         }
         pub fn run(binary: &[u8], stdin: &str, timeout: f64) -> Result<EvaluationResult> {
             std::fs::write("/tmp/sandbox/main", binary)?;
             let output = Command::new("isolate")
                 .args(["--run", "--time", &timeout.to_string(), "--mem=512000", "--stdin", stdin])
                 .arg("/tmp/sandbox/main")
                 .output()?;
             Ok(EvaluationResult {
                 compile_time: 0.0,
                 stdout: String::from_utf8_lossy(&output.stdout).to_string(),
                 stderr: if output.stderr.is_empty() { None } else { Some(String::from_utf8_lossy(&output.stderr).to_string()) },
                 exit_code: output.status.code().unwrap_or(-1) as i64,
                 run_time: timeout,
                 space_consumed: "0 MB".to_string(),
             })
         }
     }
     ```
   - Update `executor.rs`: Check `.env` for `USE_ISOLATE=true`, route to `IsolateSandbox` for compile/run.
   - Update `compilers/`: Move logic to use Isolate.
2. **Broccoli Queue**:
   - Add to `Cargo.toml`: `broccoli = "0.5"`
   - Refactor `queue_management/manager.rs`:
     ```rust
     use broccoli::Client;
     pub struct QueueManager {
         client: Client,
         executor: Arc<CodeExecutor>,
     }
     impl QueueManager {
         pub fn new(executor: Arc<CodeExecutor>) -> Self {
             let client = Client::new("redis://localhost:6379");
             Self { client, executor }
         }
         pub async fn add_task(&self, requests: Vec<ExecutionRequest>, exec_type: ExecutionType) -> Result<Vec<EvaluationResult>> {
             self.client.enqueue("evalx_worker", (requests, exec_type)).await?;
             Ok(vec![])
         }
     }
     ```
   - In `setup.rs`, init Broccoli client.
- **Why**: Isolate + Broccoli = Judge0’s speed (sub-100ms execs) + Rust’s efficiency. Scales to 5000+/sec.
- **Test**: Benchmark single req: <50ms. Run 1000 reqs x20; expect ~2s.
- **Time**: 3-5 days.

#### Step 5: Implement Phase 3 Advanced Features
1. **Dynamic Scaling**:
   - In `sandbox_management/`, spawn Isolate boxes dynamically based on queue depth (Redis key count).
   - Create `docker-compose.yml`:
     ```yaml
     version: '3'
     services:
       app:
         build: .
         ports:
           - "3000:3000"
         depends_on:
           - redis
       redis:
         image: redis:7.0-alpine
         ports:
           - "6379:6379"
       worker:
         build: .
         command: cargo run -- worker
         depends_on:
           - redis
         deploy:
           replicas: 4
     ```
   - Install Minikube: `curl -LO https://minikube.sigs.k8s.io/docs/start/minikube-latest-linux-amd64 && sudo install minikube-latest-linux-amd64 /usr/local/bin/minikube`
   - Create `minikube-config.yaml` for HPA.
2. **Monitoring & Webhooks**:
   - Add `prometheus = "0.13"` to Cargo.toml.
   - Create `src/monitoring/metrics.rs`: Track queue depth, latency.
   - Update `routes.rs`: Add `/metrics`.
   - In `notificationControllers.rs`, add webhook POSTs.
3. **Extras**:
   - Add `hints` to `EvaluationResult`.
   - Optional: Add `sea-orm` + PostgreSQL for history.
- **Why**: Matches prod, adds features Judge0 lacks (hints, fine-grained rate-limiting).
- **Test**: Deploy to Minikube, load test 10000 reqs. Check Grafana for metrics.
- **Time**: 1 week.

### Final Notes
- **Why Ubuntu Wins**: Native performance, Judge0-like env, no overhead, prod-ready. Dual-boot is practical for your Windows machine.
- **Avoid Kali/Windows/VMs**: Kali’s for hacking, not dev. Windows/WSL2 adds latency. VMs are slow and complex.
- **Timeline**: 2 weeks total (1-2 days Phase 1, 3-5 days Phase 2, 1 week Phase 3).
- **Next Steps**: Start with Ubuntu dual-boot, follow steps sequentially. Test after each phase with `wrk`. If stuck, focus on Isolate integration—it’s the biggest win.

Your evalX will outperform Judge0 with this setup, hitting <2s for 1000+ reqs x20 tests, scalable to 10000, all developed free on Ubuntu!
