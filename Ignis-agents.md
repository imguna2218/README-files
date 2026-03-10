# vessel-agent

## Overview
`vessel-agent` is the AI orchestration service responsible for processing user prompts, interacting with Large Language Models (LLMs), and generating structured code changes. It acts as the intelligence layer of IgnisOps, translating natural language instructions into actionable file operations. The service implements a three-tier agent architecture (Planner → Coder → Verifier) to ensure reliable and accurate code generation.

## Core Responsibilities
- **Prompt Processing**: Receive user prompts from `vessel-backend` with full project context
- **LLM Management**: Handle communication with multiple LLM providers (Gemini, OpenAI, etc.)
- **API Key Orchestration**: Smart rotation of API keys to manage rate limits
- **Three-Tier Agent Architecture**: Coordinate Planner, Coder, and Verifier agents
- **Structured Output Generation**: Produce JSON-formatted actions for backend execution
- **Context Management**: Utilize Gemini's context caching for efficiency
- **File Operations**: Generate precise file creation, modification, and deletion instructions

## Three-Tier Agent Architecture

### 1. Planner Agent
The Planner is the first stage of processing. It analyzes the user's prompt along with the current project context and breaks down the request into a structured plan.

**Input:**
- User prompt (from VS Code extension)
- Project context:
  - File tree structure
  - Contents of currently open file
  - Recently modified files
  - Project stack/language
- System prompt (predefined)

**Process:**
- Uses Gemini context caching to maintain conversation history
- Analyzes requirements and existing codebase
- Generates a task breakdown in JSON format

**Output Example:**
```json
{
  "plan_id": "plan_123",
  "tasks": [
    {
      "task_id": "task_1",
      "type": "create_file",
      "description": "Create a new Express route file for user authentication",
      "file_path": "routes/auth.js",
      "dependencies": ["express", "bcrypt"]
    },
    {
      "task_id": "task_2",
      "type": "modify_file",
      "description": "Update app.js to include the new auth routes",
      "file_path": "app.js",
      "changes": "Add import and use statement for auth routes"
    },
    {
      "task_id": "task_3", 
      "type": "install_package",
      "description": "Install bcrypt for password hashing",
      "command": "npm install bcrypt"
    }
  ],
  "reasoning": "User requested authentication system. Need to create routes, update main app, and install dependencies."
}
```

### 2. Coder Agent
The Coder takes each task from the Planner and generates precise implementation details.

**Input:**
- Individual task from Planner
- Relevant file contents (if modifying existing files)
- Code patterns and best practices for the stack
- System prompt (predefined for coding)

**Process:**
- For each task, generates exact code/content
- Ensures syntax correctness and follows project patterns
- Produces structured actions for the backend

**Output Example:**
```json
{
  "actions": [
    {
      "type": "create_file",
      "path": "routes/auth.js",
      "content": "const express = require('express');\nconst router = express.Router();\nconst bcrypt = require('bcrypt');\n\n// Login route\nrouter.post('/login', async (req, res) => {\n  // ... implementation\n});\n\nmodule.exports = router;",
      "task_id": "task_1"
    },
    {
      "type": "modify_file",
      "path": "app.js",
      "diff": "@@ -1,4 +1,5 @@\n const express = require('express');\n+const authRoutes = require('./routes/auth');\n const app = express();\n \n+app.use('/auth', authRoutes);",
      "task_id": "task_2"
    },
    {
      "type": "run_command",
      "command": "npm install bcrypt --save",
      "task_id": "task_3"
    }
  ]
}
```

### 3. Verifier Agent
The Verifier reviews the generated actions before they're sent to the backend.

**Input:**
- Original user prompt
- Planner's task breakdown
- Coder's generated actions
- Current project state

**Process:**
- Validates that all requested features are addressed
- Checks for potential conflicts or errors
- Ensures code follows best practices
- Verifies no security issues introduced

**Output Example:**
```json
{
  "verification_id": "ver_123",
  "status": "approved", // or "needs_revision"
  "issues": [], // empty if approved
  "suggestions": ["Consider adding input validation"],
  "approved_actions": [...] // original or modified actions
}
```

## LLM Integration & API Key Management

### Provider Support
- **Gemini**: Primary LLM with context caching support
- **OpenAI**: Fallback/secondary provider
- **Claude**: Optional for complex tasks
- **Custom Fine-tuned Models**: Future support

### API Key Rotation
The service implements intelligent API key rotation to handle rate limits:

```javascript
class KeyManager {
  constructor(keys) {
    this.keys = keys.map(key => ({
      value: key,
      usage: 0,
      lastReset: Date.now(),
      rpm: 60 // requests per minute limit
    }));
  }

  getNextKey() {
    // Round-robin with usage tracking
    // Reset counters every minute
    // Skip keys at their limit
    return availableKey;
  }

  reportUsage(keyId, tokens) {
    // Track token usage for cost management
  }
}
```

### Context Caching (Gemini)
- Cache project context to reduce token usage
- Cache common code patterns and responses
- Invalidate cache on significant file changes

## Communication Flow

```
1. User Prompt → vessel-backend → vessel-agent

2. vessel-agent:
   ├─→ Planner (with context)
   │     ↓
   │   Task JSON
   │     ↓
   ├─→ Coder (per task)
   │     ↓
   │   Actions JSON
   │     ↓
   └─→ Verifier
         ↓
      Verified Actions

3. Verified Actions → vessel-backend → VM Execution
```

## API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/agent/process` | POST | Process a user prompt (full pipeline) |
| `/agent/stream` | WS | WebSocket for streaming responses |
| `/agent/status` | GET | Agent health and queue status |
| `/agent/keys/rotate` | POST | Manually trigger key rotation |
| `/agent/cache/clear` | POST | Clear context cache for project |

## Prompt Templates

### Planner System Prompt (Simplified)
```
You are a senior software architect. Break down the user's request into specific tasks.
Consider the existing code structure and best practices for {stack}.
Output tasks in JSON format with clear descriptions and file paths.
```

### Coder System Prompt (Simplified)
```
You are an expert {language} developer. Generate precise code for the given task.
Follow the project's existing patterns and style.
Ensure all code is production-ready and includes error handling.
```

### Verifier System Prompt (Simplified)
```
You are a code reviewer. Verify that the generated code:
1. Correctly implements the user's request
2. Follows best practices and is secure
3. Integrates properly with existing code
4. Has no syntax errors or logical flaws
```

## Environment Configuration
```
# LLM Providers
GEMINI_API_KEY=key1,key2,key3  # multiple keys for rotation
OPENAI_API_KEY=key1,key2
CLAUDE_API_KEY=...

# Provider Settings
DEFAULT_PROVIDER=gemini
FALLBACK_PROVIDER=openai
MAX_TOKENS=8192
TEMPERATURE=0.2

# Key Rotation
KEY_REFRESH_INTERVAL=60000  # 1 minute
RATE_LIMIT_WINDOW=60  # seconds
MAX_REQUESTS_PER_MINUTE=60

# Caching
ENABLE_CONTEXT_CACHING=true
CACHE_TTL=300  # 5 minutes
CACHE_MAX_SIZE=100MB

# Agent Settings
PLANNER_MODEL=gemini-1.5-pro
CODER_MODEL=gemini-1.5-flash
VERIFIER_MODEL=gemini-1.5-flash
```

## Assumptions & Constraints
- Each agent operates independently but sequentially
- Verifier can reject actions and request regeneration
- API keys are provided via environment variables
- Context caching significantly reduces costs
- All agents share the same infrastructure but can be scaled separately

## Future Enhancements
- **Fine-tuned Models**: Custom models for specific stacks
- **Parallel Processing**: Multiple coder agents for large tasks
- **Learning System**: Improve prompts based on user feedback
- **Code Analysis**: Integrate static analysis tools pre-verification
- **Multi-modal Support**: Handle image-to-code scenarios
- **Custom Agent Chains**: User-configurable agent workflows
