# Code Reviewer Skills Guide

> **Skills and competencies required to effectively execute the [Code Review Checklist](CODE-REVIEW-CHECKLIST.md).**
> Each skill maps to a checklist category and describes what a reviewer needs to *know* and *do* to provide high-quality feedback.

---

## 🗺️ Skills Map

| # | Skill | Checklist Category | Difficulty |
|---|-------|--------------------|------------|
| 1 | [Spec Verification](#1-spec-verification) | Functionality | ⭐⭐ |
| 2 | [Architectural Reasoning](#2-architectural-reasoning) | Architecture & Patterns | ⭐⭐⭐ |
| 3 | [Abstraction Analysis](#3-abstraction-analysis) | Abstraction & Structure | ⭐⭐⭐ |
| 4 | [Code Readability Assessment](#4-code-readability-assessment) | Readability & Formatting | ⭐⭐ |
| 5 | [Dependency Evaluation](#5-dependency-evaluation) | Dependencies | ⭐⭐ |
| 6 | [Design Principle Application](#6-design-principle-application) | SOLID / DRY / KISS / YAGNI | ⭐⭐⭐ |
| 7 | [Logging Review](#7-logging-review) | Logging | ⭐⭐ |
| 8 | [Null Safety Reasoning](#8-null-safety-reasoning) | Null Safety | ⭐⭐ |
| 9 | [Boundary Condition Thinking](#9-boundary-condition-thinking) | Boundary Conditions | ⭐⭐⭐ |
| 10 | [Test Quality Assessment](#10-test-quality-assessment) | Unit Tests | ⭐⭐ |
| 11 | [Checkstyle & Static Analysis Literacy](#11-checkstyle--static-analysis-literacy) | Java Checkstyle Rules | ⭐ |
| 12 | [MCP Tooling Proficiency](#12-mcp-tooling-proficiency) | Using Checklist with MCP | ⭐⭐ |

---

## 1. Spec Verification

> **Ability to confirm that code does what the specification says — no more, no less.**

### What You Need to Know
- How to read and interpret acceptance criteria from Jira / issue trackers
- The difference between happy-path, error-path, and edge-case scenarios
- What constitutes over-engineering vs. fulfilling requirements

### What You Need to Do
- Cross-reference every acceptance criterion against the PR diff
- Verify that all described scenarios (happy path + error paths) are covered
- Confirm backward compatibility is preserved or breaking changes are intentional and documented
- Flag side effects that are not obvious or documented

### How to Practice
- Pick a closed PR and its linked ticket — read the acceptance criteria first, then review the diff. Did the code satisfy every criterion?
- Deliberately look for what's *missing* rather than what's wrong with what's *there*.

### MCP Accelerator
Use Rovo → `getJiraIssue` to pull acceptance criteria and compare against the PR diff:
```
"Show me the acceptance criteria for PROJ-123 and compare with the changes in PR #42"
```

---

## 2. Architectural Reasoning

> **Ability to evaluate whether code is consistent with established patterns and standards.**

### What You Need to Know
- The project's established patterns: Executors, Spring Data repositories, converters, mappers
- Dependency injection best practices (constructor injection preferred)
- Transaction boundary management
- Async patterns used by the team (`@Async`, `CompletableFuture`, Executors)

### What You Need to Do
- Check that the PR follows existing patterns instead of reinventing solutions
- Verify proper use of dependency injection
- Confirm transaction boundaries are correct and intentional
- Ask: *"We already have a pattern for this — why not follow it?"*
- Ask: *"Is this a deliberate deviation, or did you miss the existing approach?"*

### How to Practice
- Study 3–5 well-established features in the codebase — document the patterns they use.
- When reviewing a new feature, explicitly compare it against those patterns.

### MCP Accelerator
Use GitHub → `search_code` to find existing patterns:
```
"Search for how we use ExecutorService in the codebase"
```

---

## 3. Abstraction Analysis

> **Ability to assess method decomposition, layering, and package structure.**

### What You Need to Know
- Single-level-of-abstraction principle
- How to identify methods that do too many things
- Package/layer responsibilities: controller → service → repository → domain
- DTO vs. domain model boundaries

### What You Need to Do
- Flag methods that mix multiple levels of abstraction
- Verify each method does **one thing** and its name describes *what*, not *how*
- Check for layer violations (e.g., controller calling repository directly)
- Ensure DTOs don't leak into domain layers and vice versa
- Suggest extracting methods when a code block needs a comment to explain it

### Recognizing the Smell

❌ **One method doing everything:**
```java
public void processOrder(Order order) {
    // validate → calculate → save → notify all in one place
}
```

✅ **Decomposed into semantic methods:**
```java
public void processOrder(Order order) {
    validateOrder(order);
    BigDecimal total = calculateTotal(order);
    persistOrder(order, total);
    notifyCustomer(order, total);
}
```

### How to Practice
- Review any method longer than ~20 lines and try to split it into named sub-methods.
- Draw a quick layer diagram of a feature and check that call flow only goes downward.

---

## 4. Code Readability Assessment

> **Ability to judge whether another developer can understand the code quickly.**

### What You Need to Know
- Vertical spacing conventions (blank lines between logical blocks)
- Semantic line allocation (one concept per line)
- Naming conventions that reveal intent, not implementation
- When comments are valuable (explain *why*) vs. noise (explain *what*)

### What You Need to Do
- Check that variable and method names reveal intent
- Flag magic numbers / strings — they should be constants with meaningful names
- Ensure no dead code, commented-out blocks, or TODOs without ticket references
- Verify method chains are broken at logical points
- Confirm complex conditions are extracted into well-named booleans or methods

### Recognizing the Smell

❌ **Dense and hard to scan:**
```java
return items.stream().filter(i -> i.getStatus() == Status.ACTIVE && i.getPrice().compareTo(minPrice) > 0).map(i -> new ItemDto(i.getId(), i.getName(), i.getPrice())).collect(Collectors.toList());
```

✅ **One concept per line:**
```java
return items.stream()
    .filter(item -> item.isActive())
    .filter(item -> item.isPricedAbove(minPrice))
    .map(ItemDto::fromEntity)
    .collect(Collectors.toList());
```

### How to Practice
- Read a file you've never seen for 60 seconds. Note every place you got confused — those are readability issues.

---

## 5. Dependency Evaluation

> **Ability to judge whether a new dependency is justified, safe, and maintainable.**

### What You Need to Know
- Common open-source licenses and their compatibility (MIT ✅, Apache 2.0 ✅, GPL ⚠️)
- How to inspect transitive dependency trees (`mvn dependency:tree`)
- How to check for known CVEs in dependencies
- Indicators of a healthy vs. abandoned library

### What You Need to Do
- Ask: *Can this be done with existing libraries or JDK APIs?*
- Verify licensing compatibility
- Check maintenance status (last release date, issue activity)
- Evaluate the transitive dependency footprint
- Run vulnerability scanners on new dependencies

### Red Flags
- Adding a library for a single utility method
- Libraries with no releases in 2+ years
- Copyleft licenses (GPL, AGPL) in proprietary projects

### MCP Accelerator
Use GitHub → `get_file_contents` on `pom.xml` / `build.gradle` to inspect dependency changes.

---

## 6. Design Principle Application

> **Ability to recognize violations of SOLID, DRY, KISS, and YAGNI in real code.**

### What You Need to Know

| Principle | Key Question |
|-----------|-------------|
| **S** — Single Responsibility | Does this class/method have more than one reason to change? |
| **O** — Open/Closed | Is this extended via abstraction, or by modifying existing code? |
| **L** — Liskov Substitution | Can subtypes be used wherever the base type is expected? |
| **I** — Interface Segregation | Are implementations forced to provide methods they don't use? |
| **D** — Dependency Inversion | Does this depend on concretions instead of abstractions? |
| **DRY** | Is there duplicated logic that could be extracted? |
| **KISS** | Is there unnecessary complexity or clever tricks? |
| **YAGNI** | Is there speculative code for requirements that don't exist yet? |

### What You Need to Do
- Flag classes/methods with multiple responsibilities
- Identify duplicated logic across the PR (or between the PR and existing code)
- Question any abstraction or extension point that doesn't serve a current requirement
- Challenge complexity — prefer the simplest solution that works

### How to Practice
- For each principle, find one example of a violation and one example of good adherence in your codebase.

---

## 7. Logging Review

> **Ability to evaluate whether logging provides enough context without leaking sensitive data.**

### What You Need to Know

| Level | Use For |
|-------|---------|
| `ERROR` | Something failed that needs attention |
| `WARN` | Unexpected but recoverable situations |
| `INFO` | Key business events (startup, shutdown, major operations) |
| `DEBUG` | Detailed diagnostic info for troubleshooting |

### What You Need to Do
- Verify key operations are logged at appropriate levels
- Confirm log messages include **context** (IDs, operation name, relevant parameters)
- Check that **no secrets** appear in logs: passwords, tokens, API keys, PII
- Ensure exceptions are logged with the exception object (not just `e.getMessage()`).
- Look for structured logging format where applicable

### Recognizing the Smell

❌ `log.error("Failed: " + e.getMessage());` — loses the stack trace
❌ `log.debug("User token: " + user.getToken());` — **secret leak!**
✅ `log.error("Failed to process order [orderId={}]", orderId, e);`

---

## 8. Null Safety Reasoning

> **Ability to identify null-related risks and evaluate defensive coding strategies.**

### What You Need to Know
- `Optional` is for return types, **not** method parameters or fields
- Collections should never be null — return empty collections instead
- Null checks should happen **early** (fail-fast at boundaries)
- `@NonNull` / `@Nullable` annotations communicate intent

### What You Need to Do
- Check that `Optional` is used for return types that may have no value
- Verify inputs are validated at boundaries (public API methods, controller endpoints)
- Confirm `Objects.requireNonNull()` is used for constructor parameters with meaningful messages
- Ensure null checks happen early, not deep in the logic

### Recognizing the Smell

❌ **No null protection:**
```java
return user.getFirstName() + " " + user.getLastName();  // NPE if null
```

✅ **Fail-fast + safe defaults:**
```java
Objects.requireNonNull(user, "User must not be null");
String firstName = Optional.ofNullable(user.getFirstName()).orElse("");
String lastName = Optional.ofNullable(user.getLastName()).orElse("");
return String.format("%s %s", firstName, lastName).trim();
```

---

## 9. Boundary Condition Thinking

> **Ability to mentally explore the edges of input/output space and anticipate failures.**

### What You Need to Know
- The common boundary categories:
  - Empty collections and empty strings
  - Zero, negative, and overflow values
  - Maximum size / length constraints
  - Concurrent access and race conditions
  - Timeout scenarios for external calls
  - Unicode / special characters
  - First and last elements in ordered data
  - Pagination boundaries (first page, last page, beyond last page)
  - Date/time edge cases (midnight, DST transitions, timezones)

### What You Need to Do
- For every input parameter, mentally ask: *"What happens at the extremes?"*
- Check if the code handles empty, null, zero, negative, max, and concurrent scenarios
- Verify timeout handling for external calls
- Confirm pagination logic handles first, last, and out-of-range pages

### How to Practice
- Pick any method and list every possible boundary input. Then check if each is handled. This becomes instinctive with repetition.

---

## 10. Test Quality Assessment

> **Ability to evaluate whether tests are structured, meaningful, and maintainable.**

### What You Need to Know
- The **given / when / then** pattern (Arrange / Act / Assert)
- Test naming conventions: `should_<result>_when_<condition>()`
- One logical assertion per test
- Tests must be independent — no shared mutable state

### What You Need to Do
- Verify every test follows given → when → then structure
- Check test method names describe the scenario being tested
- Confirm edge cases are tested (nulls, empty, boundary values)
- Ensure mocks are used for external dependencies, **not** for the class under test
- Flag tests with multiple unrelated assertions

### Example of a Well-Structured Test

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
```

### MCP Accelerator
Use GitHub → `pull_request_read` (get_files) to check whether test files are included in the PR.

---

## 11. Checkstyle & Static Analysis Literacy

> **Ability to understand and interpret Java Checkstyle rules and static analysis output.**

### What You Need to Know
- Naming conventions for packages, types, methods, constants, locals
- Import rules: no wildcards, no unused, no redundant
- Formatting rules: max line length, no tabs, no trailing spaces
- Brace and block style rules
- Complexity limits: method length ≤ 200, parameters ≤ 16, cyclomatic complexity ≤ 50, nested if ≤ 3
- Correctness rules: `equals`/`hashCode` contract, no `==` on strings, no parameter reassignment

### What You Need to Do
- Verify the PR doesn't introduce Checkstyle violations
- Review any `@SuppressWarnings` usage — it must be justified and scoped correctly
- Confirm TODO comments are tracked with ticket references
- Check that type-level Javadocs are present

### Note
Most of these are caught automatically by Checkstyle/PMD. Your job as a reviewer is to:
1. Ensure the tools are actually running in CI
2. Verify suppressions are justified
3. Catch the edge cases tools miss (e.g., *technically* valid but *semantically* wrong naming)

---

## 12. MCP Tooling Proficiency

> **Ability to leverage MCP-powered tools to accelerate and enrich code reviews.**

### What You Need to Know
- How to configure and connect MCP servers (GitHub, Rovo)
- Available MCP tools and when to use them
- How to craft effective prompts for MCP-powered reviews

### Tool Quick Reference

| Review Task | MCP Tool |
|-------------|----------|
| Verify functionality against spec | Rovo → `getJiraIssue` |
| Check pattern consistency | GitHub → `search_code` |
| Inspect dependency changes | GitHub → `get_file_contents` on `pom.xml` / `build.gradle` |
| Verify test coverage in PR | GitHub → `pull_request_read` (get_files) |
| Check CI/CD status | GitHub → `get_check_runs` and `get_job_logs` |
| Search for related issues | Rovo → `searchJiraIssuesUsingJql` |
| Find design documentation | Rovo → `searchAtlassian` |

### Best Practices
1. **Be specific** — mention repo names, PR numbers, and issue keys
2. **Combine tools** — cross-reference Jira issues with PR changes
3. **Leverage CI logs** — ask for logs instead of manually digging through pipelines
4. **Use the checklist** — let the checklist guide *what* to review; let MCP handle *how* to gather context

---

## 📈 Skill Development Path

### Level 1 — Foundational (Start Here)
- [ ] Spec Verification (Skill 1)
- [ ] Code Readability Assessment (Skill 4)
- [ ] Logging Review (Skill 7)
- [ ] Checkstyle Literacy (Skill 11)

### Level 2 — Intermediate
- [ ] Dependency Evaluation (Skill 5)
- [ ] Null Safety Reasoning (Skill 8)
- [ ] Test Quality Assessment (Skill 10)
- [ ] MCP Tooling Proficiency (Skill 12)

### Level 3 — Advanced
- [ ] Architectural Reasoning (Skill 2)
- [ ] Abstraction Analysis (Skill 3)
- [ ] Design Principle Application (Skill 6)
- [ ] Boundary Condition Thinking (Skill 9)

---

## 🎯 How to Use This Guide

1. **Self-assess** — rate yourself on each skill (1–5) and identify gaps
2. **Focus on one level at a time** — master Level 1 before moving to Level 2
3. **Practice during reviews** — pick one skill per review to deliberately practice
4. **Pair review** — review with a more experienced developer and compare notes
5. **Use the [Code Review Checklist](CODE-REVIEW-CHECKLIST.md)** — the checklist tells you *what* to check; this guide tells you *how* to build the skill

---

**Build the skills. Trust the tooling. Deliver high-quality reviews. 🧠**