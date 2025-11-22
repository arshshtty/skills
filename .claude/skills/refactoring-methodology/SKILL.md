---
name: refactoring-methodology
description: Safely refactor code using proven patterns to improve structure without changing behavior. Apply when improving code quality, reducing technical debt, or making code easier to extend.
---

# Refactoring Methodology Skill

Systematically improve code structure, readability, and maintainability without changing external behavior, using proven refactoring patterns and safety techniques.

## Core Principles

### 1. **Behavior Preservation**
- Refactoring changes structure, not behavior
- Tests must pass before and after
- External API contracts remain the same
- If behavior changes, it's not refactoring

### 2. **Small, Incremental Steps**
- Make one change at a time
- Commit frequently
- Each step should be reversible
- Keep the system working at all times

### 3. **Test Coverage First**
- Have tests before refactoring
- Tests are your safety net
- If no tests exist, write them first
- Run tests after every change

### 4. **Refactor with Purpose**
- Have a clear goal
- Don't refactor for refactoring's sake
- Address specific code smells
- Improve specific quality attributes

## When to Refactor

### Good Times to Refactor

- **Before adding features**: Clean up code you're about to modify
- **During code review**: Address issues before merging
- **When fixing bugs**: Improve code you're already changing
- **During dedicated tech debt time**: Scheduled cleanup
- **Rule of three**: Third time you see duplication, refactor

### When NOT to Refactor

- **No tests**: Add tests first
- **Deadline pressure**: Ship first, refactor later
- **Code you don't understand**: Learn it first
- **Stable, working code**: If it ain't broke...
- **During a production crisis**: Fix the issue first

## Code Smells

Code smells are indicators that refactoring might be needed.

### Bloaters

**Long Method**
```
Smell: Function does too much, hard to understand
Signs: > 30 lines, multiple levels of abstraction
Fix: Extract Method, Replace Temp with Query
```

**Large Class**
```
Smell: Class has too many responsibilities
Signs: > 10 methods, > 5 instance variables
Fix: Extract Class, Extract Subclass
```

**Primitive Obsession**
```
Smell: Using primitives instead of small objects
Signs: String for phone, int for money
Fix: Replace Primitive with Object, Introduce Parameter Object
```

**Long Parameter List**
```
Smell: Function has too many parameters
Signs: > 3-4 parameters
Fix: Introduce Parameter Object, Replace Parameter with Method Call
```

**Data Clumps**
```
Smell: Same group of data appears together repeatedly
Signs: Same parameters in multiple methods
Fix: Extract Class, Introduce Parameter Object
```

### Object-Orientation Abusers

**Switch Statements**
```
Smell: Same switch/if-else on type in multiple places
Signs: Type checking, multiple conditionals
Fix: Replace Conditional with Polymorphism
```

**Refused Bequest**
```
Smell: Subclass doesn't use parent's methods/properties
Signs: Empty overrides, unused inheritance
Fix: Replace Inheritance with Delegation
```

**Alternative Classes with Different Interfaces**
```
Smell: Similar classes with different method names
Signs: Classes do same thing differently
Fix: Rename Method, Move Method
```

### Change Preventers

**Divergent Change**
```
Smell: One class changed for multiple unrelated reasons
Signs: Changing same class for different features
Fix: Extract Class
```

**Shotgun Surgery**
```
Smell: One change requires many small changes elsewhere
Signs: Scattered related code
Fix: Move Method, Move Field, Inline Class
```

**Parallel Inheritance Hierarchies**
```
Smell: Creating subclass in one hierarchy requires subclass in another
Signs: Matching class names in different hierarchies
Fix: Move Method, Move Field
```

### Dispensables

**Comments (Excessive)**
```
Smell: Comments used to explain bad code
Signs: Long comments explaining logic
Fix: Extract Method, Rename Method
```

**Duplicate Code**
```
Smell: Same code in multiple places
Signs: Copy-pasted logic
Fix: Extract Method, Pull Up Method, Form Template Method
```

**Dead Code**
```
Smell: Code that's never executed
Signs: Unused variables, unreachable code
Fix: Remove Dead Code
```

**Speculative Generality**
```
Smell: Code created "in case we need it"
Signs: Unused parameters, abstract classes with one subclass
Fix: Collapse Hierarchy, Inline Class, Remove Parameter
```

### Couplers

**Feature Envy**
```
Smell: Method uses more of another class than its own
Signs: Extensive use of other object's methods
Fix: Move Method, Extract Method
```

**Inappropriate Intimacy**
```
Smell: Classes too involved with each other's internals
Signs: Accessing private fields, bidirectional associations
Fix: Move Method, Move Field, Hide Delegate
```

**Message Chains**
```
Smell: Chain of method calls (a.b().c().d())
Signs: Client depends on navigation structure
Fix: Hide Delegate, Extract Method, Move Method
```

**Middle Man**
```
Smell: Class just delegates to another class
Signs: Many methods just call another object's method
Fix: Remove Middle Man, Inline Method
```

## Refactoring Patterns

### Composing Methods

**Extract Method**
```
Before:
  function printOwing(invoice) {
    printBanner();

    // Calculate outstanding
    let outstanding = 0;
    for (const order of invoice.orders) {
      outstanding += order.amount;
    }

    // Print details
    console.log(`name: ${invoice.customer}`);
    console.log(`amount: ${outstanding}`);
  }

After:
  function printOwing(invoice) {
    printBanner();
    const outstanding = calculateOutstanding(invoice);
    printDetails(invoice, outstanding);
  }

  function calculateOutstanding(invoice) {
    return invoice.orders.reduce((sum, order) => sum + order.amount, 0);
  }

  function printDetails(invoice, outstanding) {
    console.log(`name: ${invoice.customer}`);
    console.log(`amount: ${outstanding}`);
  }
```

**Inline Method**
```
Before:
  function getRating(driver) {
    return moreThanFiveLateDeliveries(driver) ? 2 : 1;
  }

  function moreThanFiveLateDeliveries(driver) {
    return driver.lateDeliveries > 5;
  }

After:
  function getRating(driver) {
    return driver.lateDeliveries > 5 ? 2 : 1;
  }
```

**Replace Temp with Query**
```
Before:
  function calculateTotal(order) {
    const basePrice = order.quantity * order.itemPrice;
    if (basePrice > 1000) {
      return basePrice * 0.95;
    }
    return basePrice;
  }

After:
  function calculateTotal(order) {
    if (basePrice(order) > 1000) {
      return basePrice(order) * 0.95;
    }
    return basePrice(order);
  }

  function basePrice(order) {
    return order.quantity * order.itemPrice;
  }
```

### Simplifying Conditional Expressions

**Decompose Conditional**
```
Before:
  if (date.before(SUMMER_START) || date.after(SUMMER_END)) {
    charge = quantity * winterRate + winterServiceCharge;
  } else {
    charge = quantity * summerRate;
  }

After:
  if (isSummer(date)) {
    charge = summerCharge(quantity);
  } else {
    charge = winterCharge(quantity);
  }
```

**Consolidate Conditional Expression**
```
Before:
  function disabilityAmount(employee) {
    if (employee.seniority < 2) return 0;
    if (employee.monthsDisabled > 12) return 0;
    if (employee.isPartTime) return 0;
    // Calculate disability amount...
  }

After:
  function disabilityAmount(employee) {
    if (isNotEligibleForDisability(employee)) return 0;
    // Calculate disability amount...
  }

  function isNotEligibleForDisability(employee) {
    return employee.seniority < 2
        || employee.monthsDisabled > 12
        || employee.isPartTime;
  }
```

**Replace Nested Conditional with Guard Clauses**
```
Before:
  function getPayAmount(employee) {
    let result;
    if (employee.isSeparated) {
      result = {amount: 0, reason: "separated"};
    } else {
      if (employee.isRetired) {
        result = {amount: 0, reason: "retired"};
      } else {
        // Calculate pay...
        result = {amount: somePay};
      }
    }
    return result;
  }

After:
  function getPayAmount(employee) {
    if (employee.isSeparated) return {amount: 0, reason: "separated"};
    if (employee.isRetired) return {amount: 0, reason: "retired"};
    // Calculate pay...
    return {amount: somePay};
  }
```

**Replace Conditional with Polymorphism**
```
Before:
  function getSpeed(vehicle) {
    switch (vehicle.type) {
      case 'car':
        return vehicle.baseSpeed;
      case 'bike':
        return vehicle.baseSpeed - 10;
      case 'plane':
        return vehicle.baseSpeed + vehicle.jetSpeed;
    }
  }

After:
  class Car {
    getSpeed() {
      return this.baseSpeed;
    }
  }

  class Bike {
    getSpeed() {
      return this.baseSpeed - 10;
    }
  }

  class Plane {
    getSpeed() {
      return this.baseSpeed + this.jetSpeed;
    }
  }
```

### Moving Features

**Move Method**
```
When: Method uses more features of another class

Before:
  class Account {
    overdraftCharge() {
      if (this.type.isPremium) {
        const baseCharge = 10;
        if (this.daysOverdrawn > 7) {
          return baseCharge + (this.daysOverdrawn - 7) * 0.85;
        }
        return baseCharge;
      }
      return this.daysOverdrawn * 1.75;
    }
  }

After:
  class AccountType {
    overdraftCharge(daysOverdrawn) {
      if (this.isPremium) {
        const baseCharge = 10;
        if (daysOverdrawn > 7) {
          return baseCharge + (daysOverdrawn - 7) * 0.85;
        }
        return baseCharge;
      }
      return daysOverdrawn * 1.75;
    }
  }

  class Account {
    overdraftCharge() {
      return this.type.overdraftCharge(this.daysOverdrawn);
    }
  }
```

**Extract Class**
```
When: Class does too much

Before:
  class Person {
    name;
    officeAreaCode;
    officeNumber;

    getTelephoneNumber() {
      return `(${this.officeAreaCode}) ${this.officeNumber}`;
    }
  }

After:
  class Person {
    name;
    officeTelephone = new TelephoneNumber();

    getTelephoneNumber() {
      return this.officeTelephone.toString();
    }
  }

  class TelephoneNumber {
    areaCode;
    number;

    toString() {
      return `(${this.areaCode}) ${this.number}`;
    }
  }
```

### Organizing Data

**Replace Magic Number with Constant**
```
Before:
  function potentialEnergy(mass, height) {
    return mass * 9.81 * height;
  }

After:
  const GRAVITATIONAL_CONSTANT = 9.81;

  function potentialEnergy(mass, height) {
    return mass * GRAVITATIONAL_CONSTANT * height;
  }
```

**Introduce Parameter Object**
```
Before:
  function amountInvoiced(startDate, endDate) { ... }
  function amountReceived(startDate, endDate) { ... }
  function amountOverdue(startDate, endDate) { ... }

After:
  class DateRange {
    constructor(start, end) {
      this.start = start;
      this.end = end;
    }
  }

  function amountInvoiced(dateRange) { ... }
  function amountReceived(dateRange) { ... }
  function amountOverdue(dateRange) { ... }
```

**Encapsulate Field**
```
Before:
  class Person {
    name;  // Public field
  }

After:
  class Person {
    #name;  // Private field

    getName() {
      return this.#name;
    }

    setName(name) {
      this.#name = name;
    }
  }
```

## Refactoring Process

### Step-by-Step Approach

**1. Identify the smell**
```
What's wrong with this code?
- Too complex?
- Duplicated?
- Hard to understand?
- Hard to change?
```

**2. Ensure test coverage**
```
Before refactoring:
- Do tests exist?
- Do they pass?
- Do they cover the code you're changing?

If not: Write tests first!
```

**3. Apply refactoring in small steps**
```
For each change:
1. Make one small change
2. Run tests
3. If tests pass, commit
4. If tests fail, undo and try smaller step
5. Repeat
```

**4. Review and clean up**
```
After refactoring:
- Is the code better?
- Are tests still passing?
- Any new smells introduced?
- Documentation updated?
```

### Safe Refactoring Techniques

**Use IDE Refactoring Tools**
- Rename (updates all references)
- Extract Method/Variable
- Move
- Change signature
- Inline

**Parallel Change (Expand and Contract)**
```
1. Add new implementation alongside old
2. Gradually migrate callers to new
3. Remove old implementation when all migrated

Example:
// Step 1: Add new method
function getName() { return this.name; }
function getFullName() { return `${this.firstName} ${this.lastName}`; }

// Step 2: Migrate callers to getFullName()

// Step 3: Remove getName() when no longer used
```

**Strangler Fig Pattern**
```
For large systems:
1. Create new implementation
2. Route some traffic to new
3. Gradually increase routing
4. Remove old when all traffic moved
```

**Branch by Abstraction**
```
1. Create abstraction for current implementation
2. Modify clients to use abstraction
3. Create new implementation of abstraction
4. Switch to new implementation
5. Remove old implementation
```

## Working with Legacy Code

### When Tests Don't Exist

**Characterization Testing**
```
Purpose: Document current behavior, not correct behavior

Process:
1. Write test that calls the code
2. Assert on actual output (even if wrong)
3. This captures existing behavior
4. Now you can refactor safely
```

**Seam Extraction**
```
Find places where you can change behavior without editing code:
- Subclass and override
- Extract interface
- Inject dependencies

This allows testing without modifying risky code
```

### Safe Modifications

**Sprout Method**
```
When adding feature to complex code:
1. Write new code in new method
2. Call from original location
3. New method can be tested easily
4. Later, refactor original code
```

**Wrap Method**
```
When adding behavior before/after existing code:
1. Rename old method
2. Create new method with old name
3. New method calls renamed method + new behavior
4. Clients unchanged
```

## Refactoring in Practice

### Refactoring to Patterns

**Replace Constructor with Factory Method**
```
When: Construction logic is complex or varies

Before:
  const employee = new Employee(type);

After:
  const employee = Employee.create(type);

  class Employee {
    static create(type) {
      switch (type) {
        case 'engineer': return new Engineer();
        case 'manager': return new Manager();
      }
    }
  }
```

**Replace Inheritance with Composition**
```
When: Inheritance is misused or too rigid

Before:
  class Stack extends ArrayList {
    push(item) { this.add(item); }
    pop() { return this.remove(this.size() - 1); }
  }
  // But Stack now has all ArrayList methods!

After:
  class Stack {
    #items = [];

    push(item) { this.#items.push(item); }
    pop() { return this.#items.pop(); }
  }
```

### Common Refactoring Sequences

**Tease Apart Inheritance**
```
1. Create separate hierarchy for each dimension of variation
2. Use composition to combine them
```

**Convert Procedural Design to Objects**
```
1. Turn each "record" into dumb data class
2. Move behavior into classes
3. Remove procedural code
```

**Separate Domain from Presentation**
```
1. Extract domain logic from UI code
2. Create domain layer
3. UI delegates to domain
```

## Refactoring Checklist

Before refactoring:
- [ ] Tests exist and pass
- [ ] You understand the code
- [ ] You have a clear goal
- [ ] Changes are committed/stashed

During refactoring:
- [ ] Make small changes
- [ ] Run tests after each change
- [ ] Commit frequently
- [ ] Keep the system working

After refactoring:
- [ ] All tests pass
- [ ] Code is measurably better
- [ ] No new smells introduced
- [ ] Documentation updated

## Anti-Patterns

**Big Bang Refactoring**
- Don't rewrite everything at once
- Small, incremental changes are safer

**Refactoring Without Tests**
- Always have tests first
- No tests = no safety net

**Gold Plating**
- Don't over-engineer
- Refactor for current needs

**Refactoring Forever**
- Have clear stopping criteria
- Don't pursue perfection

**Changing Behavior During Refactoring**
- If you're adding features, you're not refactoring
- Keep them separate

---

**Remember**: Refactoring is about improving code structure without changing behavior. Work in small steps, keep tests green, and commit often. The goal is code that's easier to understand, modify, and extend.
