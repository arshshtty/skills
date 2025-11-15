---
name: refactor-wizard
description: Guides safe, incremental code refactoring with impact analysis and testing strategies. Apply when improving code quality, reducing complexity, or preparing for new features.
---

# Refactor Wizard Skill

Safely and systematically improve code structure, readability, and maintainability through disciplined refactoring practices.

## When to Refactor

### ✅ Good Times to Refactor

**Before adding features**:
- Current code structure makes new feature hard to add
- Technical debt is blocking progress
- Pattern emerges that should be abstracted

**After adding features**:
- Clean up experimental code that worked
- Remove duplication introduced
- Extract reusable patterns

**During code review**:
- Reviewer spots improvement opportunities
- Code doesn't meet team standards
- Complexity can be reduced

**When you touch code**:
- "Boy Scout Rule": Leave code better than you found it
- Fix minor issues while you're there
- Update patterns to current standards

**When code smells**:
- Duplication everywhere
- Functions too long
- Too complex to understand
- Hard to test

### ❌ Bad Times to Refactor

**Don't refactor**:
- ❌ Right before a deadline
- ❌ When you don't have tests
- ❌ When you don't understand the code
- ❌ Just because you prefer a different style
- ❌ While actively debugging a production issue
- ❌ Code that will be deleted soon
- ❌ Without business value (perfection for perfection's sake)

## The Prime Directive of Refactoring

> **Refactoring should NOT change observable behavior**

- ✅ Change internal structure
- ❌ Change external behavior
- ✅ Improve code quality
- ❌ Add new features
- ✅ Make code easier to change
- ❌ Fix bugs (that's a separate commit)

**Separate refactoring from feature work**: Different commits!

## Refactoring Safety Net

### 1. Have Tests FIRST

**Before refactoring**:
```bash
# Ensure tests pass
npm test

# Check coverage
npm test -- --coverage

# If coverage is low, add tests FIRST
```

**No tests?**:
1. Write characterization tests (test current behavior)
2. Even if behavior is wrong, test it
3. Once safe, refactor
4. Then fix bugs in separate commit

### 2. Make Small, Incremental Changes

**Bad approach**:
```
Massive refactoring commit:
- Renamed everything
- Changed architecture
- Updated dependencies
- Moved files around
- Fixed bugs
All at once! 🔥
```

**Good approach**:
```
Commit 1: Extract helper function
Commit 2: Rename variables for clarity
Commit 3: Split large function
Commit 4: Remove duplication
Commit 5: Update tests

Each commit: Tests pass ✅
```

### 3. Run Tests After Each Change

**Workflow**:
1. Make small change
2. Run tests
3. Tests pass? Commit
4. Tests fail? Fix or revert
5. Repeat

**Use watch mode**:
```bash
npm test -- --watch
# Tests run automatically on file save
```

## Common Code Smells

### 1. Duplicated Code

**Smell**: Same or similar code in multiple places

**Impact**: Changes must be made everywhere, error-prone

**Example**:
```typescript
// Duplicated validation logic
function createUser(data) {
  if (!data.email || !data.email.includes('@')) {
    throw new Error('Invalid email')
  }
  // create user
}

function updateUser(id, data) {
  if (!data.email || !data.email.includes('@')) {
    throw new Error('Invalid email')
  }
  // update user
}
```

**Refactor**: Extract to shared function
```typescript
function validateEmail(email) {
  if (!email || !email.includes('@')) {
    throw new Error('Invalid email')
  }
}

function createUser(data) {
  validateEmail(data.email)
  // create user
}

function updateUser(id, data) {
  validateEmail(data.email)
  // update user
}
```

### 2. Long Functions

**Smell**: Function is > 50 lines, does too much

**Impact**: Hard to understand, test, and modify

**Example**:
```typescript
function processOrder(order) {
  // Validate (10 lines)
  if (!order.items || order.items.length === 0) {
    throw new Error('Order must have items')
  }
  // ... more validation

  // Calculate total (15 lines)
  let total = 0
  for (const item of order.items) {
    const price = item.quantity * item.price
    total += price
  }
  // ... discount logic

  // Save to database (20 lines)
  const connection = await db.connect()
  // ... database operations

  // Send email (15 lines)
  const emailBody = buildEmailBody(order)
  // ... email sending

  return order
}
```

**Refactor**: Extract methods
```typescript
function processOrder(order) {
  validateOrder(order)
  const total = calculateTotal(order)
  const savedOrder = await saveOrder(order, total)
  await sendOrderConfirmation(savedOrder)
  return savedOrder
}

function validateOrder(order) {
  if (!order.items || order.items.length === 0) {
    throw new Error('Order must have items')
  }
  // validation logic
}

function calculateTotal(order) {
  // calculation logic
}

async function saveOrder(order, total) {
  // database logic
}

async function sendOrderConfirmation(order) {
  // email logic
}
```

### 3. Long Parameter Lists

**Smell**: Function takes > 4 parameters

**Impact**: Hard to call, easy to pass wrong arguments

**Example**:
```typescript
function createUser(
  firstName,
  lastName,
  email,
  phone,
  address,
  city,
  zipCode,
  country
) {
  // ...
}

// Hard to call
createUser('John', 'Doe', 'john@...', '555-1234', '123 Main', 'NYC', '10001', 'USA')
```

**Refactor**: Use object parameter
```typescript
interface CreateUserParams {
  firstName: string
  lastName: string
  email: string
  phone: string
  address: string
  city: string
  zipCode: string
  country: string
}

function createUser(params: CreateUserParams) {
  // ...
}

// Easy to call, self-documenting
createUser({
  firstName: 'John',
  lastName: 'Doe',
  email: 'john@...',
  phone: '555-1234',
  address: '123 Main',
  city: 'NYC',
  zipCode: '10001',
  country: 'USA'
})
```

### 4. Deep Nesting

**Smell**: Code nested > 3 levels deep

**Impact**: Hard to follow, cognitive overload

**Example**:
```typescript
function processData(data) {
  if (data) {
    if (data.items) {
      if (data.items.length > 0) {
        for (const item of data.items) {
          if (item.active) {
            if (item.price > 0) {
              // Finally do something
            }
          }
        }
      }
    }
  }
}
```

**Refactor**: Guard clauses and early returns
```typescript
function processData(data) {
  if (!data?.items?.length) {
    return
  }

  for (const item of data.items) {
    if (!item.active || item.price <= 0) {
      continue
    }
    // Do something (at level 2 instead of 6!)
  }
}
```

### 5. Magic Numbers/Strings

**Smell**: Unexplained literals scattered in code

**Impact**: Meaning unclear, hard to change

**Example**:
```typescript
function calculateDiscount(price) {
  if (price > 100) {
    return price * 0.9
  }
  return price
}

function checkTimeout(elapsed) {
  if (elapsed > 30000) {
    throw new Error('Timeout')
  }
}
```

**Refactor**: Named constants
```typescript
const DISCOUNT_THRESHOLD = 100
const DISCOUNT_RATE = 0.1

function calculateDiscount(price) {
  if (price > DISCOUNT_THRESHOLD) {
    return price * (1 - DISCOUNT_RATE)
  }
  return price
}

const TIMEOUT_MS = 30_000 // 30 seconds

function checkTimeout(elapsed) {
  if (elapsed > TIMEOUT_MS) {
    throw new Error('Timeout')
  }
}
```

### 6. Large Classes/Modules

**Smell**: Class/module with > 500 lines, many responsibilities

**Impact**: Violates Single Responsibility Principle, hard to maintain

**Refactor**: Split by responsibility
```typescript
// Before: God class
class UserManager {
  createUser() { }
  updateUser() { }
  deleteUser() { }
  validateEmail() { }
  hashPassword() { }
  sendWelcomeEmail() { }
  generateReport() { }
  exportToCSV() { }
}

// After: Focused classes
class UserService {
  createUser() { }
  updateUser() { }
  deleteUser() { }
}

class UserValidator {
  validateEmail() { }
  validatePassword() { }
}

class UserNotifier {
  sendWelcomeEmail() { }
  sendPasswordReset() { }
}

class UserReportGenerator {
  generateReport() { }
  exportToCSV() { }
}
```

### 7. Comments Explaining What Code Does

**Smell**: Comments describing what the code does (not why)

**Impact**: Comments get stale, code should be self-documenting

**Example**:
```typescript
// Check if user is admin and has permission
if (user.role === 'admin' && user.permissions.includes('delete')) {
  // Delete the item
  db.items.delete(id)
}
```

**Refactor**: Self-documenting code
```typescript
function isAdminWithDeletePermission(user) {
  return user.role === 'admin' && user.permissions.includes('delete')
}

function deleteItem(id) {
  db.items.delete(id)
}

if (isAdminWithDeletePermission(user)) {
  deleteItem(id)
}
```

## Classic Refactoring Patterns

### Extract Method

**When**: Function does too much, extract part to new function

**Before**:
```typescript
function renderOrder(order) {
  console.log('Order ID:', order.id)
  console.log('Customer:', order.customer.name)

  let total = 0
  for (const item of order.items) {
    total += item.price * item.quantity
  }

  console.log('Total:', total)
}
```

**After**:
```typescript
function renderOrder(order) {
  printOrderHeader(order)
  const total = calculateTotal(order.items)
  printTotal(total)
}

function printOrderHeader(order) {
  console.log('Order ID:', order.id)
  console.log('Customer:', order.customer.name)
}

function calculateTotal(items) {
  return items.reduce((sum, item) => sum + item.price * item.quantity, 0)
}

function printTotal(total) {
  console.log('Total:', total)
}
```

### Rename Variable/Function

**When**: Name doesn't clearly express intent

**Before**:
```typescript
function calc(d) {
  const t = d * 24 * 60 * 60 * 1000
  return t
}
```

**After**:
```typescript
function convertDaysToMilliseconds(days) {
  const milliseconds = days * 24 * 60 * 60 * 1000
  return milliseconds
}
```

**Safe rename**:
1. Use IDE refactoring (right-click → Rename)
2. Or find-replace all occurrences
3. Run tests to ensure nothing broke

### Inline Function

**When**: Function is so simple it doesn't add value

**Before**:
```typescript
function isAdult(user) {
  return user.age >= 18
}

if (isAdult(user)) {
  // ...
}
```

**After**:
```typescript
if (user.age >= 18) {
  // ...
}
```

### Replace Conditional with Polymorphism

**When**: Complex conditionals based on type

**Before**:
```typescript
function calculateArea(shape) {
  if (shape.type === 'circle') {
    return Math.PI * shape.radius ** 2
  } else if (shape.type === 'rectangle') {
    return shape.width * shape.height
  } else if (shape.type === 'triangle') {
    return 0.5 * shape.base * shape.height
  }
}
```

**After**:
```typescript
class Circle {
  constructor(radius) {
    this.radius = radius
  }

  area() {
    return Math.PI * this.radius ** 2
  }
}

class Rectangle {
  constructor(width, height) {
    this.width = width
    this.height = height
  }

  area() {
    return this.width * this.height
  }
}

class Triangle {
  constructor(base, height) {
    this.base = base
    this.height = height
  }

  area() {
    return 0.5 * this.base * this.height
  }
}

// Usage
const shapes = [new Circle(5), new Rectangle(4, 6), new Triangle(3, 8)]
shapes.forEach(shape => console.log(shape.area()))
```

### Replace Magic Number with Named Constant

**When**: Literals appear without explanation

**Before**:
```typescript
function calculateTax(amount) {
  return amount * 0.07
}
```

**After**:
```typescript
const TAX_RATE = 0.07

function calculateTax(amount) {
  return amount * TAX_RATE
}
```

### Split Temporary Variable

**When**: Variable assigned multiple times for different purposes

**Before**:
```typescript
let temp = 2 * (height + width)
console.log('Perimeter:', temp)

temp = height * width
console.log('Area:', temp)
```

**After**:
```typescript
const perimeter = 2 * (height + width)
console.log('Perimeter:', perimeter)

const area = height * width
console.log('Area:', area)
```

### Introduce Parameter Object

**When**: Same group of parameters passed together

**Before**:
```typescript
function createRange(start, end, step) { }
function isInRange(value, start, end) { }
function formatRange(start, end) { }
```

**After**:
```typescript
class Range {
  constructor(start, end, step = 1) {
    this.start = start
    this.end = end
    this.step = step
  }

  contains(value) {
    return value >= this.start && value <= this.end
  }

  toString() {
    return `${this.start}..${this.end}`
  }
}

const range = new Range(0, 100)
```

## Refactoring Process

### Step-by-Step Approach

#### 1. Understand the Code

**Before changing anything**:
- Read the code thoroughly
- Understand what it does
- Understand why it does it
- Identify all dependencies
- Check where it's called from

**Questions to ask**:
- What's the purpose of this code?
- What are the inputs and outputs?
- What are the side effects?
- What's the error handling?
- What are the edge cases?

#### 2. Ensure Test Coverage

**Check existing tests**:
```bash
npm test -- --coverage
```

**Add tests if needed**:
- Test current behavior (even if wrong)
- Cover edge cases
- Test all code paths

**Goal**: >80% coverage before refactoring

#### 3. Make One Change at a Time

**One refactoring pattern per commit**:
- Extract method → commit
- Rename variable → commit
- Remove duplication → commit

**Why**:
- Easy to review
- Easy to revert if needed
- Clear history

#### 4. Run Tests Continuously

**After each change**:
```bash
npm test
```

**Green?** → Commit
**Red?** → Fix or revert

**Use TDD cycle**:
1. Red: Tests fail (or add new test)
2. Green: Make tests pass
3. Refactor: Improve code
4. Repeat

#### 5. Review and Iterate

**After refactoring**:
- Does code read better?
- Is complexity reduced?
- Are tests still passing?
- Any new smells introduced?

**Get feedback**:
- Self-review your diff
- Ask colleague to review
- Run linter and type checker

## Impact Analysis

### Before Refactoring: Assess Risk

**Questions**:
1. **How much code is affected?**
   - Single function? Low risk
   - Core utility used everywhere? High risk

2. **How critical is this code?**
   - Experimental feature? Lower risk
   - Payment processing? High risk

3. **How good is test coverage?**
   - >80% coverage? Lower risk
   - No tests? High risk

4. **How well do I understand it?**
   - Wrote it yesterday? Lower risk
   - Legacy code I don't understand? High risk

### Finding Usages

**Before changing a function**:

```bash
# Find all usages
grep -r "functionName" src/

# Or use IDE: right-click → Find Usages
```

**Check**:
- How many places call this?
- What contexts is it used in?
- Can I change all call sites?

### Breaking Changes

**If you must make breaking changes**:

1. **Deprecation approach**:
```typescript
// Old function - mark as deprecated
/** @deprecated Use calculateTotalV2 instead */
function calculateTotal(items) {
  console.warn('calculateTotal is deprecated, use calculateTotalV2')
  return calculateTotalV2(items)
}

// New function
function calculateTotalV2(items) {
  // new implementation
}
```

2. **Parallel implementation**:
- Add new function alongside old
- Migrate call sites gradually
- Remove old function when usage hits zero

3. **Feature flag**:
```typescript
function calculateTotal(items) {
  if (featureFlags.useNewCalculation) {
    return calculateTotalNew(items)
  }
  return calculateTotalOld(items)
}
```

## Refactoring Large Codebases

### Strangler Fig Pattern

**Gradually replace old system with new**:

1. **Identify boundary**: Where old and new will interface
2. **Build new alongside old**: Don't touch old code yet
3. **Route some traffic to new**: Use feature flags
4. **Gradually shift traffic**: 1%, 10%, 50%, 100%
5. **Remove old code**: When new is proven

**Example**:
```typescript
// Old system
function processOrderOld(order) {
  // Legacy logic (400 lines, scary)
}

// New system
function processOrderNew(order) {
  // Clean, tested implementation
}

// Router (gradually shift traffic)
function processOrder(order) {
  const useNewSystem = featureFlags.newOrderProcessing

  if (useNewSystem) {
    return processOrderNew(order)
  }
  return processOrderOld(order)
}
```

### Incremental Refactoring

**For large refactorings**:

**Week 1**: Extract helper functions
**Week 2**: Add tests for helpers
**Week 3**: Reduce duplication
**Week 4**: Split large class
**Week 5**: Update callers
**Week 6**: Remove old code

**Ship after each week**: Don't wait for everything to be perfect

## Common Refactoring Mistakes

### ❌ Refactoring Without Tests

**Problem**: No safety net, breaking changes go unnoticed

**Solution**: Write tests first, even for legacy code

### ❌ Changing Behavior While Refactoring

**Problem**: Can't tell if tests fail due to refactoring or behavior change

**Solution**: Separate commits:
1. Refactor (no behavior change)
2. Fix bug / add feature (behavior change)

### ❌ Too Much at Once

**Problem**: Massive diff, hard to review, hard to debug if something breaks

**Solution**: Small, incremental changes

### ❌ Perfection Paralysis

**Problem**: Spending weeks refactoring instead of shipping features

**Solution**:
- Fix what you touch
- Incremental improvements
- Don't refactor everything at once

### ❌ Refactoring for the Sake of It

**Problem**: No business value, just style preference

**Solution**: Refactor when it:
- Blocks new features
- Makes code hard to maintain
- Causes bugs
- Has measurable impact

## Refactoring Checklist

Before starting:
- [ ] Do I understand this code?
- [ ] Are there tests? (If no, write them)
- [ ] Do tests pass?
- [ ] Have I identified code smells?
- [ ] Is this the right time to refactor?

During refactoring:
- [ ] Making one change at a time?
- [ ] Running tests after each change?
- [ ] Committing frequently?
- [ ] Not changing behavior?
- [ ] Not adding features?

After refactoring:
- [ ] All tests still pass?
- [ ] Code is more readable?
- [ ] Complexity is reduced?
- [ ] No new code smells introduced?
- [ ] Reviewed my changes?

## Measuring Improvement

### Cyclomatic Complexity

**Measure code complexity**:

```typescript
// High complexity (bad)
function processData(data) {
  if (data) {
    if (data.type === 'A') {
      if (data.valid) {
        // ...
      } else {
        // ...
      }
    } else if (data.type === 'B') {
      // ...
    }
  }
}
// Complexity: 5

// Lower complexity (good)
function processData(data) {
  if (!data?.valid) return

  const handlers = {
    A: handleTypeA,
    B: handleTypeB
  }

  return handlers[data.type]?.(data)
}
// Complexity: 2
```

**Tools**:
- ESLint (complexity rule)
- SonarQube
- Code Climate

**Target**: Complexity < 10 per function

### Lines of Code per Function

**Aim for < 50 lines per function**

If > 50:
- Extract methods
- Split responsibilities
- Reduce nesting

### Code Coverage

**Track before and after**:
```bash
# Before
Coverage: 65%

# After refactoring (should increase)
Coverage: 85%
```

---

**Remember**: Refactoring is not about making code "perfect" – it's about making it **better, safer, and easier to change**. Refactor continuously in small increments, not in large infrequent rewrites.
