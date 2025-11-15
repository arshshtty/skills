---
name: test-strategy
description: Create comprehensive testing strategies using the testing pyramid. Apply when writing tests, planning test coverage, or setting up testing infrastructure for any codebase.
---

# Test Strategy Skill

Design and implement comprehensive testing strategies that balance coverage, speed, and maintainability using testing pyramid principles.

## The Testing Pyramid

```
       /\
      /  \     E2E Tests (Few)
     /    \    - Slow, expensive, brittle
    /------\   - Test critical user journeys
   /        \  - 5-10% of tests
  /          \
 / Integration\ Integration Tests (Some)
/    Tests     \- Medium speed/cost
------------------ Test module interactions
/                \ 10-20% of tests
/                 \
/   Unit Tests     \ Unit Tests (Many)
--------------------/ Fast, cheap, focused
                     - Test individual functions
                     - 70-80% of tests
```

## Core Principles

### 1. **Test Pyramid Structure**
- **Base (Unit)**: Fast, focused, abundant
- **Middle (Integration)**: Moderate, realistic interactions
- **Top (E2E)**: Slow, critical paths only

### 2. **What to Test**
✅ Business logic and algorithms
✅ Edge cases and boundary conditions
✅ Error handling and failure modes
✅ Public APIs and interfaces
✅ Critical user workflows
✅ Security-sensitive code

❌ Third-party library internals
❌ Framework code
❌ Trivial getters/setters
❌ Auto-generated code
❌ Private implementation details

### 3. **Test Quality Over Quantity**
- **Meaningful coverage** > percentage metrics
- Tests should catch real bugs, not just exist
- One well-designed test > ten brittle ones
- Fast feedback loops encourage testing

## Testing Strategy by Layer

### Unit Tests (70-80% of tests)

**Purpose**: Test individual units in isolation

**Characteristics**:
- Fast (milliseconds)
- No external dependencies (mock/stub them)
- Focused on one function/method/class
- Deterministic and repeatable

**What to test**:
- Pure functions with different inputs
- Business logic branches
- Edge cases (null, empty, boundary values)
- Error conditions

**Example approach** (language-agnostic):
```
test "calculateDiscount with valid coupon":
  given:
    cart with total $100
    coupon with 20% off
  when:
    calculateDiscount(cart, coupon)
  then:
    returns $20

test "calculateDiscount with expired coupon":
  given:
    cart with total $100
    expired coupon
  when:
    calculateDiscount(cart, coupon)
  then:
    throws InvalidCouponError
```

**Patterns**:
- **Arrange-Act-Assert (AAA)**: Set up → Execute → Verify
- **Given-When-Then (GWT)**: Context → Action → Outcome
- **Test one thing**: Each test validates one behavior
- **Descriptive names**: Test name explains what's being tested

### Integration Tests (10-20% of tests)

**Purpose**: Test how components work together

**Characteristics**:
- Slower (seconds)
- Use real dependencies where practical
- Test realistic scenarios
- May use test databases/APIs

**What to test**:
- Database queries and transactions
- API endpoints with real request/response
- Service-to-service communication
- File system operations
- External service integrations (with test instances)

**Example approach**:
```
test "user registration flow":
  given:
    clean test database
    valid user data
  when:
    POST /api/users with user data
  then:
    - returns 201 Created
    - user exists in database
    - confirmation email sent
    - password is hashed
```

**Patterns**:
- **Test containers**: Spin up real databases in Docker
- **Test fixtures**: Reusable test data setups
- **Cleanup**: Reset state after each test
- **Realistic data**: Use production-like test data

### E2E Tests (5-10% of tests)

**Purpose**: Test complete user workflows from UI to database

**Characteristics**:
- Slowest (minutes)
- Full stack involvement
- Fragile to UI changes
- Most expensive to maintain

**What to test**:
- Critical business workflows (checkout, signup, payment)
- Multi-step user journeys
- Cross-browser compatibility
- Accessibility requirements

**Example approach**:
```
test "complete purchase flow":
  1. Navigate to product page
  2. Add item to cart
  3. Proceed to checkout
  4. Enter shipping info
  5. Enter payment details
  6. Confirm order
  7. Verify order confirmation page
  8. Verify order in database
  9. Verify confirmation email sent
```

**Patterns**:
- **Page Object Model**: Encapsulate page interactions
- **Critical paths only**: Don't test every variation
- **Stable selectors**: Use data attributes, not CSS classes
- **Wait strategies**: Handle async operations properly

## Test Coverage Guidelines

### Coverage Metrics
- **Line coverage**: Percentage of code lines executed
- **Branch coverage**: Percentage of decision branches tested
- **Path coverage**: Percentage of execution paths tested

### Realistic Targets
- **Unit tests**: 80-90% line coverage for business logic
- **Integration tests**: 60-70% coverage of critical paths
- **E2E tests**: 100% coverage of critical user journeys

⚠️ **Warning**: 100% coverage ≠ bug-free code. Focus on meaningful tests.

### Coverage Gaps to Address
- Uncovered error handlers
- Untested edge cases
- Missing negative test cases
- Unvalidated security checks

## Testing Patterns & Best Practices

### Test Independence
- Each test should run in isolation
- No shared state between tests
- Tests should pass in any order
- Use setup/teardown hooks properly

### Test Data Management
- **Factories**: Generate test objects on demand
- **Fixtures**: Reusable test data sets
- **Builders**: Fluent interfaces for test data
- **Snapshots**: Capture expected output for comparison

### Mocking & Stubbing
**When to mock**:
- Slow external dependencies (APIs, databases)
- Non-deterministic behavior (time, randomness)
- Difficult-to-trigger scenarios (errors, edge cases)

**When NOT to mock**:
- Simple, fast functions
- Code you're specifically testing
- Everything (over-mocking makes tests brittle)

### Assertion Strategies
- **Specific assertions**: Check exact values when possible
- **Partial assertions**: Validate structure, not exact content
- **Error assertions**: Verify error type and message
- **State assertions**: Check system state after actions

## Testing Different Code Types

### API/Service Layer
- Test all HTTP methods (GET, POST, PUT, DELETE)
- Validate request/response schemas
- Test authentication and authorization
- Check error responses (400, 401, 403, 404, 500)
- Verify rate limiting and throttling

### Database Layer
- Test CRUD operations
- Verify constraints and validations
- Check transaction rollbacks
- Test query performance (for critical queries)
- Validate migrations work both ways (up/down)

### Business Logic
- Test all conditional branches
- Validate calculations and transformations
- Check state transitions
- Test invariants and constraints
- Verify error conditions

### UI Components
- Test rendering with different props
- Validate user interactions (clicks, inputs)
- Check accessibility (a11y)
- Test error states and loading states
- Verify responsive behavior

## Test Organization

### File Structure
```
project/
├── src/
│   ├── user.js
│   └── payment.js
└── tests/
    ├── unit/
    │   ├── user.test.js
    │   └── payment.test.js
    ├── integration/
    │   ├── api.test.js
    │   └── database.test.js
    └── e2e/
        └── checkout.test.js
```

### Naming Conventions
- **Test files**: `*.test.js`, `*_test.py`, `*_spec.rb`
- **Test names**: Describe behavior, not implementation
- **Good**: `"throws error when email is invalid"`
- **Bad**: `"testFunction1"`

### Grouping Tests
- Group related tests using describe/context blocks
- Share setup code with before/beforeEach hooks
- Use consistent test structure across the codebase

## Test-Driven Development (TDD)

### Red-Green-Refactor Cycle
1. **Red**: Write a failing test
2. **Green**: Write minimal code to pass
3. **Refactor**: Improve code while keeping tests green

### When to Use TDD
✅ Complex business logic
✅ Well-defined requirements
✅ Bug fixing (write test that reproduces bug)
✅ Public APIs and interfaces

⚠️ **Not everything needs TDD**: Exploratory code, prototypes, and UI polish can be tested after.

## Performance Testing

### Load Testing
- Test system under expected load
- Identify performance bottlenecks
- Measure response times and throughput

### Stress Testing
- Test system beyond normal capacity
- Find breaking points
- Validate graceful degradation

### Tools Approach (Language Agnostic)
- Benchmark critical functions
- Profile database queries
- Monitor memory usage
- Track response time percentiles (p50, p95, p99)

## Continuous Testing

### CI/CD Integration
- Run unit tests on every commit
- Run integration tests on every PR
- Run E2E tests before deployment
- Block merges if tests fail

### Test Speed Optimization
- Parallelize test execution
- Use test databases/containers
- Cache dependencies
- Split slow tests into separate jobs

### Flaky Test Management
- Identify and fix non-deterministic tests
- Don't ignore flaky tests
- Use retries sparingly (fix root cause instead)
- Track flaky test metrics

## Testing Anti-Patterns

❌ **Testing implementation details**: Test behavior, not internals
❌ **Brittle tests**: Tests that break on minor changes
❌ **Slow test suites**: Tests that take too long discourage running them
❌ **Test interdependence**: Tests that rely on other tests
❌ **Mocking everything**: Over-mocking makes tests meaningless
❌ **No assertions**: Tests that don't verify anything
❌ **Ignored tests**: Commented-out or skipped tests accumulate

## Testing Checklist

Before considering a feature complete:

- [ ] Unit tests cover business logic (80%+ coverage)
- [ ] Integration tests verify component interactions
- [ ] E2E tests validate critical user workflows
- [ ] Edge cases and error conditions tested
- [ ] Tests are fast and reliable
- [ ] Tests have clear, descriptive names
- [ ] No flaky or intermittent failures
- [ ] Tests run in CI/CD pipeline
- [ ] Security-sensitive code has dedicated tests
- [ ] Performance-critical code has benchmarks

## Language-Agnostic Testing Tools

### Unit Testing
- **Assertion libraries**: Built-in or third-party
- **Mocking frameworks**: Stubs, spies, mocks
- **Test runners**: Execute and report test results

### Integration Testing
- **Database testing**: Test databases or containers
- **HTTP testing**: Request/response validation
- **Test containers**: Isolated environments

### E2E Testing
- **Browser automation**: Headless browsers
- **Mobile testing**: Device emulators
- **Visual regression**: Screenshot comparison

## Adapting to Your Stack

This skill provides principles. Adapt to your tech stack:

- **Backend**: Focus on API, database, and business logic tests
- **Frontend**: Focus on component, integration, and E2E tests
- **Mobile**: Focus on unit, UI, and device-specific tests
- **Embedded**: Focus on unit and hardware abstraction tests

---

**Remember**: Tests are documentation, safety nets, and design tools. Invest in a solid testing strategy—it pays dividends in confidence, velocity, and quality.
