---
name: docs-lookup
description: Get up-to-date documentation for any library, framework, or technology using Context7 and Brave Search MCP servers. Apply when you need current API references, migration guides, or best practices.
---

# Documentation Lookup Skill

Retrieve accurate, up-to-date documentation for any library, framework, or technology using Context7 and Brave Search MCP servers.

## When to Use This Skill

**Use this skill when**:
- You need current API documentation (methods, parameters, return types)
- Looking up migration guides for version upgrades
- Finding best practices for a specific library
- Checking for breaking changes in new releases
- Getting code examples for unfamiliar APIs
- Verifying correct usage patterns

**Why use MCP servers**:
- Training data may be outdated
- APIs change frequently
- New libraries/versions emerge constantly
- Official documentation is authoritative

## Available MCP Servers

### Context7

**Purpose**: Direct access to library documentation and code examples

**Best for**:
- API reference documentation
- Code examples from official docs
- Library-specific guides
- SDK documentation

**MCP Function**: `mcp__context7__resolve-library-id` and `mcp__context7__get-library-docs`

### Brave Search

**Purpose**: Web search for documentation, tutorials, and discussions

**Best for**:
- Finding official documentation URLs
- Tutorial and guide discovery
- Stack Overflow solutions
- Blog posts and articles
- Community discussions

**MCP Function**: `mcp__brave-search__brave_web_search`

## Usage Patterns

### Pattern 1: Get Library Documentation

**Use Case**: Need API reference for a specific library

**Process**:
```
1. Use Context7 to resolve library ID
2. Fetch documentation for specific topic/function
3. Extract relevant information
```

**Example Query**:
```
"Show me the documentation for React's useState hook"
"What are the parameters for Express.js app.get()?"
"How do I configure Prisma database connections?"
```

**Approach**:
```
Step 1: Resolve library
  mcp__context7__resolve-library-id
  Input: "react" or "expressjs" or "prisma"

Step 2: Get specific documentation
  mcp__context7__get-library-docs
  Input: library_id, topic="useState" or "routing" or "connections"
```

### Pattern 2: Search for Documentation

**Use Case**: Find documentation, tutorials, or solutions

**Process**:
```
1. Search with Brave for relevant documentation
2. Find official sources
3. Fetch and extract information
```

**Example Query**:
```
"Find the migration guide for Next.js 14"
"Search for TypeScript configuration best practices"
"Look up AWS Lambda deployment documentation"
```

**Approach**:
```
mcp__brave-search__brave_web_search
Query: "Next.js 14 migration guide site:nextjs.org"
Query: "TypeScript tsconfig best practices"
Query: "AWS Lambda deployment documentation site:aws.amazon.com"
```

### Pattern 3: Compare Versions

**Use Case**: Understand changes between versions

**Process**:
```
1. Search for changelog/release notes
2. Look up migration guides
3. Find breaking changes documentation
```

**Example Query**:
```
"What changed between React 17 and React 18?"
"Breaking changes in Python 3.11"
"Node.js 18 to 20 migration guide"
```

### Pattern 4: Find Code Examples

**Use Case**: Get working code examples

**Process**:
```
1. Check library documentation for examples
2. Search for community examples
3. Verify against official documentation
```

**Example Query**:
```
"Show me examples of using Zod for form validation"
"How to implement authentication with NextAuth.js"
"Redis caching examples in Node.js"
```

## Search Strategies

### Effective Search Queries

**Be Specific**:
```
❌ "react documentation"
✅ "react useEffect cleanup function documentation"

❌ "python async"
✅ "python asyncio gather vs wait documentation"
```

**Use Site Filtering**:
```
Official docs:     site:reactjs.org
GitHub:            site:github.com
Stack Overflow:    site:stackoverflow.com
MDN:               site:developer.mozilla.org
```

**Include Version**:
```
"Next.js 14 app router documentation"
"TypeScript 5.0 decorators"
"Python 3.11 new features"
```

**Target Specific Content**:
```
API reference:     "API reference [library]"
Migration:         "migration guide [library] [version]"
Best practices:    "best practices [library] [topic]"
Examples:          "[library] [feature] example code"
```

### Validating Information

**Check Source Authority**:
```
Most reliable:
1. Official documentation
2. Official GitHub repository
3. Author's blog/talks
4. Well-known community resources

Less reliable:
- Random blog posts
- Old Stack Overflow answers
- Tutorial sites with ads
```

**Check Freshness**:
```
- Look for documentation date/version
- Check if applies to current version
- Verify against changelog
- Test code examples
```

**Cross-Reference**:
```
When in doubt:
1. Check official docs first
2. Verify with multiple sources
3. Test the code
4. Check GitHub issues for known problems
```

## Common Documentation Tasks

### 1. API Reference Lookup

**Goal**: Find exact function signature and behavior

**Approach**:
```
1. Context7 for direct library docs
2. Search for "[function] [library] API documentation"
3. Look for:
   - Parameters and types
   - Return value
   - Exceptions/errors
   - Usage examples
```

### 2. Migration Guide

**Goal**: Upgrade from one version to another

**Approach**:
```
1. Search "[library] [old version] to [new version] migration"
2. Look for:
   - Breaking changes
   - Deprecated features
   - New requirements
   - Code changes needed
```

### 3. Configuration Options

**Goal**: Find all configuration options

**Approach**:
```
1. Search "[library] configuration options documentation"
2. Look for:
   - Default values
   - Environment variables
   - Config file formats
   - Examples
```

### 4. Error Troubleshooting

**Goal**: Understand and fix an error

**Approach**:
```
1. Search exact error message
2. Check GitHub issues
3. Look for:
   - What causes the error
   - How to fix it
   - Workarounds
   - Related issues
```

### 5. Best Practices

**Goal**: Learn recommended patterns

**Approach**:
```
1. Search "[library] best practices [topic]"
2. Look for:
   - Official recommendations
   - Performance tips
   - Security considerations
   - Common pitfalls
```

## Handling Search Results

### Extracting Key Information

**From API Documentation**:
```
Extract:
- Function/method name
- Parameters (name, type, required/optional, default)
- Return type
- Throws/raises
- Example usage
- Related functions
```

**From Migration Guides**:
```
Extract:
- Breaking changes
- Required code changes
- New dependencies
- Deprecated features
- Timeline/urgency
```

**From Tutorials**:
```
Extract:
- Prerequisites
- Step-by-step process
- Code examples
- Common issues
- Next steps
```

### Presenting Information

**Format for Clarity**:
```markdown
## [Library] [Feature] Documentation

### Overview
Brief description of what it does

### Syntax
```language
functionName(param1: Type, param2?: Type): ReturnType
```

### Parameters
- `param1` (Type, required): Description
- `param2` (Type, optional): Description. Default: value

### Returns
Type - Description

### Example
```language
// Example code here
```

### Notes
- Important considerations
- Common pitfalls
```

## Tips for Effective Lookups

### Do

✅ **Start with official sources**
- Official documentation is most accurate
- Use Context7 for library-specific docs

✅ **Be version-specific**
- APIs change between versions
- Always note the version you're targeting

✅ **Verify code examples**
- Examples may be outdated
- Test before using in production

✅ **Check multiple sources**
- Cross-reference important information
- Different perspectives help understanding

✅ **Look at dates**
- Old tutorials may not apply
- Check when documentation was updated

### Don't

❌ **Trust outdated information**
- Old blog posts may be wrong now
- Check publication date

❌ **Ignore version mismatches**
- v1 and v2 APIs can be completely different
- Don't mix documentation versions

❌ **Copy without understanding**
- Understand what code does before using
- Modify for your use case

❌ **Skip official documentation**
- Third-party tutorials can be wrong
- Official docs are authoritative

## Example Workflows

### Workflow 1: New Library Setup

```
User: "How do I set up Prisma with PostgreSQL?"

1. Context7: Get Prisma documentation
   - Query library docs for "getting started"
   - Query for "postgresql setup"

2. Extract:
   - Installation commands
   - Configuration file setup
   - Schema definition
   - Database connection string
   - Initial migration

3. Present step-by-step guide with code
```

### Workflow 2: Debugging an Error

```
User: "I'm getting 'Cannot read property of undefined' in React"

1. Brave Search:
   - Search for error + common causes
   - Search for React-specific solutions

2. Extract:
   - Common causes (async data, wrong state)
   - Solutions (optional chaining, null checks)
   - Prevention patterns

3. Present likely causes and fixes
```

### Workflow 3: Version Migration

```
User: "Migrate from Express 4 to Express 5"

1. Brave Search:
   - "Express 5 migration guide site:expressjs.com"
   - "Express 4 vs 5 breaking changes"

2. Context7: Get Express docs
   - Query for migration/upgrade information

3. Extract:
   - Breaking changes list
   - Removed APIs
   - New APIs
   - Required changes

4. Present migration checklist with code changes
```

### Workflow 4: Best Practices Research

```
User: "Best practices for React Query caching"

1. Context7: Get React Query documentation
   - Query for "caching"
   - Query for "best practices"

2. Brave Search:
   - "React Query caching best practices"
   - "TanStack Query cache strategies"

3. Extract:
   - Recommended patterns
   - Cache configuration
   - Invalidation strategies
   - Common mistakes

4. Present organized best practices guide
```

## Quick Reference

### MCP Commands

**Context7 - Resolve Library**:
```
mcp__context7__resolve-library-id
- Find the correct library identifier
- Use before fetching docs
```

**Context7 - Get Documentation**:
```
mcp__context7__get-library-docs
- Fetch specific documentation
- Include topic for focused results
```

**Brave Search**:
```
mcp__brave-search__brave_web_search
- Web search for documentation
- Use site: for filtering
- Include version numbers
```

### Search Query Templates

```
API Reference:
"[library] [function] API reference documentation"

Migration:
"[library] migrate from [old] to [new] guide site:[official-site]"

Examples:
"[library] [feature] example code official documentation"

Best Practices:
"[library] [topic] best practices recommended"

Troubleshooting:
"[exact error message] [library] [version]"

Configuration:
"[library] configuration options reference [version]"
```

---

**Remember**: Always prioritize official documentation over third-party sources. Use Context7 for direct library access and Brave Search for broader documentation discovery. Verify version compatibility and test code examples before using them.
