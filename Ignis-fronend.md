# vessel-frontend

## Overview
`vessel-frontend` is the Next.js web application that serves as the primary user interface for IgnisOps. It provides a seamless experience similar to StackBlitz, where users can manage their cloud development environments, create new projects from templates, and access their existing workspaces. The frontend communicates with `vessel-backend` via REST APIs and WebSockets to orchestrate VM lifecycle, project management, and real-time updates.

## Core Responsibilities
- **User Authentication**: Handle sign-up, login, and session management
- **Project Dashboard**: Display list of user projects with metadata (last opened, stack, status)
- **Template Selection**: Provide modal with available project templates (static data)
- **Project Creation**: Allow users to create new projects with chosen stack and configuration
- **Project Management**: Open, delete, and manage existing projects
- **Backend Communication**: Make API calls to vessel-backend for all operations
- **Real-time Updates**: WebSocket connection for live status updates (VM state, deployment readiness)

## Key Pages & Components

### 1. Authentication Pages
- **Login/Signup**: Email/password authentication (via Supabase Auth)
- **OAuth Options**: Google/GitHub integration (optional)
- **Session Persistence**: Maintain user session across page reloads

### 2. Dashboard (Main View)
The landing page after authentication:
- **Header**: User profile, logout, settings
- **Projects Grid/List**: 
  - Project name, stack icon, last modified date
  - Status indicator (VM running/stopped)
  - Quick actions: Open, Delete, Deploy (if applicable)
- **"New Project" Button**: Triggers template selection modal
- **Search/Filter**: Filter projects by name or stack

### 3. Template Selection Modal
When user clicks "New Project":
- **Template Categories**: 
  - Full Stack (React + Java, React + Spring, React + Node)
  - Frontend (Next.js alone, React alone)
  - Backend (Node.js API, Spring Boot, Python FastAPI)
  - Blank Project (empty workspace)
- **Template Cards**: Each shows:
  - Stack name (e.g., "React + Spring Boot")
  - Description (e.g., "React frontend with Spring Boot REST API")
  - Icon/visual representation
- **Configuration Options** (if applicable):
  - Project name input
  - Advanced: CPU/Memory allocation (optional, with defaults)
- **"Create Project" Button**: Triggers creation API call

### 4. Project Workspace View
After opening a project (redirects to VS Code in new tab):
- **Loading State**: While backend provisions VM
- **Success**: Redirects to `http://user-{id}-project-{id}.localhost:3000`
- **Fallback**: If VM fails, show error with retry option

### 5. Settings/Profile Page
- User account management
- API keys for AI providers (if user brings their own)
- Billing/usage (future)

## API Integration

### REST Endpoints Consumed from vessel-backend

| Frontend Action | Backend Endpoint | Method | Description |
|-----------------|------------------|--------|-------------|
| Login/Signup | Supabase Auth directly | POST | Handle authentication |
| Fetch projects | `/api/projects/list` | GET | Get user's projects |
| Create project | `/api/projects/create` | POST | Create new project with template |
| Open project | `/api/projects/open` | POST | Get VM access info |
| Delete project | `/api/projects/delete` | POST | Delete project permanently |
| Cleanup project | `/api/cleanup` | POST | Trigger manual cleanup |

### WebSocket Connection
- Connect to `ws://vessel-backend/ws?userId={id}`
- Listen for:
  - VM status changes (provisioning/running/stopped)
  - Deployment readiness (when user app starts on a port)
  - Error notifications

## Template Data Structure (Static)
Templates are defined as static data in the frontend:

```javascript
const templates = [
  {
    id: 'react-node-fullstack',
    name: 'React + Node.js Full Stack',
    category: 'fullstack',
    stack: 'react+node',
    description: 'React frontend with Node.js/Express backend',
    icon: '/icons/react-node.png',
    boilerplate_id: 'react-node-starter' // matches Supabase storage key
  },
  {
    id: 'nextjs-standalone',
    name: 'Next.js Standalone',
    category: 'frontend',
    stack: 'nextjs',
    description: 'Next.js application with App Router',
    icon: '/icons/nextjs.png',
    boilerplate_id: 'nextjs-starter'
  },
  // ... more templates
]
```

## Key User Flows

### Flow 1: User Signs Up / Logs In
1. User visits `/login`
2. Authenticates via email/password or OAuth
3. Redirected to dashboard
4. Dashboard fetches `/api/projects/list` to display existing projects

### Flow 2: User Creates New Project
1. Clicks "New Project" on dashboard
2. Modal opens with template grid
3. Selects template (e.g., "React + Spring Boot")
4. Enters project name "My Spring App"
5. Clicks "Create"
6. Frontend calls `POST /api/projects/create` with:
   ```json
   {
     "template_id": "react-spring-fullstack",
     "project_name": "My Spring App",
     "user_id": "user_123"
   }
   ```
7. Backend provisions VM, injects template, returns VS Code URL
8. Frontend redirects to `http://user-123-project-xyz.localhost:3000`

### Flow 3: User Views and Opens Existing Project
1. Dashboard loads project list from backend
2. Each project card shows:
   - Name, stack icon, last opened timestamp
   - Status badge (Running/Stopped)
3. Clicks "Open" on a project
4. Frontend calls `POST /api/projects/open` with `project_id`
5. If VM exists: returns URL immediately
6. If VM needs restoration: shows loading spinner until backend restores
7. Redirects to VS Code URL

### Flow 4: User Deletes Project
1. Clicks delete icon on project card
2. Confirmation modal appears
3. On confirm, calls `POST /api/projects/delete` with `project_id`
4. On success, removes project from UI list

## Component Structure (Planned)
```
src/
├── components/
│   ├── auth/
│   │   ├── LoginForm.tsx
│   │   └── SignupForm.tsx
│   ├── dashboard/
│   │   ├── ProjectCard.tsx
│   │   ├── ProjectsGrid.tsx
│   │   └── DashboardHeader.tsx
│   ├── modals/
│   │   ├── TemplateModal.tsx
│   │   ├── TemplateCard.tsx
│   │   └── ConfirmDeleteModal.tsx
│   └── common/
│       ├── Loader.tsx
│       └── ErrorBoundary.tsx
├── pages/
│   ├── index.tsx (dashboard)
│   ├── login.tsx
│   ├── signup.tsx
│   └── settings.tsx
├── hooks/
│   ├── useAuth.ts
│   ├── useProjects.ts
│   └── useWebSocket.ts
├── services/
│   ├── api.ts (REST client)
│   └── websocket.ts
└── utils/
    └── constants.ts (templates, config)
```

## State Management
- **Authentication State**: React Context or Zustand store
- **Projects List**: React Query (SWR) for caching and background updates
- **Modal State**: Local component state or context
- **WebSocket Connection**: Custom hook with reconnection logic

## Environment Variables
```
NEXT_PUBLIC_SUPABASE_URL=...
NEXT_PUBLIC_SUPABASE_ANON_KEY=...
NEXT_PUBLIC_BACKEND_URL=http://localhost:8080
NEXT_PUBLIC_WS_URL=ws://localhost:8080
```

## Assumptions & Constraints
- Templates are static data in frontend (can be updated via deployment)
- Project list is fetched on every dashboard visit (or cached with SWR)
- VS Code opens in same tab or new tab (configurable)
- WebSocket connection is established after authentication for real-time updates
- Responsive design for desktop (primary) with mobile adaptation (secondary)

## Future Enhancements
- **Deployment View**: Allow one-click deployment to cloud platforms
- **Template Preview**: Show screenshot/gif of template before creation
- **Usage Analytics**: Display resource usage per project
- **Team Collaboration**: Share projects with team members
- **Custom Templates**: Allow users to save custom templates
