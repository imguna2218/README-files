# vessel-infra

## Overview
`vessel-infra` is the low-level infrastructure service responsible for managing the lifecycle of isolated development environments. It interfaces directly with Ignite (Firecracker) to create, manage, and destroy microVMs that serve as per-project development environments. Each VM comes pre-configured with Java, Python, Node.js, and an Open VS Code server, enabling users to code and run applications without worrying about underlying infrastructure.

## Core Responsibilities
- **VM Lifecycle Management**: Initialize, start, stop, and delete microVMs based on requests from the backend.
- **Network Configuration**: Set up routing and subdomains (`user-{id}-project-{x}.localhost`) for seamless access to VS Code and user applications.
- **Resource Allocation**: Assign CPU, memory, and disk resources per VM.
- **Port Management**: Expose predefined ports (3000, 5000, 8000, and up to 5 additional ports) for VS Code and user applications.
- **Image Preparation**: Ensure base VM images contain all required languages and tools (Java, Python, Node.js, Open VS Code Server).

## How It Works

### 1. Base Image Configuration
The base Ignite image includes:
- **Languages**: Java (OpenJDK 17), Python (3.10+), Node.js (18+)
- **Tools**: Open VS Code Server (running on port 3000), git, curl, wget, build-essential
- **Networking**: Preconfigured to accept connections on ports 3000, 5000, 8000 (expandable up to 5 additional ports)
- **User Environment**: A non-root user (`vessel`) with sudo privileges for package installation

This image is built once and used as the template for all user VMs, ensuring consistency and fast startup times.

### 2. VM Initialization Flow
When the backend requests a new VM:

1. **Receive Request**  
   `vessel-infra` accepts a JSON payload via HTTP POST (or eventually via internal messaging). Example:
   ```json
   {
     "action": "create",
     "project_id": "proj_123",
     "user_id": "user_456",
     "resources": {
       "cpus": 2,
       "memory_mb": 2048,
       "disk_mb": 10240
     },
     "exposed_ports": [3000, 5000, 8000, 8080, 3001]
   }
   ```

2. **Spin Up VM**  
   Using Ignite, a new microVM is created from the pre-built base image with the requested resources.

3. **Assign Subdomain**  
   A unique subdomain is generated: `user-456-project-123.localhost`.  
   This domain is mapped to the VM's IP via a local DNS resolver (or Traefik in production).

4. **Configure Routing**  
   - Port `3000` is permanently reserved for Open VS Code Server.  
   - Other exposed ports are dynamically routed:  
     `user-456-project-123.localhost:3000` → VS Code  
     `user-456-project-123.localhost:5000` → User app (if running)  
     `user-456-project-123.localhost:8000` → Another user app  
   - If the user starts a dev server on port 5173 (Vite default), they can access it via `user-456-project-123.localhost:5173` (provided the port was included in `exposed_ports`).

5. **Return Success Response**  
   ```json
   {
     "status": "success",
     "vm_id": "vm_789",
     "subdomain": "user-456-project-123.localhost",
     "ports": {
       "vscode": 3000,
       "available": [5000, 8000, 8080, 3001]
     }
   }
   ```

### 3. VM Deletion & Cleanup
When the backend requests deletion:

1. **Receive Request**  
   ```json
   {
     "action": "delete",
     "project_id": "proj_123",
     "user_id": "user_456"
   }
   ```

2. **Stop & Remove VM**  
   The microVM is stopped and removed using Ignite commands, freeing all resources.

3. **Clean Up Network Mappings**  
   Subdomain entries are removed from DNS/Traefik configuration.

4. **Acknowledge**  
   ```json
   { "status": "success", "message": "VM terminated" }
   ```

### 4. Inactivity & Auto-cleanup
While the backend tracks user inactivity, `vessel-infra` can also receive immediate delete requests if the frontend detects a tab close. The flow:

- Frontend detects inactivity/closing → notifies backend
- Backend requests `vessel-infra` to delete VM
- `vessel-infra` performs deletion and confirms
- Backend then handles project archiving (code zipping, storing in Supabase)

## API Endpoints (Initial Design)

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/vm/create` | POST | Create a new VM with given resources and ports |
| `/vm/delete` | POST | Delete a specific VM by project/user ID |
| `/vm/status` | GET | Get status of a VM (running/stopped) |
| `/vm/list` | GET | List all active VMs for a user |

## Environment & Configuration
`vessel-infra` requires:
- **Ignite** installed and configured
- **Base image** built and available
- **Network proxy** (Traefik or similar) for subdomain routing
- **Redis** (optional) for caching VM metadata

Example `.env`:
```
IGNITE_DATA_DIR=/var/lib/ignite
BASE_IMAGE_NAME=vessel-base:latest
TRAEFIK_NETWORK=traefik-proxy
SUBNET_PREFIX=10.10
```

## Assumptions & Constraints
- Each VM runs exactly one project.
- VS Code Server always runs on port 3000.
- Maximum of 5 additional ports per VM (configurable).
- Subdomain format: `user-{userId}-project-{projectId}.localhost`
- All VMs are ephemeral; persistent data is managed by the backend (Supabase storage).

## Future Considerations
- Auto-scaling pools of warm VMs for faster startup
- Direct integration with backend via message queue (Redis pub/sub)
- Enhanced monitoring and metrics per VM
- Custom image building per user request (if needed)
