---
name: clean-code-utils
description: Use when creating or modifying utility functions, helpers, services, or shared logic - covers long functions, deep nesting, large switch/if-else chains, magic values, duplication, unclear naming, and over-abstraction
---

# Clean Code Utils

## Purpose

This skill enforces clean, maintainable, readable code when creating or modifying utility functions, helpers, services, and shared logic.

The goal is **not** to blindly apply rules or over-engineer simple code.

Prioritize:

1. Readability
2. Maintainability
3. Simplicity
4. Testability
5. Clear responsibility
6. Appropriate abstraction

Do not refactor code merely to make it shorter.

---

## Core Rule

Whenever you create or modify a utility, perform a clean-code review before considering the task complete.

Ask:

* Is this function doing one clear thing?
* Is the function easy to understand without extensive comments?
* Is the nesting unnecessarily deep?
* Is the control flow difficult to follow?
* Is there duplicated logic?
* Is there a simpler data structure or pattern?
* Is the abstraction justified?
* Can this code be tested independently?
* Are names descriptive?
* Is the utility doing more than its name suggests?

---

# 1. Function Size

Avoid excessively long functions.

### Warning signs

Consider refactoring when a function:

* Exceeds roughly **30–40 lines**
* Has multiple distinct responsibilities
* Requires scrolling to understand
* Contains multiple levels of unrelated logic
* Has several large conditional branches

Do not split a function simply because it exceeds an arbitrary number of lines.

### Bad

```ts
function calculateResult(data: Data) {
  // validation
  // normalization
  // calculations
  // formatting
  // logging
  // result transformation
}
```

### Better

```ts
function calculateResult(data: Data) {
  const validated = validateData(data);
  const normalized = normalizeData(validated);
  const result = calculate(normalized);

  return formatResult(result);
}
```

The main function should read like a high-level description of the operation.

---

# 2. Single Responsibility

A utility should have one clear responsibility.

Avoid functions that:

* Validate data
* Transform data
* Calculate values
* Format output
* Perform side effects

all in the same function.

Separate responsibilities when doing so improves clarity.

---

# 3. Deep Nesting

Avoid unnecessary nesting.

### Warning threshold

Pay particular attention when nesting reaches **3+ levels**.

### Bad

```ts
for (const category of categories) {
  for (const calculator of category.calculators) {
    if (calculator.enabled) {
      if (calculator.fields.length > 0) {
        for (const field of calculator.fields) {
          if (field.required && !field.value) {
            // logic
          }
        }
      }
    }
  }
}
```

### Better

Use early returns, extracted functions, or appropriate data transformations.

```ts
for (const category of categories) {
  for (const calculator of category.calculators) {
    processCalculator(calculator);
  }
}

function processCalculator(calculator: Calculator) {
  if (!calculator.enabled) return;
  if (!calculator.fields.length) return;

  validateRequiredFields(calculator.fields);
}
```

Do not blindly eliminate every nested loop. Nested loops are acceptable when they clearly represent the problem being solved.

---

# 4. Nested Loops

Nested loops are not automatically bad.

Evaluate whether they:

* Are necessary
* Have clear variable names
* Have excessive nesting
* Contain complicated conditional logic
* Can reasonably be replaced with a lookup structure
* Can be extracted into a dedicated function

### Acceptable

```ts
for (const user of users) {
  for (const permission of user.permissions) {
    checkPermission(user, permission);
  }
}
```

### Needs review

```ts
for (...) {
  for (...) {
    for (...) {
      if (...) {
        if (...) {
          // complicated logic
        }
      }
    }
  }
}
```

Prefer extraction over clever one-liners.

---

# 5. Early Returns

Prefer guard clauses when they reduce nesting.

### Avoid

```ts
function process(data: Data | null) {
  if (data) {
    if (data.enabled) {
      if (data.items.length > 0) {
        return processItems(data.items);
      }
    }
  }

  return null;
}
```

### Prefer

```ts
function process(data: Data | null) {
  if (!data) return null;
  if (!data.enabled) return null;
  if (!data.items.length) return null;

  return processItems(data.items);
}
```

Use early returns when they make the happy path easier to understand.

---

# 6. Large switch Statements

Large `switch` statements should be reviewed.

### Warning signs

* More than roughly **5–7 cases**
* Repeated logic between cases
* Cases mainly map values to functions
* Adding a new case requires modifying a large block

### Prefer a lookup map when appropriate

```ts
const handlers: Record<Type, Handler> = {
  percentage: handlePercentage,
  currency: handleCurrency,
  interest: handleInterest,
};

const handler = handlers[type];

if (!handler) {
  throw new Error(`Unsupported type: ${type}`);
}

return handler(data);
```

Do not replace every `switch` with a map.

A small, readable `switch` is often better.

---

# 7. Long if/else Chains

Review long conditional chains.

### Bad

```ts
if (type === 'percentage') {
  // ...
} else if (type === 'currency') {
  // ...
} else if (type === 'interest') {
  // ...
} else if (type === 'loan') {
  // ...
} else if (type === 'mortgage') {
  // ...
}
```

Consider:

* Lookup maps
* Strategy functions
* Dedicated handlers
* Polymorphism

Use the simplest solution that improves readability.

---

# 8. Complex Conditions

Avoid complicated boolean expressions.

### Bad

```ts
if (
  user &&
  user.active &&
  user.permissions &&
  user.permissions.includes('admin') &&
  !user.suspended &&
  user.subscription?.active
) {
  // ...
}
```

Extract meaningful predicates.

```ts
const canAccessAdmin = isActiveAdmin(user) && hasActiveSubscription(user);

if (canAccessAdmin) {
  // ...
}
```

The extracted function should have a meaningful name.

---

# 9. Magic Values

Avoid unexplained magic numbers and strings.

### Bad

```ts
if (amount > 1000) {
  return amount * 0.15;
}
```

### Better

```ts
const HIGH_VALUE_THRESHOLD = 1000;
const HIGH_VALUE_RATE = 0.15;

if (amount > HIGH_VALUE_THRESHOLD) {
  return amount * HIGH_VALUE_RATE;
}
```

Do not create constants for values that are already self-explanatory.

---

# 10. Too Many Parameters

Review functions with many parameters.

### Warning sign

```ts
calculate(
  amount,
  rate,
  years,
  frequency,
  inflation,
  tax,
  fees,
  currency,
);
```

Prefer an options object when appropriate.

```ts
calculate({
  amount,
  rate,
  years,
  frequency,
  inflation,
  tax,
  fees,
  currency,
});
```

Do not introduce an object merely to avoid a small number of parameters.

---

# 11. Duplication

Avoid duplicated business logic.

### Bad

```ts
function calculateMonthlyLoan(...) {
  // same calculation
}

function calculateYearlyLoan(...) {
  // same calculation with duplicated logic
}
```

Extract genuinely shared logic.

```ts
function calculateLoan(params: LoanParams) {
  // shared calculation
}
```

Do not abstract code that only looks similar but represents different concepts.

---

# 12. Utility File Size

Avoid creating a single massive utility file.

### Bad

```text
utils.ts
├── date functions
├── currency functions
├── validation
├── calculator math
├── string helpers
├── URL helpers
├── formatting
└── unrelated business logic
```

Prefer focused modules.

```text
utils/
├── date.ts
├── currency.ts
├── validation.ts
├── calculator.ts
├── formatting.ts
└── url.ts
```

Organize utilities around meaningful responsibilities.

---

# 13. Naming

Names should communicate intent.

### Avoid

```ts
function process(data) {}
function handle(value) {}
function doThing(input) {}
function calc(x) {}
```

Prefer:

```ts
function calculateCompoundInterest(input) {}
function validateCalculatorInput(input) {}
function formatCurrency(value) {}
function normalizeCalculatorConfig(config) {}
```

Avoid unnecessary abbreviations.

---

# 14. Boolean Naming

Boolean variables and functions should read naturally.

### Prefer

```ts
isValid
isEnabled
hasPermission
canCalculate
shouldRedirect
```

Instead of:

```ts
valid
enabled
permission
calculate
redirect
```

when the meaning is ambiguous.

---

# 15. Avoid Clever Code

Do not sacrifice readability to reduce line count.

### Avoid

```ts
return items.filter(x => x.a && x.b).map(x => x.c).reduce((a, b) => a + b, 0);
```

if the transformation becomes difficult to understand.

Readable multi-step code is preferable.

```ts
const validItems = items.filter(isValidItem);
const values = validItems.map(getValue);

return sum(values);
```

---

# 16. Array Methods vs Loops

Do not automatically replace loops with `map`, `filter`, `reduce`, etc.

Use the construct that best communicates intent.

### Good

```ts
const activeUsers = users.filter(isActiveUser);
```

### Also good

```ts
for (const user of users) {
  if (!user.active) continue;

  processUser(user);
}
```

A loop is often preferable when:

* There are side effects
* Multiple operations occur
* Early termination is required
* The logic would become harder to understand with chained array methods

---

# 17. Mutation

Prefer immutable operations where practical.

### Avoid unnecessary mutation

```ts
const result = data;

result.items.push(item);
result.total += value;
```

Prefer:

```ts
const result = {
  ...data,
  items: [...data.items, item],
  total: data.total + value,
};
```

However, do not create excessive object copies in performance-sensitive code without reason.

---

# 18. Side Effects

Utilities should ideally be predictable.

Be cautious when a function that appears to be a simple utility:

```ts
formatCurrency(value)
```

also:

* Writes to localStorage
* Makes API requests
* Mutates global state
* Logs extensively
* Changes DOM state

Separate pure transformations from side effects where practical.

---

# 19. Comments

Do not use comments to justify unnecessarily complicated code.

### Bad

```ts
// Loop through every item and check if it is valid,
// then if it is valid check if it is enabled...
```

Instead, simplify the code.

Comments should explain:

* Why something unusual is required
* Important business rules
* Non-obvious browser/runtime behavior
* Performance constraints
* External API quirks

Comments should not explain obvious syntax.

---

# 20. Error Handling

Do not silently swallow errors.

### Bad

```ts
try {
  return calculate(data);
} catch {
  return null;
}
```

unless intentionally designed behavior.

Prefer meaningful handling:

```ts
try {
  return calculate(data);
} catch (error) {
  throw new CalculationError('Failed to calculate result', {
    cause: error,
  });
}
```

Do not over-engineer error classes for simple utilities.

---

# 21. Abstraction Rule

Do not create abstractions just because code can be abstracted.

Before extracting a function, ask:

> Does this give the code a meaningful name, isolate responsibility, improve testing, or remove duplication?

If the answer is no, keep the code simple.

---

# 22. Refactoring Priority

When multiple problems exist, prioritize:

### High priority

1. Incorrect behavior
2. Excessive complexity
3. Deep nesting
4. Large multi-responsibility functions
5. Duplicated business logic
6. Hard-to-test code

### Medium priority

7. Naming problems
8. Magic values
9. Large conditional structures
10. Poor module organization

### Low priority

11. Minor stylistic improvements
12. Small line-count reductions
13. Personal preference changes

Do not perform large refactors for low-priority issues unless explicitly requested.

---

# 23. Do Not Over-Refactor

The skill must avoid turning simple code into unnecessarily complex architecture.

### Bad refactoring

```ts
calculate()
  -> CalculationService
      -> CalculationStrategyFactory
          -> CalculationStrategyRegistry
              -> CalculationStrategyResolver
                  -> CalculationHandler
```

for a simple calculation.

Prefer:

```ts
function calculate(input: Input) {
  // clear calculation
}
```

Clean code means **simple code when simple code is enough**.

---

# 24. Before/After Review

Whenever modifying a complex utility, review the final implementation.

Check:

```text
[ ] Function has a clear responsibility
[ ] Function is not unnecessarily long
[ ] Nesting is reasonable
[ ] No unnecessary nested loops
[ ] No excessive if/else chains
[ ] Switch statements are appropriately sized
[ ] Complex conditions have meaningful names
[ ] No unnecessary duplication
[ ] No unexplained magic values
[ ] Parameters are manageable
[ ] Names communicate intent
[ ] Abstractions are justified
[ ] Side effects are controlled
[ ] Error handling is intentional
[ ] Code is easy to test
[ ] No unnecessary cleverness
[ ] No over-engineering
```

---

# 25. Final Rule

When Claude creates or modifies utility code:

**First make it work. Then make it clear. Then remove unnecessary complexity.**

Do not optimize for the fewest lines.

Optimize for the code being easy for another developer to understand, modify, test, and safely extend.

The best refactoring is the one that makes the next change easier without making the current code harder to understand.
