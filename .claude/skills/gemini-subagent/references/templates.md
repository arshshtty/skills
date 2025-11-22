# Gemini CLI Prompt Templates

Ready-to-use prompt templates for common development tasks. Copy, customize, and execute.

## Table of Contents

- [Code Generation](#code-generation)
- [Code Review](#code-review)
- [Bug Fixing](#bug-fixing)
- [Test Generation](#test-generation)
- [Documentation](#documentation)
- [Code Transformation](#code-transformation)
- [Web Research](#web-research)
- [Architecture Analysis](#architecture-analysis)
- [Specialized Tasks](#specialized-tasks)

## Code Generation

### Single File Application

```bash
gemini -y -o text -m gemini-2.5-flash -p "Create a complete [language] application that [description].

Requirements:
- [feature 1]
- [feature 2]
- [feature 3]

Include:
- Error handling
- Input validation
- Clear comments

Output as a single file." 2>/dev/null
```

**Example:**
```bash
gemini -y -o text -m gemini-2.5-flash -p "Create a complete Python application that manages a todo list using SQLite.

Requirements:
- Add, delete, update, and list todos
- Mark todos as complete
- Filter by status (all, active, completed)

Include:
- Error handling
- Input validation
- Clear comments

Output as a single file." 2>/dev/null
```

### Multi-File Project

```bash
gemini -y -m gemini-2.5-flash -p "Create a [framework] project for [description].

Structure:
- [folder/file 1]
- [folder/file 2]
- [folder/file 3]

Features:
- [feature 1]
- [feature 2]

Use [specific libraries/patterns].
Include README with setup instructions." 2>/dev/null
```

**Example:**
```bash
gemini -y -m gemini-2.5-flash -p "Create a React project for a user authentication system.

Structure:
- src/components/LoginForm.tsx
- src/components/RegisterForm.tsx
- src/hooks/useAuth.ts
- src/contexts/AuthContext.tsx
- src/services/api.ts

Features:
- Login and registration forms
- JWT token management
- Protected routes
- Auto-refresh tokens

Use React 18, TypeScript, React Router v6.
Include README with setup instructions." 2>/dev/null
```

### Component/Module

```bash
gemini -y -m gemini-2.5-flash -p "Create a [language] [component/module/class] for [purpose].

Inputs: [describe inputs]
Outputs: [describe outputs]
Behavior: [describe behavior]

Requirements:
- [requirement 1]
- [requirement 2]

Include TypeScript types and JSDoc comments." 2>/dev/null
```

**Example:**
```bash
gemini -y -m gemini-2.5-flash -p "Create a TypeScript class for rate limiting API requests.

Inputs: maxRequests (number), windowMs (number)
Outputs: allow(): boolean, getRemainingRequests(): number
Behavior: Track requests per time window, reset after window expires

Requirements:
- Thread-safe for concurrent requests
- Configurable window size and request limit
- Memory efficient

Include TypeScript types and JSDoc comments." 2>/dev/null
```

### API Endpoint

```bash
gemini -y -m gemini-2.5-flash -p "Create a REST API endpoint for [purpose] using [framework].

Method: [GET/POST/PUT/DELETE]
Path: [/api/path]
Request: [describe request body/params]
Response: [describe response format]

Include:
- Input validation
- Error handling
- Authentication check
- TypeScript types" 2>/dev/null
```

**Example:**
```bash
gemini -y -m gemini-2.5-flash -p "Create a REST API endpoint for user registration using Express.js.

Method: POST
Path: /api/auth/register
Request: { email: string, password: string, name: string }
Response: { user: User, token: string } or { error: string }

Include:
- Email format validation
- Password strength requirements
- Duplicate email check
- Password hashing with bcrypt
- JWT token generation
- TypeScript types" 2>/dev/null
```

## Code Review

### Comprehensive Review

```bash
gemini -y -m gemini-2.5-pro -p "Review this [language] code for:
- Bugs and logic errors
- Performance issues
- Security vulnerabilities
- Code quality and best practices
- Potential edge cases

Code:
\`\`\`[language]
[paste code here]
\`\`\`

Provide:
1. Issue severity (Critical/High/Medium/Low)
2. Specific line references
3. Recommended fixes
4. Explanation of each issue" 2>/dev/null
```

**Example:**
```bash
gemini -y -m gemini-2.5-pro -p "Review this TypeScript code for:
- Bugs and logic errors
- Performance issues
- Security vulnerabilities
- Code quality and best practices
- Potential edge cases

Code:
\`\`\`typescript
export async function updateUserBalance(userId: string, amount: number) {
  const user = await db.users.findById(userId);
  user.balance += amount;
  await db.users.update(userId, { balance: user.balance });
  return user.balance;
}
\`\`\`

Provide:
1. Issue severity (Critical/High/Medium/Low)
2. Specific line references
3. Recommended fixes
4. Explanation of each issue" 2>/dev/null
```

### Security Focused Review

```bash
gemini -y -m gemini-2.5-pro -p "Perform a security audit on this code:

\`\`\`[language]
[paste code here]
\`\`\`

Check for:
- SQL injection
- XSS vulnerabilities
- CSRF issues
- Authentication/authorization flaws
- Input validation gaps
- Sensitive data exposure
- Insecure dependencies

For each finding:
1. Vulnerability type
2. Severity (Critical/High/Medium/Low)
3. Attack vector
4. Mitigation strategy" 2>/dev/null
```

### Performance Review

```bash
gemini -y -m gemini-2.5-pro -p "Analyze this code for performance issues:

\`\`\`[language]
[paste code here]
\`\`\`

Identify:
- Inefficient algorithms (with Big-O analysis)
- Unnecessary computations
- Memory leaks
- Blocking operations
- Database query optimization opportunities
- Caching opportunities

For each issue, provide:
1. Performance impact
2. Specific optimization
3. Expected improvement" 2>/dev/null
```

### Before Commit Review

```bash
gemini -y -m gemini-2.5-flash -p "Quick review before commit:

\`\`\`[language]
[paste changed code]
\`\`\`

Check for:
- Syntax errors
- Obvious bugs
- Breaking changes
- Missing error handling
- TODO/FIXME comments
- Console.log/debugging statements
- Sensitive data (API keys, passwords)

Brief summary only." 2>/dev/null
```

## Bug Fixing

### Fix Specific Bug

```bash
gemini -y -m gemini-2.5-flash -p "Fix this bug in the code:

Bug: [describe the bug]
Expected: [expected behavior]
Actual: [actual behavior]

Code:
\`\`\`[language]
[paste code]
\`\`\`

Provide:
1. Root cause explanation
2. Fixed code
3. Why the fix works" 2>/dev/null
```

**Example:**
```bash
gemini -y -m gemini-2.5-flash -p "Fix this bug in the code:

Bug: User authentication fails intermittently
Expected: Users should login successfully with correct credentials
Actual: Sometimes returns 'Invalid credentials' even with correct password

Code:
\`\`\`typescript
async function validatePassword(input: string, hash: string): Promise<boolean> {
  return bcrypt.compare(input, hash);
}
\`\`\`

Provide:
1. Root cause explanation
2. Fixed code
3. Why the fix works" 2>/dev/null
```

### Auto-Detect and Fix

```bash
gemini -y -m gemini-2.5-flash -p "Analyze this code, find all bugs, and provide fixed version:

\`\`\`[language]
[paste code]
\`\`\`

For each bug found:
1. What's wrong
2. Why it's a problem
3. How to fix it

Then provide the complete corrected code." 2>/dev/null
```

### Debug Error

```bash
gemini -y -m gemini-2.5-flash -p "Help debug this error:

Error Message:
\`\`\`
[paste error message and stack trace]
\`\`\`

Code:
\`\`\`[language]
[paste relevant code]
\`\`\`

Context: [describe what the code is supposed to do]

Provide:
1. Root cause
2. Why it's happening
3. Fix with explanation
4. How to prevent similar issues" 2>/dev/null
```

## Test Generation

### Unit Tests

```bash
gemini -y -m gemini-2.5-flash -p "Generate comprehensive unit tests for this code:

\`\`\`[language]
[paste code to test]
\`\`\`

Use [testing framework, e.g., Jest, pytest, JUnit].

Include tests for:
- Happy path cases
- Edge cases
- Error conditions
- Boundary values
- Invalid inputs

Use descriptive test names and arrange-act-assert pattern." 2>/dev/null
```

**Example:**
```bash
gemini -y -m gemini-2.5-flash -p "Generate comprehensive unit tests for this code:

\`\`\`typescript
function calculateDiscount(price: number, discountPercent: number): number {
  if (price < 0 || discountPercent < 0 || discountPercent > 100) {
    throw new Error('Invalid input');
  }
  return price * (1 - discountPercent / 100);
}
\`\`\`

Use Jest.

Include tests for:
- Happy path cases
- Edge cases
- Error conditions
- Boundary values
- Invalid inputs

Use descriptive test names and arrange-act-assert pattern." 2>/dev/null
```

### Integration Tests

```bash
gemini -y -m gemini-2.5-flash -p "Create integration tests for this [component/API/service]:

Code:
\`\`\`[language]
[paste code]
\`\`\`

Test scenarios:
- [scenario 1]
- [scenario 2]
- [scenario 3]

Use [framework] with mocking for external dependencies.
Include setup and teardown." 2>/dev/null
```

### Test Edge Cases

```bash
gemini -y -m gemini-2.5-flash -p "Generate tests specifically for edge cases and error conditions:

\`\`\`[language]
[paste code]
\`\`\`

Test:
- Null/undefined inputs
- Empty arrays/strings
- Very large numbers
- Concurrent access
- Network failures
- Timeout scenarios
- Invalid data types

Use [framework]." 2>/dev/null
```

## Documentation

### JSDoc/TSDoc Comments

```bash
gemini -y -m gemini-2.5-flash -p "Add comprehensive JSDoc comments to this code:

\`\`\`[language]
[paste code]
\`\`\`

Include:
- Function/class descriptions
- @param tags with types and descriptions
- @returns tag with type and description
- @throws tag for exceptions
- @example with usage examples
- @deprecated if applicable" 2>/dev/null
```

### README Generation

```bash
gemini -y -m gemini-2.5-flash -p "Create a comprehensive README.md for this project:

Project: [name]
Description: [brief description]
Tech Stack: [list technologies]

Include sections for:
- Overview
- Features
- Installation
- Usage with examples
- API documentation (if applicable)
- Configuration
- Contributing guidelines
- License

Make it professional and clear." 2>/dev/null
```

### API Documentation

```bash
gemini -y -m gemini-2.5-flash -p "Generate API documentation for these endpoints:

\`\`\`[language]
[paste API code]
\`\`\`

Format: [Markdown/OpenAPI/Swagger]

For each endpoint:
- Method and path
- Description
- Request parameters
- Request body schema
- Response codes and schemas
- Example requests/responses
- Authentication requirements
- Error responses" 2>/dev/null
```

### Code Explanation

```bash
gemini -y -m gemini-2.5-flash -p "Explain what this code does in clear, simple terms:

\`\`\`[language]
[paste code]
\`\`\`

Audience: [junior developer/non-technical stakeholder/etc.]

Include:
- High-level purpose
- Step-by-step walkthrough
- Key concepts explained
- Why certain approaches were used" 2>/dev/null
```

## Code Transformation

### Refactoring

```bash
gemini -y -m gemini-2.5-flash -p "Refactor this code to improve [clarity/performance/maintainability]:

\`\`\`[language]
[paste code]
\`\`\`

Goals:
- [goal 1]
- [goal 2]

Principles:
- Follow [SOLID/DRY/KISS] principles
- Improve readability
- Maintain existing functionality
- Add comments explaining changes" 2>/dev/null
```

**Example:**
```bash
gemini -y -m gemini-2.5-flash -p "Refactor this code to improve maintainability and testability:

\`\`\`typescript
function processOrder(orderId) {
  const order = db.orders.find(orderId);
  const user = db.users.find(order.userId);
  const total = order.items.reduce((sum, item) => sum + item.price, 0);
  const tax = total * 0.08;
  const discount = user.isPremium ? total * 0.1 : 0;
  const final = total + tax - discount;
  db.orders.update(orderId, { total: final });
  sendEmail(user.email, `Order total: ${final}`);
  return final;
}
\`\`\`

Goals:
- Separate concerns
- Make testable
- Improve error handling

Principles:
- Follow SOLID principles
- Improve readability
- Maintain existing functionality
- Add comments explaining changes" 2>/dev/null
```

### Language Translation

```bash
gemini -y -m gemini-2.5-flash -p "Translate this code from [source language] to [target language]:

\`\`\`[source language]
[paste code]
\`\`\`

Requirements:
- Maintain exact functionality
- Use idiomatic [target language] patterns
- Include equivalent libraries
- Preserve comments (translated)
- Add notes about any unavoidable differences" 2>/dev/null
```

**Example:**
```bash
gemini -y -m gemini-2.5-flash -p "Translate this code from JavaScript to TypeScript:

\`\`\`javascript
function fetchUser(id) {
  return fetch(\`/api/users/\${id}\`)
    .then(res => res.json())
    .then(data => data.user)
    .catch(err => console.error(err));
}
\`\`\`

Requirements:
- Add proper type annotations
- Use async/await instead of promises
- Define interface for User
- Improve error handling" 2>/dev/null
```

### Framework Migration

```bash
gemini -y -m gemini-2.5-flash -p "Migrate this component from [old framework] to [new framework]:

\`\`\`[language]
[paste code]
\`\`\`

Ensure:
- Equivalent functionality
- Follow [new framework] best practices
- Use [new framework version] features
- Maintain styling
- Update dependencies" 2>/dev/null
```

### Modernization

```bash
gemini -y -m gemini-2.5-flash -p "Modernize this legacy code to use current [language] best practices:

\`\`\`[language]
[paste legacy code]
\`\`\`

Update:
- Use modern syntax (ES2024, etc.)
- Replace deprecated APIs
- Improve patterns and practices
- Add type safety
- Enhance error handling

Explain what was changed and why." 2>/dev/null
```

## Web Research

### Latest Best Practices

```bash
gemini -y -m gemini-2.5-flash -p "Search for the latest best practices for [technology/pattern] in 2025.

Focus on:
- Official recommendations
- Industry standards
- Common pitfalls to avoid
- Breaking changes from previous versions

Provide a summary with sources." 2>/dev/null
```

**Example:**
```bash
gemini -y -m gemini-2.5-flash -p "Search for the latest best practices for React Server Components in 2025.

Focus on:
- Official React team recommendations
- Data fetching patterns
- State management approaches
- Common pitfalls to avoid
- Breaking changes from React 18

Provide a summary with sources." 2>/dev/null
```

### Library Research

```bash
gemini -y -m gemini-2.5-flash -p "Research [library/framework name]:

Find:
- Current stable version
- Key features
- Installation and setup
- Basic usage example
- Pros and cons
- Active maintenance status
- Community adoption
- Alternatives" 2>/dev/null
```

### Technology Comparison

```bash
gemini -y -m gemini-2.5-flash -p "Compare [technology A] vs [technology B] for [use case]:

Compare:
- Performance
- Learning curve
- Community support
- Ecosystem
- Production readiness
- Scalability
- Cost

Provide a recommendation with reasoning." 2>/dev/null
```

**Example:**
```bash
gemini -y -m gemini-2.5-flash -p "Compare Next.js 15 vs Remix for building a large-scale e-commerce platform:

Compare:
- Performance (SSR, routing, caching)
- Learning curve
- Community support and ecosystem
- Production readiness
- Scalability
- Developer experience
- Deployment options

Provide a recommendation with reasoning." 2>/dev/null
```

### Breaking Changes

```bash
gemini -y -m gemini-2.5-flash -p "Search for breaking changes in [library] version [X] to version [Y]:

Find:
- Migration guide
- Deprecated features
- API changes
- New requirements
- Common migration issues

Provide actionable migration steps." 2>/dev/null
```

### Security Advisories

```bash
gemini -y -m gemini-2.5-flash -p "Search for recent security vulnerabilities in [library/framework]:

Find:
- CVE numbers
- Severity levels
- Affected versions
- Recommended patches
- Workarounds if no patch available

Focus on last 12 months." 2>/dev/null
```

## Architecture Analysis

### Project Structure Analysis

```bash
gemini -y --include-directories ./src -m gemini-2.5-pro -p "Analyze the project structure and architecture:

Evaluate:
- Folder organization
- Module boundaries
- Separation of concerns
- Dependencies and coupling
- Scalability of structure

Provide:
1. Current structure overview
2. Strengths
3. Issues and anti-patterns
4. Recommended improvements" 2>/dev/null
```

### Dependency Analysis

```bash
gemini -y --include-directories ./src -m gemini-2.5-pro -p "Analyze dependencies in this codebase:

Find:
- Circular dependencies
- Tight coupling
- Dependency injection opportunities
- Unused dependencies
- Missing abstractions

Provide dependency graph and recommendations." 2>/dev/null
```

### Performance Bottleneck Analysis

```bash
gemini -y --include-directories ./src -m gemini-2.5-pro -p "Identify potential performance bottlenecks:

Analyze:
- Database query patterns
- N+1 query problems
- Inefficient algorithms
- Missing caching opportunities
- Blocking operations
- Memory leaks

For each bottleneck:
1. Location
2. Impact
3. Recommended fix" 2>/dev/null
```

### Security Architecture Review

```bash
gemini -y --include-directories ./src -m gemini-2.5-pro -p "Review the security architecture:

Check:
- Authentication implementation
- Authorization patterns
- Input validation approach
- Sensitive data handling
- API security
- OWASP Top 10 compliance

Provide security score and prioritized fixes." 2>/dev/null
```

## Specialized Tasks

### Git Commit Message

```bash
gemini -y -m gemini-2.5-flash -p "Generate a conventional commit message for these changes:

Changes:
\`\`\`
[paste git diff or describe changes]
\`\`\`

Format: <type>(<scope>): <subject>

Types: feat, fix, docs, style, refactor, test, chore

Include:
- Clear, concise subject line
- Body explaining WHY (not what)
- Breaking changes if applicable" 2>/dev/null
```

### Error Message Explanation

```bash
gemini -y -m gemini-2.5-flash -p "Explain this error in simple terms:

\`\`\`
[paste error message]
\`\`\`

Provide:
1. What the error means
2. Common causes
3. How to fix it
4. How to prevent it

For audience: [junior developer/non-technical/etc.]" 2>/dev/null
```

### Code Snippet Explanation

```bash
gemini -y -m gemini-2.5-flash -p "Explain this code snippet:

\`\`\`[language]
[paste code snippet]
\`\`\`

Explain:
- What it does
- How it works
- Why it's written this way
- Potential issues or gotchas

Keep it clear and concise." 2>/dev/null
```

### SQL Query Optimization

```bash
gemini -y -m gemini-2.5-flash -p "Optimize this SQL query:

\`\`\`sql
[paste SQL query]
\`\`\`

Table schema:
\`\`\`
[paste relevant schema]
\`\`\`

Provide:
1. Current query issues
2. Optimized query
3. Explanation of improvements
4. Index recommendations" 2>/dev/null
```

### Regex Generation

```bash
gemini -y -m gemini-2.5-flash -p "Create a regex pattern for [purpose]:

Requirements:
- [requirement 1]
- [requirement 2]

Valid examples:
- [example 1]
- [example 2]

Invalid examples:
- [example 1]
- [example 2]

Provide:
1. Regex pattern
2. Explanation
3. Test cases
4. [Language] code to use it" 2>/dev/null
```

**Example:**
```bash
gemini -y -m gemini-2.5-flash -p "Create a regex pattern for validating email addresses:

Requirements:
- Follow RFC 5322 standard
- Support common formats
- Reject obvious invalid patterns

Valid examples:
- user@example.com
- user.name+tag@example.co.uk
- user123@sub.example.com

Invalid examples:
- @example.com
- user@
- user@.com

Provide:
1. Regex pattern
2. Explanation
3. Test cases
4. TypeScript code to use it" 2>/dev/null
```

## Template Variables Guide

Common placeholders used in templates:

- `[language]` - Programming language (TypeScript, Python, etc.)
- `[framework]` - Framework name (React, Express, Django, etc.)
- `[description]` - Project/feature description
- `[feature N]` - Specific features to implement
- `[requirement N]` - Specific requirements
- `[paste code]` - Placeholder for code to insert
- `[use case]` - Specific use case or scenario
- `[purpose]` - Intent or goal of code

## Tips for Using Templates

1. **Customize before executing** - Replace all `[placeholders]` with actual values
2. **Add context** - Include relevant background information for better results
3. **Specify constraints** - Mention libraries, patterns, or standards to follow
4. **Use appropriate models** - Pro for architecture/security, Flash for generation
5. **Enable auto-approval** - Use `-y` for trusted operations
6. **Chain templates** - Use Generate → Review → Fix workflow
7. **Save successful prompts** - Build your own template library
8. **Iterate** - Refine prompts based on results

---

**Note:** These templates are starting points. Customize them for your specific needs and context.
