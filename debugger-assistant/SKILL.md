---
name: debugger-assistant
description: Analyzes errors, stack traces, and unexpected behavior to identify root causes and suggest debugging strategies. Apply when facing bugs, crashes, or mysterious issues.
---

# Debugger Assistant Skill

Systematically analyze bugs, interpret stack traces, identify root causes, and guide you through effective debugging strategies to resolve issues quickly.

## When to Use

✅ **Analyzing error messages**: Understand cryptic errors
✅ **Stack trace analysis**: Trace error origin through call stack
✅ **Unexpected behavior**: Figure out why code isn't working as expected
✅ **Crashes and exceptions**: Diagnose crash causes
✅ **Performance issues**: Identify bottlenecks and slowdowns
✅ **Intermittent bugs**: Debug flaky, non-deterministic issues
✅ **Production incidents**: Post-mortem analysis

## Debugging Process

### 1. Reproduce the Bug

**Before you can fix it, you must be able to trigger it reliably**

Steps:
1. **Document exact steps** to reproduce
2. **Identify minimal conditions** that trigger the bug
3. **Note environment details**: OS, browser, versions, data state
4. **Check consistency**: Does it always happen? Sometimes? Under what conditions?

**Reproduction Quality**:
- ✅ **Reliable**: Happens 100% of the time with same steps
- ⚠️ **Intermittent**: Happens sometimes (harder to debug)
- ❌ **Unreproducible**: Can't trigger it (very hard to fix)

For intermittent bugs:
- Look for race conditions
- Check time-dependent logic
- Examine randomness or external data
- Review concurrent operations

### 2. Gather Information

**Collect all relevant data before diving into code**

#### Error Messages
- Full error text
- Error type/class
- Error code (if any)

#### Stack Trace
- Full stack trace (don't truncate!)
- Line numbers and file names
- Call sequence

#### Context
- What was the user trying to do?
- What input data was provided?
- What's the application state?
- Recent code changes

#### Environment
- OS and version
- Browser/Node version
- Framework versions
- Environment variables
- Database state

### 3. Analyze the Stack Trace

**The stack trace is your map - learn to read it**

#### Stack Trace Anatomy

```
Error: User not found
    at UserService.getUser (/app/services/user.service.ts:45:11)
    at UserController.show (/app/controllers/user.controller.ts:23:28)
    at Layer.handle (/app/node_modules/express/lib/router/layer.js:95:5)
    at next (/app/node_modules/express/lib/router/route.js:137:13)
    ...
```

**How to read it**:
1. **Error message**: "User not found" - what went wrong
2. **Origin**: First line (UserService.getUser:45) - where it was thrown
3. **Call path**: Follow upward to see how we got there
4. **Your code vs library code**: Focus on your files first

**Reading strategy**:
1. Start at the **top** (where error was thrown)
2. Identify the **first file in YOUR codebase** (ignore library files)
3. Look at that **line number** - what's happening there?
4. Trace **backwards through your code** to see the call chain
5. Identify the **earliest point where things went wrong**

### 4. Form a Hypothesis

**Based on evidence, what do you think is wrong?**

Common hypotheses:
- "Variable X is null/undefined when it shouldn't be"
- "Function Y is called with wrong arguments"
- "Async operation Z isn't being awaited"
- "Race condition between operations A and B"
- "Data format changed but code wasn't updated"

**Make specific, testable predictions**:
- ❌ Bad: "Something's wrong with the database"
- ✅ Good: "User ID is null because session middleware isn't running first"

### 5. Test Your Hypothesis

**Verify your theory with evidence**

Methods:
- **Add logging**: Print values at key points
- **Use debugger**: Set breakpoints and inspect
- **Add assertions**: Validate assumptions
- **Simplify**: Remove code until bug disappears
- **Isolate**: Create minimal reproduction

### 6. Fix and Verify

1. **Implement fix** based on confirmed hypothesis
2. **Test the fix** with original reproduction steps
3. **Test edge cases** to ensure fix is complete
4. **Add regression test** to prevent bug from returning
5. **Document** the bug and fix if non-obvious

## Common Bug Patterns

### 1. Null/Undefined Reference Errors

**Symptoms**:
```
TypeError: Cannot read property 'name' of undefined
TypeError: Cannot read properties of null (reading 'id')
```

**Causes**:
- Object doesn't exist (null/undefined)
- API returned different shape than expected
- Async data not loaded yet
- Optional chaining missing

**Debug strategy**:
```typescript
// Add defensive logging
console.log('User object:', user)
console.log('User type:', typeof user)
console.log('User keys:', user ? Object.keys(user) : 'null/undefined')

// Check assumptions
if (!user) {
  console.error('User is null/undefined at line X')
  console.trace() // Print stack trace
}
```

**Common fixes**:
```typescript
// Use optional chaining
const name = user?.profile?.name

// Provide defaults
const name = user?.profile?.name ?? 'Unknown'

// Guard clauses
if (!user) {
  throw new Error('User is required')
}

// Type checking
if (typeof user === 'object' && user !== null) {
  // Safe to use
}
```

### 2. Async/Await Issues

**Symptoms**:
- "Promise {<pending>}" instead of value
- Race conditions
- Unhandled promise rejections
- Code runs in wrong order

**Missing await**:
```typescript
// Bug - missing await
async function getUser() {
  const user = fetchUser(1) // Returns Promise!
  console.log(user.name) // undefined.name → error
}

// Fix
async function getUser() {
  const user = await fetchUser(1)
  console.log(user.name) // Works
}
```

**Forgetting async**:
```typescript
// Bug - not async
function getUser() {
  const user = await fetchUser(1) // SyntaxError
}

// Fix
async function getUser() {
  const user = await fetchUser(1)
}
```

**Unhandled rejections**:
```typescript
// Bug - no error handling
async function getUser() {
  const user = await fetchUser(1) // Throws, crashes app
}

// Fix
async function getUser() {
  try {
    const user = await fetchUser(1)
    return user
  } catch (error) {
    console.error('Failed to fetch user:', error)
    throw error // or handle appropriately
  }
}
```

### 3. Race Conditions

**Symptoms**:
- Bug happens sometimes, not always
- Different results on repeated runs
- Works in dev, fails in production
- Issues with concurrent operations

**Example**:
```typescript
// Bug - race condition
let counter = 0

async function increment() {
  const current = counter // Read
  await delay(100)
  counter = current + 1 // Write (lost update!)
}

// Called concurrently
await Promise.all([increment(), increment(), increment()])
console.log(counter) // 1 instead of 3!
```

**Debug strategy**:
1. **Add timing logs**:
```typescript
async function increment() {
  console.log('Start increment', Date.now())
  const current = counter
  console.log('Read counter:', current, Date.now())
  await delay(100)
  counter = current + 1
  console.log('Wrote counter:', counter, Date.now())
}
```

2. **Look for shared state** accessed by multiple async operations
3. **Check for awaits** between read and write
4. **Use locks or atomic operations**

**Fixes**:
```typescript
// Use atomic operations
counter++

// Use locks (mutex)
await lock.acquire()
try {
  counter++
} finally {
  lock.release()
}

// Use queue
const queue = new Queue()
await queue.add(() => counter++)
```

### 4. Off-by-One Errors

**Symptoms**:
- Array index out of bounds
- Loop runs too many/few times
- Fencepost errors

**Common mistakes**:
```typescript
// Bug - should be < not <=
for (let i = 0; i <= arr.length; i++) {
  console.log(arr[i]) // undefined on last iteration
}

// Bug - should be length - 1
const last = arr[arr.length] // undefined

// Bug - wrong slice boundary
const chunk = arr.slice(0, 5) // Gets 5 items (0-4), not 6
```

**Debug strategy**:
- Print loop indices
- Check boundary conditions (0, length, length-1)
- Test with small arrays [1], [1, 2], [1, 2, 3]

### 5. Type Coercion Issues

**JavaScript's implicit conversions cause bugs**:

```typescript
// Unexpected string concatenation
const total = 1 + 2 + '3' // "33" not "123"

// Truthy/falsy surprises
if (user.age) { } // Fails when age is 0!
if (items.length) { } // Fails for empty array (length 0)

// Equality confusion
0 == '0' // true (type coercion)
0 === '0' // false (strict equality)
null == undefined // true
null === undefined // false

// NaN comparisons
NaN === NaN // false (!)
isNaN(NaN) // true
Number.isNaN(NaN) // true (safer)
```

**Debug strategy**:
```typescript
// Log types
console.log('Value:', value, 'Type:', typeof value)

// Use strict equality
if (value === 0) { } // Explicit check

// Validate types
if (typeof age === 'number' && !isNaN(age)) { }
```

### 6. Memory Leaks

**Symptoms**:
- Memory usage grows over time
- Eventual crashes or slowdowns
- Performance degradation

**Common causes**:

**Event listeners not removed**:
```typescript
// Bug - listener never removed
element.addEventListener('click', handler)
// Element removed from DOM, but listener still in memory

// Fix
element.addEventListener('click', handler)
// Later:
element.removeEventListener('click', handler)
```

**Closures retaining references**:
```typescript
// Bug - closure keeps large object in memory
function createHandler(data) { // data is huge
  return () => {
    console.log(data.id) // Only need id, but whole object retained
  }
}

// Fix
function createHandler(data) {
  const id = data.id // Extract only what's needed
  return () => {
    console.log(id) // Only id is retained
  }
}
```

**Cache without eviction**:
```typescript
// Bug - cache grows forever
const cache = new Map()

function getUser(id) {
  if (!cache.has(id)) {
    cache.set(id, fetchUser(id))
  }
  return cache.get(id)
}

// Fix - add size limit and LRU eviction
const cache = new LRUCache({ max: 1000 })
```

**Debug strategy**:
1. **Heap snapshots**: Take before/after snapshots, compare
2. **Monitor memory**: Track process.memoryUsage()
3. **Look for unbounded growth**: Arrays, Maps, Sets, caches
4. **Check event listeners**: Use Chrome DevTools → Memory → Event Listeners
5. **Profile over time**: Does memory keep growing or stabilize?

### 7. Scope and Closure Issues

**Symptoms**:
- Variable has unexpected value
- "Variable is not defined"
- Counter doesn't increment properly

**Example**:
```typescript
// Bug - closure in loop
for (var i = 0; i < 5; i++) {
  setTimeout(() => {
    console.log(i) // Prints 5, 5, 5, 5, 5
  }, 100)
}

// Fix 1 - use let (block scope)
for (let i = 0; i < 5; i++) {
  setTimeout(() => {
    console.log(i) // Prints 0, 1, 2, 3, 4
  }, 100)
}

// Fix 2 - use IIFE to capture value
for (var i = 0; i < 5; i++) {
  (function(i) {
    setTimeout(() => {
      console.log(i) // Prints 0, 1, 2, 3, 4
    }, 100)
  })(i)
}
```

## Debugging Tools and Techniques

### Console Logging

**Strategic logging**:
```typescript
// Log function entry
function processOrder(order) {
  console.log('processOrder called with:', order)

  // Log intermediate values
  const total = calculateTotal(order.items)
  console.log('Calculated total:', total)

  // Log branches
  if (total > 100) {
    console.log('Applying discount')
  }

  // Log exit
  console.log('processOrder returning:', result)
  return result
}
```

**Advanced logging**:
```typescript
// Conditional logging
const DEBUG = process.env.NODE_ENV === 'development'
if (DEBUG) console.log('Debug info:', data)

// Trace call stack
console.trace('How did we get here?')

// Time operations
console.time('operation')
doExpensiveOperation()
console.timeEnd('operation') // Prints elapsed time

// Table formatting
console.table(users)

// Group related logs
console.group('User processing')
console.log('User:', user)
console.log('Permissions:', permissions)
console.groupEnd()
```

### Using Debugger

**Breakpoints**:
```typescript
function buggyFunction(x) {
  debugger // Execution pauses here
  const result = x * 2
  return result
}
```

**Debugging workflow**:
1. Set breakpoint in IDE or add `debugger` statement
2. Run in debug mode
3. When execution pauses:
   - Inspect variables
   - Evaluate expressions in console
   - Step through code line by line
4. Use controls:
   - **Step Over**: Execute current line, move to next
   - **Step Into**: Enter function call
   - **Step Out**: Exit current function
   - **Continue**: Run until next breakpoint

**Conditional breakpoints**:
```typescript
// In IDE: Set breakpoint, add condition
// Example: only break when user.id === 123
```

### Binary Search Debugging

**When you don't know where the bug is**:

1. **Find working and broken points**
   - Working: Last known good state
   - Broken: Current state with bug

2. **Test midpoint**
   - Comment out half the code
   - Does bug still occur?

3. **Narrow down**
   - If yes: Bug in first half
   - If no: Bug in second half

4. **Repeat** until you find the problematic line

**Example**:
```typescript
// Bug somewhere in this function
function process(data) {
  const cleaned = cleanData(data)     // Line 1
  const validated = validate(cleaned) // Line 2
  const transformed = transform(validated) // Line 3
  const saved = save(transformed)     // Line 4
  return saved                        // Line 5
}

// Test: Comment out lines 3-5, does bug occur?
// - Yes: Bug in lines 1-2
// - No: Bug in lines 3-5
// Repeat until found
```

### Rubber Duck Debugging

**Explain the problem to an inanimate object (or person)**

Why it works:
- Forces you to articulate the issue clearly
- Often reveals assumptions you're making
- Helps you see the problem from another angle

Process:
1. Explain what the code *should* do
2. Explain what it *actually* does
3. Walk through the code line by line
4. Often you'll spot the bug while explaining

### Minimal Reproduction

**Simplify until you have the smallest possible bug**

Steps:
1. **Remove unrelated code**: Delete anything not required for bug
2. **Use hardcoded data**: Replace API calls with static data
3. **Isolate component**: Extract to standalone file
4. **Remove dependencies**: Strip out libraries if possible

Benefits:
- Easier to understand
- Faster to test
- Often reveals the root cause
- Makes it easier to ask for help

## Error Message Interpretation

### Common Patterns

**"Cannot read property 'X' of undefined"**:
- Object is undefined
- Check why object doesn't exist
- Look at the previous operation that should have created it

**"X is not a function"**:
- Variable is not a function (check type)
- Function name misspelled
- Function not imported
- Wrong property accessed

**"Maximum call stack size exceeded"**:
- Infinite recursion
- Check base case in recursive function
- Look for function calling itself without exit condition

**"Unexpected token"**:
- Syntax error (missing bracket, comma, etc.)
- Check the line number in error
- Often the real error is on the previous line

**"Cannot find module 'X'"**:
- Module not installed
- Incorrect import path
- Typo in module name

**"CORS error"**:
- Cross-origin request blocked
- Backend needs to set CORS headers
- Check if API allows requests from your domain

**"404 Not Found"**:
- URL/endpoint incorrect
- Route not defined
- Check API documentation

**"401 Unauthorized"**:
- Missing authentication
- Invalid token
- Token expired

**"500 Internal Server Error"**:
- Backend error
- Check server logs
- Look at request payload

## Debugging Checklist

When stuck, go through this systematically:

### Input/Output
- [ ] Is the input what you expect? (Log it!)
- [ ] Is the output what you expect? (Log it!)
- [ ] Are there intermediate transformations? (Log them!)

### Assumptions
- [ ] What assumptions am I making?
- [ ] Are those assumptions valid?
- [ ] Test each assumption explicitly

### State
- [ ] What's the application state when bug occurs?
- [ ] Is state being mutated unexpectedly?
- [ ] Are multiple operations modifying same state?

### Timing
- [ ] Is this a race condition?
- [ ] Are async operations completing in expected order?
- [ ] Is timing-dependent code involved?

### Environment
- [ ] Does it work in different environment?
- [ ] Are environment variables set correctly?
- [ ] Are versions matching (Node, packages, etc.)?

### Recent Changes
- [ ] What changed recently?
- [ ] Can I rollback to working version?
- [ ] What's different between working and broken?

### External Dependencies
- [ ] Are API responses what I expect?
- [ ] Is database in correct state?
- [ ] Are third-party services working?

## Advanced Debugging Scenarios

### Production Debugging

**You can't use debugger in production, so:**

1. **Comprehensive logging**:
   - Log error details
   - Log context (user ID, request ID, etc.)
   - Use log levels (error, warn, info, debug)

2. **Error tracking** (Sentry, Rollbar):
   - Captures stack traces
   - Groups similar errors
   - Shows frequency and user impact

3. **Reproduce locally**:
   - Use production data (sanitized)
   - Match production environment
   - Check production logs for clues

4. **Feature flags**:
   - Roll back problematic feature
   - Test fix with small % of users
   - Gradually roll out

### Debugging Performance Issues

**Tools**:
- Browser DevTools Performance tab
- Chrome Lighthouse
- Node.js profiler
- `console.time()` / `console.timeEnd()`

**Process**:
1. **Measure**: Don't guess, profile!
2. **Identify bottleneck**: What's taking the most time?
3. **Optimize**: Focus on the slowest part
4. **Measure again**: Did it actually improve?

### Debugging Flaky Tests

**Flaky tests pass sometimes, fail sometimes**:

Common causes:
- Race conditions
- Timing assumptions
- Shared state between tests
- Reliance on external services
- Random data

Debug strategy:
1. **Run many times**: `npm test -- --repeat 100`
2. **Look for timing**: Add delays, does it change?
3. **Check test isolation**: Run single test, does it pass?
4. **Review setup/teardown**: Is state cleaned properly?

## Getting Help

When you need to ask someone:

**Provide**:
1. **Clear problem description**: What's wrong?
2. **Expected vs actual**: What should happen vs what does?
3. **Reproduction steps**: How to trigger the bug
4. **Code**: Minimal, reproducible example
5. **Error message**: Full text and stack trace
6. **What you've tried**: Show you've debugged
7. **Environment**: Versions, OS, etc.

**Example**:
```markdown
## Problem
User creation fails with "duplicate key" error even when email is unique.

## Expected
User should be created successfully with unique email.

## Actual
Getting: `E11000 duplicate key error collection: users index: email_1`

## Reproduction
1. Create user with email test@example.com
2. Delete that user from database
3. Try to create new user with same email
4. Error occurs

## Code
[Minimal code example]

## What I've tried
- Verified user is deleted from database
- Checked email is unique in DB
- Looked for email in other collections
- Restarted database

## Environment
- Node.js v18.0.0
- MongoDB v6.0
- Mongoose v7.0
```

---

**Remember**: Debugging is a skill. The more you practice systematic debugging, the faster you'll find bugs. Stay curious, be methodical, and don't give up!
