# Code Review Checklist — Manual Review Criteria

> These are items that **must be reviewed manually** — PMD and other static analysis tools won't or can't reliably enforce them.
> Use this checklist during every code review alongside MCP-powered tooling.

---

## 🔍 At a Glance

| Category | Key Question |
|----------|-------------|
| [Functionality](#1-functionality) | Does it actually do what the spec says? |
| [Architecture & Patterns](#2-architecture--patterns) | Is it consistent with our established patterns? |
| [Abstraction & Structure](#3-abstraction--structure) | Are layers, packages, and methods well-organized? |
| [Readability](#4-readability--formatting) | Can another developer understand this quickly? |
| [Dependencies](#5-dependencies) | Is every new dependency justified and approved? |
| [Design Principles](#6-design-principles-solid--dry--kiss--yagni) | Does it follow SOLID / DRY / KISS / YAGNI? |
| [Logging](#7-logging) | Is there enough context without leaking secrets? |
| [Null Safety](#8-null-safety) | Are null cases handled beyond simple NPE checks? |
| [Boundary Conditions](#9-boundary-conditions) | Are edge cases and limits properly handled? |
| [Unit Tests](#10-unit-tests) | Do tests follow given / when / then + assert? |

---

## 1. Functionality

> **Does it actually fulfill the spec?**

- [ ] The implementation matches the acceptance criteria in the ticket
- [ ] All described scenarios are covered (happy path + error paths)
- [ ] No over-engineering — only what the spec requires is built
- [ ] Side effects are documented or obvious
- [ ] Backward compatibility is preserved (or breaking changes are intentional and documented)

**MCP Tip:** Use Rovo to pull the Jira issue and cross-reference the acceptance criteria against the PR diff.

```
"Show me the acceptance criteria for PROJ-123 and compare with the changes in PR #42"
```

---

## 2. Architecture & Patterns

> **Is it consistent with higher-level patterns and standards?**

- [ ] Follows established patterns: Executors, Spring Data repositories, converters, mappers
- [ ] Uses project-standard frameworks/utilities instead of reinventing
- [ ] Consistent with how similar features are implemented elsewhere in the codebase
- [ ] Proper use of dependency injection (constructor injection preferred)
- [ ] Transaction boundaries are correct and intentional
- [ ] Async patterns (Executors, `@Async`, CompletableFuture) follow team conventions

**Questions to ask:**
- "We already have a pattern for this — why not follow it?"
- "Is this a deliberate deviation, or did you miss the existing approach?"

---

## 3. Abstraction & Structure

> **Are levels of abstraction, layering, and package structure clean?**

### Method Size & Decomposition
- [ ] No huge methods — logic is split into semantically named methods
- [ ] Each method does **one thing** at **one level of abstraction**
- [ ] Method names describe *what* is done, not *how*
- [ ] Extract method when a code block needs a comment to explain it

### Package & Layering
- [ ] Classes are in the correct package (controller / service / repository / domain)
- [ ] No layer violations (e.g., controller calling repository directly)
- [ ] DTOs don't leak into domain layers and vice versa

### Example — Before & After

❌ **Bad:** One method doing everything
```java
public void processOrder(Order order) {
    // validate
    if (order.getItems() == null || order.getItems().isEmpty()) {
        throw new ValidationException("No items");
    }
    // calculate total
    BigDecimal total = BigDecimal.ZERO;
    for (Item item : order.getItems()) {
        total = total.add(item.getPrice().multiply(BigDecimal.valueOf(item.getQuantity())));
    }
    // save
    order.setTotal(total);
    orderRepository.save(order);
    // notify
    emailService.send(order.getCustomerEmail(), "Order confirmed", total.toString());
}
```

✅ **Good:** Split into semantic methods
```java
public void processOrder(Order order) {
    validateOrder(order);
    BigDecimal total = calculateTotal(order);
    persistOrder(order, total);
    notifyCustomer(order, total);
}
```

---

## 4. Readability & Formatting

> **Can another developer understand the intent quickly?**

### Vertical Spacing
- [ ] Blank lines separate logical blocks within a method
- [ ] Related statements are grouped together (no random blank lines)
- [ ] Consistent spacing between methods and class sections

### Semantic Line Allocation
- [ ] One concept per line — don't cram multiple operations on one line
- [ ] Method chains are broken at logical points
- [ ] Complex conditions are extracted into well-named boolean variables or methods

### Intent Clarity
- [ ] Variable and method names reveal intent (not implementation)
- [ ] No magic numbers or strings — use constants with meaningful names
- [ ] Comments explain **why**, not **what** (the code should explain *what*)
- [ ] No dead code, commented-out blocks, or TODOs without ticket references

### Example — Semantic Lines

❌ **Bad:** Dense and hard to scan
```java
return items.stream().filter(i -> i.getStatus() == Status.ACTIVE && i.getPrice().compareTo(minPrice) > 0).map(i -> new ItemDto(i.getId(), i.getName(), i.getPrice())).collect(Collectors.toList());
```

✅ **Good:** One concept per line
```java
return items.stream()
    .filter(item -> item.isActive())
    .filter(item -> item.isPricedAbove(minPrice))
    .map(ItemDto::fromEntity)
    .collect(Collectors.toList());
```

---

## 5. Dependencies

> **Is every new dependency justified?**

- [ ] **Necessity:** Can this be done with existing libraries or JDK APIs?
- [ ] **Licensing:** Is the license compatible with our project? (MIT, Apache 2.0 → ✅ | GPL → ⚠️)
- [ ] **Vendor approval:** Has the library been approved by the team/org?
- [ ] **Maintenance:** Is the library actively maintained? Check last release date and issue activity
- [ ] **Size & transitives:** Does it pull in a large transitive dependency tree?
- [ ] **Security:** Any known CVEs? Check with `mvn dependency:tree` and vulnerability scanners

**Red flags:**
- Adding a library for a single utility method
- Libraries with no releases in 2+ years
- Copyleft licenses (GPL, AGPL) in proprietary projects

---

## 6. Design Principles (SOLID / DRY / KISS / YAGNI)

> **Does the code follow fundamental design principles?**

### SOLID
- [ ] **S — Single Responsibility:** Each class/method has one reason to change
- [ ] **O — Open/Closed:** Extended via abstraction, not modification of existing code
- [ ] **L — Liskov Substitution:** Subtypes are substitutable for their base types
- [ ] **I — Interface Segregation:** No fat interfaces forcing unused method implementations
- [ ] **D — Dependency Inversion:** Depend on abstractions, not concretions

### DRY (Don't Repeat Yourself)
- [ ] No duplicated logic — extract shared behavior into reusable methods/classes
- [ ] Similar code in multiple places → candidate for a shared utility or base class

### KISS (Keep It Simple, Stupid)
- [ ] Simplest solution that works — no unnecessary complexity
- [ ] Avoid clever tricks that sacrifice readability
- [ ] Prefer well-known patterns over custom abstractions

### YAGNI (You Ain't Gonna Need It)
- [ ] No speculative code for "future" requirements that don't exist yet
- [ ] No unused parameters, methods, or classes
- [ ] Configuration/extension points exist only where there's a real need

---

## 7. Logging

> **Is there enough context without leaking secrets?**

- [ ] Key operations are logged (entry, exit, decision points)
- [ ] Log levels are appropriate:
  | Level | Use For |
  |-------|---------|
  | `ERROR` | Something failed that needs attention |
  | `WARN` | Unexpected but recoverable situations |
  | `INFO` | Key business events (startup, shutdown, major operations) |
  | `DEBUG` | Detailed diagnostic info for troubleshooting |
- [ ] Log messages include **context** (IDs, operation name, relevant parameters)
- [ ] **No secrets** in logs: passwords, tokens, API keys, PII
- [ ] Structured logging format where applicable (key-value pairs)
- [ ] Exception logging includes the exception object (not just `e.getMessage()`)

### Example

❌ **Bad:**
```java
log.info("Processing...");
log.error("Failed: " + e.getMessage());
log.debug("User token: " + user.getToken());  // SECRET LEAK!
```

✅ **Good:**
```java
log.info("Processing order [orderId={}] for customer [customerId={}]", orderId, customerId);
log.error("Failed to process order [orderId={}]", orderId, e);
log.debug("User authenticated [userId={}, roles={}]", userId, roles);
```

---

## 8. Null Safety

> **Are null cases handled beyond simple NPE detection?**

- [ ] Use `Optional` for return types that may have no value
- [ ] Validate inputs at boundaries (public API methods, controller endpoints)
- [ ] Use `@NonNull` / `@Nullable` annotations to communicate intent
- [ ] Collections are never null — return empty collections instead
- [ ] Avoid `Optional` as method parameters or fields — it's for return types
- [ ] Null checks are done **early** (fail-fast) rather than deep in the logic
- [ ] `Objects.requireNonNull()` for constructor parameters with meaningful messages

### Example

❌ **Bad:**
```java
public String getDisplayName(User user) {
    return user.getFirstName() + " " + user.getLastName();  // NPE if user or names are null
}
```

✅ **Good:**
```java
public String getDisplayName(User user) {
    Objects.requireNonNull(user, "User must not be null");

    String firstName = Optional.ofNullable(user.getFirstName()).orElse("");
    String lastName = Optional.ofNullable(user.getLastName()).orElse("");

    return String.format("%s %s", firstName, lastName).trim();
}
```

---

## 9. Boundary Conditions

> **Are edge cases and limits properly handled?**

- [ ] Empty collections and empty strings
- [ ] Null inputs (see [Null Safety](#8-null-safety))
- [ ] Zero, negative, and overflow values for numeric inputs
- [ ] Maximum size / length constraints
- [ ] Concurrent access and race conditions
- [ ] Timeout scenarios for external calls
- [ ] Unicode / special characters in string processing
- [ ] First and last elements in ordered data
- [ ] Pagination boundaries (first page, last page, beyond last page)
- [ ] Date/time edge cases (midnight, DST transitions, timezone differences)

---

## 10. Unit Tests

> **Do tests follow the given / when / then + assert pattern?**

- [ ] Every test method follows the structure:
  1. **given** — set up test data and preconditions
  2. **when** — execute the action under test
  3. **then** — assert the expected outcome
- [ ] Test method names describe the scenario: `should_returnEmpty_when_noItemsFound()`
- [ ] One logical assertion per test (multiple `assertThat` for the same concept is fine)
- [ ] Tests are independent — no shared mutable state between tests
- [ ] Edge cases are tested (nulls, empty, boundary values)
- [ ] Mocks are used for external dependencies, not for the class under test

### Example

```java
@Test
void should_calculateTotal_when_orderHasMultipleItems() {
    // given
    Item laptop = new Item("Laptop", BigDecimal.valueOf(999.99), 1);
    Item mouse = new Item("Mouse", BigDecimal.valueOf(29.99), 2);
    Order order = new Order(List.of(laptop, mouse));

    // when
    BigDecimal total = orderService.calculateTotal(order);

    // then
    assertThat(total).isEqualByComparingTo(BigDecimal.valueOf(1059.97));
}

@Test
void should_throwException_when_orderHasNoItems() {
    // given
    Order order = new Order(Collections.emptyList());

    // when / then
    assertThatThrownBy(() -> orderService.calculateTotal(order))
        .isInstanceOf(ValidationException.class)
        .hasMessageContaining("No items");
}
```

---

## 🤖 Using This Checklist with MCP

During code reviews, combine this checklist with MCP-powered tools:

| Checklist Item | MCP Tool to Help |
|---------------|-----------------|
| Functionality vs spec | Rovo → `getJiraIssue` to fetch acceptance criteria |
| Pattern consistency | GitHub → `search_code` to find existing patterns |
| New dependencies | GitHub → `get_file_contents` on `pom.xml` / `build.gradle` |
| Test coverage | GitHub → `pull_request_read` (get_files) to check for test files |
| CI status | GitHub → `get_check_runs` and `get_job_logs` |

---

**Use this checklist. Trust the tooling for what it can catch. Focus your human review on what it can't. 🧠**
