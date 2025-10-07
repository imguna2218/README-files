After deep analysis of your codebase and requirements, I'll be honest: **your current architecture has fundamental flaws that make language addition painful**. The solutions you considered are over-engineered. Here's a simple, robust alternative.

## Current Solution:
- Hardcoded language logic scattered across multiple files
- Complex match statements in isolate.rs
- Manual worker registration in setup.rs
- No separation between language definitions and execution logic

## My Proposed Solution:
**Configuration-Driven Language Registry**
- Centralized language definitions in config files
- Dynamic language discovery at startup  
- Clean separation between language specs and execution
- Zero code changes for new languages

## Differences:
- **Simplicity**: No complex trait systems or plugins
- **Maintainability**: All language specs in one place
- **Flexibility**: Hot-reload capable without recompilation
- **Minimal Changes**: Preserves your exact current workflow

---

## 'Configuration-Driven Language Registry' Implementation:

### Step 1.1: Create Language Configuration System
**Goal**: Centralize all language definitions in external config files

**What to do**:
- Create `config/languages/` directory with individual language TOML files
- Define language specs: compile commands, run commands, resource limits, file extensions
- Create language registry module that loads these configs at startup
- Replace all hardcoded language logic with config lookups

**Implementation Details**:
- Use TOML for language configuration (simple, readable)
- Load configs during application initialization  
- Validate language configurations at startup
- Fallback to default values for missing configs

**Why it helps**:
- Single source of truth for all language definitions
- Add new languages by simply creating a config file
- No code changes required for new language support

**Files to change**:
- New: `config/languages/` directory with `python.toml`, `java.toml`, etc.
- New: `src/language_registry.rs` - loads and manages language configs
- Modify: `src/setup.rs` - use registry instead of hardcoded language list

### Step 1.2: Refactor Isolate Sandbox to be Language-Agnostic  
**Goal**: Remove all hardcoded language logic from isolate.rs

**What to do**:
- Replace massive match statements with config lookups
- Extract language-specific commands and paths from config registry
- Make compile() and run() methods generic, driven by language config
- Keep sandbox security and process logic intact

**Implementation Details**:
- Pass LanguageConfig to sandbox methods instead of language strings
- Build isolate commands dynamically from config values
- Move all file naming and path logic to language configs
- Preserve existing Java optimization logic but make it config-driven

**Why it helps**:
- Isolate.rs becomes truly generic and stable
- Language changes don't require sandbox code modifications
- Clean separation between execution engine and language specs

**Files to change**:
- Modify: `src/sandbox/isolate.rs` - remove match statements, add config parameter
- Modify: `src/executor.rs` - pass language config to sandbox calls
- Modify: `src/language_registry.rs` - add config validation logic

### Step 1.3: Dynamic Worker Registration
**Goal**: Automatically start workers for all configured languages

**What to do**:
- Replace hardcoded language list in setup.rs with dynamic registry query
- Start one worker per language-version combination from configs
- Add health checks for language availability
- Implement graceful degradation for misconfigured languages

**Implementation Details**:
- Query language registry for all available languages and versions
- Spawn workers dynamically based on registry contents
- Log warnings for languages with missing dependencies
- Maintain current worker scaling and queue management unchanged

**Why it helps**:
- Workers automatically adapt to available languages
- No code changes needed when adding/removing languages
- Better error handling for configuration issues

**Files to change**:
- Modify: `src/setup.rs` - dynamic worker spawning from registry
- Modify: `src/queue_management/manager.rs` - update queue key generation
- Modify: `src/main.rs` - initialize language registry early

### Step 1.4: Enhanced Language Configuration Schema
**Goal**: Comprehensive language specification covering all execution aspects

**What to do**:
- Define structured TOML schema for language properties
- Include compilation commands, execution commands, resource limits
- Add dependency checks and environment requirements
- Support multiple versions of same language

**Implementation Details**:
- Standardized TOML structure across all language files
- Validation rules for required and optional fields
- Default values for common properties
- Version-specific overrides and extensions

**Why it helps**:
- Consistent configuration experience
- Clear documentation through schema
- Easy troubleshooting of language issues
- Support for complex language setups

**Files to change**:
- New: `config/languages/python.toml`, `java.toml`, `cpp.toml` etc.
- New: `config/languages/README.md` - configuration guide
- Modify: `src/language_registry.rs` - add schema validation

---

## Why This Solution Beats Your Previous Ideas:

**Simplicity**: No complex trait systems, dependency injection, or plugin architectures
**Robustness**: Configuration validation catches errors early
**Maintainability**: All language logic in one place, clear separation of concerns
**Zero Workflow Changes**: Your current API, queue system, monitoring all remain identical
**Future-Proof**: Easy to extend config schema for new requirements

This approach gives you 90% of the benefits of more complex solutions with 10% of the complexity and risk. You can implement it incrementally without breaking existing functionality.
