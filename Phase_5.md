This is the **Architectural Blueprint for Phase 5**.

**Phase Name:** The Frontend Interface (The "Face" of Vessel)
**Goal:** Build the actual website that users interact with.
**The Transformation:**

* **Currently:** You trigger the backend via Postman/Curl. You see JSON responses.
* **After Phase 5:** You have a modern, dark-mode web application. You click "Launch," watch a hacker-style terminal stream the boot logs in real-time (using the WebSockets from Phase 4), and then seamlessly transition into the VS Code environment.

---

### **1. The Architecture**

We are building a **Single Page Application (SPA)** using **React**.

* **Communication:**
* **REST API:** Used for the initial `POST /start` trigger.
* **WebSocket:** Used to listen for the live "Booting..." logs.
* **Iframe:** Used to embed the VS Code instance once it is ready.



---

### **2. The Directory Structure**

We are adding the `/frontend` folder to the root.

```text
/project-vessel
  ├── /backend          # (Existing) The API
  ├── /infra            # (Existing) The Engine
  └── /frontend         # (NEW) The Website
       ├── public/
       ├── src/
       │    ├── components/
       │    │    ├── TerminalView.jsx   # Xterm.js implementation
       │    │    └── IDEView.jsx        # The full-screen iframe
       │    ├── services/
       │    │    ├── api.js             # HTTP calls
       │    │    └── socket.js          # WebSocket connection
       │    └── App.jsx                 # The Main State Machine
       ├── package.json
       └── vite.config.js

```

---

### **3. Detailed Implementation Steps**

#### **Step 1: Initialize the Frontend**

* **Action:** In the root `/project-vessel`, create a React project.
* Command: `npm create vite@latest frontend -- --template react`


* **Dependencies:** You need specific libraries for the "Cloud IDE" feel.
* `axios`: For HTTP requests.
* `socket.io-client`: To talk to your Phase 4 backend.
* `xterm` & `xterm-addon-fit`: To render the "Boot Logs" like a real terminal (not just a text box).
* `tailwindcss` (Optional but recommended): For rapid dark-mode styling.



#### **Step 2: The Service Layer (`services/`)**

Don't clutter your UI code with connection logic. Isolate it.

* **`api.js`:**
* Create an Axios instance pointing to your backend (e.g., `http://localhost:4000`).
* Export a function `startSession(userId)` that sends the POST request.


* **`socket.js`:**
* Initialize the socket connection to `http://localhost:4000`.
* Export the socket instance so components can subscribe to events.



#### **Step 3: The "Hacker" Terminal Component (`TerminalView.jsx`)**

This is the "Wow Factor." When the user clicks start, they shouldn't just see a spinner. They should see the raw power of what you built.

* **Logic:**
1. Initialize `Xterm.js` on mount.
2. **Subscribe** to the `socket.on('log', (data) => ...)` event.
3. When data arrives, write it to the terminal: `term.write(data + '\r\n')`.
4. **Styling:** Make it black background, green/white text, monospace font.



#### **Step 4: The IDE Wrapper (`IDEView.jsx`)**

Once the VM is ready, we hide the terminal and show this.

* **Logic:**
1. Accept the `url` (e.g., `http://user-1.localhost`) as a prop.
2. Render a full-screen `<iframe>`.
3. **Crucial:** Set the `src` to the URL.
4. **Security:** Allow necessary permissions (clipboard, etc.) in the iframe attributes.



#### **Step 5: The Main State Machine (`App.jsx`)**

This controls the flow of the application.

* **States:**
* `IDLE`: Show a big "Launch Environment" button.
* `BOOTING`: Show the `TerminalView`.
* `READY`: Show the `IDEView`.
* `ERROR`: Show error message.


* **The Workflow (to implement):**
1. **User Clicks Start:**
* Set State -> `BOOTING`.
* Call `api.startSession()`.


2. **Listen to Socket:**
* On `log` -> Pass data to Terminal component.
* On `session:ready` ->
* Save the URL from the payload.
* Set State -> `READY`.




3. **Render:**
* If `READY`, render `<IDEView url={sessionUrl} />`.





---

### **4. Expected Outcome of Phase 5**

When this phase is complete, you have a **Demo-Ready Product**.

**The User Journey:**

1. Open `http://localhost:5173` (Frontend).
2. Click **"Start Coding"**.
3. **Instant Feedback:** The screen turns black. A terminal appears.
* `> Allocating resources...`
* `> Starting MicroVM...`
* `> Syncing codebase...`


4. **The Reveal:** The terminal vanishes, and a full VS Code environment loads instantly in the browser.

**Summary for the Builder:**

* "Initialize a Vite React app."
* "Install `socket.io-client` and `xterm`."
* "Create a `TerminalView` component that listens to socket logs and renders them using Xterm.js."
* "Create a main `App` component that manages the transition from 'Booting' (Terminal) to 'Ready' (Iframe) based on WebSocket events."

---

**This concludes the Core Development.**
Your phases now cover:

1. **Infra:** The Engine.
2. **API:** The Brain.
3. **Persistence:** The Memory.
4. **Real-Time:** The Nervous System.
5. **Frontend:** The Face.

You have the complete blueprint for **Vessel**.
