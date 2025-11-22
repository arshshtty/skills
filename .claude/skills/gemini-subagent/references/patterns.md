# Gemini CLI Integration Patterns

Advanced patterns for integrating Gemini CLI into development workflows. These patterns demonstrate how to maximize effectiveness while managing resources.

## Table of Contents

- [Pattern 1: Generate-Review-Fix Cycle](#pattern-1-generate-review-fix-cycle)
- [Pattern 2: JSON Output Processing](#pattern-2-json-output-processing)
- [Pattern 3: Background Execution](#pattern-3-background-execution)
- [Pattern 4: Model Selection Strategy](#pattern-4-model-selection-strategy)
- [Pattern 5: Rate Limit Management](#pattern-5-rate-limit-management)
- [Pattern 6: Context Enrichment](#pattern-6-context-enrichment)
- [Pattern 7: Validation Pipeline](#pattern-7-validation-pipeline)
- [Pattern 8: Incremental Refinement](#pattern-8-incremental-refinement)
- [Pattern 9: Cross-Validation with Claude](#pattern-9-cross-validation-with-claude)
- [Pattern 10: Session Continuity](#pattern-10-session-continuity)

---

## Pattern 1: Generate-Review-Fix Cycle

**Purpose:** Improve code quality by using Gemini in different cognitive modes.

**Problem:** Single-pass code generation may miss edge cases, security issues, or optimization opportunities.

**Solution:** Separate generation, review, and fixing into distinct steps, potentially using different models.

### Implementation

```bash
#!/bin/bash
# Three-step quality assurance cycle

# Step 1: Generate code (Flash for speed)
echo "Generating implementation..."
implementation=$(gemini -y -m gemini-2.5-flash -p "Create a TypeScript function for user authentication with JWT tokens. Include login, token generation, and verification." 2>/dev/null)

echo "$implementation" > auth.ts

# Step 2: Review code (Pro for thorough analysis)
echo "Reviewing implementation..."
review=$(gemini -y -m gemini-2.5-pro -p "Review this authentication code for security vulnerabilities, edge cases, and best practices:

\`\`\`typescript
$(cat auth.ts)
\`\`\`

Focus on:
- JWT security
- Token expiration handling
- Input validation
- Error cases
- OWASP authentication best practices" 2>/dev/null)

echo "$review" > review.txt

# Step 3: Fix identified issues (Flash for implementation)
echo "Fixing issues..."
fixed=$(gemini -y -m gemini-2.5-flash -p "Fix these issues in the authentication code:

Original code:
\`\`\`typescript
$(cat auth.ts)
\`\`\`

Review findings:
$(cat review.txt)

Provide the corrected code." 2>/dev/null)

echo "$fixed" > auth.fixed.ts

echo "Done! Check auth.fixed.ts for final implementation"
```

### Why This Works

- **Generation mode**: Focuses on functionality and getting code working
- **Review mode**: Critical analysis without implementation pressure
- **Fix mode**: Targeted corrections based on specific feedback
- **Different models**: Pro for analysis, Flash for generation (cost-effective)

### When to Use

- Critical code (authentication, payments, security)
- Complex algorithms
- Public APIs
- Production deployments

### Variations

**Quick Review (Same Model):**
```bash
# For less critical code
gemini -y -m gemini-2.5-flash -p "Generate auth function" 2>/dev/null > auth.ts
gemini -y -m gemini-2.5-flash -p "Quick review: $(cat auth.ts)" 2>/dev/null
```

**Extended Cycle:**
```bash
# Add test generation step
gemini -y -m gemini-2.5-flash -p "Generate tests for: $(cat auth.fixed.ts)" 2>/dev/null > auth.test.ts
```

---

## Pattern 2: JSON Output Processing

**Purpose:** Enable programmatic processing of Gemini responses for automation and integration.

**Problem:** Plain text output is hard to parse and process automatically.

**Solution:** Use JSON output format and process with tools like `jq`.

### Implementation

```bash
# Get structured response
response=$(gemini -y -o json -m gemini-2.5-flash -p "Analyze this code and identify all functions, their parameters, and return types:

\`\`\`typescript
export function calculateTotal(items: Item[]): number {
  return items.reduce((sum, item) => sum + item.price, 0);
}

export function applyDiscount(total: number, percent: number): number {
  return total * (1 - percent / 100);
}
\`\`\`

Return as JSON: {functions: [{name, params, returnType, description}]}" 2>/dev/null)

# Extract just the response text
text=$(echo "$response" | jq -r '.candidates[0].content.parts[0].text')

# Parse the extracted JSON (assuming LLM returns valid JSON)
echo "$text" | jq '.functions[] | "\(.name): \(.description)"'

# Get token usage
tokens=$(echo "$response" | jq '.usageMetadata.totalTokenCount')
echo "Tokens used: $tokens"
```

### Advanced Usage

**Build CI/CD Pipeline:**
```bash
#!/bin/bash
# Automated code review in CI

# Review code
review=$(gemini -y -o json -m gemini-2.5-pro -p "Review this PR for critical issues. Return JSON: {critical: [], high: [], medium: [], low: []}" 2>/dev/null)

# Extract critical issues
critical=$(echo "$review" | jq -r '.candidates[0].content.parts[0].text' | jq '.critical | length')

# Fail CI if critical issues found
if [ "$critical" -gt 0 ]; then
  echo "FAILED: $critical critical issues found"
  exit 1
fi

echo "PASSED: No critical issues"
```

**Aggregate Multiple Analyses:**
```bash
# Analyze multiple files
for file in src/*.ts; do
  result=$(gemini -y -o json -p "Count functions in: $(cat $file)" 2>/dev/null)
  count=$(echo "$result" | jq -r '.candidates[0].content.parts[0].text')
  echo "$file: $count functions"
done
```

### JSON Response Structure

```json
{
  "candidates": [
    {
      "content": {
        "parts": [
          {
            "text": "Response text or JSON string here"
          }
        ],
        "role": "model"
      },
      "finishReason": "STOP"
    }
  ],
  "usageMetadata": {
    "promptTokenCount": 234,
    "candidatesTokenCount": 567,
    "totalTokenCount": 801
  }
}
```

### Extracting Common Fields

```bash
# Response text
echo "$response" | jq -r '.candidates[0].content.parts[0].text'

# Token counts
echo "$response" | jq '.usageMetadata'

# Finish reason
echo "$response" | jq -r '.candidates[0].finishReason'
```

---

## Pattern 3: Background Execution

**Purpose:** Run long-running Gemini tasks without blocking other work.

**Problem:** Comprehensive analyses can take 1-2 minutes, blocking terminal.

**Solution:** Execute Gemini commands in background and check results later.

### Implementation

```bash
# Start comprehensive security audit in background
echo "Starting security audit..."
gemini -y -m gemini-2.5-pro -p "Perform comprehensive security audit of:
$(cat src/auth/*.ts)

Check all OWASP Top 10 vulnerabilities." 2>/dev/null > /tmp/security_audit.txt &

AUDIT_PID=$!
echo "Audit running (PID: $AUDIT_PID)"

# Continue with other work...
echo "Doing other work while audit runs..."

# Check if still running
if ps -p $AUDIT_PID > /dev/null; then
  echo "Audit still in progress..."
else
  echo "Audit complete! Results:"
  cat /tmp/security_audit.txt
fi

# Or wait for completion
wait $AUDIT_PID
echo "Audit finished:"
cat /tmp/security_audit.txt
```

### Parallel Execution

**Run multiple independent analyses:**
```bash
# Start multiple background tasks
gemini -y -m gemini-2.5-flash -p "Analyze frontend performance" 2>/dev/null > /tmp/frontend.txt &
PID1=$!

gemini -y -m gemini-2.5-flash -p "Analyze backend performance" 2>/dev/null > /tmp/backend.txt &
PID2=$!

gemini -y -m gemini-2.5-flash -p "Analyze database queries" 2>/dev/null > /tmp/database.txt &
PID3=$!

# Wait for all to complete
wait $PID1 $PID2 $PID3

# Combine results
cat /tmp/frontend.txt /tmp/backend.txt /tmp/database.txt > /tmp/full_analysis.txt
echo "Complete analysis ready!"
```

### With Notifications

```bash
# Long-running task with notification
{
  gemini -y -m gemini-2.5-pro -p "Comprehensive codebase analysis" 2>/dev/null > /tmp/analysis.txt
  notify-send "Gemini Analysis Complete" "Results in /tmp/analysis.txt"
} &
```

### Status Monitoring

```bash
#!/bin/bash
# Monitor background Gemini task

gemini -y -m gemini-2.5-pro -p "Long analysis..." 2>/dev/null > /tmp/result.txt &
PID=$!

while ps -p $PID > /dev/null; do
  echo -n "."
  sleep 5
done

echo "\nComplete!"
cat /tmp/result.txt
```

---

## Pattern 4: Model Selection Strategy

**Purpose:** Optimize cost, speed, and quality by choosing appropriate models.

**Problem:** Using Pro for everything exhausts rate limits; using Flash-Lite for critical tasks reduces quality.

**Solution:** Match model to task complexity and importance.

### Decision Matrix

| Task Type | Model | Reasoning |
|-----------|-------|-----------|
| Architecture design | `gemini-2.5-pro` | Critical decisions need best analysis |
| Security audits | `gemini-2.5-pro` | Vulnerabilities have high impact |
| Code review (critical) | `gemini-2.5-pro` | Payment, auth, security code |
| Code generation | `gemini-2.5-flash` | Good quality, 4x more requests |
| Code review (standard) | `gemini-2.5-flash` | Adequate for most code |
| Test generation | `gemini-2.5-flash` | Tests are validated by running |
| Documentation | `gemini-2.5-flash` | Lower stakes, easier to review |
| Simple queries | `gemini-2.5-flash-lite` | Conserve quota |
| Explanations | `gemini-2.5-flash-lite` | Speed over depth |

### Implementation

```bash
#!/bin/bash
# Smart model selection based on task

task_type=$1
content=$2

case $task_type in
  "architecture"|"security"|"critical")
    model="gemini-2.5-pro"
    ;;
  "generate"|"review"|"test"|"docs")
    model="gemini-2.5-flash"
    ;;
  "explain"|"simple")
    model="gemini-2.5-flash-lite"
    ;;
  *)
    model="gemini-2.5-flash"  # default
    ;;
esac

echo "Using model: $model"
gemini -y -m "$model" -p "$content" 2>/dev/null
```

**Usage:**
```bash
./smart_gemini.sh security "Audit this auth code: ..."
./smart_gemini.sh generate "Create a login form"
./smart_gemini.sh explain "What does this function do?"
```

### Cost-Aware Workflow

```bash
# Use Flash for generation, Pro for final review
gemini -y -m gemini-2.5-flash -p "Generate payment processor" 2>/dev/null > payment.ts

# Only use Pro for critical review
gemini -y -m gemini-2.5-pro -p "Security review: $(cat payment.ts)" 2>/dev/null
```

---

## Pattern 5: Rate Limit Management

**Purpose:** Avoid hitting API rate limits during intensive operations.

**Problem:** Free tier has 15 req/min (Pro) or 60 req/min (Flash) limits.

**Solution:** Implement strategies to spread load and handle limits gracefully.

### Strategy 1: Built-in Auto-Retry

Gemini CLI has automatic retry logic:
```bash
# CLI handles rate limits automatically
gemini -y -m gemini-2.5-flash -p "your prompt" 2>/dev/null
# Will retry automatically if rate limited
```

### Strategy 2: Model Switching

```bash
# Try Pro, fall back to Flash if rate limited
gemini -y -m gemini-2.5-pro -p "analysis task" 2>/dev/null || \
  gemini -y -m gemini-2.5-flash -p "analysis task" 2>/dev/null
```

### Strategy 3: Batch with Delays

```bash
#!/bin/bash
# Process multiple files with rate limiting

for file in src/**/*.ts; do
  echo "Reviewing $file..."
  gemini -y -m gemini-2.5-flash -p "Review: $(cat $file)" 2>/dev/null > "reviews/${file}.txt"

  # Wait 1 second between requests
  sleep 1
done
```

### Strategy 4: Priority Queuing

```bash
#!/bin/bash
# Process critical files with Pro, others with Flash

critical_files=("auth.ts" "payment.ts" "security.ts")
other_files=($(ls src/*.ts | grep -v -E "auth|payment|security"))

# Process critical files first (15 req/min limit)
for file in "${critical_files[@]}"; do
  gemini -y -m gemini-2.5-pro -p "Review: $(cat $file)" 2>/dev/null
  sleep 4  # 15 requests/min = 4s spacing
done

# Process others with Flash (60 req/min limit)
for file in "${other_files[@]}"; do
  gemini -y -m gemini-2.5-flash -p "Review: $(cat $file)" 2>/dev/null
  sleep 1  # 60 requests/min = 1s spacing
done
```

### Strategy 5: Parallel with Limits

```bash
#!/bin/bash
# Process files in parallel with concurrency limit

export -f process_file

process_file() {
  file=$1
  gemini -y -m gemini-2.5-flash -p "Review: $(cat $file)" 2>/dev/null > "reviews/$file.txt"
}

# Process max 10 files at once (well within 60 req/min)
find src -name "*.ts" | xargs -n 1 -P 10 -I {} bash -c 'process_file "{}"'
```

### Rate Limit Detection and Handling

```bash
#!/bin/bash
# Detect rate limit and implement backoff

attempt=1
max_attempts=5
delay=10

while [ $attempt -le $max_attempts ]; do
  response=$(gemini -y -m gemini-2.5-pro -p "your task" 2>&1)

  if echo "$response" | grep -q "rate limit"; then
    echo "Rate limited, waiting ${delay}s (attempt $attempt/$max_attempts)"
    sleep $delay
    delay=$((delay * 2))  # Exponential backoff
    attempt=$((attempt + 1))
  else
    echo "$response"
    break
  fi
done
```

---

## Pattern 6: Context Enrichment

**Purpose:** Improve response quality by providing comprehensive context.

**Problem:** Generic prompts produce generic results.

**Solution:** Enrich prompts with project-specific context, constraints, and requirements.

### Method 1: Project Context File

Create `.gemini/GEMINI.md` in project root:

```markdown
# Project Context

## Technology Stack
- TypeScript 5.3
- React 18 with Server Components
- Next.js 15
- PostgreSQL with Prisma ORM
- Redis for caching
- Jest for testing

## Architecture
- Clean Architecture pattern
- Domain-Driven Design
- Repository pattern for data access
- Service layer for business logic

## Coding Standards
- Functional programming preferred
- Immutable data structures
- Comprehensive error handling
- JSDoc for public APIs
- 80% test coverage minimum

## Security Requirements
- JWT authentication with httpOnly cookies
- CSRF protection on all mutations
- Input sanitization and validation
- Rate limiting on all endpoints
- SQL injection prevention via ORM

## Performance Targets
- Page load < 2s
- API response < 200ms
- Database queries < 50ms
```

Gemini automatically uses this context.

### Method 2: Explicit Context in Prompts

```bash
gemini -y -m gemini-2.5-flash -p "Create a user authentication service.

Project context:
- Using TypeScript with Next.js 15
- PostgreSQL with Prisma ORM
- Clean Architecture (service -> repository)
- JWT in httpOnly cookies
- Must follow our error handling patterns

Requirements:
- Registration with email verification
- Login with rate limiting
- Token refresh mechanism
- Password reset flow

Follow our existing patterns in src/services/AuthService.ts" 2>/dev/null
```

### Method 3: Include Existing Code Patterns

```bash
gemini -y -m gemini-2.5-flash -p "Create a new ProductService following this pattern:

Existing service pattern:
\`\`\`typescript
$(cat src/services/UserService.ts)
\`\`\`

New service should:
- Follow same structure
- Use same error handling
- Implement repository pattern
- Include logging

Create ProductService with CRUD operations." 2>/dev/null
```

### Method 4: Use --include-directories

```bash
# Give Gemini access to project structure
gemini -y --include-directories ./src,./lib -m gemini-2.5-pro -p "Analyze our current authentication flow and suggest improvements based on the actual implementation" 2>/dev/null
```

### Context Enrichment Checklist

When crafting prompts, include:

- ✅ **Tech stack** - Languages, frameworks, versions
- ✅ **Architecture** - Patterns, structure, organization
- ✅ **Standards** - Coding style, conventions, best practices
- ✅ **Constraints** - Security, performance, compatibility requirements
- ✅ **Existing patterns** - Examples of similar code
- ✅ **Dependencies** - Libraries and tools to use/avoid
- ✅ **Context files** - Project-specific documentation

---

## Pattern 7: Validation Pipeline

**Purpose:** Ensure AI-generated code meets quality and security standards before deployment.

**Problem:** AI can generate insecure, buggy, or non-compliant code.

**Solution:** Multi-stage validation pipeline.

### Complete Pipeline

```bash
#!/bin/bash
# Comprehensive validation pipeline

CODE_FILE="$1"
VALIDATION_PASSED=true

echo "=== Validation Pipeline ==="

# Stage 1: Generate Code
echo "1. Generating code..."
gemini -y -m gemini-2.5-flash -p "Generate user registration endpoint with email verification" 2>/dev/null > "$CODE_FILE"

# Stage 2: Security Review
echo "2. Security review..."
security_issues=$(gemini -y -m gemini-2.5-pro -p "Security audit (count critical issues): $(cat $CODE_FILE)" 2>/dev/null)

if echo "$security_issues" | grep -q "Critical"; then
  echo "❌ FAILED: Critical security issues found"
  echo "$security_issues"
  VALIDATION_PASSED=false
fi

# Stage 3: Syntax Check
echo "3. Syntax validation..."
if ! npx tsc --noEmit "$CODE_FILE" 2>/dev/null; then
  echo "❌ FAILED: TypeScript compilation errors"
  VALIDATION_PASSED=false
fi

# Stage 4: Linting
echo "4. Linting..."
if ! npx eslint "$CODE_FILE" 2>/dev/null; then
  echo "❌ FAILED: Linting errors"
  VALIDATION_PASSED=false
fi

# Stage 5: Unit Tests
echo "5. Running tests..."
test_file="${CODE_FILE%.ts}.test.ts"
gemini -y -m gemini-2.5-flash -p "Generate tests for: $(cat $CODE_FILE)" 2>/dev/null > "$test_file"

if ! npm test "$test_file" 2>/dev/null; then
  echo "❌ FAILED: Tests failed"
  VALIDATION_PASSED=false
fi

# Stage 6: Manual Review Reminder
echo "6. Manual review required..."
echo "📋 Please review: $CODE_FILE"

# Final Result
if [ "$VALIDATION_PASSED" = true ]; then
  echo "✅ PASSED: All automated checks passed"
  echo "⚠️  Still requires manual code review before deployment"
  exit 0
else
  echo "❌ FAILED: Validation pipeline failed"
  exit 1
fi
```

### Minimal Pipeline

```bash
# Quick validation for non-critical code
gemini -y -m gemini-2.5-flash -p "Generate helper function" 2>/dev/null > helper.ts
npx tsc --noEmit helper.ts && echo "✅ Valid TypeScript"
```

### Security-Focused Pipeline

```bash
# Extra security validation for critical code
gemini -y -m gemini-2.5-flash -p "Generate payment processing" 2>/dev/null > payment.ts

# Multiple security reviews
gemini -y -m gemini-2.5-pro -p "OWASP Top 10 review: $(cat payment.ts)" 2>/dev/null > security1.txt
gemini -y -m gemini-2.5-pro -p "Race condition review: $(cat payment.ts)" 2>/dev/null > security2.txt
gemini -y -m gemini-2.5-pro -p "Input validation review: $(cat payment.ts)" 2>/dev/null > security3.txt

# Manual review
echo "Security reviews complete. Manual review required before deployment."
cat security1.txt security2.txt security3.txt
```

---

## Pattern 8: Incremental Refinement

**Purpose:** Build complex outputs through iterative improvement.

**Problem:** Complex tasks are hard to get right in one shot.

**Solution:** Start simple, then incrementally refine.

### Implementation

```bash
#!/bin/bash
# Iterative refinement workflow

# Iteration 1: Basic implementation
echo "Iteration 1: Basic structure..."
gemini -y -m gemini-2.5-flash -p "Create basic REST API for user management. Just structure, no implementation." 2>/dev/null > api.ts

# Iteration 2: Add authentication
echo "Iteration 2: Adding authentication..."
gemini -y -m gemini-2.5-flash -p "Add JWT authentication to this API:
$(cat api.ts)

Add middleware for token verification." 2>/dev/null > api.ts

# Iteration 3: Add authorization
echo "Iteration 3: Adding authorization..."
gemini -y -m gemini-2.5-flash -p "Add role-based authorization to this API:
$(cat api.ts)

Roles: admin, user, guest" 2>/dev/null > api.ts

# Iteration 4: Add error handling
echo "Iteration 4: Adding error handling..."
gemini -y -m gemini-2.5-flash -p "Add comprehensive error handling:
$(cat api.ts)

Include: validation errors, auth errors, server errors" 2>/dev/null > api.ts

# Iteration 5: Add logging
echo "Iteration 5: Adding logging..."
gemini -y -m gemini-2.5-flash -p "Add structured logging:
$(cat api.ts)

Log: requests, errors, auth events" 2>/dev/null > api.ts

# Final review
echo "Final review..."
gemini -y -m gemini-2.5-pro -p "Final review of complete API: $(cat api.ts)" 2>/dev/null
```

### With Validation Between Steps

```bash
# Validate after each iteration
for step in "basic" "auth" "validation" "error-handling"; do
  echo "Adding: $step"
  gemini -y -m gemini-2.5-flash -p "Add $step to: $(cat api.ts)" 2>/dev/null > api.ts

  # Validate
  if npx tsc --noEmit api.ts; then
    echo "✅ $step validated"
  else
    echo "❌ $step failed validation"
    break
  fi
done
```

### Benefits

- **Clearer audit trail** - See what was added in each step
- **Easier debugging** - Identify which iteration introduced issues
- **Better quality** - Focus on one concern at a time
- **Manageable complexity** - Break large tasks into small steps

---

## Pattern 9: Cross-Validation with Claude

**Purpose:** Leverage strengths of both AI systems for optimal results.

**Problem:** Each AI has unique strengths and weaknesses.

**Solution:** Use both systems for complementary tasks.

### Strengths Comparison

| Capability | Claude | Gemini |
|------------|--------|--------|
| Codebase reasoning | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| File operations | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| Current web knowledge | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| Google search integration | ❌ | ✅ |
| Multi-turn context | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Code generation | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |

### Workflow Pattern

```
1. Claude analyzes codebase (superior reasoning, file access)
2. Gemini researches current best practices (web search)
3. Claude implements changes (file operations)
4. Gemini reviews implementation (fresh perspective)
5. Claude finalizes (integrate feedback)
```

### Implementation Example

**Scenario: Implementing a new feature**

```bash
# Step 1: Claude analyzes existing codebase
# (Claude uses Read tool to understand current architecture)

# Step 2: Gemini researches latest approaches
gemini -y -m gemini-2.5-flash -p "Search for latest best practices for implementing WebSocket authentication in Next.js 15 with JWT tokens. What are the security considerations in 2025?" 2>/dev/null > research.txt

# Step 3: Claude generates implementation based on research
# (Claude reads research.txt and implements following project patterns)

# Step 4: Gemini provides fresh review perspective
gemini -y -m gemini-2.5-pro -p "Review this WebSocket implementation against 2025 security standards:

$(cat src/websocket-auth.ts)

Check against latest OWASP recommendations." 2>/dev/null > review.txt

# Step 5: Claude incorporates feedback
# (Claude reads review.txt and refines implementation)
```

### Use Cases

**Web-Grounded Decisions:**
```bash
# Gemini for current information
gemini -y -m gemini-2.5-flash -p "What are the breaking changes in React 19?" 2>/dev/null

# Claude for implementation
# (Claude updates code based on Gemini's research)
```

**Architecture Review:**
```bash
# Claude analyzes actual codebase
# (Claude reads all files, understands relationships)

# Gemini provides industry perspective
gemini -y -m gemini-2.5-pro -p "Evaluate this architecture against current industry best practices for microservices..." 2>/dev/null

# Synthesize both perspectives
```

### Best Practices

- Use **Claude** for: Codebase analysis, file operations, complex reasoning
- Use **Gemini** for: Web research, current information, cross-validation
- **Document decisions**: Note which AI provided which insights
- **Synthesize perspectives**: Combine best ideas from both
- **Validate independently**: Don't assume either is always correct

---

## Pattern 10: Session Continuity

**Purpose:** Build complex understanding across multiple interactions.

**Problem:** Complex analysis requires building context over time.

**Solution:** Use Gemini's session management for multi-turn workflows.

### Basic Session Usage

```bash
# Initial analysis (creates new session)
gemini -p "Analyze the authentication system architecture" 2>/dev/null

# Continue analysis (new session)
gemini --resume latest -p "What security vulnerabilities do you see?" 2>/dev/null

# Deep dive (remembers previous context)
gemini --resume latest -p "How would you implement rate limiting for this system?" 2>/dev/null

# Another iteration
gemini --resume latest -p "Show me the code for that rate limiting implementation" 2>/dev/null
```

### Multi-Step Investigation

```bash
#!/bin/bash
# Progressive analysis with session continuity

# Step 1: Initial overview
gemini -p "Analyze the overall project structure" 2>/dev/null > step1.txt
echo "Session created. Step 1 complete."

# Step 2: Zoom into auth
gemini --resume latest -p "Now focus on the authentication implementation. What patterns are being used?" 2>/dev/null > step2.txt
echo "Step 2 complete."

# Step 3: Security analysis
gemini --resume latest -p "Based on that authentication implementation, what are the security risks?" 2>/dev/null > step3.txt
echo "Step 3 complete."

# Step 4: Recommendations
gemini --resume latest -p "Given everything we've discussed, provide prioritized security improvements" 2>/dev/null > step4.txt
echo "Step 4 complete."

# Combine all steps
cat step1.txt step2.txt step3.txt step4.txt > full_analysis.txt
```

### Session Management

```bash
# List all sessions
gemini --list-sessions

# Resume specific session by index
gemini --resume 2 -p "continue from session 2"

# Resume latest
gemini --resume latest -p "continue from last session"

# Delete old sessions
gemini --delete-session 5
```

### Use Cases

**Architectural Discovery:**
```bash
gemini -p "Explain the data flow in this application" 2>/dev/null
gemini --resume latest -p "Where are the potential bottlenecks?" 2>/dev/null
gemini --resume latest -p "How would you optimize the bottlenecks?" 2>/dev/null
```

**Iterative Design:**
```bash
gemini -p "Design a caching strategy for this API" 2>/dev/null
gemini --resume latest -p "What if we have multiple servers?" 2>/dev/null
gemini --resume latest -p "How do we handle cache invalidation?" 2>/dev/null
gemini --resume latest -p "Show me the Redis implementation" 2>/dev/null
```

**Learning/Exploration:**
```bash
gemini -p "Explain how our authorization system works" 2>/dev/null
gemini --resume latest -p "What are the limitations of this approach?" 2>/dev/null
gemini --resume latest -p "How does OAuth 2.0 compare?" 2>/dev/null
gemini --resume latest -p "Should we migrate? What's involved?" 2>/dev/null
```

---

## Combining Patterns

These patterns are most powerful when combined:

```bash
#!/bin/bash
# Ultimate workflow: Multiple patterns combined

# Pattern 6: Context Enrichment (include project dirs)
# Pattern 4: Model Selection (Pro for architecture)
# Pattern 10: Session Continuity (multi-turn analysis)
gemini --include-directories ./src -m gemini-2.5-pro -p "Analyze our microservices architecture" 2>/dev/null

# Pattern 1: Generate-Review-Fix
# Pattern 4: Model Selection (Flash for generation, Pro for review)
implementation=$(gemini -y -m gemini-2.5-flash -p "Create API gateway based on analysis" 2>/dev/null)
echo "$implementation" > gateway.ts

review=$(gemini -y -m gemini-2.5-pro --resume latest -p "Review this gateway implementation: $(cat gateway.ts)" 2>/dev/null)

# Pattern 7: Validation Pipeline
npx tsc --noEmit gateway.ts && npx eslint gateway.ts

# Pattern 2: JSON Output Processing
metrics=$(gemini -y -o json --resume latest -p "Provide metrics for this implementation" 2>/dev/null)
echo "$metrics" | jq '.usageMetadata'

echo "Complete workflow finished!"
```

---

## Pattern Selection Guide

| Scenario | Recommended Patterns |
|----------|---------------------|
| Critical code (auth, payments) | 1, 4, 7 (Generate-Review-Fix, Model Selection, Validation) |
| Large codebase analysis | 3, 6, 10 (Background, Context, Sessions) |
| CI/CD integration | 2, 5, 7 (JSON, Rate Limits, Validation) |
| Learning new codebase | 6, 9, 10 (Context, Cross-Validation, Sessions) |
| Quick code generation | 1, 4 (Generate-Review-Fix, Model Selection) |
| Research and implementation | 9, 10 (Cross-Validation, Sessions) |
| Batch processing | 3, 5 (Background, Rate Limits) |

---

**Remember:** These patterns are frameworks, not rigid rules. Adapt and combine them based on your specific needs and context.
