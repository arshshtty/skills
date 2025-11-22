---
name: debugging-methodology
description: Systematic debugging methodology to efficiently identify and fix bugs. Apply when encountering unexpected behavior, errors, or performance issues in any codebase.
---

# Debugging Methodology Skill

Master systematic debugging techniques to find and fix bugs efficiently, reducing frustration and wasted time.

## The Scientific Method of Debugging

```
1. Observe    → What's actually happening?
2. Reproduce  → Can you make it happen again?
3. Isolate    → Where exactly is the problem?
4. Hypothesize → What might be causing it?
5. Test       → Does the fix work?
6. Verify     → Is it really solved?
```

## Core Principles

### 1. **Stay Calm and Systematic**
- Don't panic or randomly change things
- Follow a methodical process
- Document your findings as you go
- Take breaks when stuck

### 2. **Understand Before Fixing**
- Never fix a bug you don't understand
- Rushed fixes create new bugs
- Root cause > symptoms

### 3. **Reproduce First**
- If you can't reproduce it, you can't fix it
- Consistent reproduction = 50% of the work done
- Document exact steps to reproduce

### 4. **One Change at a Time**
- Change one thing, test, observe
- Multiple simultaneous changes = confusion
- Revert changes that don't help

## Step-by-Step Debugging Process

### Step 1: Observe the Problem

**Ask**:
- What is the expected behavior?
- What is the actual behavior?
- When did it start happening?
- Has anything changed recently?

**Gather Information**:
- Error messages (full stack traces)
- Logs and console output
- User reports and descriptions
- Screenshots or recordings
- System metrics (memory, CPU, network)

**Example**:
```
Expected: User submits form, sees success message
Actual: Form submission shows "Network Error"
Started: After deployment yesterday
Changed: Updated authentication library
```

### Step 2: Reproduce the Bug

**Create Minimal Reproduction**:
- Simplify the scenario as much as possible
- Remove unnecessary variables
- Identify exact conditions needed

**Reproduction Checklist**:
- [ ] Can you trigger the bug on demand?
- [ ] Does it happen every time or intermittently?
- [ ] Does it happen in different environments?
- [ ] Can you reproduce it in isolation?

**Intermittent Bugs**:
- Look for race conditions
- Check for timing dependencies
- Consider external factors (network, load, time-based logic)
- Add more logging to capture state

**Example**:
```
Steps to reproduce:
1. Log in as user with role "admin"
2. Navigate to /dashboard
3. Click "Export Data" button
4. Wait 5 seconds
Result: Error 500 appears
Frequency: 100% reproducible
```

### Step 3: Isolate the Problem

**Binary Search Technique**:
- Divide the codebase in half
- Determine which half contains the bug
- Repeat until you find the exact location

**Questions to Ask**:
- Is it frontend or backend?
- Is it in the application or infrastructure?
- Is it new code or existing code?
- Is it in the logic or the data?

**Techniques**:
1. **Comment out code**: Disable sections to narrow down
2. **Print/log statements**: Add debugging output
3. **Debugger breakpoints**: Step through execution
4. **Git bisect**: Find the commit that introduced the bug
5. **Diff comparison**: Compare working vs broken states

**Example**:
```
1. Error is 500 from backend (not frontend)
2. Add logging in controller → request reaches here
3. Add logging before DB query → reaches here
4. Add logging after DB query → doesn't reach here
Conclusion: Bug is in the database query
```

### Step 4: Form Hypotheses

**Generate Possible Causes**:
- What could cause this behavior?
- What assumptions might be wrong?
- What edge cases weren't considered?

**Common Bug Categories**:
- **Logic errors**: Wrong condition, off-by-one
- **State issues**: Uninitialized variables, stale data
- **Timing**: Race conditions, async issues
- **Input**: Null/undefined, wrong type, out of range
- **Environment**: Config differences, missing dependencies
- **External**: API changes, network issues, database problems

**Example Hypotheses**:
```
Hypothesis 1: Database connection pool is exhausted
Hypothesis 2: Query is missing WHERE clause
Hypothesis 3: User doesn't have required permissions
Hypothesis 4: Query timeout due to missing index
```

### Step 5: Test Each Hypothesis

**Prioritize**:
- Test most likely hypotheses first
- Test easiest to verify first
- Test potentially dangerous issues first (security, data loss)

**Test Systematically**:
```
Hypothesis 1: DB connection pool exhausted
Test: Check pool size and active connections
Result: Pool has 8/10 available ❌ Not the cause

Hypothesis 2: Query missing WHERE clause
Test: Log the generated SQL query
Result: Query includes WHERE clause ❌ Not the cause

Hypothesis 3: Missing permissions
Test: Run query with admin account
Result: Still fails ❌ Not the cause

Hypothesis 4: Query timeout from missing index
Test: Run EXPLAIN on query
Result: Full table scan on 10M rows ✅ Found it!
```

### Step 6: Implement Fix

**Before Fixing**:
- Understand the root cause completely
- Consider side effects of the fix
- Think about similar bugs elsewhere
- Plan how to verify the fix works

**Write the Fix**:
- Keep it simple and focused
- Fix the root cause, not symptoms
- Add comments explaining why
- Consider adding tests to prevent regression

**Example Fix**:
```sql
-- Add index to prevent full table scan
CREATE INDEX idx_users_last_login ON users(last_login_at);

-- Or optimize the query
-- Before: SELECT * FROM users WHERE last_login_at > '2024-01-01'
-- After: Use pagination or add appropriate index
```

### Step 7: Verify the Fix

**Test Thoroughly**:
- [ ] Original bug is fixed
- [ ] No new bugs introduced
- [ ] Works in all environments
- [ ] Performance is acceptable
- [ ] Edge cases are handled
- [ ] Related functionality still works

**Regression Testing**:
- Run existing test suite
- Manually test related features
- Check for side effects

**Document**:
- What was the bug?
- What caused it?
- How was it fixed?
- What tests were added?

## Debugging Techniques

### Print/Log Debugging

**When to Use**: Quick insights, production issues

**Technique**:
```
function processOrder(order) {
  console.log('Processing order:', order.id)
  console.log('Order state:', order.status)

  const total = calculateTotal(order.items)
  console.log('Calculated total:', total)

  if (total > 1000) {
    console.log('Applying discount')
    applyDiscount(order)
  }

  console.log('Final order:', order)
}
```

**Best Practices**:
- Use structured logging (JSON)
- Include context (user ID, request ID)
- Add timestamps
- Use log levels (DEBUG, INFO, WARN, ERROR)
- Remove debug logs before committing

### Debugger Usage

**When to Use**: Complex logic, need to inspect state

**Techniques**:
- **Breakpoints**: Pause execution at specific lines
- **Step Over**: Execute next line
- **Step Into**: Enter function calls
- **Step Out**: Exit current function
- **Watch Expressions**: Monitor variable values
- **Conditional Breakpoints**: Break only when condition is true

**Example Workflow**:
```
1. Set breakpoint at suspected line
2. Run in debug mode
3. When paused, inspect variables
4. Step through code line by line
5. Watch how state changes
6. Identify where behavior diverges from expectations
```

### Rubber Duck Debugging

**When to Use**: When stuck, before asking for help

**How**:
1. Explain the problem out loud (to a rubber duck, colleague, or yourself)
2. Walk through your code line by line
3. Describe what each part does
4. Often you'll realize the issue while explaining

### Binary Search Debugging

**When to Use**: Bug somewhere in large codebase

**Process**:
```
1. Identify working and broken states
2. Find midpoint between them
3. Test if midpoint works
4. If yes, bug is in second half
5. If no, bug is in first half
6. Repeat until found
```

**Git Bisect** (automated binary search):
```bash
git bisect start
git bisect bad              # Current commit is broken
git bisect good <commit>    # Known working commit
# Git checks out middle commit
# Test it
git bisect good             # If it works
# or
git bisect bad              # If it's broken
# Repeat until git identifies the breaking commit
```

### Differential Debugging

**When to Use**: Works in one place but not another

**Compare**:
- Development vs Production
- Your machine vs colleague's machine
- Old version vs new version
- Different inputs or data

**Look for differences in**:
- Environment variables
- Configuration files
- Dependencies and versions
- Data states
- System resources

### Trace/Stack Analysis

**When to Use**: Crashes, exceptions, unexpected behavior

**Stack Trace Reading**:
```
Error: Cannot read property 'name' of undefined
  at processUser (user.js:45:18)
  at handleRequest (api.js:120:5)
  at Router.handle (express.js:500:10)
```

**Read from bottom to top**:
1. Express received request
2. Routed to handleRequest
3. Called processUser
4. Error occurred at user.js:45

**Find**:
- Where the error originated
- The call chain that led to it
- Context and parameters

### Performance Profiling

**When to Use**: Slow performance, memory leaks

**Techniques**:
- **CPU Profiling**: Find hot spots (functions consuming most time)
- **Memory Profiling**: Find leaks and high memory usage
- **Network Analysis**: Check API call times and sizes

**Tools** (Language Agnostic Concepts):
- Built-in profilers
- Performance monitoring tools
- Memory heap snapshots
- Network request watchers

### Logging Best Practices

**Structured Logging**:
```json
{
  "timestamp": "2024-01-15T10:30:00Z",
  "level": "ERROR",
  "message": "Failed to process payment",
  "context": {
    "userId": "123",
    "orderId": "456",
    "amount": 99.99,
    "error": "Payment gateway timeout"
  },
  "requestId": "req_abc123"
}
```

**What to Log**:
- Application lifecycle events (startup, shutdown)
- Errors and exceptions
- Security events (login, auth failures)
- Business transactions
- External service calls
- Performance metrics

**What NOT to Log**:
- Passwords or secrets
- Personal data (PII) without necessity
- Every single request in production
- Sensitive financial information

## Debugging Different Types of Bugs

### Logic Bugs

**Symptoms**: Wrong output, incorrect behavior

**Approach**:
- Trace through the logic manually
- Check conditional statements
- Verify loop boundaries
- Test with edge cases

**Example**:
```
Bug: Discount calculated incorrectly
Debug:
  - Input: $100, 20% discount
  - Expected: $80
  - Actual: $120
  - Code: total = total + (total * discount)
  - Issue: Should be subtraction, not addition
  - Fix: total = total - (total * discount)
```

### Race Conditions

**Symptoms**: Intermittent failures, works sometimes

**Approach**:
- Look for shared state
- Check async/concurrent operations
- Add synchronization/locking
- Use atomic operations

**Example**:
```
Bug: Counter sometimes shows wrong value
Debug:
  - Multiple threads increment counter
  - counter++ is not atomic (read, increment, write)
  - Two threads read 10, both write 11
  - Fix: Use atomic increment or lock
```

### Memory Leaks

**Symptoms**: Memory usage grows over time

**Approach**:
- Take memory snapshots over time
- Look for growing collections
- Check for event listeners not removed
- Find circular references

**Example**:
```
Bug: Node.js app memory grows from 50MB to 2GB
Debug:
  - Heap snapshot shows 10,000 event listeners
  - Event listeners added but never removed
  - Fix: Remove listeners when component unmounts
```

### Integration Issues

**Symptoms**: Works in isolation, fails when integrated

**Approach**:
- Check interface contracts
- Verify data formats
- Look for version mismatches
- Test integration points

**Example**:
```
Bug: API call fails after deploying new service
Debug:
  - Service A sends: { "userId": 123 }
  - Service B expects: { "user_id": "123" }
  - Field name changed from camelCase to snake_case
  - Type changed from number to string
  - Fix: Align data contracts or add adapter
```

### Environment-Specific Bugs

**Symptoms**: Works locally, fails in production

**Approach**:
- Compare environment configurations
- Check for hardcoded values
- Verify environment variables
- Look for data differences

**Example**:
```
Bug: Feature works in dev, breaks in production
Debug:
  - Dev uses SQLite, prod uses PostgreSQL
  - Query uses SQLite-specific syntax
  - PostgreSQL rejects the query
  - Fix: Use database-agnostic query or detect DB type
```

## When You're Stuck

### Take a Break
- Step away for 10-15 minutes
- Get coffee, walk around
- Your subconscious keeps working
- Fresh eyes often see the issue

### Simplify
- Create minimal reproduction
- Remove complexity
- Test in isolation
- Start from scratch if needed

### Ask for Help
**Before asking**:
- Document what you've tried
- Prepare a minimal example
- Explain expected vs actual behavior
- Share relevant code and error messages

**Good question format**:
```
I'm trying to [goal]
I expected [expected behavior]
Instead, [actual behavior] happens
I've tried [what you've tried]
Here's a minimal example: [code]
Error message: [full stack trace]
```

### Check the Basics
- Is the code actually running? (console.log to verify)
- Are you editing the right file?
- Did you save the file?
- Did you restart the server?
- Are you looking at the right environment?
- Is cache causing issues?

### Read Documentation
- Check official docs for the library/framework
- Look for breaking changes in release notes
- Search issue trackers for similar problems
- Review examples and tutorials

### Search Effectively
- Copy exact error message
- Include relevant technology names
- Use Stack Overflow, GitHub issues
- Check recent discussions (avoid outdated answers)

## Debugging Checklist

When debugging, systematically check:

- [ ] Can you reproduce the bug consistently?
- [ ] Do you understand the expected behavior?
- [ ] Have you read the error message completely?
- [ ] Have you checked the logs?
- [ ] Have you verified your assumptions?
- [ ] Have you tested in isolation?
- [ ] Have you checked recent changes (git diff)?
- [ ] Have you tried reverting recent changes?
- [ ] Have you tested with different inputs?
- [ ] Have you checked for typos?
- [ ] Have you verified environment configuration?
- [ ] Have you consulted documentation?
- [ ] Have you searched for similar issues?
- [ ] Have you taken a break if stuck for >30 minutes?

## Common Debugging Mistakes

❌ **Random changes**: Changing things without understanding
❌ **Multiple simultaneous changes**: Can't identify what fixed it
❌ **Ignoring error messages**: They often tell you exactly what's wrong
❌ **Not reproducing**: Fixing without consistent reproduction
❌ **Assuming**: "It must be X" without verification
❌ **Skipping the basics**: Not checking if code is even running
❌ **Fixation**: Stuck on one hypothesis, ignoring alternatives
❌ **No documentation**: Forgetting what you tried

## Prevention is Better Than Cure

- **Write tests**: Catch bugs before they reach production
- **Use linters**: Catch common mistakes automatically
- **Code reviews**: Fresh eyes catch issues
- **Type systems**: Prevent type-related bugs
- **Logging**: Add logging as you write code
- **Error handling**: Fail gracefully with clear messages
- **Monitoring**: Detect issues in production quickly

---

**Remember**: Debugging is a skill that improves with practice. Stay systematic, stay patient, and learn from each bug you fix. The most complex bugs often have simple causes—keep an open mind.
