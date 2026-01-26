# **PROJECT VESSEL: SYSTEM ARCHITECTURE & TECHNICAL MANIFESTO**

### **1. Core Vision & Architectural Philosophy**

**Project Vessel** is a high-performance, cost-optimized Cloud Development Environment (CDE) designed to run on bare-metal Linux infrastructure using **Firecracker MicroVMs** via **Weaveworks Ignite**. Unlike traditional SaaS IDEs that rely on heavy, permanent Docker containers, Vessel adopts a **"Session-Based Ephemeral Architecture."** This means no user environment exists permanently. When a user requests a project, the system "hydrates" a MicroVM in <2 seconds, mounts their code, and establishes a secure tunnel. When the session ends, the state is persisted to cold storage (Supabase), and the compute resources are immediately incinerated, ensuring zero idle costs and NSA-level isolation.

### **2. The Infrastructure Layer (The Metal)**

The foundation runs on **Ubuntu 22.04 LTS** (with KVM virtualization enabled). We reject standard Docker for user workloads due to security and density concerns. Instead, we use **Ignite**, which allows us to run OCI-compliant Docker images as fully isolated Firecracker VMs.

* **The Golden Image:** A unified Docker image (`vessel/base:v1`) containing a stripped-down OS (Alpine/Ubuntu), **OpenVSCode Server** (listening on port 3000), `git`, `node`, and language runtimes. This image is immutable and cached on the host.
* **Snapshotting:** We utilize **Device Mapper** snapshotting. When User A and User B both request a "React" project, they share the read-only Golden Image. The system creates a tiny copy-on-write overlay for each user, consuming mere megabytes of storage per active session.

### **3. The Networking & Routing Layer (The Gateway)**

We bypass the complexity of assigning public IPs to every VM. Instead, we utilize a **Reverse Proxy Architecture** on a single entry point (Port 80).

* **The Gateway:** **Traefik v2.10** runs as the edge router on the host machine, listening on `0.0.0.0:80`.
* **Dynamic Discovery:** Since Firecracker VMs do not natively talk to Docker's discovery service, we implement **File-Based Dynamic Configuration**.
* When a VM boots, the Orchestrator extracts its internal interface IP (e.g., `172.16.0.5`).
* The Orchestrator writes a YAML configuration file to a watched directory (`/etc/traefik/dynamic/<project-id>.yml`).
* This file explicitly routes the Host Rule `Host(<project-id>.localhost)` to the internal destination `http://172.16.0.5:3000`.


* **Zero-Port Conflict:** The host machine *only* exposes Port 80. Users access their unique environments via subdomains (e.g., `http://proj-123.localhost`), eliminating the need for port mapping (e.g., `:3001`, `:3002`) and preventing collision.

### **4. The Storage Strategy (Zip-In / Zip-Out)**

To maintain the "Zero Cost" promise, we strictly separate **Compute** (RAM/CPU) from **Storage** (Disk).

* **Cold Storage:** All inactive project files reside in **Supabase Storage** (S3-compatible) as compressed `.zip` archives.
* **Hydration (Boot Sequence):**
1. User clicks "Open".
2. Orchestrator downloads `project-<id>.zip` from Supabase to the Host's temporary directory `/var/vessel/projects/<id>`.
3. Orchestrator unzips the contents.
4. The Ignite VM is started with a **Bind Mount**, mapping the Host path `/var/vessel/projects/<id>` to the VM path `/home/workspace`.


* **Dehydration (Shutdown Sequence):**
1. User clicks "Exit" or timeout occurs.
2. Orchestrator zips the `/var/vessel/projects/<id>` folder (excluding heavy artifacts if configured).
3. Archive is uploaded back to Supabase, overwriting the old state.
4. Local folder is wiped. VM is killed.



### **5. The Microservices Triad**

The system is divided into three strictly decoupled services to allow parallel development:

* **Service A (Infra-Manager):** A Node.js/Bash-based local agent that wraps the `ignite` CLI. It handles the raw creation of VMs, IP extraction, and Traefik YAML generation. It knows *nothing* about users or databases.
* **Service B (Orchestrator-API):** A Node.js/Express API that manages the business logic. It handles Supabase authentication, file download/upload, and commands Service A to start/stop resources. It maintains the "State of the World" in the database.
* **Service C (Dashboard-UI):** A Next.js frontend that provides the user interface. It handles the "Loading State" visualization (via WebSockets), authentication flows, and embeds the VS Code environment via secure iframes.

---

### **DETAILED AI PROMPTS FOR THE 3 ROLES**

Copy these prompts *exactly* into a new chat window for each person. They contain the full context required for that specific microservice.

#### **Prompt for Person A (The Infrastructure Engineer)**

> **Role:** You are the Senior Infrastructure Architect for "Project Vessel".
> **Project Context:** We are building a local-first Cloud IDE using Firecracker MicroVMs.
> **Your Mission:** You are responsible for building the **"VM Control Plane"**. You must create the scripts and configurations that run on the Ubuntu Host to manage the lifecycle of Weaveworks Ignite VMs and Traefik routing.
> **Technical Constraints:**
> * **Host OS:** Ubuntu 22.04 (Bare Metal).
> * **Virtualization:** Weaveworks Ignite (`ignite`).
> * **Networking:** Traefik v2.10 (File Provider Mode).
> * **Language:** Node.js (for the wrapper script) + Bash.
> 
> 
> **Your Deliverables:**
> 1. **The Base Dockerfile:**
> * Create a Dockerfile based on `ubuntu:22.04`.
> * Install: `curl`, `git`, `nodejs` (v18), `npm`, `nano`.
> * **Crucial:** Download and install the latest `openvscode-server` binary.
> * Configure it to run on Port 3000 as a non-root user (`workspace`).
> * Provide the command to build this image and import it into ignite (`ignite image import...`).
> 
> 
> 2. **The Traefik Configuration:**
> * Provide a `docker-compose.yml` that runs Traefik.
> * It must expose Port 80:80.
> * It must be configured to watch a dynamic directory: `./traefik/dynamic/`.
> 
> 
> 3. **The Node.js VM Manager (`infra-agent.js`):**
> * Write a script that accepts command line arguments.
> * **Command:** `start <vm-id> <host-mount-path>`
> * Logic: Run `ignite run vessel/base --name <vm-id> --cpus 2 --memory 1GB -v <host-mount-path>:/home/workspace`.
> * Logic: specific logic to parse `ignite inspect <vm-id>` to find the assigned **IP Address**.
> * Logic: Generate a YAML file at `./traefik/dynamic/<vm-id>.yml` that routes `Host('<vm-id>.localhost')` to `http://<VM_IP>:3000`.
> * Output: JSON `{ "status": "success", "ip": "...", "url": "http://<vm-id>.localhost" }`.
> 
> 
> * **Command:** `stop <vm-id>`
> * Logic: Delete the Traefik YAML file (killing the route).
> * Logic: Run `ignite stop <vm-id>` and `ignite rm <vm-id>`.
> 
> 
> 
> 
> 
> 
> **Output Requirement:** Give me the exact code for the Dockerfile, the docker-compose.yml, and the complete infra-agent.js script. Do not summarize.

---

#### **Prompt for Person B (The Backend Orchestrator)**

> **Role:** You are the Lead Backend Engineer for "Project Vessel".
> **Project Context:** We are building a session-based Cloud IDE. You control the API that orchestrates the user's session.
> **Your Mission:** Build the **Node.js/Express API** that handles Data Persistence and coordinates the Infrastructure.
> **Technical Constraints:**
> * **Runtime:** Node.js (Express).
> * **Database:** Supabase (PostgreSQL).
> * **Storage:** Supabase Storage.
> * **Infra Connection:** You will call a local script `infra-agent.js` (managed by another team member) to start VMs.
> 
> 
> **Your Deliverables:**
> 1. **Supabase Schema:**
> * SQL to create `projects` table (id, name, user_id, template_type, storage_path).
> * SQL to create `sessions` table (project_id, vm_id, status, ip_address, created_at).
> 
> 
> 2. **The "Zip-In" Logic (Hydration):**
> * Write a function `hydrateProject(projectId)` that:
> * Downloads the zip file from Supabase bucket `user-projects`.
> * Unzips it using `adm-zip` to a local directory `/var/vessel/projects/<projectId>`.
> 
> 
> 3. **The "Zip-Out" Logic (Dehydration):**
> * Write a function `dehydrateProject(projectId)` that:
> * Zips the folder `/var/vessel/projects/<projectId>` (excluding `node_modules`).
> * Uploads it back to Supabase.
> * Deletes the local folder.
> 
> 
> 4. **The API Endpoints:**
> * `POST /api/boot`: Accepts `projectId`.
> * Calls `hydrateProject()`.
> * Executes child_process: `node ../infra/infra-agent.js start <projectId> <path>`.
> * Returns `{ url: "http://<projectId>.localhost" }`.
> 
> 
> * `POST /api/shutdown`: Accepts `projectId`.
> * Executes child_process: `node ../infra/infra-agent.js stop <projectId>`.
> * Calls `dehydrateProject()`.
> 
> 
> 
> 
> 
> 
> **Output Requirement:** Provide the full SQL schema code and the complete `server.js` file with all the fs/child_process logic implemented.

---

#### **Prompt for Person C (The Frontend Architect)**

> **Role:** You are the Lead Frontend Developer for "Project Vessel".
> **Project Context:** We are building a Cloud IDE. You are building the "Face" of the application.
> **Your Mission:** Build the **Next.js Dashboard** that manages authentication and provides the "Loading Experience" while the backend spins up the VM.
> **Technical Constraints:**
> * **Framework:** Next.js 14 (App Router).
> * **Auth:** Supabase Auth (Email/Password).
> * **Styling:** Tailwind CSS.
> * **Real-time:** Socket.io-client (for receiving boot logs).
> 
> 
> **Your Deliverables:**
> 1. **The Dashboard Page:**
> * Fetch the list of projects from Supabase.
> * Display them in a Grid Card layout.
> * Include a "Create New Project" button that calls the Backend API to create a DB entry.
> 
> 
> 2. **The Boot Screen (The Critical UI):**
> * When a user clicks "Open", do not just redirect.
> * Show a **Terminal-like Overlay** (Black background, Green text).
> * Connect to the backend WebSocket.
> * Display real-time steps: "Downloading Files...", "Starting MicroVM...", "Configuring Gateway...".
> * When the "Ready" event is received, redirect to the IDE page.
> 
> 
> 3. **The IDE Page:**
> * Create a dynamic route `/editor/[projectId]`.
> * Embed a full-screen `<iframe>`.
> * Set the `src` to `http://<projectId>.localhost`.
> * Overlay a "Floating Action Button" (FAB) in the bottom-right corner that allows the user to "Save & Exit" (calling the Shutdown API).
> 
> 
> 
> 
> **Output Requirement:** Provide the Next.js page components for the Dashboard, the Boot Screen logic with WebSockets, and the Editor iframe implementation.
