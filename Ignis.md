# IgnisOps - AI Integrated Cloud Dev Environment

## GOAL
Provide an AI-powered cloud development environment where users can create, edit, and manage projects of any stack in isolated VMs, with an integrated AI assistant that generates and modifies code based on natural language instructions.

## PROBLEM & SOLUTION
**The Problem:**  
Existing cloud IDEs like StackBlitz are limited to frontend frameworks and run entirely in the browser, making them unsuitable for backend development (Java, Python, Node.js) or running full-stack applications. Users often need to set up local environments, deal with dependency hell, and cannot leverage AI assistance deeply integrated into their workflow.

**What IgnisOps Solves:**  
- **Full-Stack Support**: Run Java, Python, Node.js – the "big three" – in isolated microVMs.  
- **AI-Native Experience**: Built-in AI agent (Planner → Coder → Verifier) that understands your codebase and performs file operations, not just chat.  
- **Ephemeral yet Persistent**: VMs are destroyed on inactivity, but your code is automatically saved and restored.  
- **No Port Hassles**: Each project gets a unique subdomain (`user-{id}-project-{x}.localhost`) with automatic port mapping.  
- **VS Code in the Browser**: Full IDE experience with a custom AI extension.

## STACK
- **Frontend**: Next.js (latest)
- **Backend Services**: Node.js (latest)
- **Database & Storage**: Supabase (PostgreSQL + Storage)
- **Caching & Queue**: Redis (latest)
- **Containerization**: Docker (for base images)
- **MicroVM Orchestration**: Ignite Firecracker
- **AI Integration**: Gemini / other LLMs via API keys

## ARCHITECTURE
Microservices architecture with the following core components:
```
/
├── vessel-infra/
│   – Low-level VM lifecycle management (initialization, deletion, cleanup)
│   – Interfaces with Ignite Firecracker to create isolated environments per project
│   – Handles network setup and resource allocation
│
├── vessel-backend/
│   – Orchestrates communication between frontend, agent service, and infra
│   – Manages project metadata, user sessions, and VM state
│   – Handles inactivity timeouts and triggers cleanup (with project persistence)
│
├── vessel-agent/
│   – AI agent service that receives user prompts and communicates with LLMs
│   – Integrates with VS Code server extension for in-editor AI assistance
│   – Processes generated code and instructs backend to create/update files
│
├── vessel-frontend/
│   – Next.js web application (similar to StackBlitz)
│   – Lists the users projects, Deploying Feasibility, Ability to choose the required Template for a Fresh start.
|   – Communicates with backend via REST/WebSockets
│
└── supabase/
    – Stores boilerplate zip files (per stack) and user project archives
    – Maintains project metadata, user data, and session info
```
## KEY NOTES
- **Isolation**: Each project runs in its own Firecracker microVM for strong isolation and resource control.
- **AI Integration**: AI assistance is provided via a dedicated agent service and a VS Code extension; users interact with AI through chat or inline prompts.
- **Project Persistence**: On VM shutdown (e.g., inactivity timeout), the project files are cleaned (node_modules, caches removed), zipped, and stored in Supabase with a project ID. The frontend fetches this list to let users resume projects.
- **Inactivity Cleanup**: If the browser tab is closed or idle for 30 seconds, the frontend notifies the backend, triggering VM destruction after ensuring project state is saved.
- **Templates**: Boilerplate code for common stacks (React+Node, Next.js, Spring, etc.) is stored as zip files in Supabase and used to initialize new projects.
- **Technology Choices**: Using Ignite Firecracker for fast-booting lightweight VMs; Supabase for simplicity (PostgreSQL + object storage); Redis for real-time coordination and job queuing.
