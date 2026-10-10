---
name: code-quality
description: Comprehensive code review for Java - clean code principles, API contracts, null safety, exception handling, and performance. Use when user says "review code", "refactor", "check API", or before merging changes.
---

# Code Quality Review Skill

Systematic code review combining clean code principles, API design, and Java best practices.

## When to Use
- "review this code" / "code review" / "check this PR"
- "refactor" / "clean this code" / "improve readability"
- "review API" / "check endpoints" / "REST review"
- Before merging PR or releasing API changes

## Review Strategy

1. **Quick scan** - Understand intent, identify scope
2. **Checklist pass** - Apply relevant categories below
3. **Summary** - List findings by severity (Critical → Minor → Good)

---

## Workspace-specific checks

General review knowledge is assumed. Check these, which are specific to the gv services:

- Controllers implement the OpenAPI-generated interface, and DTOs come from the generated artifact. Flag hand-written copies.
- Beans use constructor injection only, with no field `@Autowired`.
- Spring test mocks use `@MockitoBean`; `@MockBean` is deprecated.
- Where a repo enables NullAway, Error Prone or ArchUnit, those are hard gates; review against them. For example, scan-manager's `FipsArchRulesTest` forbids `org.bouncycastle` imports and requires an explicit `SunPKCS11-FIPS` provider for `Cipher`/`SecretKeyFactory` in FIPS-profiled classes.
- Where the repo runs Spotless (google-java-format), formatting is owned by it. Don't raise formatting findings.

---

## Review Output Format

```markdown
## Code Review: [Component/Feature Name]

### Critical Issues
- **Null safety violation** (UserService.java:42) - `user.getName().toUpperCase()` can NPE. Use Optional or null check.
- **Resource leak** (FileHandler.java:15) - FileInputStream not closed. Use try-with-resources.

### Important Improvements
- **API design** - POST used for idempotent update (UserController.java:28). Use PUT instead.
- **Transaction missing** - Multi-step operation needs @Transactional (OrderService.java:56).
- **N+1 query** - Loop fetches orders individually (line 89). Use JOIN FETCH.

### Code Smells
- **Long method** - extractUserData() is 80 lines. Consider extracting sub-methods.
- **Magic number** - Use named constant instead of `86400` (line 123).
- **Inconsistent naming** - Mix of camelCase and snake_case in variables.

### Good Practices Observed
- ✅ Constructor injection used throughout
- ✅ DTOs properly separate from entities
- ✅ Comprehensive validation on all endpoints
- ✅ Good test coverage (87%)
```

---

## Quick Reference Flags

| Category | Red Flags |
|----------|-----------|
| **Null Safety** | Chained calls, Optional.get(), returning null |
| **Exceptions** | Empty catch, broad catch, lost stack trace |
| **Resources** | Manual close(), missing try-with-resources |
| **API Design** | Wrong HTTP verb, no versioning, entity exposure |
| **Transactions** | Multi-step writes without @Transactional |
| **Performance** | N+1 queries, loading all data, missing indexes |
| **Clean Code** | Code duplication, magic numbers, unclear names |

---

## Severity Levels

- **Critical** - Security, data loss, crash risk → Must fix before merge
- **Important** - Performance, maintainability, correctness → Should fix
- **Code Smell** - Style, complexity, minor issues → Nice to have
- **Good** - Positive feedback to reinforce good practices
