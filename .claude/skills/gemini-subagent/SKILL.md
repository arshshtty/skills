---
name: gemini-subagent
description: Use Gemini CLI as a sub-agent to brainstorm ideas, get feedback, plan implementations, and perform code reviews. Apply when you need a second perspective or alternative approach.
---

# Gemini Sub-Agent Skill

Use Gemini CLI as a collaborative sub-agent to get alternative perspectives, brainstorm ideas, gather feedback, and perform code reviews.

## When to Use Gemini

Consult Gemini as a sub-agent in these scenarios:

✅ **Brainstorming**: Generate alternative approaches or creative solutions
✅ **Feedback**: Get a second opinion on your proposed implementation
✅ **Planning**: Validate architecture decisions or implementation strategies
✅ **Code Review**: Have Gemini review code for issues, improvements, or best practices
✅ **Alternative Perspective**: When you want a different model's take on a problem

⚠️ **Don't overuse**: Use Gemini for meaningful collaboration, not every trivial task

## Command Format

### Basic Usage
```bash
gemini -p "your prompt here" 2>/dev/null
```

**Important Notes:**
- Use `2>/dev/null` to suppress stderr logging and get clean responses
- Responses can take up to **2 minutes** - set appropriate timeouts
- Output is on stdout, ready to capture and process
- Use clear, specific prompts for best results

### Advanced Usage with Flags

#### Auto-Approval Mode (--yolo)
Skip confirmation prompts for tool calls - critical for automation:
```bash
gemini --yolo -p "your prompt here" 2>/dev/null
# or short form
gemini -y -p "your prompt here" 2>/dev/null
```

#### JSON Output (--output-format)
Get structured output for programmatic processing:
```bash
gemini -o json -p "your prompt here" 2>/dev/null
```

#### Model Selection (--model)
Choose the right model for your task:
```bash
# For complex architecture, security reviews, critical decisions
gemini -m gemini-2.5-pro -p "your prompt" 2>/dev/null

# For standard tasks, code reviews, test generation (faster, 60 req/min free tier)
gemini -m gemini-2.5-flash -p "your prompt" 2>/dev/null

# For trivial tasks, quota preservation
gemini -m gemini-2.5-flash-lite -p "your prompt" 2>/dev/null
```

#### Session Continuity (--resume)
Continue previous conversations:
```bash
# Resume latest session
gemini --resume latest -p "continue from where we left off" 2>/dev/null

# List available sessions
gemini --list-sessions 2>/dev/null
```

#### Combined Flags
```bash
# Auto-approve + JSON output + specific model
gemini -y -o json -m gemini-2.5-flash -p "your prompt" 2>/dev/null
```

### Model Selection Strategy

Choose models based on task complexity and rate limits:

| Model | Use Case | Rate Limit (Free) | When to Use |
|-------|----------|-------------------|-------------|
| `gemini-2.5-pro` | Complex architecture, security audits, critical reviews | 15 req/min | High-stakes decisions, deep analysis |
| `gemini-2.5-flash` | Standard code generation/review, testing | 60 req/min | Most development tasks |
| `gemini-2.5-flash-lite` | Simple queries, trivial tasks | Higher limit | Quota preservation |

**Tip**: Default to `gemini-2.5-flash` for most tasks. Use `gemini-2.5-pro` when you need the absolute best analysis.

## Use Case Patterns

### 1. Brainstorming Ideas

**When to use**: Need creative alternatives or multiple approaches

**Pattern**:
```bash
gemini -p "I'm building [description]. Brainstorm 3-5 different approaches to [specific challenge]. Consider trade-offs for each." 2>/dev/null
```

**Example**:
```bash
gemini -p "I'm building a real-time collaborative editor. Brainstorm 3-5 different approaches to handle conflict resolution when multiple users edit the same document. Consider trade-offs for latency, complexity, and data consistency." 2>/dev/null
```

**After receiving response**:
- Review Gemini's suggestions
- Compare with your initial approach
- Synthesize the best ideas from both perspectives
- Proceed with implementation or ask follow-up questions

### 2. Getting Feedback on Plans

**When to use**: Before implementing a significant feature or architecture

**Pattern**:
```bash
gemini -p "I'm planning to [your plan]. Here's my approach: [details]. What potential issues or improvements do you see?" 2>/dev/null
```

**Example**:
```bash
gemini -p "I'm planning to implement user authentication using JWT tokens stored in httpOnly cookies with refresh token rotation. The access token expires in 15 minutes, refresh token in 7 days. What potential security issues or improvements do you see?" 2>/dev/null
```

**After receiving response**:
- Address any concerns raised
- Incorporate valid suggestions
- Refine your plan based on feedback

### 3. Planning Implementation

**When to use**: Starting a complex feature or refactoring

**Pattern**:
```bash
gemini -p "I need to implement [feature]. Given [constraints/context], create a step-by-step implementation plan. Consider [specific concerns]." 2>/dev/null
```

**Example**:
```bash
gemini -p "I need to implement a rate limiting system for an API with 100k requests/second. Given we use Node.js and Redis, create a step-by-step implementation plan. Consider distributed systems, race conditions, and performance." 2>/dev/null
```

**After receiving response**:
- Compare with your own planning approach
- Merge both plans, taking the best steps from each
- Identify any gaps or risks Gemini highlighted
- Create a final implementation plan

### 4. Code Review

**When to use**: After writing significant code, before committing

**Pattern**:
```bash
gemini -p "Review this code for bugs, performance issues, security vulnerabilities, and best practices:

\`\`\`[language]
[your code]
\`\`\`

Focus on: [specific areas of concern]" 2>/dev/null
```

**Example**:
```bash
gemini -p "Review this code for bugs, performance issues, security vulnerabilities, and best practices:

\`\`\`typescript
export async function processPayment(userId: string, amount: number) {
  const user = await db.users.findById(userId);
  const balance = user.balance - amount;
  await db.users.update(userId, { balance });
  await db.transactions.create({ userId, amount, type: 'debit' });
  return { success: true, newBalance: balance };
}
\`\`\`

Focus on: race conditions, error handling, transaction safety" 2>/dev/null
```

**After receiving response**:
- Address critical bugs or security issues immediately
- Evaluate suggested improvements
- Refactor based on valid feedback
- Document any intentional design decisions that differ from suggestions

### 5. Architecture Validation

**When to use**: Designing system architecture or making major technical decisions

**Pattern**:
```bash
gemini -p "I'm designing [system]. Here's the architecture: [description]. Evaluate this for scalability, maintainability, and potential bottlenecks." 2>/dev/null
```

**Example**:
```bash
gemini -p "I'm designing a microservices architecture with: API Gateway -> [Auth Service, User Service, Order Service, Payment Service] -> PostgreSQL databases (one per service) -> Redis cache -> RabbitMQ for async tasks. Evaluate this for scalability, maintainability, and potential bottlenecks." 2>/dev/null
```

### 6. Debugging Assistance

**When to use**: Stuck on a complex bug or unexpected behavior

**Pattern**:
```bash
gemini -p "I'm experiencing [problem]. Here's the relevant code and error: [details]. What could be causing this and how should I debug it?" 2>/dev/null
```

**Example**:
```bash
gemini -p "I'm experiencing memory leaks in my Node.js app. Heap usage grows from 50MB to 2GB over 24 hours. Here's the relevant code for event listeners:

\`\`\`javascript
class DataProcessor {
  constructor() {
    eventEmitter.on('data', this.handleData.bind(this));
  }
  handleData(data) { /* process */ }
}
\`\`\`

New instances are created every hour. What could be causing this and how should I debug it?" 2>/dev/null
```

## Gemini's Built-in Tools

Gemini CLI provides unique tools not available in Claude Code:

### google_web_search
Real-time internet search for current information:
```bash
gemini -y -p "Search for the latest best practices for React Server Components in 2025" 2>/dev/null
```

**Use when:**
- Need current information (library versions, best practices, breaking changes)
- Researching new technologies or frameworks
- Finding real-world examples or solutions
- Comparing libraries or approaches

### codebase_investigator
Deep architecture analysis of your project:
```bash
gemini -y --include-directories ./src -p "Analyze the architecture of this codebase and identify potential bottlenecks" 2>/dev/null
```

**Use when:**
- Understanding unfamiliar codebases
- Identifying architectural issues
- Planning refactoring efforts
- Analyzing dependencies and relationships

### save_memory
Persist information across sessions:
```bash
gemini -p "Save this architectural decision: We use Redis for session storage because..." 2>/dev/null
```

**Use when:**
- Building context across multiple sessions
- Documenting decisions for future reference
- Creating project-specific knowledge base

## Advanced Integration Patterns

### Pattern 1: Generate-Review-Fix Cycle

Use Gemini in different modes for quality assurance:

```bash
# Step 1: Generate code (Flash model for speed)
gemini -y -m gemini-2.5-flash -p "Create a REST API endpoint for user authentication with JWT" 2>/dev/null

# Step 2: Review generated code (Pro model for thorough analysis)
gemini -y -m gemini-2.5-pro -p "Review this code for security vulnerabilities, race conditions, and best practices:
\`\`\`typescript
[paste generated code]
\`\`\`
Focus on: authentication security, token handling, error cases" 2>/dev/null

# Step 3: Fix identified issues (Flash model for implementation)
gemini -y -m gemini-2.5-flash -p "Fix these issues in the code: [paste review findings]" 2>/dev/null
```

**Why this works:**
- Generation mode focuses on functionality
- Review mode focuses on quality and security
- Separation catches issues that single-pass generation misses

### Pattern 2: JSON Output for Automation

Extract structured data for programmatic processing:

```bash
# Get structured analysis
response=$(gemini -o json -p "Analyze this code and return: {findings: [], severity: string, recommendation: string}" 2>/dev/null)

# Process with jq
echo "$response" | jq '.candidates[0].content.parts[0].text'
```

**Use cases:**
- Extracting metrics and statistics
- Building CI/CD pipelines with AI validation
- Aggregating multiple AI responses
- Creating structured reports

### Pattern 3: Background Execution

Run long-running tasks while continuing work:

```bash
# Start background task (if supported by your shell)
gemini -y -m gemini-2.5-pro -p "Comprehensive security audit of the entire authentication system" 2>/dev/null > /tmp/gemini_audit.txt &

# Continue with other work...

# Check results later
cat /tmp/gemini_audit.txt
```

### Pattern 4: Cross-Validation with Claude

Leverage both AI systems for complementary strengths:

```bash
# 1. Claude analyzes codebase (superior reasoning, file ops)
# [Claude reads files, analyzes structure]

# 2. Gemini provides web-grounded perspective
gemini -y -p "Review this authentication implementation against 2025 OWASP best practices and current security standards" 2>/dev/null

# 3. Synthesize both perspectives for optimal solution
```

**Strengths:**
- **Claude**: Superior reasoning, codebase context, file operations
- **Gemini**: Current web knowledge, Google search integration, unique tools

### Pattern 5: Session Continuity for Complex Analysis

Build context across multiple interactions:

```bash
# Session 1: Initial analysis
gemini -p "Analyze the user service architecture" 2>/dev/null

# Session 2: Continue analysis (remembers context)
gemini --resume latest -p "Now analyze how authentication integrates with the user service" 2>/dev/null

# Session 3: Deep dive
gemini --resume latest -p "What are the security implications of this integration?" 2>/dev/null
```

## Best Practices

### Crafting Effective Prompts

**Do**:
- ✅ Be specific about the context and constraints
- ✅ Ask focused questions with clear goals
- ✅ Provide relevant code snippets or architecture details
- ✅ Specify what aspects to focus on
- ✅ Include any specific concerns or requirements

**Don't**:
- ❌ Ask vague, open-ended questions
- ❌ Dump entire files without context
- ❌ Expect Gemini to make decisions for you
- ❌ Skip providing necessary background information

### Handling Responses

1. **Always review critically**: Gemini provides suggestions, not absolute truth
2. **Synthesize perspectives**: Combine your approach with Gemini's ideas
3. **Validate suggestions**: Test recommendations before applying them
4. **Document decisions**: Note why you chose or rejected Gemini's advice
5. **Follow up if needed**: Ask clarifying questions if responses are unclear

### Validation Pipeline

**Always validate AI-generated code before deployment:**

```bash
# 1. Generate code
gemini -y -m gemini-2.5-flash -p "Create a payment processing function" 2>/dev/null

# 2. Security review (use Pro model for critical code)
gemini -y -m gemini-2.5-pro -p "Security audit this payment code: [code]" 2>/dev/null

# 3. Syntax check
# [Use language-specific linters/compilers]

# 4. Functional testing
# [Run unit tests, integration tests]

# 5. Manual review
# [Review changes yourself before committing]
```

**Critical: Never deploy AI-generated code without:**
- ✅ Security review for vulnerabilities
- ✅ Syntax validation with linters/compilers
- ✅ Functional testing with test suites
- ✅ Manual code review by developer
- ✅ Understanding what the code does

### Timeout Management

```bash
# For Bash tool, set timeout to 120000ms (2 minutes)
# Gemini can take close to 2 minutes to respond
```

In your Bash tool calls, use:
```json
{
  "command": "gemini -p \"your prompt\" 2>/dev/null",
  "timeout": 120000
}
```

### Error Handling

If Gemini fails or times out:
- Check if gemini-cli is installed and configured
- Verify credentials are set up (`Loaded cached credentials` should appear on stderr)
- Ensure the prompt doesn't contain special characters that break the command
- Try a simpler prompt to test connectivity
- Fall back to proceeding without Gemini's input if necessary

## Workflow Example

Here's a complete workflow for building a new feature with advanced patterns:

```bash
# 1. Research current best practices (Web search tool)
gemini -y -m gemini-2.5-flash -p "Search for the latest best practices for real-time notifications in React apps in 2025. Compare SSE, WebSockets, and polling." 2>/dev/null

# [Review response, decide on WebSockets]

# 2. Get feedback on architectural plan (Pro model for architecture)
gemini -m gemini-2.5-pro -p "Planning to use Socket.io for WebSockets with Redis adapter for horizontal scaling. Auth via JWT in connection handshake. Room-based notifications. Evaluate this architecture for scalability, security, and potential bottlenecks." 2>/dev/null

# [Review feedback, refine plan]

# 3. Generate implementation (Flash model for speed, auto-approve)
gemini -y -m gemini-2.5-flash -p "Create a Socket.io server with Redis adapter, JWT authentication in handshake, and room-based notifications. Include TypeScript types." 2>/dev/null

# [Claude integrates code into project]

# 4. Security review (Pro model for thorough analysis)
gemini -y -m gemini-2.5-pro -p "Review this Socket.io server code for security vulnerabilities, race conditions, and best practices:
\`\`\`typescript
[code here]
\`\`\`
Focus on: JWT validation, room authorization, memory leaks, DoS vectors" 2>/dev/null

# [Address critical findings]

# 5. Generate tests (Flash model, auto-approve)
gemini -y -m gemini-2.5-flash -p "Generate unit and integration tests for this Socket.io implementation. Include: connection tests, auth tests, room tests, error cases." 2>/dev/null

# 6. Final validation
# [Run tests, verify functionality, commit]
```

**This workflow demonstrates:**
- ✅ Using web search for current information
- ✅ Choosing appropriate models (Pro for architecture, Flash for generation)
- ✅ Auto-approval (`-y`) for trusted operations
- ✅ Generate-Review-Fix cycle for quality
- ✅ Comprehensive testing coverage

## Output Processing

Gemini's output is plain text on stdout. Process it naturally:

```bash
# Capture response in variable (if needed in a script)
response=$(gemini -p "your prompt" 2>/dev/null)

# Direct output (typical for CLI usage)
gemini -p "your prompt" 2>/dev/null
```

## Integration Tips

- **Before major commits**: Use Gemini for code review
- **At planning phase**: Get alternative architectural perspectives
- **When stuck**: Consult for debugging assistance
- **For creative tasks**: Leverage for brainstorming
- **During refactoring**: Validate approach and identify edge cases

## Rate Limiting & Quota Management

Gemini API has rate limits on the free tier:

| Model | Free Tier Limit | Strategy |
|-------|----------------|----------|
| `gemini-2.5-pro` | 15 requests/minute | Reserve for critical analysis |
| `gemini-2.5-flash` | 60 requests/minute | Use for most tasks |
| `gemini-2.5-flash-lite` | Higher limit | Use for trivial queries |

**Rate Limit Handling Strategies:**

1. **Model Selection**: Use Flash for routine tasks, Pro for critical decisions
2. **Batch Operations**: Group related requests together
3. **Auto-Retry**: Gemini CLI has built-in retry logic for rate limits
4. **Spread Load**: Add delays in automated scripts: `sleep 1 && gemini ...`

**If you hit rate limits:**
```bash
# Wait and retry (CLI handles this automatically)
# Or switch to faster model
gemini -y -m gemini-2.5-flash-lite -p "your prompt" 2>/dev/null
```

## Limitations

- **Response time**: Up to 2 minutes - factor this into your workflow
- **Context limit**: Very long prompts may hit token limits (~1M tokens for Pro)
- **Not deterministic**: Responses may vary between calls
- **Requires installation**: gemini-cli must be installed and configured (`npm install -g @google/gemini-cli`)
- **Credential requirement**: Must have valid Gemini API credentials (set via `GEMINI_API_KEY` env var or OAuth)
- **Rate limits**: Free tier has request-per-minute limits (see above)

---

**Remember**: Gemini is a collaborative tool, not a replacement for your judgment. Use it to enhance your decision-making, validate approaches, and gain alternative perspectives - but always think critically about its suggestions.
