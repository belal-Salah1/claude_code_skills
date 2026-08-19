---
name: clean-code
version: 3.0.0
description: '[Code Quality] Use when writing, reviewing, or refactoring code — utilities, helpers, services, components, shared logic. Covers long functions, deep nesting, switch/if-else chains, complex conditions, magic values, duplication, naming, side effects, error handling, over-abstraction, and behavior-preserving refactoring (extract / move / inline / simplify) with evidence, safety, and verification gates.'
---

# Clean Code

## Purpose

Enforce clean, maintainable, readable code when writing or modifying code, and
restructure existing code without changing its behavior.

Two modes, one standard:

- **Writing mode** — new or modified code must pass the quality rules in Part A.
- **Refactoring mode** — structural change with **no behavior change**, run
  through the workflow and gates in Part B.

Prioritize:

1. Readability
2. Maintainability
3. Simplicity
4. Testability
5. Clear responsibility
6. Appropriate abstraction

The goal is **not** to blindly apply rules or over-engineer simple code.
Do not refactor code merely to make it shorter.

---

## Quick Summary

**Writing:** produce the code → run the Part A review → fix high-priority
findings → run the Before/After checklist.

**Refactoring:**

1. **Analysis** — identify target, map every usage with grep, assess impact, verify test coverage
2. **Plan** — document refactoring type, changes, and risks
3. **Execute** — apply the change (extract method/class, move to the owning layer, simplify conditionals)
4. **Verify** — run tests, confirm no behavior change, check it compiles/type-checks

**Key rules:**

- Establish test coverage first, then refactor — never refactor code with no
  existing tests. Tests are the only proof the refactor preserved behavior.
- Make small incremental changes; never mix refactoring with feature work.
- Place logic in the lowest appropriate layer (Entity/Model > Service > Component).

---

## First Principle — Easy to Change

> **The success metric of every coding decision is _future change cost_.**
> DRY, SRP, abstraction, design patterns, naming, layering, tests — every
> technique exists to serve one goal: **making the next change cheaper**.

When evaluating code, a refactor, a test, or an abstraction, ask:
**does this make the next change cheaper or more expensive?**

- Reject "best practices" that raise change cost (premature abstraction,
  speculative generality, leaky indirection, ceremony without payoff).
- Name the real enemies in findings: **coupling, hidden state, duplicated
  knowledge, unclear intent, irreversible decisions exposed too early**.
- A simpler design that is easy to change beats a sophisticated design that isn't.

Apply this lens **before** invoking any specific rule, pattern, or checklist
below — if a downstream rule would raise change cost, this principle wins.

---

## Core Rule

Whenever you create or modify code, perform a clean-code review before
considering the task complete.

Ask:

- Is this function doing one clear thing?
- Is it easy to understand without extensive comments?
- Is the nesting unnecessarily deep?
- Is the control flow difficult to follow?
- Is there duplicated logic?
- Is there a simpler data structure or pattern?
- Is the abstraction justified?
- Can this code be tested independently?
- Are names descriptive?
- Is it doing more than its name suggests?

---

# Ground Rules (non-negotiable)

## Understand code first

**HARD GATE: do not write, plan, or fix until you have read the existing code.**

1. Search for 3+ similar patterns (grep/glob) — cite `file:line` evidence.
2. Read the existing files in the target area — understand structure, base
   classes, and conventions.
3. Map dependencies — know what depends on your target before you touch it.
4. Read the project's own conventions first (`CLAUDE.md`, `AGENTS.md`, `docs/`,
   lint/format config). **Project conventions override generic defaults,
   including the defaults in this skill.**
5. For non-trivial work (3+ files), write the investigation to a notes file and
   re-read it before implementing — never work from memory alone. Long context
   drifts from the files; the files are ground truth.
6. NEVER invent a new pattern when an existing one works — match it exactly, or
   document why you deviated. Divergent patterns fragment the codebase and slow
   every future reader.

**Blocked until:** target files read · 3+ patterns grepped · assumptions
verified with evidence.

## Investigation mindset

Be skeptical. Every claim needs traced proof.

- Verify any "unused" code with grep across the whole repo before touching it —
  do NOT assume it is dead. Dynamic dispatch, reflection, string-keyed
  registries, templates, and cross-service callers don't surface in a casual read.
- Every recommendation includes `file:line` evidence.
- If you cannot prove a code path is safe to change, say
  "unverified, needs investigation".
- Question assumptions: "Is this really dead code?" → trace all usages.
- Challenge completeness: "Have I checked every caller / every package?"
- No "should be refactored" without proof — demonstrate the improvement.

## Evidence & confidence gate

Declare `Confidence: X%` with an evidence list and `file:line` proof for every
non-obvious claim.

| Confidence | Action |
| --- | --- |
| 95%+ | Recommend freely |
| 80–94% | Recommend with caveats |
| 60–79% | List the unknowns |
| <60% | **STOP — gather more evidence** |

Breaking changes (removing an exported symbol, changing a public interface,
altering a shared contract) require **95%+ confidence** with a full trace of
every consumer.

Forbidden without proof: "obviously", "I think", "should be", "probably".
If evidence is incomplete, output:
`Insufficient evidence. Verified: [...]. Not verified: [...].`

## Assume existing values are intentional

Before changing **or flagging** a constant, limit, flag, cutoff, wording, or
pattern, read the nearby context and history, the caller's ordering, and 2+
sibling call sites of the same convention. A comment stating WHAT without WHY is
missing rationale, not proof of a missing guard. Ask WHY before changing.

## AI mistake prevention

- **Re-read files after context loss.** Compaction, resume, or long-running work
  makes memory stale; verify current file contents before acting.
- **Verify generated content against source.** APIs, names, and claims get
  hallucinated — check the source before documenting or referencing it.
- **Check downstream references before deleting or renaming.** Removing an
  artifact can stale docs, generated files, configs, and callers — map the
  references first.
- **Trace the full impact chain after edits.** Changing a definition can miss
  derived outputs and consumers.
- **Verify ALL affected outputs, not just the first.** One green check is not
  all green checks.
- **New artifact = wired artifact.** Created something? Prove it is exported,
  imported, registered, and reachable by every consumer.
- **Surface ambiguity before acting.** Multiple valid readings require an
  explicit question, or a stated assumption with its risk.

## Task planning

Break the work into small todos **before** starting, including a todo per file
to read for long files — this prevents context loss. Add a final review todo to
verify quality. For genuinely simple tasks, skip the ceremony. When running as
one step of a larger workflow, still create the per-phase todos for this skill's
own work, keep exactly one in progress, and complete each only after its
evidence is written.

---

# Part A — Code Quality Rules

## 1. Function size

Avoid excessively long functions. Consider refactoring when a function:

- Exceeds roughly **30–40 lines**
- Has multiple distinct responsibilities
- Requires scrolling to understand
- Contains multiple levels of unrelated logic
- Has several large conditional branches

Do not split a function simply because it exceeds an arbitrary line count.

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

## 2. Single responsibility

A unit of code should have one clear responsibility.

Avoid functions that validate, transform, calculate, format, **and** perform
side effects all at once. Separate responsibilities when doing so improves
clarity.

## 3. Deep nesting

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

Do not blindly eliminate every nested loop. Nested loops are acceptable when
they clearly represent the problem being solved.

## 4. Nested loops

Nested loops are not automatically bad. Evaluate whether they:

- Are necessary
- Have clear variable names
- Have excessive nesting
- Contain complicated conditional logic
- Can reasonably be replaced with a lookup structure
- Can be extracted into a dedicated function

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

## 5. Early returns

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

## 6. Large switch statements

Review a `switch` when it has:

- More than roughly **5–7 cases**
- Repeated logic between cases
- Cases that mainly map values to functions
- A large block that must be edited to add a case

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

Do not replace every `switch` with a map. A small, readable `switch` is often better.

## 7. Long if/else chains

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

Consider lookup maps, strategy functions, dedicated handlers, or polymorphism.
Use the simplest solution that improves readability.

## 8. Complex conditions

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

The extracted function must have a meaningful name.

## 9. Magic values

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

## 10. Too many parameters

### Warning sign

```ts
calculate(amount, rate, years, frequency, inflation, tax, fees, currency);
```

Prefer an options object when appropriate.

```ts
calculate({ amount, rate, years, frequency, inflation, tax, fees, currency });
```

Do not introduce an object merely to avoid a small number of parameters.

## 11. Duplication

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

## 12. File size and module organization

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

Organize around meaningful responsibilities.

## 13. Naming

### Avoid

```ts
function process(data) {}
function handle(value) {}
function doThing(input) {}
function calc(x) {}
```

### Prefer

```ts
function calculateCompoundInterest(input) {}
function validateCalculatorInput(input) {}
function formatCurrency(value) {}
function normalizeCalculatorConfig(config) {}
```

Avoid unnecessary abbreviations.

## 14. Boolean naming

### Prefer

```ts
isValid
isEnabled
hasPermission
canCalculate
shouldRedirect
```

Instead of `valid`, `enabled`, `permission`, `calculate`, `redirect` when the
meaning is ambiguous.

## 15. Avoid clever code

### Avoid

```ts
return items.filter(x => x.a && x.b).map(x => x.c).reduce((a, b) => a + b, 0);
```

when the transformation becomes difficult to understand. Readable multi-step
code is preferable.

```ts
const validItems = items.filter(isValidItem);
const values = validItems.map(getValue);

return sum(values);
```

## 16. Array methods vs loops

Do not automatically replace loops with `map` / `filter` / `reduce`. Use the
construct that best communicates intent.

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

A loop is often preferable when there are side effects, multiple operations,
early termination, or the logic would get harder to follow when chained.

## 17. Mutation

Prefer immutable operations where practical.

### Avoid unnecessary mutation

```ts
const result = data;

result.items.push(item);
result.total += value;
```

### Prefer

```ts
const result = {
  ...data,
  items: [...data.items, item],
  total: data.total + value,
};
```

Do not create excessive object copies in performance-sensitive code without reason.

## 18. Side effects

Utilities should be predictable. Be cautious when something that looks like a
simple transformation:

```ts
formatCurrency(value)
```

also writes to storage, makes requests, mutates global state, logs extensively,
or changes DOM state. Separate pure transformations from side effects where
practical.

## 19. Comments

Do not use comments to justify unnecessarily complicated code.

### Bad

```ts
// Loop through every item and check if it is valid,
// then if it is valid check if it is enabled...
```

Simplify the code instead. Comments should explain:

- Why something unusual is required
- Important business rules
- Non-obvious browser/runtime behavior
- Performance constraints
- External API quirks

Comments should not explain obvious syntax.

## 20. Error handling

Do not silently swallow errors.

### Bad

```ts
try {
  return calculate(data);
} catch {
  return null;
}
```

unless that is intentionally designed behavior.

### Prefer meaningful handling

```ts
try {
  return calculate(data);
} catch (error) {
  throw new CalculationError('Failed to calculate result', { cause: error });
}
```

Do not over-engineer error classes for simple utilities.

## 21. Abstraction rule

Do not create an abstraction just because code *can* be abstracted.

Before extracting, ask:

> Does this give the code a meaningful name, isolate a responsibility, improve
> testing, or remove duplication?

If no, keep the code simple. **YAGNI gate: do not extract a shared abstraction
until 3+ real occurrences exist.** Never extract for hypothetical future use.

## 22. Priority order

When multiple problems exist, prioritize:

**High**

1. Incorrect behavior
2. Excessive complexity
3. Deep nesting
4. Large multi-responsibility functions
5. Duplicated business logic
6. Hard-to-test code

**Medium**

7. Naming problems
8. Magic values
9. Large conditional structures
10. Poor module organization

**Low**

11. Minor stylistic improvements
12. Small line-count reductions
13. Personal preference changes

Do not perform large refactors for low-priority issues unless explicitly requested.

## 23. Do not over-refactor

### Bad refactoring

```ts
calculate()
  -> CalculationService
      -> CalculationStrategyFactory
          -> CalculationStrategyRegistry
              -> CalculationStrategyResolver
                  -> CalculationHandler
```

for a simple calculation. Prefer:

```ts
function calculate(input: Input) {
  // clear calculation
}
```

Clean code means **simple code when simple code is enough**.

---

# Part B — Refactoring

Structural change that improves quality **without modifying behavior**.

## Refactoring catalog

### Extract patterns

| Pattern | When to use | Example |
| --- | --- | --- |
| **Extract Method** | Long method, duplicated code | Move logic to a private/local function |
| **Extract Class/Module** | Unit has multiple responsibilities | Create a Helper, Service, or Strategy |
| **Extract Interface/Type** | Need an abstraction for testing or DI | Define the contract the callers depend on |
| **Extract Expression** | Complex inline expression | Move to a named predicate on the model |
| **Extract Validator** | Repeated validation logic | Create a shared validator |

### Move patterns

| Pattern | When to use | Example |
| --- | --- | --- |
| **Move Method** | Method belongs to a different unit | Move from handler to helper/model |
| **Move to Data Layer** | Reusable query logic | Named query on the repository |
| **Move to DTO/Type** | Mapping logic in a handler | The type owns its own conversion |
| **Move to Model/Entity** | Business logic in a handler | Add a method or named predicate |

### Simplify patterns

| Pattern | When to use | Example |
| --- | --- | --- |
| **Inline Variable** | Temporary used once | Remove the intermediate |
| **Inline Method** | Body is obvious | Replace the call with the body |
| **Replace Conditional** | Complex if/switch | Strategy map or named predicate |
| **Introduce Parameter Object** | Many parameters | Options / Command / Query object |

## Workflow

### Phase 1 — Analysis

1. **Identify target** — locate the code to refactor.
2. **Map dependencies** — find every usage with grep, including dynamic and
   string-keyed callers.
3. **Assess impact** — list affected files and tests.
4. **Verify tests** — confirm coverage exists. No coverage → write tests first
   or stop.
5. **External memory** — for non-trivial work, write the analysis to a notes
   file and re-read it before planning.

### Phase 2 — Plan

```markdown
## Refactoring Plan

**Target**: [file:line]
**Type**: [Extract Method | Move to Data Layer | ...]
**Reason**: [why this improves the code / lowers change cost]

### Changes

1. [ ] Create/modify [file]
2. [ ] Update usages in [files]
3. [ ] Run tests

### Risks

- [Potential issues]
```

### Phase 3 — Execute

Small, incremental steps. One refactoring at a time. Never bundle a feature change.

### Phase 4 — Verify

1. Run the affected tests.
2. Verify no behavior change.
3. Confirm it compiles / type-checks / lints.
4. Review for consistency with surrounding code.

## Layer-down refactorings

One principle, several shapes: **push logic down to the layer that owns the
data or the concern**, out of the orchestration layer (handler, controller,
use-case, component). Reused logic goes to a shared collaborator, query logic to
the data-access layer, mapping to the type being mapped.

Examples are illustrative — translate the shape to your stack and match the
project's existing primitives.

### Business rule to the model that owns it

```ts
// BEFORE: rule inlined in the handler
const isValid = order.status === 'active' && order.user?.isActive && !order.isDeleted;
if (!isValid) throw new Error('Order not active');

// AFTER: the rule lives with the thing it describes
// order.ts
export const isActiveOrder = (o: Order) =>
  o.status === 'active' && !!o.user?.isActive && !o.isDeleted;

// handler
const order = await orders.findOne(id);
if (!isActiveOrder(order)) throw new NotFoundError('Order not active');
```

### Reused logic to a shared collaborator

```ts
// BEFORE: the same get-or-create dance in several handlers
const order =
  (await orders.findOne({ userId, customerId })) ??
  (await createOrder({ userId, customerId }));

// AFTER: one owner for the rule
// orders/getOrCreateOrder.ts
export async function getOrCreateOrder({ userId, customerId }: OrderKey) {
  return (
    (await orders.findOne({ userId, customerId })) ??
    (await createOrder({ userId, customerId }))
  );
}
```

### Query logic to the data-access layer

```ts
// BEFORE: query shape spelled out in the handler
const rows = await db.orders.findMany({
  where: { customerId, status: 'active', warehouseIds: { has: warehouseId } },
});

// AFTER: a named query owned by the repository
// orders/ordersRepository.ts
export function findActiveByWarehouse(customerId: string, warehouseId: string) {
  return db.orders.findMany({
    where: { customerId, status: 'active', warehouseIds: { has: warehouseId } },
  });
}
```

### Mapping to the type that owns it

```ts
// BEFORE: mapping inline in the handler
const config = { clientId: dto.clientId, secret: encrypt(dto.secret) };

// AFTER: the DTO owns its shape; the handler owns the side effect
// authConfigDto.ts
export const toAuthConfig = (dto: AuthConfigDto): AuthConfig => ({
  clientId: dto.clientId,
  secret: dto.secret,
});

// handler
const config = { ...toAuthConfig(dto), secret: encrypt(dto.secret) };
```

## Design-pattern quality passes

Scan **one quality dimension at a time**, in serial passes — split attention
misses violations that single-focus passes catch.

1. **Pick the applicable dimensions** from what the code actually does — DRY,
   SRP/OCP/LSP/ISP/DIP, cohesion vs coupling, Law of Demeter, language idioms,
   command/query separation. The list is not fixed.
2. **One focused pass per dimension.** Do not mix concerns across passes.
3. **Right responsibility** — logic in the LOWEST appropriate layer
   (Entity/Model > Domain Service > Application Service > Controller/Component).
   Never business logic in a controller or a template.
4. **DRY via the idiomatic abstraction** for the language — base class, mixin,
   trait, composable, decorator, higher-order function. 3+ similar occurrences →
   extract. Fewer → leave it.
5. **After every extraction, move, or rename** — grep the entire scope for
   dangling references. Zero tolerance.
6. **2+ violations of the same kind** = one structural finding ("pattern
   problem"), not a list of instances.

Anti-patterns to flag by name: God Object, copy-paste inheritance, circular
dependency, leaky abstraction.

## Data-access impact check

When extracting a predicate or moving a query, the data-access shape moves with it:

- [ ] List/collection queries paginated — no unbounded `findMany` / `all()` /
      `ToList()` without limit+offset or a cursor?
- [ ] Every filter field, foreign key, and sort column indexed?
- [ ] Moved queries still hit indexed fields?
- [ ] Refactored filters keep the index's selectivity order?
- [ ] No N+1 introduced by moving a query into a loop or a per-item helper?

## Safety checklist

Before any refactoring:

- [ ] Searched every usage across the whole repo (static + dynamic + reflection + templates)?
- [ ] Test coverage exists?
- [ ] Work broken into todos?
- [ ] Changes are incremental?
- [ ] No behavior change verified?
- [ ] `Confidence: X%` declared with an evidence list?

**If ANY item is incomplete → STOP. State "Insufficient evidence to proceed."**

## Anti-patterns

- **Big-bang refactoring** — make small, incremental changes.
- **Refactoring without tests** — establish coverage first.
- **Mixing refactoring with features** — do one or the other.
- **Breaking public APIs** — maintain backward compatibility, or trace and
  update every consumer.
- **Logic in the wrong layer** — leads to duplication; move it down.
- **Renaming without grepping** — leaves dangling references and stale docs.

## Optional tooling

If the project provides a code-graph, dependency-graph, or call-graph tool, use
it to trace callers and consumers before refactoring, then verify details with
grep. Pattern: **grep finds files → graph reveals the flow → grep verifies the
details.** Flag any consumer not covered by the plan — it can break silently.

---

# Part C — Completion Gates

## Before/After review

Whenever you create or modify non-trivial code, review the final implementation:

```text
[ ] Clear single responsibility
[ ] Not unnecessarily long
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
[ ] Easy to test
[ ] No unnecessary cleverness
[ ] No over-engineering
[ ] Logic sits in the lowest appropriate layer
[ ] Queries are paginated and indexed
```

## Source/test drift

When behavior changes, inspect the affected unit / integration / E2E tests and
decide **from evidence** whether the tests should change to match intended
behavior, or the source change is an unintended bug to fix. Do not write tests
for one-off migration code.

## Integrity gate

> **Completion ≠ correctness.** Before reporting any work done, prove it:
>
> 1. **Grep every removed name.** Extraction, rename, or delete touched N files?
>    Grep confirms zero dangling references across all file types.
> 2. **Ask WHY before changing.** Existing values are intentional until proven
>    otherwise. No "fix" without traced rationale.
> 3. **Verify ALL outputs.** One build passing ≠ every build passing. Check
>    every affected package/stack.
> 4. **Evaluate pattern fit.** Copying nearby code? Verify the preconditions
>    match — same scope, lifetime, base class, constraints.
> 5. **New artifact = wired artifact.** Prove it is registered, imported, and
>    reachable by every consumer.

## Final rule

**First make it work. Then make it clear. Then remove unnecessary complexity.**

Do not optimize for the fewest lines. Optimize for code that is easy for another
developer to understand, modify, test, and safely extend.

> **Easy to Change is the success metric.** Every finding, test, refactor, and
> abstraction answers one question: _does this make the next change cheaper or
> more expensive?_ If it doesn't reduce future change cost, reject it. Coupling,
> hidden state, duplicated knowledge, and unclear intent are the real enemies —
> call them out by name.

## Related skills

- `performance-review` — when the concern is speed, not structure
- `ui-review` — component/SCSS/BEM and async-state review
