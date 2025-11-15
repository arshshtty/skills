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

### With Timeout
```bash
# Set timeout to 120000ms (2 minutes) for Gemini responses
gemini -p "your prompt here" 2>/dev/null
```

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

Here's a complete workflow for building a new feature:

```bash
# 1. Brainstorm approaches
gemini -p "I need to add real-time notifications to a React app. Brainstorm approaches considering: SSE, WebSockets, polling. Trade-offs for each?" 2>/dev/null

# [Review response, decide on WebSockets]

# 2. Get feedback on plan
gemini -p "Planning to use Socket.io for WebSockets with Redis adapter for horizontal scaling. Auth via JWT in connection handshake. Room-based notifications. Any issues or improvements?" 2>/dev/null

# [Review feedback, refine plan]

# 3. Implement the feature
# [Write code...]

# 4. Code review
gemini -p "Review this Socket.io server code for security, performance, and best practices:
\`\`\`typescript
[code here]
\`\`\`
Focus on: auth handling, room management, memory leaks" 2>/dev/null

# [Review suggestions, refactor]

# 5. Final validation
# [Test, commit, deploy]
```

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

## Limitations

- **Response time**: Up to 2 minutes - factor this into your workflow
- **Context limit**: Very long prompts may hit token limits
- **Not deterministic**: Responses may vary between calls
- **Requires installation**: gemini-cli must be installed and configured
- **Credential requirement**: Must have valid Gemini API credentials

---

**Remember**: Gemini is a collaborative tool, not a replacement for your judgment. Use it to enhance your decision-making, validate approaches, and gain alternative perspectives - but always think critically about its suggestions.
