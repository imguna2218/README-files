This is the **Architectural Blueprint for Phase 2**.

The Goal: We are moving from **Manual Command Line Interface (CLI)** to **Programmatic API Control**.
Currently, you type `sudo node infra-manager.js start...`.
After Phase 2, a Frontend or Postman will send a JSON request, and the server will automatically launch the VM.

---

### **Phase 2: The API Gateway ("The Brain")**

#### **1. The Objective**

Create a **Node.js/Express Backend** that acts as the control center. It will:

1. Accept HTTP Requests (`POST /start`, `POST /stop`).
2. Import your existing infrastructure logic (`vm.js`, `proxy.js`).
3. Execute the heavy lifting (creating VMs, configuring routes).
4. Return the "Magic URL" to the user.

#### **2. The New Directory Structure**

We are adding a new folder named `backend` parallel to `infra` and `proxy`.

```text
/project-vessel
  ├── /infra           # (Existing) The Muscle (VM + Proxy logic)
  ├── /proxy           # (Existing) The Router (Traefik)
  └── /backend         # (NEW) The Brain (API Server)
       ├── package.json
       ├── server.js   # The Entry Point
       └── routes.js   # The API Definitions

```

---

### **3. Detailed Implementation Steps**

Follow these steps exactly to build the backend.

#### **Step 1: Initialize the Backend**

* **Action:** Create the `/backend` folder.
* **Action:** Initialize a Node.js project inside it (`npm init -y`).
* **Dependencies:** You must install:
* `express`: To handle web requests.
* `cors`: To allow your future frontend to talk to this backend.
* `body-parser` (optional, usually built into Express now): To read JSON data from requests.



#### **Step 2: The "Import Strategy" (Crucial)**

We will **NOT** rewrite the logic you already built. We will **reuse** it.
Your `infra/vm.js` and `infra/proxy.js` verify that they export functions like `create`, `copyTo`, and `addRoute`.

* **Requirement:** The Backend needs to require these files. Since they are in a sibling directory (`../infra/vm`), ensure the pathing in your code handles this "step back" correctly.

#### **Step 3: Create the Controller Logic (`sessionController.js` or inside `routes.js`)**

This is where the logic lives. You need to write a function that handles the "Start Session" request.

**Logic Flow for `POST /start`:**

1. **Receive Inputs:** Get `userId`, `projectId` (or generic `vmId`), and `projectPath` from the request body.
2. **Validation:** Check if `vmId` and `projectPath` exist. If not, return 400 Error.
3. **Orchestration (The Chain Reaction):**
* Call `vm.create(vmId)` -> Starts the Ignite VM.
* Call `vm.getIP(vmId)` -> Retreives the new IP (e.g., 10.61.0.50).
* Call `vm.copyTo(vmId, projectPath, ip)` -> Syncs the user's code (excluding node_modules).
* Call `proxy.addRoute(vmId, ip)` -> Updates Traefik to open ports 3000, 5173, and 8000.


4. **Response:** Send back a JSON object:
* `status: "success"`
* `ideUrl`: `http://user-1.localhost`
* `appUrl`: `http://5173-user-1.localhost`



**Logic Flow for `POST /stop`:**

1. **Receive Inputs:** Get `vmId` and `projectPath` (optional, for saving back).
2. **Orchestration:**
* Call `vm.getIP(vmId)` (Needed to copy data out).
* (Optional) Call `vm.copyFrom(...)` to save work back to the host.
* Call `proxy.removeRoute(vmId)` -> Kills the URL immediately.
* Call `vm.remove(vmId)` -> Destroys the VM.


3. **Response:** JSON `{ status: "stopped" }`.

#### **Step 4: Create the Server Entry Point (`server.js`)**

This file ties it all together.

1. **Setup Express:** Initialize the app.
2. **Middleware:** Enable `cors` and `express.json()`.
3. **Mount Routes:** Tell the app to use the routes defined in Step 3.
4. **The "Root" Requirement:**
* **Critical Note:** This backend calls `ignite` and `docker`, which require **Root/Sudo** privileges.
* You must code this server to listen on a specific port (e.g., `3001` or `4000`) so it doesn't conflict with the VMs or Traefik.
* **Execution Rule:** You will explicitly state that this server MUST be started with `sudo node server.js`.



---

### **4. Expected Outcome of Phase 2**

At the end of this phase, you will have a running API server.

**Validation Test:**
You (or the AI) will perform this test to prove completion:

1. **Start the Backend:** `sudo node backend/server.js` (Server listening on Port 4000).
2. **Send a Request (via Curl or Postman):**
```json
POST http://localhost:4000/start
{
  "vmId": "user-api-test",
  "projectPath": "/tmp/vessel-app"
}

```


3. **Result:**
* The server logs show: "Creating VM...", "Syncing Code...", "Adding Routes...".
* The Response returns: `{ "ideUrl": "http://user-api-test.localhost", ... }`
* You can open that URL in the browser and it works.



---

### **5. Summary Checklist for the Builder**

If you give this to an AI, tell them:

1. "Create a `backend` folder."
2. "Install `express` and `cors`."
3. "Create a server that imports `../infra/vm.js` and `../infra/proxy.js`."
4. "Implement a `POST /start` endpoint that chains the `create -> getIP -> copyTo -> addRoute` functions."
5. "Implement a `POST /stop` endpoint that chains `copyFrom -> removeRoute -> remove`."
6. "Ensure the server runs on Port 4000 (or any non-3000 port)."
7. "Remind me that I must run this server with `sudo`."

This is the complete definition of Phase 2.
