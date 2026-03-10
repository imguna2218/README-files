# IgnisOps - AI Integrated Cloud Dev Environment

## GOAL
Provide an AI-powered cloud development environment where users can create, edit, and manage projects of any stack in isolated VMs, with an integrated AI assistant that generates and modifies code based on natural language instructions.

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
## Key Notes : 
- **Isolation**: Each project runs in its own Firecracker microVM for strong isolation and resource control.
- **AI Integration**: AI assistance is provided via a dedicated agent service and a VS Code extension; users interact with AI through chat or inline prompts.
- **Project Persistence**: On VM shutdown (e.g., inactivity timeout), the project files are cleaned (node_modules, caches removed), zipped, and stored in Supabase with a project ID. The frontend fetches this list to let users resume projects.
- **Inactivity Cleanup**: If the browser tab is closed or idle for 30 seconds, the frontend notifies the backend, triggering VM destruction after ensuring project state is saved.
- **Templates**: Boilerplate code for common stacks (React+Node, Next.js, Spring, etc.) is stored as zip files in Supabase and used to initialize new projects.
- **Technology Choices**: Using Ignite Firecracker for fast-booting lightweight VMs; Supabase for simplicity (PostgreSQL + object storage); Redis for real-time coordination and job queuing.
