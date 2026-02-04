This is the **Architectural Blueprint for Phase 3**.

**Phase Name:** The Orchestrator (State Management & Persistence)
**Goal:** Transform the API from a "Dumb Trigger" into a "Smart Manager."
**Problem it Solves:** Currently, your server has **Amnesia**. If the Node process restarts, it forgets which VMs are running. It also has no idea if a VM has been abandoned by the user, wasting resources.

---

### **1. The Architecture**

We are introducing a **Persistence Layer** using **Redis**.

* **Before Phase 3:** API receives request -> Executes Script -> Returns URL. (Fire and Forget).
* **After Phase 3:** API receives request -> Checks Redis (Is user already running?) -> If No, Execute Script -> **Save State to Redis** -> Return URL.

**The Data Flow:**

1. **Client** calls `POST /start`.
2. **Session Manager** checks Redis for `session:userId`.
* *Hit:* Return existing URL immediately (0s latency).
* *Miss:* Call Infra to create VM.


3. **Session Manager** saves VM details (IP, ID, Port) to Redis with a **Heartbeat Timestamp**.
4. **The Reaper** (Background Process) runs every minute. It checks Redis for sessions where `lastActive > 30 mins`. It kills them to save RAM.

---

### **2. The New Directory Structure**

We are adding the **Database** connection and the **Logic Class** to the backend.

```text
/project-vessel
  ├── /backend
       ├── server.js           # (Modified) Uses the Manager instead of direct calls
       ├── routes.js           # (Modified) Routes traffic to the Manager
       ├── redisClient.js      # (NEW) The Connection to the Database
       ├── sessionManager.js   # (NEW) The Brain (Logic Layer)
       └── cronJobs.js         # (NEW) The Reaper (Cleanup Task)

```

---

### **3. Detailed Implementation Steps**

#### **Step 1: Install the Persistence Engine**

We need Redis to store the state in RAM for high speed.

1. **System Level:** Install Redis on your Host Machine.
* Command: `sudo apt install redis-server`
* Command: `sudo systemctl enable redis-server && sudo systemctl start redis-server`


2. **Project Level:** Install the client in your backend.
* Command: `cd backend && npm install redis`



#### **Step 2: Create the Database Connector (`redisClient.js`)**

This file ensures we have a stable connection.

* **Action:** Create a standard Redis client connection.
* **Logic:** Export the `client` object so other files can perform `client.get()` and `client.set()`.
* **Constraint:** Handle connection errors gracefully (log them, don't crash the server).

#### **Step 3: Build the Orchestrator (`sessionManager.js`)**

This is the most critical file. It wraps your low-level infrastructure.

**Function A: `startSession(userId, projectPath)**`

1. **Check Cache:** Query Redis for key `session:{userId}`.
* If found: Parse the JSON. Ping the VM IP to ensure it's actually alive. Return the existing data.


2. **Cold Start:** If not found:
* Generate a unique `vmId` (e.g., `user-{userId}-session`).
* Call `infra/vm.js` -> `create`, `copyTo`.
* Call `infra/proxy.js` -> `addRoute`.


3. **Persist:** Save to Redis:
* Key: `session:{userId}`
* Value: `{ vmId: "...", ip: "...", url: "...", lastHeartbeat: Date.now() }`


4. **Return:** The Session Object.

**Function B: `stopSession(userId)**`

1. **Lookup:** Get `vmId` from Redis using `userId`.
2. **Teardown:**
* Call `infra/proxy.js` -> `removeRoute`.
* Call `infra/vm.js` -> `remove`.


3. **Clean DB:** Delete `session:{userId}` from Redis.

**Function C: `heartbeat(userId)**`

1. **Update:** Find the session in Redis and update `lastHeartbeat` to `Date.now()`.
2. **Purpose:** The Frontend will call this every 5 minutes to say "I am still here, don't kill me."

#### **Step 4: implement "The Reaper" (`cronJobs.js`)**

This prevents your server from running out of RAM due to abandoned VMs.

* **Logic:**
1. Use `setInterval` to run a function every 60 seconds.
2. Get **ALL** keys matching `session:*`.
3. Loop through them.
4. Check: `if (Date.now() - session.lastHeartbeat > 30 * 60 * 1000)` (30 Minutes).
5. If True: Call `sessionManager.stopSession(userId)`.
6. Log: "Reaper killed zombie session for User X".



#### **Step 5: Wiring it to the API (`routes.js`)**

Refactor your routes to stop calling `infra` directly.

* `POST /start`: Calls `sessionManager.startSession`.
* `POST /stop`: Calls `sessionManager.stopSession`.
* `POST /heartbeat`: Calls `sessionManager.heartbeat`. (This is the new endpoint).
* `GET /status/:userId`: Returns the data from Redis (for the frontend loading screen).

---

### **4. Expected Outcome of Phase 3**

When this phase is complete, your system becomes **Production Grade**.

**Validation Test:**

1. **Start:** Call `POST /start` for User A. (Takes 5 seconds to boot).
2. **Repeat:** Call `POST /start` for User A *again*. (Takes 0.01 seconds - returns cached URL).
3. **Status:** Call `GET /status/User-A`. Returns `{ status: "active", ip: "10.61.0.x" }`.
4. **Reaper:** Manually set the `lastHeartbeat` in Redis to 1 hour ago. Wait 1 minute.
* **Result:** The VM should disappear from `sudo ignite ps`, and the route should vanish.



**Summary for the Builder:**

* "Implement Redis integration."
* "Create a `SessionManager` class to handle caching and wrapping infra calls."
* "Implement a `heartbeat` mechanism."
* "Implement a background `cron` job to auto-kill inactive VMs."

This completes the backend architecture. The next phase (Phase 4) would be the **Frontend Integration**.
