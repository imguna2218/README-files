This is the **Architectural Blueprint for Phase 4**.

**Phase Name:** Asynchronous Process Management & Real-Time Observability
**Goal:** Transform the Backend from a "Blocking Script Runner" to a "Real-Time Streaming Server".
**The Problem:** Currently, your `infra/vm.js` uses `execSync`. This means when a user starts a VM, your entire Node.js server **freezes** for 5-10 seconds until the VM is ready. If 10 users click "Start" at once, the server crashes or hangs.
**The Solution:** We must move to **Asynchronous Streams** (Node.js `spawn`) and use **WebSockets** to stream the "Booting..." logs to the frontend.

---

### **1. The Architecture**

We are changing the communication protocol.

* **Before Phase 4 (HTTP Only):**
* Frontend sends `POST /start`.
* ... (Browser spins and waits 10s) ...
* Backend responds `{ url: "..." }`.


* **After Phase 4 (HTTP + WebSockets):**
* **Frontend** connects to WebSocket (`socket.io`).
* **Frontend** sends `POST /start`.
* **Backend** immediately responds: `{ status: "processing", jobId: "123" }`.
* **Backend** spawns a background process to create the VM.
* **Backend** streams logs via WebSocket:
* `event: log` -> "Creating VM..."
* `event: log` -> "Syncing files..."
* `event: success` -> `{ url: "..." }`.





---

### **2. The New Directory Structure**

We need a Socket Manager and we need to refactor the Infra layer to support streaming.

```text
/project-vessel
  ├── /backend
       ├── server.js          # (Modified) Mounts the Socket.io server
       ├── socketManager.js   # (NEW) Handles the live connections
       └── ...
  ├── /infra
       ├── vm.js              # (HEAVY REFACTOR) execSync -> spawn
       └── ...

```

---

### **3. Detailed Implementation Steps**

#### **Step 1: Install WebSocket Libraries**

We need `socket.io` to handle the real-time pipe between the user and the server.

* **Action:** Inside `/backend`, run `npm install socket.io`.

#### **Step 2: Create the `socketManager.js**`

This file manages the "Live Lines" to the users.

* **Logic:**
1. Initialize `io` (Socket.io instance).
2. Listen for `connection`.
3. Join the user to a "Room" based on their UserID (e.g., `room-user-1`).
4. Export a function `emitToUser(userId, event, data)` that allows other parts of your backend to send messages to specific users.



#### **Step 3: Refactor `server.js` to support WebSockets**

Express and Socket.io need to share the same HTTP server.

* **Change:** instead of `app.listen()`, you will create a standard HTTP server:
```javascript
const server = http.createServer(app);
const io = new Server(server);

```


* **Change:** Initialize your `socketManager` with this `io` instance.

#### **Step 4: Refactor `infra/vm.js` (The Critical Step)**

We must stop using `execSync` for long tasks. We will switch to `child_process.spawn`.

* **Why?** `execSync` waits. `spawn` streams.
* **New Function Signature:** Change `create(id)` to accept a **callback** or return a **Stream**.
* **Logic:**
1. Instead of just running the command, spawn it: `const process = spawn('sudo', ['ignite', 'run', ...])`.
2. Listen to `process.stdout.on('data', ...)`:
* When data comes in, convert it to a string.
* Call `socketManager.emitToUser(userId, 'log', logData)`.


3. Listen to `process.on('close', ...)`:
* When the command finishes, trigger the next step (getting IP).





#### **Step 5: Update the `SessionManager**`

Your Orchestrator (Phase 3) needs to handle this non-blocking flow.

* **Logic for `startSession`:**
1. Check Redis.
2. If Miss:
* Call `vm.create` (Async Mode).
* **Do not wait.** Return `{ status: "initializing" }` to the API immediately.
* Inside the `vm.create` callbacks, chain the next steps (`copyTo` -> `addRoute`).
* On final success, emit `socket.emit('session:ready', { url: ... })`.





---

### **4. Expected Outcome of Phase 4**

When this phase is complete, your system feels "Alive".

**Validation Test:**

1. **Frontend/Postman:** Connect to the WebSocket.
2. **Action:** Send `POST /start`.
3. **Result:**
* API returns `200 OK` instantly.
* WebSocket receives a stream of messages:
* `[LOG] Cleanup old VM...`
* `[LOG] Starting Ignite...`
* `[LOG] Syncing files...`
* `[READY] { url: "http://user-1.localhost" }`





**Summary for the Builder:**

* "Install `socket.io`."
* "Refactor `server.js` to bind Socket.io to the HTTP server."
* "Create `socketManager.js` to handle user rooms."
* "**Refactor `infra/vm.js**`: Replace `execSync` with `spawn` for the `create` and `copyTo` functions to prevent server freezing."
* "Pipe the `stdout` of the spawn commands to the Socket Manager so the user sees the logs."

This completes the **System Core**. You now have a non-blocking, scalable, real-time Cloud IDE backend.
