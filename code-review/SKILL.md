---
name: code-review
description: Perform thorough code reviews focusing on correctness, maintainability, performance, and best practices. Apply when reviewing pull requests, auditing code quality, or mentoring developers.
---

# Code Review Skill

Conduct effective code reviews that improve code quality, share knowledge, and maintain team standards while being constructive and respectful.

## Core Principles

### 1. **Review the Code, Not the Person**
- Focus on the code, not the author
- Use "we" language, not "you"
- Assume good intent
- Be constructive, not critical

### 2. **Prioritize What Matters**
- Correctness > Performance > Style
- Security issues are critical
- Focus on substance over nitpicks
- Consider the context and constraints

### 3. **Be Specific and Actionable**
- Explain why something is an issue
- Suggest concrete improvements
- Provide examples when helpful
- Link to documentation or resources

### 4. **Balance Thoroughness with Timeliness**
- Don't block progress unnecessarily
- Distinguish blocking vs non-blocking feedback
- Review promptly (within 24 hours)
- Break large PRs into smaller reviews

## Review Checklist

### 1. Correctness

**Does the code work correctly?**

- [ ] Logic is sound and handles all cases
- [ ] Edge cases are considered (null, empty, boundary values)
- [ ] Error conditions are handled appropriately
- [ ] Code does what the PR description says
- [ ] No obvious bugs or logical errors
- [ ] Concurrency issues addressed (race conditions, deadlocks)

**Questions to ask**:
```
- What happens if this input is null/empty/negative?
- What happens if this operation fails?
- Are there any race conditions?
- Does this handle the edge cases mentioned in requirements?
```

### 2. Security

**Is the code secure?**

- [ ] No SQL injection vulnerabilities
- [ ] No XSS vulnerabilities
- [ ] Input is validated and sanitized
- [ ] Authentication/authorization properly enforced
- [ ] Sensitive data not logged or exposed
- [ ] Dependencies don't have known vulnerabilities
- [ ] Secrets not hardcoded

**Red flags**:
```
- String concatenation for SQL queries
- User input directly in HTML output
- Missing authorization checks
- Hardcoded credentials or API keys
- Overly permissive CORS settings
- eval() or dynamic code execution with user input
```

### 3. Performance

**Is the code efficient?**

- [ ] No N+1 query problems
- [ ] Appropriate use of indexes
- [ ] No unnecessary database calls
- [ ] Efficient algorithms and data structures
- [ ] Large operations are paginated or batched
- [ ] Caching used where appropriate
- [ ] No memory leaks

**Questions to ask**:
```
- How does this perform with 10x, 100x data?
- Are there any database queries in loops?
- Is this doing work that could be cached?
- Are we loading more data than needed?
```

### 4. Maintainability

**Is the code easy to understand and modify?**

- [ ] Code is readable and well-organized
- [ ] Functions/methods have single responsibility
- [ ] No excessive complexity (deep nesting, long functions)
- [ ] Naming is clear and consistent
- [ ] Magic numbers/strings are constants
- [ ] No unnecessary duplication
- [ ] Dependencies are appropriate

**Complexity indicators**:
```
- Functions longer than 30-50 lines
- More than 3 levels of nesting
- Too many parameters (> 4-5)
- Complex conditionals
- Tight coupling between components
```

### 5. Testing

**Is the code properly tested?**

- [ ] Unit tests cover critical logic
- [ ] Edge cases and error conditions tested
- [ ] Tests are clear and maintainable
- [ ] Test names describe what's being tested
- [ ] No testing of implementation details
- [ ] Integration tests for component interactions
- [ ] Mocks/stubs used appropriately

**Test quality checks**:
```
- Do tests actually assert something meaningful?
- Are tests independent of each other?
- Do test names explain the scenario?
- Are edge cases covered?
```

### 6. Documentation

**Is the code properly documented?**

- [ ] Public APIs have documentation
- [ ] Complex logic is explained
- [ ] Non-obvious decisions have comments
- [ ] README updated if needed
- [ ] API documentation updated
- [ ] No outdated comments
- [ ] Changelog updated for user-facing changes

**Documentation guidelines**:
```
Good: Explains "why" not "what"
Bad: Comments that just repeat the code

Good: // Using exponential backoff to avoid overwhelming the server
Bad: // Increment retry count
```

### 7. Design & Architecture

**Does the code fit well in the codebase?**

- [ ] Follows existing patterns and conventions
- [ ] Appropriate abstraction level
- [ ] Components have clear responsibilities
- [ ] Dependencies flow in the right direction
- [ ] Interfaces are well-designed
- [ ] Changes are backward compatible

**Questions to ask**:
```
- Does this belong in this module/service?
- Is this the right level of abstraction?
- Are we following our established patterns?
- Will this be easy to extend in the future?
```

### 8. Error Handling

**Are errors handled appropriately?**

- [ ] Errors are caught and handled
- [ ] Error messages are helpful
- [ ] Failures don't crash the system
- [ ] Resources are cleaned up on error
- [ ] Errors are logged appropriately
- [ ] User-facing errors are friendly

**Error handling patterns**:
```
Good:
- Specific error types for different failures
- Contextual error messages
- Proper cleanup in finally blocks
- Graceful degradation

Bad:
- Catching and ignoring errors
- Generic error messages
- Resource leaks on failure
- Crashing on recoverable errors
```

## Review Process

### Before Starting

1. **Understand the context**
   - Read the PR description and linked issues
   - Understand the problem being solved
   - Check if there are special considerations

2. **Get the big picture**
   - Review the file list to understand scope
   - Identify the main changes
   - Note areas requiring careful review

### During Review

1. **First pass: Architecture and design**
   - Does the overall approach make sense?
   - Are there fundamental issues?
   - Is the scope appropriate?

2. **Second pass: Implementation details**
   - Check each file thoroughly
   - Look for bugs and issues
   - Note improvements

3. **Third pass: Polish**
   - Style and naming consistency
   - Documentation completeness
   - Test coverage

### Providing Feedback

**Be Clear About Severity**:
```
🔴 Blocking: Must be fixed before merge
   "This has a SQL injection vulnerability that must be fixed."

🟡 Should Fix: Important but could be follow-up
   "This function is getting complex. Consider refactoring,
   but could be a separate PR."

🟢 Suggestion: Nice to have, optional
   "Consider using a more descriptive variable name here."

💭 Question: Seeking clarification
   "I don't understand the reasoning here. Could you explain?"

📝 Nitpick: Minor style preference
   "Nit: Extra blank line here."
```

**Provide Constructive Feedback**:

```
❌ Bad: "This is wrong."

✅ Good: "This will throw a null pointer exception if user
is not found. Consider adding a null check:
if (user != null) { ... }"

❌ Bad: "Why did you do it this way?"

✅ Good: "I'm curious about the choice to use polling here
instead of websockets. What were the trade-offs you considered?"

❌ Bad: "This is inefficient."

✅ Good: "This query inside the loop will cause N+1 queries.
Consider fetching all users upfront:
const users = await getUsers(ids)
Then look them up from the map."
```

### After Review

1. **Summarize your review**
   - Overall assessment
   - Key points to address
   - Approval status

2. **Follow up**
   - Respond to questions promptly
   - Re-review fixes
   - Don't block unnecessarily

## Common Issues to Look For

### Logic Errors

```javascript
// Off-by-one error
for (let i = 0; i <= array.length; i++) {  // Should be <
  console.log(array[i]);
}

// Wrong boolean logic
if (!user.isActive && !user.isAdmin) {  // Should be ||?
  denyAccess();
}

// Missing break in switch
switch (type) {
  case 'A':
    handleA();
    // Missing break - falls through!
  case 'B':
    handleB();
    break;
}

// Integer division issues
const percentage = count / total * 100;  // May need float division

// Null/undefined issues
const name = user.profile.name;  // What if profile is null?
```

### Security Issues

```javascript
// SQL Injection
const query = `SELECT * FROM users WHERE id = ${userId}`;

// XSS
element.innerHTML = userInput;

// Missing authorization
app.get('/admin/users', (req, res) => {
  // No check if user is admin!
  return db.users.findAll();
});

// Hardcoded secrets
const apiKey = 'sk_live_abc123';

// Sensitive data exposure
logger.info('User login', { user: user, password: password });
```

### Performance Issues

```javascript
// N+1 queries
const posts = await Post.findAll();
for (const post of posts) {
  post.author = await User.findById(post.authorId);  // Query per post!
}

// Unnecessary work in loops
for (const item of items) {
  const config = JSON.parse(fs.readFileSync('config.json'));  // Read once outside!
  process(item, config);
}

// Loading too much data
const users = await db.users.findAll();  // Loading millions of users?
return users.slice(0, 10);  // Just to show 10?

// Missing index usage
SELECT * FROM logs WHERE message LIKE '%error%';  // Can't use index
```

### Maintainability Issues

```javascript
// Magic numbers
if (retries > 3) {  // What does 3 mean?
  throw new Error();
}
// Better:
const MAX_RETRIES = 3;
if (retries > MAX_RETRIES) { ... }

// Deep nesting
if (user) {
  if (user.isActive) {
    if (user.hasPermission) {
      if (resource.isAvailable) {
        // Actual logic buried here
      }
    }
  }
}
// Better: Use early returns

// Long functions
function processOrder(order) {
  // 200 lines of code...
}
// Better: Break into smaller, focused functions

// Duplicated logic
// Same validation in 3 different files
// Better: Extract to shared utility
```

### Error Handling Issues

```javascript
// Swallowing errors
try {
  await riskyOperation();
} catch (e) {
  // Silently ignored!
}

// Generic error messages
throw new Error('An error occurred');
// Better: throw new Error(`Failed to fetch user ${userId}: ${e.message}`);

// Missing cleanup
const file = await openFile(path);
await processFile(file);  // If this throws, file is never closed!
// Better: Use try/finally or RAII pattern

// Wrong error type
if (!user) {
  throw new Error('User not found');
}
// Better: throw new NotFoundError('User', userId);
```

## Review Etiquette

### For Reviewers

**Do**:
- Be respectful and constructive
- Explain the "why" behind feedback
- Acknowledge good solutions
- Ask questions instead of making accusations
- Offer to discuss complex issues
- Approve when issues are minor

**Don't**:
- Be condescending or sarcastic
- Focus only on negatives
- Rewrite code in your style
- Delay reviews unnecessarily
- Require unnecessary perfection
- Make it personal

### For Authors

**Do**:
- Provide context in PR description
- Self-review before requesting review
- Keep PRs small and focused
- Respond to feedback constructively
- Ask for clarification if needed
- Thank reviewers for their time

**Don't**:
- Take feedback personally
- Get defensive
- Submit massive PRs
- Ignore reviewer comments
- Merge without addressing blocking issues

## Review Templates

### PR Description Template

```markdown
## Summary
Brief description of what this PR does

## Problem
What problem does this solve? Link to issue.

## Solution
How does this PR solve the problem?

## Testing
How was this tested? What should reviewers test?

## Screenshots (if applicable)
Before/after screenshots for UI changes

## Checklist
- [ ] Tests added/updated
- [ ] Documentation updated
- [ ] No breaking changes (or documented)
- [ ] Self-reviewed
```

### Review Summary Template

```markdown
## Overall
[Approve / Request Changes / Comment]

Brief summary of the review.

## What I Liked
- Good things about the PR

## Must Fix (Blocking)
- Critical issues that must be addressed

## Should Fix
- Important issues, could be follow-up

## Suggestions
- Optional improvements

## Questions
- Things I'd like clarified
```

## Review Metrics

### Healthy Review Practices

- **Review time**: < 24 hours for initial review
- **PR size**: < 400 lines of code changed
- **Review cycles**: < 3 rounds typically
- **Coverage**: All PRs reviewed before merge

### Signs of Issues

- Reviews taking days
- Heated discussions in comments
- Same issues repeatedly
- PRs merged without review
- Large PRs that are hard to review

## Automation Support

### Automate What You Can

- **Linters**: Code style enforcement
- **Formatters**: Consistent formatting
- **Type checkers**: Type errors
- **Security scanners**: Known vulnerabilities
- **Test coverage**: Coverage requirements
- **PR checks**: Size limits, required sections

### Focus Human Review On

- Logic and correctness
- Design decisions
- Security implications
- Performance considerations
- Knowledge sharing
- Mentorship

---

**Remember**: Code review is about improving code quality and sharing knowledge. Be thorough but respectful, specific but constructive. A good review helps the author grow and makes the codebase better.
