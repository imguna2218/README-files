Here is the **Master Plan** for your "Zero Cost" Microservices Cloud IDE.

### **1. The "Caching" Verdict (Acceptable for MVP)**

You are right. For Day 1 (MVP), **ignore the cache**.

* User opens project -> System unzips code -> System runs `npm install`.
* It might take 30-60 seconds. **This is acceptable for a student project.**
* *Optimization Strategy:* We will add the "Shared Volume" in Week 4 only if it is too slow.

### **2. The Development Workflow (How 3 Friends Work)**

You asked: *"If I develop in my dual boot, will my friends be able to run the code?"*

**The Problem:** Your friends (if they are on Windows or weak VMs) **CANNOT** run Firecracker/Ignite. It requires bare-metal Linux (your Dual Boot).

**The Solution: "The Mock & Remote Strategy"**

1. **Person A (You - Infra):** You are the **Server**. You run the Firecracker VMs on your laptop.
2. **Person B (Backend):** Developing the API. He points his code to **YOUR** laptop (via **Tailscale** or **Ngrok**) to start VMs. Or, he uses a "Mock" function that pretends to start a VM.
3. **Person C (Frontend):** Developing the UI. He points his Dashboard to Person B's API.

**Crucial Rule:** You must agree on **Constants**.

* Create a file `shared-config.js` in your git repo.
* `const REACT_IMAGE_NAME = "vessel/react-template:v1";`
* `const TRAEFIK_DOMAIN = ".localhost";`
* Everyone uses these variables. If you change the image name, everyone updates.

---

### **3. The Three Roles (Equal Suffering)**

Here is the exact division. Copy-paste this to your friends.

#### **Role 1: The Infrastructure Architect (Person A - YOU)**

* **The Mission:** "I control the Metal."
* **The Suffering:** You fight the Linux Kernel, Networking, and Dockerfiles. If the VM doesn't boot, it's your fault.
* **Tech Stack:** Ubuntu 22.04, Ignite (Firecracker), Docker, Traefik, Bash.
* **Key Deliverables:**
1. **The Base Image:** A Dockerfile that installs `Node.js`, `Git`, and `OpenVSCode Server`. Build it as `vessel/react-base`.
2. **The Ignite Wrapper:** A standalone Node.js script (`vm-manager.js`) that acts as a tool:
* `node vm-manager.js start <project-id>` -> Boots VM, returns IP.
* `node vm-manager.js stop <project-id>` -> Kills VM.


3. **The Gateway:** A `docker-compose.yml` file running **Traefik**. It must accept traffic on `*.localhost` and route it to the Firecracker VM IPs.



#### **Role 2: The Backend Orchestrator (Person B)**

* **The Mission:** "I control the Logic."
* **The Suffering:** You fight the State. What if the user closes the tab? What if the zip file is corrupt? You handle the messy "Business Logic."
* **Tech Stack:** Node.js (Express), Supabase (Auth, DB, Storage), `adm-zip` (for zipping).
* **Key Deliverables:**
1. **Storage Logic:**
* `downloadProject(projectId)`: Pulls zip from Supabase -> Unzips to `/tmp/projects/{id}`.
* `saveProject(projectId)`: Zips `/tmp/projects/{id}` -> Uploads to Supabase.


2. **The API:**
* `POST /project/open`: Auth user -> Check DB -> Trigger Role 1's Start Script -> Return URL.


3. **Database Design:**
* Table `projects`: `id`, `user_id`, `storage_path`, `template_type`.
* Table `active_sessions`: `project_id`, `container_ip`, `started_at`.





#### **Role 3: The Frontend & Experience (Person C)**

* **The Mission:** "I control the User."
* **The Suffering:** You fight the Browser. You must make the user *feel* like the system is instant, even when it takes 10 seconds to boot.
* **Tech Stack:** Next.js (React), Tailwind CSS, Supabase Auth UI, Framer Motion (for loaders).
* **Key Deliverables:**
1. **Authentication:** Fully working Login/Signup using Supabase.
2. **The Dashboard:** A grid view of "My Projects" and a "Create New" wizard (Select React/Node -> Enter Name).
3. **The Loading Screen:** This is critical. When User clicks "Open", show a "Terminal-style" log:
* *Initializing Container... Done*
* *Mounting Files... Done*
* *Starting VS Code... Ready!*


4. **The Editor Page:** An `iframe` that points to `http://{project-id}.localhost`.



---

### **4. The Perfect Parallel Timeline (30 Days)**

**Phase 1: Isolation (Day 1 - 7)**

* **Goal:** Everyone works ALONE. No integration yet.
* **Person A:** Gets 1 Firecracker VM running manually. Can visit `localhost:3000` to see VS Code.
* **Person B:** Can run a script `node test-storage.js` that downloads a zip from Supabase and unzips it.
* **Person C:** Can log in via Supabase and see a fake list of projects.

**Phase 2: The Handshake (Day 8 - 14)**

* **Goal:** Connect A to B.
* **Person B** updates their API to call Person A's `vm-manager.js` script (instead of logging "Fake Start").
* **Person A** ensures Traefik is running so B's API can verify the URL is reachable.
* **Person C** connects the "Create Project" button to Person B's API (even if it just returns a success message for now).

**Phase 3: The Full Loop (Day 15 - 21)**

* **Goal:** A User can open a project.
* **Integration:**
1. Person C clicks "Open".
2. Person B downloads code & calls A.
3. Person A boots VM.
4. Person C redirects browser to `project.localhost`.


* **Bug Fixing:** It will fail. Permissions will be wrong. Networking will break. Fix it.

**Phase 4: Persistence & Polish (Day 22 - 30)**

* **Goal:** Saving work.
* **Person B:** Adds the `saveProject` logic when the API receives a "Stop" signal.
* **Person A:** Adds a script to detect "Zombie VMs" (running > 1 hour) and kill them.
* **Person C:** Makes the UI look beautiful.

---

### **5. Immediate Action Plan (Right Now)**

1. **Create 1 GitHub Repository:** `team/vessel-cloud-ide`.
2. **Create 3 Folders:** `/infra`, `/backend`, `/frontend`.
3. **Person A (You):**
* Boot your Ubuntu Dual Boot.
* Install **Ignite** and **Traefik**.
* Run: `ignite run weaveworks/ignite-ubuntu --name test-vm --cpus 2 --memory 1GB --ssh`.
* *Success Check:* Can you SSH into it?



**Are you ready to execute Phase 1?**
