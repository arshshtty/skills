---
name: code-reviewer
description: Performs systematic code review analyzing security, performance, maintainability, and best practices. Apply after implementing features, before commits, or when reviewing pull requests.
---

# Code Reviewer Skill

Perform thorough, systematic code reviews that catch bugs, security vulnerabilities, performance issues, and maintainability concerns before they reach production.

## When to Use

✅ **After implementing a feature**: Review your code before committing
✅ **Before creating a PR**: Ensure quality before requesting human review
✅ **During PR review**: Supplement human review with systematic analysis
✅ **When debugging**: Identify potential issues in problematic code
✅ **After refactoring**: Verify changes maintain quality standards
✅ **Legacy code analysis**: Assess quality of inherited code

⚠️ **Don't use for**: Trivial changes like typo fixes or simple config updates

## Review Categories

### 1. Security Vulnerabilities

**Critical Issues**:
- SQL injection vulnerabilities
- XSS (Cross-Site Scripting) attack vectors
- CSRF token absence
- Authentication/authorization bypasses
- Insecure data exposure
- Hardcoded secrets or credentials
- Insecure cryptography usage
- Path traversal vulnerabilities
- Command injection risks

**Medium Issues**:
- Missing input validation
- Insufficient rate limiting
- Weak password policies
- Insecure session management
- Missing security headers
- Overly permissive CORS policies

### 2. Performance Issues

**Algorithmic**:
- O(n²) or worse algorithms where better exists
- Unnecessary nested loops
- Redundant computations
- Missing memoization/caching opportunities

**Database**:
- N+1 query problems
- Missing indexes
- Inefficient query patterns
- Unbounded result sets
- Missing pagination

**Resource Management**:
- Memory leaks (event listeners, closures, cache)
- File handle leaks
- Database connection leaks
- Unnecessary large object allocations

**Frontend**:
- Unnecessary re-renders
- Missing virtualization for long lists
- Unoptimized images
- Bundle size issues
- Missing code splitting

### 3. Bugs and Logic Errors

**Common Patterns**:
- Off-by-one errors
- Race conditions
- Null/undefined reference errors
- Type coercion issues
- Incorrect boundary conditions
- Missing error handling
- Unhandled promise rejections
- Infinite loops
- Integer overflow/underflow
- Timezone and date handling bugs

**Async Issues**:
- Missing await keywords
- Uncaught promise rejections
- Race conditions in async code
- Callback hell
- Missing error handling in async operations

### 4. Code Quality & Maintainability

**Structure**:
- Functions/methods too long (>50 lines)
- Too many parameters (>4)
- Excessive nesting depth (>3 levels)
- God objects/classes
- Tight coupling
- Low cohesion

**Naming**:
- Unclear variable/function names
- Inconsistent naming conventions
- Misleading names
- Magic numbers/strings without constants

**Best Practices**:
- Missing error handling
- Poor error messages
- Inadequate logging
- Missing input validation
- Hardcoded configuration
- Commented-out code
- TODO comments without tickets

**Testing**:
- Missing test coverage for critical paths
- Tests that don't test behavior
- Flaky tests
- Missing edge case coverage

### 5. Language/Framework-Specific

**JavaScript/TypeScript**:
- Using `var` instead of `const`/`let`
- Missing TypeScript types (`any` usage)
- Not using optional chaining
- Unnecessary type assertions
- Missing strict mode

**Python**:
- Mutable default arguments
- Incorrect `__eq__` without `__hash__`
- Using `==` instead of `is` for None
- Missing context managers for resources
- Not following PEP 8

**React**:
- Missing dependency arrays in hooks
- Unnecessary useEffect calls
- Missing key props in lists
- Inline function definitions in JSX
- State mutation

**Node.js**:
- Blocking the event loop
- Missing error handling in callbacks
- Unsafe `eval()` usage
- Missing input sanitization

## Review Process

### Step 1: Initial Assessment

Quickly scan the code to understand:
- What is the purpose of this code?
- What are the main components/functions?
- What are the inputs/outputs?
- What are the side effects?

### Step 2: Systematic Review

Go through each category:

1. **Security First**: Look for vulnerabilities
2. **Correctness**: Verify logic is sound
3. **Performance**: Identify inefficiencies
4. **Maintainability**: Assess code quality
5. **Tests**: Verify adequate coverage

### Step 3: Prioritize Findings

Classify issues by severity:

**🔴 Critical**: Must fix before merge
- Security vulnerabilities
- Data loss bugs
- Crash-inducing errors
- Performance killers (O(n²) on user input)

**🟡 Medium**: Should fix soon
- Minor bugs
- Moderate performance issues
- Maintainability concerns
- Missing error handling

**🟢 Low**: Nice to have
- Style inconsistencies
- Minor optimizations
- Documentation improvements

### Step 4: Provide Actionable Feedback

For each issue:
- **What**: Clearly describe the problem
- **Why**: Explain the impact/risk
- **Where**: Point to specific line numbers
- **How**: Suggest a concrete fix

## Output Format

Structure review feedback as:

```markdown
## Code Review Summary

**Files Reviewed**: [list files]
**Overall Assessment**: [Brief summary]

---

## 🔴 Critical Issues

### 1. SQL Injection Vulnerability
**File**: `src/api/users.ts:45`
**Issue**: User input directly interpolated into SQL query
**Impact**: Attackers can execute arbitrary SQL commands
**Fix**: Use parameterized queries
```typescript
// Before
db.query(`SELECT * FROM users WHERE id = ${userId}`)

// After
db.query('SELECT * FROM users WHERE id = ?', [userId])
```

---

## 🟡 Medium Issues

### 1. N+1 Query Problem
**File**: `src/services/orders.ts:23-30`
**Issue**: Loading user data in a loop creates N+1 queries
**Impact**: Scales poorly, slow for large datasets
**Fix**: Use eager loading or batch query
```typescript
// Use JOIN or batch query to load all users at once
const users = await db.users.findMany({
  where: { id: { in: userIds } }
})
```

---

## 🟢 Suggestions

### 1. Extract Magic Number
**File**: `src/utils/validator.ts:12`
**Suggestion**: Extract magic number 256 to named constant
```typescript
const MAX_USERNAME_LENGTH = 256
```

---

## Positive Highlights

- Excellent error handling in authentication flow
- Good use of TypeScript types throughout
- Clear separation of concerns
```

## Best Practices for Reviewers

### Do:
✅ Focus on substantial issues, not nitpicks
✅ Provide context and reasoning
✅ Suggest specific improvements with code examples
✅ Acknowledge good patterns when you see them
✅ Consider the broader system architecture
✅ Think about edge cases and failure modes
✅ Verify error handling paths
✅ Check for security implications

### Don't:
❌ Just point out problems without explaining why
❌ Be overly pedantic about style (use linters for that)
❌ Suggest rewrites without strong justification
❌ Ignore the context and constraints
❌ Review more than ~500 lines at once (break it up)
❌ Rush - take time to understand the code

## Common Pitfalls to Watch For

### Security Red Flags
```javascript
// ❌ Direct user input in queries
db.query(`SELECT * FROM users WHERE name = '${req.body.name}'`)

// ❌ Missing authentication checks
app.delete('/api/users/:id', deleteUser)

// ❌ Hardcoded secrets
const API_KEY = 'sk_live_abc123'

// ❌ Eval on user input
eval(req.body.code)
```

### Performance Red Flags
```javascript
// ❌ N+1 queries
for (const order of orders) {
  order.user = await db.users.findById(order.userId)
}

// ❌ O(n²) when O(n) possible
for (const item of items) {
  if (selected.includes(item)) { ... }  // includes is O(n)
}

// ❌ Missing pagination
const allUsers = await db.users.findAll()  // Could be millions
```

### Bug Red Flags
```javascript
// ❌ Missing await
async function getUser() {
  const user = fetchUser()  // Returns Promise, not user!
  return user.name  // undefined.name → error
}

// ❌ Race condition
let counter = 0
async function increment() {
  const current = counter
  await delay(100)
  counter = current + 1  // Lost updates!
}

// ❌ Mutating state
function addItem(state, item) {
  state.items.push(item)  // Mutates!
  return state
}
```

## Integration with Workflow

### Pre-Commit Review
```bash
# Review staged changes before committing
git diff --cached | # use code-reviewer skill
```

### PR Review
```bash
# Review changes in a PR
gh pr diff 123 | # use code-reviewer skill
```

### File Review
Simply provide the file contents or git diff to review.

## Handling Review Feedback

### As Code Author

When receiving review feedback:
1. **Don't take it personally** - reviews improve code quality
2. **Ask questions** if feedback is unclear
3. **Push back** if you disagree, but with reasoning
4. **Fix critical issues** immediately
5. **Plan medium issues** for this or next PR
6. **Consider suggestions** but they're optional

### As Reviewer

When giving feedback:
1. **Be respectful** and constructive
2. **Distinguish** must-fix from suggestions
3. **Explain reasoning** - help them learn
4. **Acknowledge constraints** - perfection isn't always possible
5. **Highlight good work** - positive reinforcement matters

## Review Checklist

Before marking review complete, verify:

- [ ] No critical security vulnerabilities
- [ ] No obvious bugs or logic errors
- [ ] Error handling is present and appropriate
- [ ] Performance is acceptable for expected scale
- [ ] Code is reasonably maintainable
- [ ] Tests cover critical paths
- [ ] No sensitive data is logged or exposed
- [ ] Database queries are efficient
- [ ] API contracts are backward compatible (if applicable)
- [ ] Configuration is externalized (not hardcoded)

## Language-Specific Checklists

### JavaScript/TypeScript
- [ ] No `any` types without justification
- [ ] Async functions properly handle errors
- [ ] React hooks have correct dependencies
- [ ] No blocking operations on event loop
- [ ] Proper TypeScript strict mode compliance

### Python
- [ ] No mutable default arguments
- [ ] Context managers used for resources
- [ ] Exceptions are specific, not bare `except:`
- [ ] Type hints present (Python 3.5+)
- [ ] Following PEP 8 style guide

### Go
- [ ] Errors are checked, not ignored
- [ ] Defer used for cleanup
- [ ] Goroutines don't leak
- [ ] Contexts passed for cancellation
- [ ] No data races (use -race flag)

### Rust
- [ ] No unsafe without justification
- [ ] Proper error handling with Result
- [ ] No unwrap() in production code
- [ ] Lifetime annotations are correct
- [ ] No memory leaks from Rc cycles

## Example Review Scenarios

### Scenario 1: Authentication Endpoint

**Code**:
```typescript
app.post('/login', async (req, res) => {
  const user = await db.query(
    `SELECT * FROM users WHERE email = '${req.body.email}'`
  )
  if (user && user.password === req.body.password) {
    res.json({ token: user.id })
  }
  res.status(401).send('Invalid')
})
```

**Review**:
- 🔴 **SQL Injection**: Use parameterized queries
- 🔴 **Plaintext Password**: Should use bcrypt/argon2
- 🔴 **Weak Token**: User ID is not a secure token, use JWT
- 🟡 **Missing Rate Limiting**: Brute force vulnerability
- 🟡 **Error Handling**: Doesn't handle DB errors

### Scenario 2: Data Processing

**Code**:
```python
def process_orders(order_ids):
    results = []
    for order_id in order_ids:
        order = db.get_order(order_id)  # N+1 query
        customer = db.get_customer(order.customer_id)  # N+1 query
        results.append({
            'order': order,
            'customer': customer
        })
    return results
```

**Review**:
- 🔴 **N+1 Queries**: Load all orders and customers in batch
- 🟡 **Missing Error Handling**: What if order or customer not found?
- 🟡 **No Input Validation**: Check order_ids is valid list
- 🟢 **Consider**: Return generator for large datasets

---

**Remember**: The goal of code review is to **improve quality, catch bugs, and share knowledge** - not to assert dominance or nitpick. Be thorough but kind, critical but constructive.
