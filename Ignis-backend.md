# vessel-backend

## Overview
`vessel-backend` is the orchestration layer and control plane for IgnisOps. It acts as the central nervous system, coordinating all communication between the frontend, the AI agent service, and the infrastructure service (`vessel-infra`). It manages user sessions, project lifecycle, VM state, and ensures that code persists across sessions. Additionally, it injects the Vessel AI extension into every VS Code instance and handles all communication between the user and the AI agents.

## Core Responsibilities
- **Orchestration**: Coordinate between frontend, agents, and infra services
- **Session Management**: Track user activity and VM assignments
- **Project Lifecycle**: Handle project creation (templates), saving, and restoration
- **File Management**: Inject zip files into VMs, extract code, and archive projects on shutdown
- **AI Communication**: Relay prompts from VS Code extension to agent service and stream responses back
- **Extension Management**: Inject and configure the Vessel AI extension in every VS Code server
- **Inactivity Handling**: Monitor idle time and trigger cleanup workflows

## How It Works

### 1. Project Creation Flow
When a user creates a new project from the frontend:

1. **Frontend Request**
   ```json
   {
     "action": "create_project",
     "user_id": "user_456",
     "project_name": "My Spring App",
     "template": "spring-react-fullstack",  // or null for empty project
     "stack": "java+react"
   }
   ```

2. **Fetch Boilerplate**  
   Backend queries Supabase for the requested template zip (if any) or prepares an empty workspace structure.

3. **Request VM from Infra**  
   Backend calls `vessel-infra` to spin up a new VM:
   ```json
   {
     "action": "create",
     "project_id": "proj_123",
     "user_id": "user_456",
     "resources": { "cpus": 2, "memory_mb": 2048 },
     "exposed_ports": [3000, 5000, 8000, 8080]
   }
   ```
   Infra responds with `vm_id`, `subdomain`, and port mappings.

4. **Inject Project Files**  
   Backend streams the template zip (or empty structure) into the VM at `/home/workspace` and unzips it via SSH/exec.

5. **Inject Vessel AI Extension**  
   The Vessel AI extension folder is copied into the VS Code extensions directory (`~/.vscode-server/extensions/`).  
   This extension handles:
   - Capturing user prompts from the VS Code interface
   - Sending prompts to `vessel-backend` (which forwards to agent service)
   - Receiving and applying AI-generated code changes

6. **Return VM Access Info to Frontend**
   ```json
   {
     "status": "success",
     "project_id": "proj_123",
     "vscode_url": "http://user-456-project-123.localhost:3000",
     "subdomain": "user-456-project-123.localhost",
     "available_ports": [5000, 8000, 8080]
   }
   ```

### 2. AI Prompt Handling Flow
When a user types a prompt in the Vessel AI extension:

1. **Extension Sends Prompt**  
   VS Code extension makes an HTTP call to `vessel-backend`:
   ```json
   {
     "action": "ai_prompt",
     "project_id": "proj_123",
     "user_id": "user_456",
     "prompt": "Create a REST API with Spring Boot",
     "context": {
       "current_files": ["pom.xml", "src/main/java/..."],
       "open_file": "src/main/java/Application.java"
     }
   }
   ```

2. **Backend Routes to Agent Service**  
   Backend forwards the prompt to `vessel-agent` (via HTTP or message queue), including project context.

3. **Agent Processes & Returns**  
   Agent communicates with LLM, generates file changes, and returns structured response:
   ```json
   {
     "actions": [
       { "type": "create_file", "path": "src/main/java/Controller.java", "content": "..." },
       { "type": "modify_file", "path": "pom.xml", "diff": "..." },
       { "type": "run_command", "command": "mvn spring-boot:run" }
     ]
   }
   ```

4. **Backend Applies Changes**  
   Backend executes the actions in the VM:
   - Writes/updates files via SSH/exec
   - Runs terminal commands if needed
   - Returns confirmation to extension

5. **Extension Updates UI**  
   VS Code extension refreshes the file tree, opens new files, and shows AI responses.

### 3. Project Persistence & Cleanup
When a user closes the tab or becomes inactive:

1. **Inactivity Detection**  
   Frontend detects 30 seconds of inactivity or tab close → calls backend:
   ```json
   { "action": "cleanup_project", "project_id": "proj_123", "user_id": "user_456" }
   ```

2. **Extract Code from VM**  
   Backend connects to the VM and:
   - Runs cleanup commands: `rm -rf /home/workspace/node_modules /home/workspace/target /home/workspace/__pycache__`
   - Zips the remaining `/home/workspace` contents

3. **Store in Supabase**  
   The cleaned zip is uploaded to Supabase storage under:
   `users/{user_id}/projects/{project_id}/{timestamp}.zip`  
   Metadata is updated in the database.

4. **Request VM Deletion**  
   Backend calls `vessel-infra` to delete the VM:
   ```json
   { "action": "delete", "project_id": "proj_123", "user_id": "user_456" }
   ```

5. **Confirm to Frontend**  
   Backend responds with success, and frontend updates the project list.

### 4. Project Restoration
When a user resumes a project:

1. **Frontend Requests Open**  
   ```json
   { "action": "open_project", "project_id": "proj_123", "user_id": "user_456" }
   ```

2. **Backend Checks Existing VM**  
   If VM exists and is running, return existing access info.  
   If not:

   a. Request new VM from infra (same as creation)  
   b. Fetch latest project zip from Supabase  
   c. Inject and unzip into `/home/workspace`  
   d. Inject Vessel AI extension  
   e. Return access info

## API Endpoints (Core)

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/projects/create` | POST | Create new project with template |
| `/api/projects/open` | POST | Open existing project (restore VM) |
| `/api/projects/list` | GET | List user's projects |
| `/api/projects/delete` | POST | Permanently delete project |
| `/api/ai/prompt` | POST | Handle AI prompt from extension |
| `/api/ai/stream` | WS | WebSocket for streaming AI responses |
| `/api/cleanup` | POST | Trigger project cleanup |
| `/api/status` | GET | Health check and VM status |

## Vessel AI Extension
The extension injected into VS Code is a custom VS Code extension that:
- Adds a chat panel/sidebar for AI interaction
- Captures context (open files, selected code, terminal output)
- Sends prompts to `vessel-backend`
- Receives and applies file changes
- Displays AI responses and suggestions
- Supports inline code completion (future)

The extension is built separately and its build output is stored in Supabase or directly in the backend repository for injection.

## How `vessel-agent` Works (Brief)

`vessel-agent` is a dedicated microservice that:

1. **Receives Prompts** from `vessel-backend` with full project context
2. **Manages LLM Connections** (Gemini, OpenAI, etc.) via API keys
3. **Constructs Effective Prompts** including file tree, open file content, and user intent
4. **Parses LLM Responses** into structured actions (file creations, modifications, commands)
5. **Returns Actions** to `vessel-backend` for execution

It is stateless and can scale independently. Communication between `vessel-backend` and `vessel-agent` is via REST or Redis pub/sub for reliability.

## Environment & Configuration
```
SUPABASE_URL=...
SUPABASE_SERVICE_KEY=...
REDIS_URL=...
INFRA_SERVICE_URL=http://vessel-infra:8080
AGENT_SERVICE_URL=http://vessel-agent:8081
VSCODE_EXTENSION_PATH=./extensions/vessel-ai
DEFAULT_RESOURCES_CPU=2
DEFAULT_RESOURCES_MEM=2048
INACTIVITY_TIMEOUT=30
```

## Assumptions
- Each VM has SSH or exec access for the backend to run commands
- VS Code server accepts extension injection via filesystem copy
- Supabase stores both boilerplate templates and user project archives
- The backend maintains a mapping of project_id ↔ vm_id ↔ subdomain
