---
name: test-writer
description: Generates comprehensive unit, integration, and E2E tests with edge cases, fixtures, and mocks. Apply when implementing features, fixing bugs, or improving test coverage.
---

# Test Writer Skill

Write comprehensive, maintainable tests that catch bugs, prevent regressions, and serve as living documentation of your code's behavior.

## When to Use

✅ **After implementing a feature**: Write tests for new functionality
✅ **Before fixing a bug**: Write failing test, then fix (TDD)
✅ **Improving coverage**: Fill gaps in test suite
✅ **During refactoring**: Ensure behavior doesn't change
✅ **For critical paths**: Payment, auth, data loss scenarios
✅ **When coverage is low**: Bring untested code under test

⚠️ **Prioritize testing**:
- Critical business logic
- Security-sensitive code
- Complex algorithms
- Bug-prone areas
- Public APIs

## Test Types

### 1. Unit Tests

**What**: Test individual functions/methods in isolation

**Characteristics**:
- Fast (milliseconds)
- No external dependencies (mock DB, API, filesystem)
- Test one thing at a time
- High code coverage

**When to use**:
- Pure functions
- Business logic
- Utility functions
- Validation logic
- State management

**Example** (TypeScript/Jest):
```typescript
// Function to test
export function calculateDiscount(price: number, discountPercent: number): number {
  if (price < 0 || discountPercent < 0 || discountPercent > 100) {
    throw new Error('Invalid input')
  }
  return price * (1 - discountPercent / 100)
}

// Unit tests
describe('calculateDiscount', () => {
  it('should calculate discount correctly', () => {
    expect(calculateDiscount(100, 10)).toBe(90)
    expect(calculateDiscount(50, 20)).toBe(40)
  })

  it('should handle 0% discount', () => {
    expect(calculateDiscount(100, 0)).toBe(100)
  })

  it('should handle 100% discount', () => {
    expect(calculateDiscount(100, 100)).toBe(0)
  })

  it('should throw on negative price', () => {
    expect(() => calculateDiscount(-10, 10)).toThrow('Invalid input')
  })

  it('should throw on negative discount', () => {
    expect(() => calculateDiscount(100, -10)).toThrow('Invalid input')
  })

  it('should throw on discount > 100', () => {
    expect(() => calculateDiscount(100, 150)).toThrow('Invalid input')
  })

  it('should handle decimal values', () => {
    expect(calculateDiscount(99.99, 15.5)).toBeCloseTo(84.49, 2)
  })
})
```

### 2. Integration Tests

**What**: Test multiple components working together

**Characteristics**:
- Medium speed (seconds)
- May use real dependencies (test DB, local services)
- Test interaction between components
- Verify integrations work

**When to use**:
- Database operations
- API endpoints
- Service integrations
- File system operations
- Message queues

**Example** (Node.js/Supertest):
```typescript
// API endpoint to test
app.post('/api/users', async (req, res) => {
  const user = await db.users.create(req.body)
  res.status(201).json(user)
})

// Integration tests
describe('POST /api/users', () => {
  beforeEach(async () => {
    await db.users.deleteAll() // Clean database
  })

  it('should create a new user', async () => {
    const response = await request(app)
      .post('/api/users')
      .send({ name: 'John Doe', email: 'john@example.com' })
      .expect(201)

    expect(response.body).toMatchObject({
      name: 'John Doe',
      email: 'john@example.com'
    })

    // Verify in database
    const user = await db.users.findByEmail('john@example.com')
    expect(user).toBeDefined()
  })

  it('should reject duplicate email', async () => {
    await db.users.create({ name: 'Jane', email: 'test@example.com' })

    await request(app)
      .post('/api/users')
      .send({ name: 'John', email: 'test@example.com' })
      .expect(409) // Conflict
  })

  it('should validate required fields', async () => {
    await request(app)
      .post('/api/users')
      .send({ name: 'John' }) // Missing email
      .expect(400)
  })
})
```

### 3. End-to-End (E2E) Tests

**What**: Test complete user workflows through the UI

**Characteristics**:
- Slow (seconds to minutes)
- Uses real browser, full stack
- Tests from user perspective
- Catches integration issues

**When to use**:
- Critical user journeys (signup, checkout)
- Cross-browser compatibility
- UI interactions
- Full-stack workflows

**Example** (Playwright):
```typescript
test.describe('User Registration Flow', () => {
  test('should register new user successfully', async ({ page }) => {
    await page.goto('/signup')

    // Fill form
    await page.fill('[name="email"]', 'newuser@example.com')
    await page.fill('[name="password"]', 'SecurePass123!')
    await page.fill('[name="confirmPassword"]', 'SecurePass123!')

    // Submit
    await page.click('button[type="submit"]')

    // Verify success
    await expect(page.locator('.success-message')).toContainText(
      'Registration successful'
    )

    // Verify redirect
    await expect(page).toHaveURL('/dashboard')

    // Verify user is logged in
    await expect(page.locator('.user-menu')).toContainText('newuser@example.com')
  })

  test('should show error for weak password', async ({ page }) => {
    await page.goto('/signup')

    await page.fill('[name="email"]', 'test@example.com')
    await page.fill('[name="password"]', '123') // Weak password
    await page.fill('[name="confirmPassword"]', '123')

    await page.click('button[type="submit"]')

    await expect(page.locator('.error-message')).toContainText(
      'Password must be at least 8 characters'
    )
  })
})
```

## Test Coverage Strategy

### Aim for Meaningful Coverage, Not 100%

**High Priority** (Aim for >90% coverage):
- Business logic
- Security-critical code
- Payment processing
- Data validation
- Authentication/authorization
- API contracts

**Medium Priority** (Aim for >70% coverage):
- CRUD operations
- Utilities
- Services
- Middleware

**Low Priority** (Coverage optional):
- Simple getters/setters
- Configuration files
- Type definitions
- Third-party library wrappers

### Coverage Metrics

```bash
# Run tests with coverage
npm test -- --coverage

# Common coverage types:
# - Line coverage: % of lines executed
# - Branch coverage: % of if/else paths taken
# - Function coverage: % of functions called
# - Statement coverage: % of statements executed
```

**Target**: 80% overall coverage, 100% on critical paths

## Edge Cases and Boundary Conditions

Always test:

### Boundary Values
```typescript
describe('isPrime', () => {
  // Boundaries
  it('should return false for 0', () => expect(isPrime(0)).toBe(false))
  it('should return false for 1', () => expect(isPrime(1)).toBe(false))
  it('should return true for 2', () => expect(isPrime(2)).toBe(true))

  // Edge of range
  it('should handle large primes', () => expect(isPrime(7919)).toBe(true))
  it('should handle large composites', () => expect(isPrime(7920)).toBe(false))
})
```

### Empty/Null/Undefined
```typescript
describe('processItems', () => {
  it('should handle empty array', () => {
    expect(processItems([])).toEqual([])
  })

  it('should handle null', () => {
    expect(processItems(null)).toEqual([])
  })

  it('should handle undefined', () => {
    expect(processItems(undefined)).toEqual([])
  })
})
```

### Invalid Input
```typescript
describe('parseDate', () => {
  it('should reject invalid date string', () => {
    expect(() => parseDate('not-a-date')).toThrow()
  })

  it('should reject wrong type', () => {
    expect(() => parseDate(123 as any)).toThrow()
  })
})
```

### Size Limits
```typescript
describe('truncateString', () => {
  it('should handle string at limit', () => {
    expect(truncateString('a'.repeat(100), 100)).toHaveLength(100)
  })

  it('should truncate string over limit', () => {
    expect(truncateString('a'.repeat(101), 100)).toHaveLength(100)
  })
})
```

### Concurrent Operations
```typescript
describe('Counter (concurrency)', () => {
  it('should handle concurrent increments', async () => {
    const counter = new Counter()
    await Promise.all([
      counter.increment(),
      counter.increment(),
      counter.increment()
    ])
    expect(counter.value).toBe(3) // Not 2 or 1!
  })
})
```

## Mocking and Test Doubles

### When to Mock

**Mock**: External dependencies
- Databases
- APIs
- File system
- Current time
- Random number generators
- External services

**Don't Mock**: Internal logic you're testing

### Mocking Patterns

#### Database Mocking
```typescript
import { jest } from '@jest/globals'

// Mock database
const mockDb = {
  users: {
    findById: jest.fn(),
    create: jest.fn(),
    update: jest.fn()
  }
}

describe('UserService', () => {
  beforeEach(() => {
    jest.clearAllMocks()
  })

  it('should get user by id', async () => {
    const mockUser = { id: 1, name: 'John' }
    mockDb.users.findById.mockResolvedValue(mockUser)

    const service = new UserService(mockDb)
    const user = await service.getUser(1)

    expect(user).toEqual(mockUser)
    expect(mockDb.users.findById).toHaveBeenCalledWith(1)
  })
})
```

#### API Mocking
```typescript
import nock from 'nock'

describe('WeatherService', () => {
  it('should fetch weather data', async () => {
    nock('https://api.weather.com')
      .get('/forecast')
      .query({ city: 'London' })
      .reply(200, { temp: 20, condition: 'sunny' })

    const service = new WeatherService()
    const weather = await service.getWeather('London')

    expect(weather.temp).toBe(20)
  })
})
```

#### Time Mocking
```typescript
import { jest } from '@jest/globals'

describe('ExpirationChecker', () => {
  beforeEach(() => {
    jest.useFakeTimers()
  })

  afterEach(() => {
    jest.useRealTimers()
  })

  it('should mark item as expired after 1 hour', () => {
    const checker = new ExpirationChecker()
    const item = checker.create()

    expect(item.isExpired()).toBe(false)

    // Advance time by 1 hour
    jest.advanceTimersByTime(60 * 60 * 1000)

    expect(item.isExpired()).toBe(true)
  })
})
```

## Test Organization

### File Structure

```
src/
  services/
    user.service.ts
    user.service.test.ts        # Unit tests
    user.service.integration.ts # Integration tests

tests/
  e2e/
    user-registration.spec.ts   # E2E tests
  fixtures/
    users.json                  # Test data
  helpers/
    setup.ts                    # Test setup utilities
```

### Test Naming

**Pattern**: `should [expected behavior] when [condition]`

```typescript
describe('UserService', () => {
  describe('getUser', () => {
    it('should return user when id exists', async () => {})
    it('should throw NotFoundError when id does not exist', async () => {})
    it('should throw ValidationError when id is invalid', async () => {})
  })

  describe('createUser', () => {
    it('should create user with valid data', async () => {})
    it('should hash password before storing', async () => {})
    it('should reject duplicate email', async () => {})
  })
})
```

### Setup and Teardown

```typescript
describe('DatabaseTests', () => {
  // Run once before all tests
  beforeAll(async () => {
    await db.connect()
  })

  // Run before each test
  beforeEach(async () => {
    await db.clearAll()
    await db.seed(fixtures.users)
  })

  // Run after each test
  afterEach(async () => {
    await db.clearAll()
  })

  // Run once after all tests
  afterAll(async () => {
    await db.disconnect()
  })

  it('should...', () => {})
})
```

## Test Fixtures

### Static Fixtures
```typescript
// fixtures/users.ts
export const testUsers = {
  admin: {
    id: 1,
    email: 'admin@example.com',
    role: 'admin',
    name: 'Admin User'
  },
  regular: {
    id: 2,
    email: 'user@example.com',
    role: 'user',
    name: 'Regular User'
  }
}
```

### Factory Pattern
```typescript
// factories/user.factory.ts
let nextId = 1

export function createUser(overrides = {}) {
  return {
    id: nextId++,
    email: `user${nextId}@example.com`,
    name: `User ${nextId}`,
    role: 'user',
    createdAt: new Date(),
    ...overrides
  }
}

// Usage in tests
it('should...', () => {
  const admin = createUser({ role: 'admin' })
  const user = createUser({ email: 'custom@example.com' })
})
```

## Testing Best Practices

### Do:
✅ Test behavior, not implementation
✅ Write tests before fixing bugs (reproduce first)
✅ Keep tests simple and readable
✅ Use descriptive test names
✅ Test one thing per test
✅ Make tests independent (no shared state)
✅ Use AAA pattern (Arrange, Act, Assert)
✅ Mock external dependencies
✅ Test error cases, not just happy path

### Don't:
❌ Test private methods directly
❌ Make tests depend on execution order
❌ Use sleep/timeouts (use proper async)
❌ Test framework code (trust React, Express, etc.)
❌ Duplicate tests across suites
❌ Leave commented-out tests
❌ Write flaky tests
❌ Over-mock (mock only external deps)

## AAA Pattern

**Arrange, Act, Assert** - structure every test clearly:

```typescript
it('should calculate total price with tax', () => {
  // Arrange - Set up test data
  const cart = new ShoppingCart()
  cart.addItem({ price: 100, quantity: 2 })
  const taxRate = 0.1

  // Act - Execute the behavior
  const total = cart.calculateTotal(taxRate)

  // Assert - Verify the result
  expect(total).toBe(220) // (100 * 2) * 1.1
})
```

## Common Testing Patterns

### Testing Async Code
```typescript
// Using async/await
it('should fetch user', async () => {
  const user = await userService.getUser(1)
  expect(user.name).toBe('John')
})

// Testing promise rejection
it('should reject invalid id', async () => {
  await expect(userService.getUser(-1)).rejects.toThrow('Invalid ID')
})
```

### Testing Callbacks
```typescript
it('should call callback with result', (done) => {
  fetchUser(1, (err, user) => {
    expect(err).toBeNull()
    expect(user.name).toBe('John')
    done()
  })
})
```

### Testing Events
```typescript
it('should emit event when user created', (done) => {
  const emitter = new UserService()

  emitter.on('user:created', (user) => {
    expect(user.name).toBe('John')
    done()
  })

  emitter.createUser({ name: 'John' })
})
```

### Parametrized Tests
```typescript
describe.each([
  [0, 0],
  [1, 1],
  [2, 4],
  [3, 9],
  [10, 100]
])('square(%i)', (input, expected) => {
  it(`should return ${expected}`, () => {
    expect(square(input)).toBe(expected)
  })
})
```

### Snapshot Testing
```typescript
it('should render user card correctly', () => {
  const component = render(<UserCard user={mockUser} />)
  expect(component.container).toMatchSnapshot()
})

// Use sparingly - snapshots can be brittle
```

## Framework-Specific Patterns

### React Testing Library
```typescript
import { render, screen, fireEvent } from '@testing-library/react'

it('should toggle visibility on button click', () => {
  render(<ToggleComponent />)

  // Query elements
  const button = screen.getByRole('button', { name: /toggle/i })
  const content = screen.getByText(/secret content/i)

  // Initial state
  expect(content).not.toBeVisible()

  // Interact
  fireEvent.click(button)

  // Assert new state
  expect(content).toBeVisible()
})
```

### Python pytest
```python
import pytest

def test_divide():
    assert divide(10, 2) == 5

def test_divide_by_zero():
    with pytest.raises(ZeroDivisionError):
        divide(10, 0)

@pytest.mark.parametrize("a,b,expected", [
    (10, 2, 5),
    (20, 4, 5),
    (100, 10, 10),
])
def test_divide_parametrized(a, b, expected):
    assert divide(a, b) == expected
```

### Go testing
```go
func TestCalculateDiscount(t *testing.T) {
    tests := []struct {
        name     string
        price    float64
        discount float64
        want     float64
        wantErr  bool
    }{
        {"valid 10%", 100, 10, 90, false},
        {"negative price", -10, 10, 0, true},
        {"over 100%", 100, 150, 0, true},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := CalculateDiscount(tt.price, tt.discount)
            if (err != nil) != tt.wantErr {
                t.Errorf("want error %v, got %v", tt.wantErr, err)
            }
            if got != tt.want {
                t.Errorf("want %v, got %v", tt.want, got)
            }
        })
    }
}
```

## TDD Workflow (Red-Green-Refactor)

### 1. Red - Write Failing Test
```typescript
it('should calculate compound interest', () => {
  expect(calculateCompoundInterest(1000, 0.05, 2)).toBe(1102.50)
})
// Test fails - function doesn't exist yet
```

### 2. Green - Make It Pass
```typescript
function calculateCompoundInterest(principal, rate, years) {
  return principal * Math.pow(1 + rate, years)
}
// Test passes
```

### 3. Refactor - Improve Code
```typescript
function calculateCompoundInterest(
  principal: number,
  rate: number,
  years: number
): number {
  if (principal < 0 || rate < 0 || years < 0) {
    throw new Error('Values must be non-negative')
  }
  return parseFloat((principal * Math.pow(1 + rate, years)).toFixed(2))
}
// Test still passes, code is better
```

## Coverage Gaps Analysis

Identify untested code:

```bash
# Generate coverage report
npm test -- --coverage

# View HTML report
open coverage/lcov-report/index.html

# Check specific file
npm test -- --coverage --collectCoverageFrom=src/services/user.service.ts
```

**Look for**:
- Uncovered lines (red in report)
- Uncovered branches (yellow in report)
- Uncovered functions
- High complexity, low coverage (risky!)

## Testing Anti-Patterns

### ❌ Testing Implementation Details
```typescript
// Bad - tests implementation
it('should call internal method', () => {
  const spy = jest.spyOn(service, '_internalMethod')
  service.publicMethod()
  expect(spy).toHaveBeenCalled()
})

// Good - tests behavior
it('should return processed data', () => {
  const result = service.publicMethod()
  expect(result).toEqual(expectedOutput)
})
```

### ❌ Shared State Between Tests
```typescript
// Bad - tests affect each other
let user
beforeAll(() => {
  user = createUser() // Shared across tests!
})

// Good - isolated state
beforeEach(() => {
  user = createUser() // Fresh for each test
})
```

### ❌ Testing Too Much at Once
```typescript
// Bad - tests everything
it('should handle user workflow', async () => {
  const user = await createUser()
  await user.login()
  await user.updateProfile()
  await user.makePurchase()
  // Too much!
})

// Good - focused tests
it('should create user', async () => {})
it('should login user', async () => {})
it('should update profile', async () => {})
```

---

**Remember**: Good tests are **fast, independent, repeatable, self-validating, and timely** (FIRST). Write tests that give you confidence to refactor and ship with peace of mind.
