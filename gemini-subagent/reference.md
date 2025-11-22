# Gemini CLI Complete Reference

Complete command-line reference for `@google/gemini-cli` v0.16.0+.

## Table of Contents

- [Installation & Setup](#installation--setup)
- [Core Command Flags](#core-command-flags)
- [Model Options](#model-options)
- [Session Management](#session-management)
- [Context & Tool Control](#context--tool-control)
- [Configuration](#configuration)
- [Interactive Commands](#interactive-commands)
- [Output Formats](#output-formats)
- [Examples](#examples)

## Installation & Setup

### Installation

```bash
# Install globally
npm install -g @google/gemini-cli

# Or use via npx (no installation)
npx @google/gemini-cli
```

### Authentication

**Option 1: API Key (Recommended for CI/CD)**
```bash
export GEMINI_API_KEY="your-api-key-here"
gemini -p "test prompt"
```

**Option 2: OAuth (Interactive)**
```bash
# First run will prompt for authentication
gemini -p "test prompt"
# Follow browser authentication flow
```

**Verify Authentication:**
```bash
# Should see "Loaded cached credentials" in stderr
gemini -p "hello" 2>&1 | grep -i credential
```

## Core Command Flags

### Essential Operations

#### `--prompt` / `-p`
Provide the prompt/query to Gemini:
```bash
gemini -p "Explain how async/await works in JavaScript"
```

#### `--yolo` / `-y`
Auto-approve all tool calls (bypass confirmation prompts):
```bash
gemini -y -p "Create a React component for user login"
```

**When to use:**
- Automation and CI/CD pipelines
- Trusted operations where manual approval slows workflow
- When you've reviewed the prompt and trust the operation

**When NOT to use:**
- Untrusted or experimental prompts
- Operations that modify critical files
- First-time testing of new commands

#### `--output-format` / `-o`
Control output format:
```bash
# Plain text (default)
gemini -o text -p "your prompt"

# JSON (for programmatic processing)
gemini -o json -p "your prompt"

# Streaming JSON (real-time response)
gemini -o stream-json -p "your prompt"
```

**JSON Output Structure:**
```json
{
  "candidates": [
    {
      "content": {
        "parts": [
          {
            "text": "Response text here..."
          }
        ]
      }
    }
  ],
  "usageMetadata": {
    "promptTokenCount": 123,
    "candidatesTokenCount": 456,
    "totalTokenCount": 579
  }
}
```

#### `--model` / `-m`
Select the AI model:
```bash
# Pro model (best quality, 15 req/min free tier)
gemini -m gemini-2.5-pro -p "your prompt"

# Flash model (fast, 60 req/min free tier)
gemini -m gemini-2.5-flash -p "your prompt"

# Flash Lite model (fastest, higher limits)
gemini -m gemini-2.5-flash-lite -p "your prompt"
```

See [Model Options](#model-options) for detailed comparison.

### Execution Control

#### `--sandbox` / `-s`
Run in isolated environment:
```bash
gemini -s -p "your prompt"
```

**Use when:**
- Testing untrusted code
- Preventing file system modifications
- Isolating experimental operations

#### `--approval-mode`
Set validation level for tool calls:
```bash
# Default: Manual approval for each tool call
gemini --approval-mode default -p "your prompt"

# Auto-edit: Auto-approve file edits, ask for others
gemini --approval-mode auto_edit -p "your prompt"

# YOLO: Auto-approve everything (same as -y flag)
gemini --approval-mode yolo -p "your prompt"
```

#### `--timeout`
Set request timeout in milliseconds:
```bash
# 5 minute timeout
gemini --timeout 300000 -p "complex analysis task"
```

**Note:** Gemini responses can take up to 2 minutes. Default timeout is usually sufficient.

#### `--checkpointing`
Enable file state tracking:
```bash
gemini --checkpointing -p "your prompt"
```

Saves file states before modifications for potential rollback.

### Diagnostic & Help

#### `--debug` / `-d`
Enable verbose logging:
```bash
gemini -d -p "your prompt"
```

Shows detailed execution information, API calls, and internal operations.

#### `--version` / `-v`
Display CLI version:
```bash
gemini -v
# Output: 0.16.0
```

#### `--help` / `-h`
Show help information:
```bash
gemini -h
```

#### `--list-extensions` / `-l`
Show installed extensions:
```bash
gemini -l
```

## Model Options

### Available Models

| Model | Context Window | Speed | Quality | Free Tier Limit | Best For |
|-------|---------------|-------|---------|----------------|----------|
| `gemini-2.5-pro` | 1M tokens | Slower | Highest | 15 req/min | Architecture, security audits, critical reviews |
| `gemini-2.5-flash` | 1M tokens | Fast | High | 60 req/min | Standard development tasks, code generation |
| `gemini-2.5-flash-lite` | 1M tokens | Fastest | Good | Higher limit | Simple queries, quota preservation |

### Model Selection Strategy

```bash
# Complex architecture decisions
gemini -m gemini-2.5-pro -p "Design a microservices architecture for..."

# Standard code generation
gemini -m gemini-2.5-flash -p "Create a React component for..."

# Simple queries
gemini -m gemini-2.5-flash-lite -p "Explain what this function does"
```

**Default Model:** If no model is specified, uses the default from config (typically `gemini-2.5-flash`).

## Session Management

### `--resume` / `-r`
Resume previous conversation:
```bash
# Resume latest session
gemini --resume latest -p "continue from where we left off"

# Resume specific session by index
gemini --resume 3 -p "add more details"
```

### `--list-sessions`
Display available sessions:
```bash
gemini --list-sessions
```

**Output example:**
```
0: Session from 2025-01-15 14:30 (5 messages)
1: Session from 2025-01-15 10:15 (12 messages)
2: Session from 2025-01-14 16:45 (3 messages)
```

### `--delete-session`
Remove specific session:
```bash
gemini --delete-session 2
```

### Session Use Cases

**Multi-turn analysis:**
```bash
# First interaction
gemini -p "Analyze the authentication system architecture" 2>/dev/null

# Continue analysis (remembers context)
gemini --resume latest -p "Now review the security implications" 2>/dev/null

# Deep dive
gemini --resume latest -p "Suggest improvements" 2>/dev/null
```

## Context & Tool Control

### `--include-directories`
Add workspace folders to context:
```bash
gemini --include-directories ./src,./lib -p "Analyze the codebase structure"

# Multiple directories
gemini --include-directories ./frontend,./backend,./shared -p "your prompt"
```

**Best practice:** Include only relevant directories to avoid context bloat.

### `--allowed-tools`
Restrict available tools:
```bash
# Only allow specific tools
gemini --allowed-tools "read_file,list_directory" -p "your prompt"

# Disable all tools
gemini --allowed-tools "" -p "your prompt"
```

**Available tools:**
- `read_file`
- `write_file`
- `list_directory`
- `google_web_search`
- `codebase_investigator`
- `save_memory`

### `--allowed-mcp-server-names`
Limit MCP server access:
```bash
gemini --allowed-mcp-server-names "server1,server2" -p "your prompt"
```

## Configuration

### Configuration Files

Gemini CLI loads config from (in priority order):

1. **Project level:** `./.gemini/config.json`
2. **User level:** `~/.config/gemini/config.json`
3. **System level:** `/etc/gemini/config.json`

### Configuration Example

```json
{
  "model": "gemini-2.5-flash",
  "approvalMode": "default",
  "timeout": 120000,
  "allowedTools": ["read_file", "write_file", "google_web_search"],
  "includedDirectories": ["./src"]
}
```

### Project Context (GEMINI.md)

Create `.gemini/GEMINI.md` in your project root:

```markdown
# Project Context

This is a Node.js e-commerce platform built with:
- TypeScript
- Express.js
- PostgreSQL
- Redis for caching
- JWT authentication

## Architecture

- `/src/routes` - API endpoints
- `/src/services` - Business logic
- `/src/models` - Database models
- `/src/middleware` - Auth, validation, error handling

## Coding Standards

- Use TypeScript strict mode
- Async/await for async operations
- Comprehensive error handling
```

Gemini will automatically use this context when working in the project.

### File Exclusion (.geminiignore)

Create `.geminiignore` to exclude files (gitignore syntax):

```
node_modules/
dist/
.env
*.log
coverage/
```

## Interactive Commands

When running Gemini interactively, use these commands:

| Command | Description |
|---------|-------------|
| `/help` | Show available commands |
| `/tools` | List available tools |
| `/stats` | Show token usage statistics |
| `/compress` | Compress conversation history |
| `/memory show` | Display saved memories |
| `/save [name]` | Save current session |
| `/resume [name]` | Resume saved session |
| `/exit` | Exit interactive mode |

### Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl+L` | Clear screen |
| `Ctrl+V` | Paste from clipboard |
| `Ctrl+Y` | Toggle YOLO mode |
| `Ctrl+C` | Cancel current operation |
| `Ctrl+D` | Exit |

## Output Formats

### Text Output (Default)

```bash
gemini -p "explain async/await" 2>/dev/null
```

Output: Plain text response, easy to read.

### JSON Output

```bash
gemini -o json -p "explain async/await" 2>/dev/null
```

**Use for:**
- Programmatic processing
- Extracting metadata (token counts)
- Building automation pipelines
- Structured data extraction

**Process JSON with jq:**
```bash
# Extract response text
gemini -o json -p "your prompt" 2>/dev/null | jq -r '.candidates[0].content.parts[0].text'

# Extract token count
gemini -o json -p "your prompt" 2>/dev/null | jq '.usageMetadata.totalTokenCount'
```

### Stream JSON Output

```bash
gemini -o stream-json -p "long analysis task" 2>/dev/null
```

**Use for:**
- Real-time response streaming
- Long-running tasks
- Building interactive UIs

## Examples

### Basic Usage

```bash
# Simple query
gemini -p "What are React hooks?" 2>/dev/null

# With specific model
gemini -m gemini-2.5-pro -p "Review this architecture" 2>/dev/null

# Auto-approve mode
gemini -y -p "Create a login component" 2>/dev/null
```

### Code Generation

```bash
# Generate code with auto-approval
gemini -y -m gemini-2.5-flash -p "Create a TypeScript function that validates email addresses using regex" 2>/dev/null

# Generate with JSON output
gemini -y -o json -m gemini-2.5-flash -p "Create a REST API endpoint" 2>/dev/null
```

### Code Review

```bash
# Thorough security review
gemini -m gemini-2.5-pro -p "Review this code for security vulnerabilities:

\`\`\`typescript
$(cat src/auth.ts)
\`\`\`

Focus on: authentication, authorization, input validation" 2>/dev/null
```

### Web Research

```bash
# Latest information
gemini -y -m gemini-2.5-flash -p "Search for the latest React 19 features and breaking changes" 2>/dev/null

# Compare technologies
gemini -y -p "Compare Next.js 15 vs Remix for server-side rendering in 2025" 2>/dev/null
```

### Architecture Analysis

```bash
# Analyze codebase
gemini -y --include-directories ./src -p "Analyze the architecture and identify potential bottlenecks" 2>/dev/null

# With specific focus
gemini --include-directories ./src,./lib -p "Identify circular dependencies and coupling issues" 2>/dev/null
```

### Session Continuity

```bash
# Multi-turn analysis
gemini -p "Analyze the authentication flow" 2>/dev/null
gemini --resume latest -p "What security improvements would you suggest?" 2>/dev/null
gemini --resume latest -p "How would you implement rate limiting?" 2>/dev/null
```

### Automation Pipeline

```bash
#!/bin/bash
# Automated code review pipeline

# Generate code
code=$(gemini -y -o text -m gemini-2.5-flash -p "Create a user registration endpoint" 2>/dev/null)

# Review generated code
review=$(gemini -y -o json -m gemini-2.5-pro -p "Review this code for security issues: $code" 2>/dev/null)

# Extract findings
findings=$(echo "$review" | jq -r '.candidates[0].content.parts[0].text')

# Check for critical issues
if echo "$findings" | grep -i "critical"; then
  echo "Critical security issues found!"
  exit 1
fi
```

### Background Execution

```bash
# Start long-running task in background
gemini -y -m gemini-2.5-pro -p "Comprehensive security audit of the entire codebase" 2>/dev/null > /tmp/audit.txt &
AUDIT_PID=$!

# Continue with other work...

# Check if completed
if ps -p $AUDIT_PID > /dev/null; then
  echo "Audit still running..."
else
  echo "Audit complete!"
  cat /tmp/audit.txt
fi
```

### Rate Limit Handling

```bash
# Batch operations with delays
for file in src/*.ts; do
  gemini -y -m gemini-2.5-flash -p "Review $file for issues" 2>/dev/null
  sleep 1  # Avoid rate limits
done

# Fallback to faster model
gemini -y -m gemini-2.5-flash -p "your prompt" 2>/dev/null || \
  gemini -y -m gemini-2.5-flash-lite -p "your prompt" 2>/dev/null
```

## Common Issues & Solutions

### "Rate limit exceeded"
**Solution:** Wait 60 seconds or switch to a faster model (`gemini-2.5-flash-lite`)

### "Authentication failed"
**Solution:** Set `GEMINI_API_KEY` environment variable or run interactive auth

### "Timeout error"
**Solution:** Increase timeout with `--timeout 300000` (5 minutes)

### "Context too long"
**Solution:** Reduce included directories or split prompt into smaller chunks

### Empty/incomplete responses
**Solution:** Check stderr for errors: `gemini -p "prompt" 2>&1 | less`

## Best Practices

1. **Suppress stderr in production:** Always use `2>/dev/null` for clean output
2. **Choose appropriate models:** Use Flash for routine tasks, Pro for critical analysis
3. **Set timeouts:** Use `--timeout` for long-running operations
4. **Use auto-approval wisely:** Only `-y` for trusted operations
5. **Validate all generated code:** Never deploy without security review and testing
6. **Manage rate limits:** Spread operations, use appropriate models
7. **Leverage sessions:** Use `--resume` for multi-turn analysis
8. **Include context:** Use `.gemini/GEMINI.md` for project-specific guidance
9. **Process JSON programmatically:** Use `-o json` with `jq` for automation
10. **Monitor token usage:** Use `-o json` to track token consumption

## Additional Resources

- **Official Docs:** https://github.com/google/generative-ai-js
- **API Reference:** https://ai.google.dev/api
- **Installation Guide:** `npm install -g @google/gemini-cli`
- **Issue Tracker:** https://github.com/google/generative-ai-js/issues

---

**Version:** This reference covers `@google/gemini-cli` v0.16.0+

**Last Updated:** 2025-01-22
