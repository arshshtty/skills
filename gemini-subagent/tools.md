# Gemini CLI Built-in Tools

Gemini CLI provides unique built-in tools that extend its capabilities beyond code generation and analysis. This guide covers each tool, when to use it, and how to leverage it effectively.

## Table of Contents

- [Overview](#overview)
- [google_web_search](#google_web_search)
- [codebase_investigator](#codebase_investigator)
- [save_memory](#save_memory)
- [Standard File Tools](#standard-file-tools)
- [Tool Control](#tool-control)
- [Best Practices](#best-practices)

---

## Overview

Gemini CLI has access to several tools that enable it to:
- 🌐 Search the internet for current information
- 🔍 Analyze codebase architecture and dependencies
- 💾 Persist information across sessions
- 📁 Read and write files
- 📂 Navigate directory structures

These tools are automatically invoked when needed based on your prompts. You can also explicitly control which tools are available.

---

## google_web_search

**Purpose:** Search the internet for current, real-time information using Google Search.

**Unique Advantage:** Unlike Claude, Gemini has direct access to current web information through Google Search integration.

### When to Use

✅ **Use google_web_search for:**
- Latest library/framework versions and updates
- Current best practices and standards
- Breaking changes in new releases
- Recent security vulnerabilities (CVEs)
- Comparing current technologies
- Finding real-world examples and solutions
- API documentation for external services
- Industry trends and benchmarks

❌ **Don't use for:**
- Information already in your codebase
- Timeless programming concepts
- Tasks where local code analysis is sufficient

### How to Trigger

Simply ask questions that require current information:

```bash
# Automatic invocation - Gemini uses web search as needed
gemini -y -p "What are the latest best practices for React Server Components in 2025?" 2>/dev/null

gemini -y -p "What are the breaking changes in Next.js 15?" 2>/dev/null

gemini -y -p "Search for current OWASP Top 10 web security risks" 2>/dev/null
```

### Examples

**Finding Latest Version:**
```bash
gemini -y -m gemini-2.5-flash -p "What is the latest stable version of TypeScript and what are the new features?" 2>/dev/null
```

**Researching Best Practices:**
```bash
gemini -y -m gemini-2.5-flash -p "Search for 2025 best practices for implementing JWT authentication in Node.js. Include security considerations." 2>/dev/null
```

**Security Research:**
```bash
gemini -y -m gemini-2.5-flash -p "Are there any known security vulnerabilities in express.js version 4.18.x? Check recent CVEs." 2>/dev/null
```

**Technology Comparison:**
```bash
gemini -y -m gemini-2.5-flash -p "Compare Next.js vs Remix vs Astro for building a content-heavy website in 2025. Search for recent benchmarks and community feedback." 2>/dev/null
```

**Finding Solutions:**
```bash
gemini -y -m gemini-2.5-flash -p "Search for how to implement real-time notifications in a React application. What are developers using in 2025?" 2>/dev/null
```

**API Documentation:**
```bash
gemini -y -m gemini-2.5-flash -p "Look up the Stripe API documentation for creating subscription billing. What's the recommended approach?" 2>/dev/null
```

### Output Format

Web search results are integrated into Gemini's response. You'll typically see:
- Direct answers synthesized from search results
- References to sources (when Gemini cites specific information)
- Multiple perspectives from different sources
- Current dates and version numbers

### Best Practices

1. **Be specific about dates/versions:**
   ```bash
   # Good
   gemini -y -p "React 19 features released in 2025" 2>/dev/null

   # Less effective
   gemini -y -p "React features" 2>/dev/null
   ```

2. **Ask for comparisons:**
   ```bash
   gemini -y -p "Compare Prisma vs Drizzle ORM in 2025: performance, DX, ecosystem" 2>/dev/null
   ```

3. **Request recent information:**
   ```bash
   gemini -y -p "Latest security best practices for GraphQL APIs in 2025" 2>/dev/null
   ```

4. **Verify critical information:**
   - Always verify security recommendations
   - Cross-reference version numbers with official sources
   - Test code examples before using in production

### Use Cases

**Pre-Implementation Research:**
```bash
#!/bin/bash
# Research before building feature

# Search for current approaches
gemini -y -m gemini-2.5-flash -p "Search for best practices for implementing rate limiting in Express.js in 2025. What libraries are recommended?" 2>/dev/null > research.txt

# Review research
cat research.txt

# Implement based on findings
# (Use Claude or Gemini for implementation)
```

**Dependency Updates:**
```bash
# Check for breaking changes before updating
gemini -y -m gemini-2.5-flash -p "What are the breaking changes when upgrading from React Router v6.20 to v6.30?" 2>/dev/null
```

**Security Audits:**
```bash
# Check for known vulnerabilities
gemini -y -m gemini-2.5-pro -p "Search for security vulnerabilities in our dependencies:
- express@4.18.2
- jsonwebtoken@9.0.0
- bcrypt@5.1.0

List any CVEs or security advisories." 2>/dev/null
```

---

## codebase_investigator

**Purpose:** Perform deep architecture analysis of your codebase, identifying structure, dependencies, patterns, and relationships.

**Unique Advantage:** Automated codebase exploration and architectural analysis without manual file reading.

### When to Use

✅ **Use codebase_investigator for:**
- Understanding unfamiliar codebases
- Architectural assessment and documentation
- Identifying circular dependencies
- Finding coupling and cohesion issues
- Discovering code patterns and anti-patterns
- Planning refactoring efforts
- Onboarding documentation
- Dependency graph visualization

❌ **Don't use for:**
- Analyzing single files (use file reading instead)
- Making code changes (use file tools)
- Line-by-line code review (too broad)

### How to Trigger

Use `--include-directories` flag to specify codebase scope:

```bash
# Analyze specific directories
gemini -y --include-directories ./src -p "Analyze the architecture of this codebase" 2>/dev/null

# Multiple directories
gemini -y --include-directories ./src,./lib,./components -p "Identify architectural patterns" 2>/dev/null
```

Or explicitly request codebase analysis:

```bash
gemini -y -m gemini-2.5-pro -p "Use the codebase investigator to analyze the structure of this project and identify potential issues" 2>/dev/null
```

### Examples

**Overall Architecture Analysis:**
```bash
gemini -y --include-directories ./src -m gemini-2.5-pro -p "Analyze the overall architecture of this codebase. Describe:
1. Project structure and organization
2. Architectural patterns being used
3. Main modules and their responsibilities
4. Data flow and dependencies" 2>/dev/null
```

**Dependency Analysis:**
```bash
gemini -y --include-directories ./src -m gemini-2.5-pro -p "Investigate the dependency relationships in this codebase. Find:
- Circular dependencies
- Tightly coupled modules
- God objects or classes
- Dependency injection opportunities" 2>/dev/null
```

**Identifying Bottlenecks:**
```bash
gemini -y --include-directories ./src -m gemini-2.5-pro -p "Analyze this codebase for potential performance bottlenecks:
- Database query patterns
- Inefficient algorithms
- Missing caching opportunities
- N+1 query problems
- Resource-intensive operations" 2>/dev/null
```

**Code Pattern Discovery:**
```bash
gemini -y --include-directories ./src -m gemini-2.5-flash -p "Identify the common patterns used in this codebase:
- Design patterns (Factory, Singleton, Observer, etc.)
- Architectural patterns (MVC, Repository, Service Layer, etc.)
- Custom patterns specific to this project

Provide examples of each." 2>/dev/null
```

**Security Architecture Review:**
```bash
gemini -y --include-directories ./src -m gemini-2.5-pro -p "Investigate the security architecture:
- Authentication implementation
- Authorization patterns
- Input validation approach
- Sensitive data handling
- Security vulnerabilities

Provide a security assessment." 2>/dev/null
```

**Refactoring Planning:**
```bash
gemini -y --include-directories ./src -m gemini-2.5-pro -p "Analyze this codebase to plan a refactoring effort:
- Identify code smells
- Find duplication
- Spot overly complex modules
- Suggest extraction opportunities
- Prioritize refactoring targets" 2>/dev/null
```

**Onboarding Documentation:**
```bash
gemini -y --include-directories ./src,./lib -m gemini-2.5-flash -p "Create an architectural overview for new developers:
- High-level architecture diagram (in text/ASCII)
- Key modules and their purposes
- Important patterns to understand
- How components interact
- Where to start for common tasks" 2>/dev/null > ARCHITECTURE.md
```

### Configuration

**Include Specific Directories:**
```bash
# Only analyze source code
gemini --include-directories ./src -p "your prompt" 2>/dev/null

# Multiple directories
gemini --include-directories ./frontend,./backend,./shared -p "your prompt" 2>/dev/null
```

**Exclude Directories (.geminiignore):**
Create `.geminiignore` in project root:
```
node_modules/
dist/
build/
coverage/
.next/
*.test.ts
*.spec.ts
```

### Output Examples

Typical codebase_investigator output includes:

```
Project Structure:
├── src/
│   ├── controllers/     # HTTP request handlers
│   ├── services/        # Business logic layer
│   ├── repositories/    # Data access layer
│   ├── models/          # Data models
│   └── utils/           # Helper functions

Architectural Patterns:
- Layered Architecture (Controller → Service → Repository)
- Repository Pattern for data access
- Dependency Injection via constructor

Dependencies:
- High coupling between UserService and OrderService
- Circular dependency: AuthService ↔ UserService
- Payment module is well-isolated

Recommendations:
1. Break circular dependency with AuthService
2. Extract shared logic from UserService and OrderService
3. Add interface abstractions for better testability
```

### Use Cases

**New Team Member Onboarding:**
```bash
#!/bin/bash
# Generate onboarding documentation

gemini -y --include-directories ./src -m gemini-2.5-flash -p "Create comprehensive onboarding documentation for this codebase including architecture overview, key patterns, and how to get started" 2>/dev/null > docs/ONBOARDING.md
```

**Pre-Refactoring Analysis:**
```bash
# Understand current state before refactoring
gemini -y --include-directories ./src -m gemini-2.5-pro -p "Analyze the current architecture focusing on areas that need refactoring. Prioritize issues by impact." 2>/dev/null > refactoring-plan.txt
```

**Technical Debt Assessment:**
```bash
# Identify technical debt
gemini -y --include-directories ./src,./lib -m gemini-2.5-pro -p "Assess technical debt in this codebase:
- Code smells and anti-patterns
- Missing tests
- Outdated dependencies
- Poor separation of concerns
- Hard-to-maintain areas

Provide a prioritized technical debt backlog." 2>/dev/null
```

---

## save_memory

**Purpose:** Persist important information, decisions, and context across Gemini sessions.

**Unique Advantage:** Build project-specific knowledge base that persists across multiple interactions.

### When to Use

✅ **Use save_memory for:**
- Documenting architectural decisions
- Recording important project context
- Saving answers to recurring questions
- Building institutional knowledge
- Tracking patterns and conventions
- Remembering user preferences
- Creating project-specific context

❌ **Don't use for:**
- Storing code (use files)
- Temporary information (not needed across sessions)
- Sensitive data (security risk)

### How to Trigger

Ask Gemini to save or remember information:

```bash
gemini -p "Save this information: We use JWT tokens with 15-minute expiration and 7-day refresh tokens. Tokens are stored in httpOnly cookies." 2>/dev/null

gemini -p "Remember that we follow Clean Architecture with strict layer separation: Controllers → Services → Repositories" 2>/dev/null
```

### Examples

**Architectural Decisions:**
```bash
gemini -p "Save this architectural decision:

Decision: Use Redis for session storage instead of in-memory
Reasoning: Need horizontal scaling, session persistence across restarts
Trade-offs: Added dependency, slight latency increase
Date: 2025-01-22
Decided by: Tech lead review" 2>/dev/null
```

**Coding Standards:**
```bash
gemini -p "Remember our coding standards:
- All async functions must have comprehensive error handling
- Database queries must have timeouts (max 5s)
- API responses must follow RFC 7807 problem details format
- Authentication uses JWT in httpOnly cookies only
- All inputs must be validated with Zod schemas" 2>/dev/null
```

**Project Context:**
```bash
gemini -p "Save project context:

Project: E-commerce platform
Tech Stack: Next.js 15, TypeScript, PostgreSQL, Redis, Stripe
Architecture: Clean Architecture with DDD
Team Size: 5 developers
Deployment: Vercel with database on Railway
Peak traffic: 10k concurrent users

This helps you understand the scale and constraints." 2>/dev/null
```

**Common Patterns:**
```bash
gemini -p "Remember this pattern we use for API endpoints:

All endpoints follow this structure:
1. Input validation with Zod
2. Authentication check
3. Authorization check
4. Business logic in service layer
5. Error handling with custom error classes
6. Structured logging

Example in src/controllers/UserController.ts" 2>/dev/null
```

**Recurring Answers:**
```bash
gemini -p "Save FAQ:

Q: Why do we use Prisma instead of raw SQL?
A: Type safety, automatic migrations, better DX. We evaluated TypeORM and Drizzle but chose Prisma for ecosystem maturity.

Q: Why httpOnly cookies instead of localStorage for tokens?
A: XSS protection. Even if site has XSS vulnerability, tokens can't be stolen via JavaScript." 2>/dev/null
```

### Retrieving Saved Information

Gemini automatically recalls saved memories in relevant contexts:

```bash
# Gemini will remember your project context
gemini -p "How should I implement authentication in this project?" 2>/dev/null
# Response will reference your saved JWT + httpOnly cookie decision

# Explicitly ask to recall
gemini -p "What do you remember about our architectural decisions?" 2>/dev/null

# Check all saved memories
gemini -p "Show me all the information you've saved about this project" 2>/dev/null
```

### Use Cases

**New Team Member Context:**
```bash
# Save comprehensive context for future sessions
gemini -p "Save all important context about this project:

Architecture: Microservices with API gateway
Key Services: Auth, User, Order, Payment, Notification
Database: PostgreSQL per service + shared Redis
Message Queue: RabbitMQ for async operations
Authentication: JWT with Keycloak
Frontend: React with Server Components

Critical Patterns:
- Event-driven architecture for service communication
- Saga pattern for distributed transactions
- Circuit breaker for external API calls
- CQRS for complex read scenarios

Deployment:
- Kubernetes on AWS
- Separate clusters for staging and production
- Blue-green deployments
- Automated rollbacks on health check failures" 2>/dev/null
```

**Decision Log:**
```bash
# Keep track of important decisions
gemini -p "Save decision log entry:

Date: 2025-01-22
Decision: Migrate from REST to GraphQL for mobile API
Reason: Reduce over-fetching, better mobile performance
Trade-offs: Learning curve, additional complexity
Status: Approved, implementation starts Q2
Reviewers: Tech lead, mobile team lead" 2>/dev/null
```

**Style Guide:**
```bash
# Remember project-specific conventions
gemini -p "Remember our style guide:

Naming Conventions:
- Components: PascalCase (UserProfile.tsx)
- Files: kebab-case (user-profile.ts)
- Functions: camelCase (getUserById)
- Constants: SCREAMING_SNAKE_CASE (API_BASE_URL)
- Interfaces: PascalCase with 'I' prefix (IUserService)

Code Organization:
- Max file length: 300 lines
- Max function length: 50 lines
- One component per file
- Co-locate tests with source files

Testing:
- Unit test coverage: 80% minimum
- Integration tests for all API endpoints
- E2E tests for critical user flows" 2>/dev/null
```

### Managing Memories

```bash
# View all memories
gemini -p "List all saved memories" 2>/dev/null

# Update a memory
gemini -p "Update the memory about JWT tokens: We now use 30-minute expiration instead of 15 minutes" 2>/dev/null

# Clear specific memory
gemini -p "Forget the memory about Redis session storage" 2>/dev/null
```

### Best Practices

1. **Be specific and structured** - Save well-organized information
2. **Include dates** - Track when decisions were made
3. **Add context** - Explain why, not just what
4. **Update regularly** - Keep information current
5. **Don't save secrets** - Never save API keys, passwords, or sensitive data
6. **Use for cross-session context** - Things that help in future conversations

---

## Standard File Tools

Gemini also has standard file operation tools:

### read_file
Read file contents:
```bash
gemini -p "Read the file src/auth/jwt.ts and explain how token validation works" 2>/dev/null
```

### write_file
Create or modify files:
```bash
gemini -y -p "Create a new file src/utils/logger.ts with a Winston logger configuration" 2>/dev/null
```

### list_directory
List directory contents:
```bash
gemini -p "List all files in the src/services directory" 2>/dev/null
```

These tools are automatically invoked when needed based on your prompts.

---

## Tool Control

### Restricting Available Tools

Limit which tools Gemini can use with `--allowed-tools`:

```bash
# Only allow reading files (no modifications)
gemini --allowed-tools "read_file,list_directory" -p "Analyze the codebase" 2>/dev/null

# Disable all tools (pure conversation)
gemini --allowed-tools "" -p "Explain how JWT works" 2>/dev/null

# Allow everything (default)
gemini -p "your prompt" 2>/dev/null
```

### Available Tools List

- `read_file` - Read file contents
- `write_file` - Create/modify files
- `list_directory` - List directory contents
- `google_web_search` - Internet search
- `codebase_investigator` - Architecture analysis
- `save_memory` - Persist information across sessions

### Use Cases for Restrictions

**Read-only Analysis:**
```bash
# Prevent accidental file modifications during analysis
gemini --allowed-tools "read_file,list_directory,codebase_investigator" --include-directories ./src -p "Analyze the codebase for security issues" 2>/dev/null
```

**No Internet Access:**
```bash
# Disable web search for offline work
gemini --allowed-tools "read_file,write_file,list_directory" -p "Refactor this code" 2>/dev/null
```

**Pure Conversation:**
```bash
# No tools, just AI reasoning
gemini --allowed-tools "" -p "Explain the SOLID principles with examples" 2>/dev/null
```

---

## Best Practices

### General Tool Usage

1. **Let Gemini choose tools automatically** - It's good at deciding which tools to use
2. **Be explicit when needed** - "Use web search to find..." or "Use codebase investigator to analyze..."
3. **Combine tools** - Web search + codebase analysis for well-informed implementations
4. **Verify results** - Especially for web search results and architectural recommendations
5. **Use appropriate models** - Pro for in-depth codebase analysis, Flash for simple searches

### google_web_search Best Practices

- ✅ Request current/recent information
- ✅ Ask for comparisons
- ✅ Specify dates and versions
- ✅ Verify critical information independently
- ❌ Don't rely solely on search for architecture decisions
- ❌ Don't assume search results are always current

### codebase_investigator Best Practices

- ✅ Use `--include-directories` to scope analysis
- ✅ Use Pro model for comprehensive analysis
- ✅ Create `.geminiignore` to exclude irrelevant files
- ✅ Ask specific architectural questions
- ✅ Combine with manual code review
- ❌ Don't analyze generated/dist folders
- ❌ Don't expect line-by-line code review (too broad)

### save_memory Best Practices

- ✅ Save architectural decisions with rationale
- ✅ Document project-specific conventions
- ✅ Include dates and context
- ✅ Keep information well-structured
- ✅ Update memories when things change
- ❌ Never save sensitive data (API keys, passwords)
- ❌ Don't save temporary information
- ❌ Don't save code (use files instead)

---

## Tool Comparison with Claude

| Capability | Gemini Tools | Claude Tools |
|------------|--------------|--------------|
| Web Search | ✅ google_web_search | ❌ No direct search |
| File Operations | ✅ Basic | ✅ Advanced |
| Codebase Analysis | ✅ codebase_investigator | ✅ Manual reading |
| Memory Persistence | ✅ save_memory | ❌ Per-session only |
| Code Generation | ✅ Via write_file | ✅ Via Write tool |
| Architecture Analysis | ✅ Automated | ✅ Manual + reasoning |

**When to use Gemini's tools:**
- Need current web information
- Want automated codebase analysis
- Need cross-session memory
- Quick architecture overview

**When to use Claude's tools:**
- Need detailed file manipulation
- Superior reasoning about code
- Complex multi-file refactoring
- Precise code changes

---

## Advanced Tool Usage

### Combining Multiple Tools in Workflow

```bash
#!/bin/bash
# Comprehensive analysis using all tools

# 1. Web search for best practices
echo "Researching best practices..."
gemini -y -m gemini-2.5-flash -p "Search for 2025 best practices for Node.js microservices architecture" 2>/dev/null > research.txt

# 2. Analyze current codebase
echo "Analyzing current architecture..."
gemini -y --include-directories ./src -m gemini-2.5-pro -p "Use codebase investigator to analyze our microservices architecture. Compare against the best practices in research.txt" 2>/dev/null > analysis.txt

# 3. Save recommendations
echo "Saving architectural recommendations..."
gemini -p "Based on analysis.txt, save our architectural improvement plan as a memory for future reference" 2>/dev/null

# 4. Generate implementation plan
echo "Creating implementation plan..."
gemini --resume latest -m gemini-2.5-pro -p "Create a prioritized implementation plan for the architectural improvements" 2>/dev/null > implementation-plan.md

echo "Complete! Check implementation-plan.md"
```

### Tool-Specific Workflows

See [patterns.md](./patterns.md) for advanced integration patterns using these tools.

---

**Remember:** These tools are most powerful when used together. Combine web search for current knowledge, codebase analysis for understanding, and memory for preserving context across sessions.
