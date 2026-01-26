This is the **War Room Plan**.

You asked for **3 Parallel Lanes**, **Equal Suffering**, and a **Ruthless Timeline**.
You also chose **Ubuntu 22.04 Dedicated VM** as your battleground. This is excellent. It is the native home of Firecracker.

---

### **The Architecture: "The Hydra"**

Before assigning roles, everyone must agree on this picture.

* **The Head (Frontend):** A Next.js Dashboard + The "WebContainer" IDE (Runs in Browser).
* **The Body (Backend):** A Node.js API that manages Users, Projects, and Storage.
* **The Legs (Infrastructure):** A Firecracker Cluster (Ignite) + Traefik Gateway.

---

### **Part 1: The Role Split (Equal Suffering)**

We split the team into **Infrastructure**, **Backend Logic**, and **Frontend Experience**.

#### **Person A: The System Architect (Infrastructure)**

* **The "Suffering":** You are fighting the Linux Kernel. You must make Firecracker networking talk to the outside world.
* **Core Responsibility:** The `ignite` Cluster & Traefik Gateway.
* **Specific Tasks:**
1. **Host Setup:** Configure the Ubuntu VM for KVM virtualization (checking `/dev/kvm`).
2. **Ignite Installation:** Manually install `ignite`, `containerd`, and CNI plugins (networking).
3. **Base Image Creation:** Build a Docker image containing `OpenVSCode Server` + `Node/Python`, then import it into Ignite.
4. **Traefik Routing:** Configure Traefik to route `*.localhost` traffic to the dynamic IPs of the Firecracker VMs.
5. **The "Kill Switch":** Write a script to kill VMs that have been idle for 30 minutes.



#### **Person B: The Backend Lead (The Brain)**

* **The "Suffering":** You are fighting State and Storage. You must ensure files don't disappear when the VM dies.
* **Core Responsibility:** The API (Node/Express), Database (Supabase), and File Sync.
* **Specific Tasks:**
1. **Auth & DB:** Design the Supabase Schema (`Users`, `Projects`, `VM_Status`).
2. **The "Orchestrator" API:** Write the Node.js code that calls Person A's Ignite commands (`exec('ignite run...')`).
3. **Storage Sync (The Hardest Part):**
* *On Boot:* Download Zip from Supabase -> Inject into VM.
* *On Close:* SSH into VM -> Zip folder -> Upload to Supabase -> Destroy VM.


4. **WebSocket Status:** Build the socket connection to tell the Frontend "VM is Ready."



#### **Person C: The Frontend Architect (The Face)**

* **The "Suffering":** You are fighting the Browser. You must build an IDE that looks real but runs in Chrome.
* **Core Responsibility:** The Dashboard, WebContainer IDE integration, and User Experience.
* **Specific Tasks:**
1. **The "Fake" IDE:** Fork and customize `Sunny-117/webcontainer-ide`. Connect it to Supabase Storage to load files.
2. **The Dashboard:** Build the Project List, Login Page, and "New Project" Wizard.
3. **The "Router" Logic:**
* If Project = React -> Load WebContainer Component.
* If Project = Java -> Show "Booting Server..." Loader -> Redirect to Person A's Traefik URL.


4. **Terminal Integration:** Integrate `xterm.js` to look exactly like VS Code's terminal.



---

### **Part 2: The Timeline (30 Days)**

You work in **Parallel** phases. Do not wait for each other.

#### **Phase 1: The Setup & Hello World (Days 1-3)**

* **Goal:** Everyone proves their tech stack works in isolation.
* **Person A:** Get **ONE** Firecracker VM running on Ubuntu and SSH into it.
* **Person B:** Get Supabase connected and write a script to Upload/Download a Zip file.
* **Person C:** Clone `webcontainer-ide` and run it locally. Verify `npm install` works inside the browser.

#### **Phase 2: The Builders (Days 4-15)**

* **Goal:** Build the core components.
* **Person A:** Build the custom `ignite-vscode` image. Set up Traefik so `test.localhost` routes to a manually started VM.
* **Person B:** Build the API endpoints: `/create-project`, `/start-vm`. (Mock the VM creation part for now).
* **Person C:** Build the Dashboard. Make the WebContainer IDE load files from a URL instead of hardcoded strings.

#### **Phase 3: The Integration (Days 16-25)**

* **Goal:** Connect the wires. **(This is where conflicts happen).**
* **Dependency:** Person B needs Person A's `ignite` commands to finish the API.
* **Dependency:** Person C needs Person B's API to know *where* to redirect the user.
* **Action:**
* B connects API to A's Ignite CLI.
* C connects Dashboard to B's API.
* **Test:** Click "Open Project" -> API spins VM -> Traefik routes -> Browser opens.



#### **Phase 4: Optimization & "Reaper" (Days 26-30)**

* **Goal:** Handle the "Closing" flow.
* **Person A & B:** Implement the logic to save files and kill the VM when the user clicks "Exit".

---

### **Part 3: The Environment Setup (Ubuntu 22.04)**

**WARNING:** Firecracker requires **KVM Virtualization**.
If you are running Ubuntu inside VMware/VirtualBox, you MUST enable "Nested Virtualization" in the VM settings.

**The "God Script" for Person A (Run this on the Ubuntu VM):**
This installs Ignite (The Firecracker Manager) and dependencies.

```bash
# 1. Check if your CPU supports virtualization
egrep -c '(vmx|svm)' /proc/cpuinfo
# If output is 0, STOP. You cannot use Firecracker. Enable VT-x in BIOS/VM settings.

# 2. Install Dependencies (Go, CNI plugins, Containerd)
sudo apt-get update && sudo apt-get install -y dmsetup openssh-client git binutils containerd

# 3. Install CNI Plugins (Networking)
export CNI_VERSION=v1.1.1
sudo mkdir -p /opt/cni/bin
curl -sSL "https://github.com/containernetworking/plugins/releases/download/${CNI_VERSION}/cni-plugins-linux-amd64-${CNI_VERSION}.tgz" | sudo tar -xz -C /opt/cni/bin

# 4. Install Ignite (The Firecracker Manager)
export VERSION=v0.10.0
curl -fLo ignite https://github.com/weaveworks/ignite/releases/download/${VERSION}/ignite-amd64
chmod +x ignite
sudo mv ignite /usr/local/bin

# 5. Verify it works
sudo ignite version

```

---

### **Summary Checklist for Day 1**

1. **Person A:** Run the script above. If `ignite version` fails, debug KVM.
2. **Person B:** Create a Supabase project. Get the API Keys.
3. **Person C:** `git clone https://github.com/Sunny-117/webcontainer-ide` and `npm run dev`.

**Go. Parallel execution starts now.**
