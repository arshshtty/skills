# Claude Code Skills & Agents Repository

A comprehensive collection of professional skills and specialized agents for [Claude Code](https://claude.ai/code), designed to enhance your development workflow with expert-level guidance across multiple domains.

## 📁 Repository Structure

```
.claude/
├── agents/                    # Specialized sub-agents
│   ├── code-reviewer.md
│   ├── debugger-assistant.md
│   ├── refactor-wizard.md
│   ├── security-auditor.md
│   └── test-writer.md
│
└── skills/                    # Domain expertise skills
    ├── api-design/
    │   └── SKILL.md
    ├── code-review/
    │   └── SKILL.md
    ├── database-design/
    │   └── SKILL.md
    ├── debugging-methodology/
    │   └── SKILL.md
    ├── devops-cicd/
    │   └── SKILL.md
    ├── docs-lookup/
    │   └── SKILL.md
    ├── frontend-design/
    │   └── SKILL.md
    ├── gemini-subagent/
    │   ├── SKILL.md
    │   └── references/        # Additional reference materials
    ├── performance-optimization/
    │   └── SKILL.md
    ├── refactoring-methodology/
    │   └── SKILL.md
    ├── security-audit/
    │   └── SKILL.md
    ├── technical-writing/
    │   └── SKILL.md
    └── test-strategy/
        └── SKILL.md
```

## 🤖 Agents

Agents are specialized sub-agents that can be explicitly invoked during your Claude Code sessions to perform specific tasks.

| Agent | Description |
|-------|-------------|
| **code-reviewer** | Performs systematic code review analyzing security vulnerabilities (SQL injection, XSS, auth issues), performance problems (N+1 queries, algorithmic complexity), bugs (race conditions, null references), and maintainability concerns. Returns structured feedback with severity levels and concrete fixes. |
| **debugger-assistant** | Analyzes error messages, stack traces, and logs to identify root causes. Provides debugging strategies for common patterns like race conditions, memory leaks, async issues, and null references. Suggests concrete next steps. |
| **refactor-wizard** | Guides safe, incremental refactoring with impact analysis. Detects code smells (duplication, long functions, deep nesting), suggests classic refactoring patterns, and ensures tests pass at each step. **Critical: Refactoring must NOT change behavior.** |
| **security-auditor** | Identifies OWASP Top 10 vulnerabilities, injection attacks, broken access control, cryptographic failures, and insecure dependencies. Provides concrete remediation code examples. **Must be used before production deployments.** |
| **test-writer** | Generates comprehensive unit, integration, and E2E tests with edge cases, mocks, and fixtures. Analyzes coverage gaps and suggests missing test scenarios. Follows TDD best practices. |

## 🎯 Skills

Skills are activated automatically based on task relevance, providing domain expertise when needed.

### Development & Code Quality

| Skill | Description |
|-------|-------------|
| **api-design** | Design consistent, maintainable REST and GraphQL APIs with proper versioning, error handling, and documentation. Apply when creating new APIs or refactoring existing ones. |
| **code-review** | Perform thorough code reviews focusing on correctness, maintainability, performance, and best practices. Apply when reviewing pull requests, auditing code quality, or mentoring developers. |
| **debugging-methodology** | Systematic debugging methodology to efficiently identify and fix bugs. Apply when encountering unexpected behavior, errors, or performance issues in any codebase. |
| **performance-optimization** | Optimize application performance through profiling, benchmarking, caching, and resource management. Apply when addressing slow response times, high memory usage, or scalability concerns. |
| **refactoring-methodology** | Safely refactor code using proven patterns to improve structure without changing behavior. Apply when improving code quality, reducing technical debt, or making code easier to extend. |
| **test-strategy** | Create comprehensive testing strategies using the testing pyramid. Apply when writing tests, planning test coverage, or setting up testing infrastructure for any codebase. |

### Architecture & Infrastructure

| Skill | Description |
|-------|-------------|
| **database-design** | Design efficient database schemas with proper normalization, indexing, and query optimization. Apply when creating new databases, optimizing queries, or planning data migrations. |
| **devops-cicd** | Design and implement CI/CD pipelines, containerization, and deployment strategies. Apply when setting up automation, improving deployment processes, or implementing infrastructure as code. |

### Frontend & UI

| Skill | Description |
|-------|-------------|
| **frontend-design** | Creates minimalistic, production-grade frontend interfaces using shadcn/ui with themes from tweakcn.com. Apply when building React components, landing pages, dashboards, or any frontend UI work. |

### Security & Documentation

| Skill | Description |
|-------|-------------|
| **security-audit** | Perform security audits using OWASP Top 10 principles. Check for SQL injection, XSS, CSRF, authentication issues, and other vulnerabilities. Apply when reviewing code or building security-sensitive features. |
| **technical-writing** | Write clear, comprehensive technical documentation including READMEs, API docs, architecture decision records, and onboarding guides. Apply when creating or updating any project documentation. |

### Utilities

| Skill | Description |
|-------|-------------|
| **docs-lookup** | Get up-to-date documentation for any library, framework, or technology using Context7 and Brave Search MCP servers. Apply when you need current API references, migration guides, or best practices. |
| **gemini-subagent** | Use Gemini CLI as a sub-agent to brainstorm ideas, get feedback, plan implementations, and perform code reviews. Apply when you need a second perspective or alternative approach. |

## 🔗 Installation via Symlinks

Symlinking allows you to keep your Claude Code configuration always in sync with this repository. Any updates to skills or agents in this repo will automatically be reflected in your Claude Code environment.

### Why Symlink?

- **Always up-to-date**: Pull latest changes and they're immediately available
- **Single source of truth**: Manage skills and agents in one place
- **Easy versioning**: Use git to track and revert changes
- **Shareable**: Team members can use the same configuration

### Prerequisites

1. Clone this repository:
   ```bash
   git clone https://github.com/arshshtty/skills.git
   cd skills
   ```

2. Locate your Claude Code configuration directory:
   - **macOS/Linux**: `~/.claude/`
   - **Windows**: `%USERPROFILE%\.claude\`

### Symlinking Instructions

Choose the appropriate method for your operating system:

#### macOS / Linux

```bash
# Navigate to your Claude Code config directory
cd ~/.claude

# Backup existing skills/agents (if any)
[ -d skills ] && mv skills skills.backup
[ -d agents ] && mv agents agents.backup

# Create symlinks to the repository
ln -s /path/to/skills/.claude/skills skills
ln -s /path/to/skills/.claude/agents agents

# Verify symlinks
ls -l skills agents
```

**Example with actual path:**
```bash
cd ~/.claude
ln -s ~/projects/skills/.claude/skills skills
ln -s ~/projects/skills/.claude/agents agents
```

#### Windows (Command Prompt - Run as Administrator)

```cmd
:: Navigate to Claude Code config directory
cd %USERPROFILE%\.claude

:: Backup existing skills/agents (if any)
if exist skills ren skills skills.backup
if exist agents ren agents agents.backup

:: Create symlinks to the repository
mklink /D skills C:\path\to\skills\.claude\skills
mklink /D agents C:\path\to\skills\.claude\agents

:: Verify symlinks
dir skills
dir agents
```

#### Windows (PowerShell - Run as Administrator)

```powershell
# Navigate to Claude Code config directory
cd $env:USERPROFILE\.claude

# Backup existing skills/agents (if any)
if (Test-Path skills) { Rename-Item skills skills.backup }
if (Test-Path agents) { Rename-Item agents agents.backup }

# Create symlinks to the repository
New-Item -ItemType SymbolicLink -Path skills -Target C:\path\to\skills\.claude\skills
New-Item -ItemType SymbolicLink -Path agents -Target C:\path\to\skills\.claude\agents

# Verify symlinks
Get-Item skills, agents
```

### Verification

After creating symlinks, verify they're working:

```bash
# Check that files are accessible through symlinks
ls ~/.claude/skills/
ls ~/.claude/agents/

# Verify the symlinks point to the correct location
# macOS/Linux:
readlink ~/.claude/skills
readlink ~/.claude/agents

# Windows PowerShell:
Get-Item ~/.claude/skills | Select-Object Target
Get-Item ~/.claude/agents | Select-Object Target
```

### Updating Skills and Agents

To get the latest skills and agents:

```bash
# Navigate to the repository
cd /path/to/skills

# Pull latest changes
git pull origin main

# Changes are immediately available in Claude Code (no restart needed)
```

### Troubleshooting Symlinks

**Permission Issues (Linux/macOS):**
```bash
# Ensure you have read permissions
chmod -R u+r /path/to/skills/.claude
```

**Permission Issues (Windows):**
- Ensure you're running Command Prompt or PowerShell as Administrator
- Check that Developer Mode is enabled (Windows 10+): Settings → Update & Security → For Developers → Developer Mode

**Broken Symlinks:**
```bash
# Remove broken symlinks
rm ~/.claude/skills ~/.claude/agents

# Recreate with correct path
ln -s /correct/path/to/skills/.claude/skills ~/.claude/skills
ln -s /correct/path/to/skills/.claude/agents ~/.claude/agents
```

**Path Issues:**
- Use absolute paths, not relative paths
- Ensure the path doesn't contain spaces (or properly quote it)
- On Windows, use backslashes (`\`) in paths or forward slashes with quotes

## 🚀 Usage

### Using Skills

Skills are automatically activated by Claude Code based on your request context. Simply describe what you want to do:

```
"Design a REST API for a task management system"
→ Activates: api-design skill

"Review this pull request for security issues"
→ Activates: code-review, security-audit skills

"Optimize this slow database query"
→ Activates: database-design, performance-optimization skills
```

### Using Agents

Invoke agents explicitly using the Task tool or mention them in your request:

```
"Use the code-reviewer agent to analyze this module"
"Run the security-auditor on the authentication code"
"Have the test-writer create tests for this function"
```

## 📝 Contributing

Contributions are welcome! To add or improve skills/agents:

1. Fork this repository
2. Create your feature branch: `git checkout -b feature/amazing-skill`
3. Make changes to files in `.claude/skills/` or `.claude/agents/`
4. Commit your changes: `git commit -m 'Add amazing skill'`
5. Push to the branch: `git push origin feature/amazing-skill`
6. Open a Pull Request

### Skill Format

Skills must have a `SKILL.md` file with YAML frontmatter:

```markdown
---
name: skill-name
description: Brief description of when to use this skill
---

# Skill Name

Detailed instructions and guidelines for the skill...
```

### Agent Format

Agents are Markdown files with YAML frontmatter:

```markdown
---
name: agent-name
description: Description of what this agent does and when to use it
---

# Agent Name

System prompt and detailed instructions for the agent...
```

## 📄 License

MIT License - feel free to use these skills and agents in your projects.

## 🤝 Acknowledgments

- Built for [Claude Code](https://claude.ai/code)
- Inspired by software engineering best practices and industry standards
- Community contributions and feedback

## 🔗 Related Resources

- [Claude Code Documentation](https://docs.anthropic.com/claude/docs)
- [Claude Code Skills Guide](https://support.anthropic.com/en/articles/12512198-how-to-create-custom-skills)
- [Agent Development Guide](https://code.claude.com/docs/en/sub-agents)

---

**Made with ❤️ for the Claude Code community**
