---
name: coding-constitution
description: Mandatory operating laws for autonomous AI coding agents. Apply these rules to ALL coding tasks — features, bug fixes, refactoring, performance improvements, infrastructure changes, and documentation updates. Violation is considered invalid work.
---

# Autonomous AI Coding Constitution
Version: 1.0

This constitution defines the **mandatory operating laws** for any autonomous coding agent working within this repository. The purpose is to guarantee:

- Predictable software development
- Architectural consistency
- Security and reliability
- Traceable and auditable code changes
- Prevention of hallucinated implementations
- Production-grade pull requests

These rules apply to **all tasks**. Violation of these rules is considered **invalid work**.

---

## Fundamental Principles

### 1. Deterministic Development
Always produce **predictable, explainable, and reproducible changes**.

MUST:
- Follow existing patterns
- Use documented APIs only

MUST NOT:
- Invent undocumented APIs
- Create new architectural patterns without justification

### 2. Minimal Surface Change
Every task must modify the **smallest possible number of files**.

MUST:
- Only modify files required for the task
- Avoid formatting unrelated code
- Avoid refactoring unless explicitly requested

### 3. Architectural Respect
The existing system architecture is the **source of truth**.

MUST follow this layer separation:
- `controllers/` → request handling
- `services/` → business logic
- `repositories/` → database logic
- `utils/` → reusable helpers
- `types/` → shared types

MUST NOT:
- Place business logic in controllers
- Access databases directly from UI layers
- Duplicate services

---

## Law 1 — Task Understanding
Before writing code, determine:

**Task Type:** Feature | Bug Fix | Refactor | Performance Improvement | Documentation Update

Produce an internal summary:
- Objective
- Affected Components
- Risk Level (Low / Medium / High)

If the objective is unclear → **stop and request clarification**.

---

## Law 2 — Planning Before Implementation
Create an **implementation plan before coding**.

Example:
1. Update schema
2. Add service logic
3. Expose API endpoint
4. Add validation
5. Write tests
6. Update documentation

**MUST NOT begin coding without a plan.**

---

## Law 3 — Architecture Validation
Before writing code, validate:
- Does this change fit the architecture?
- Is there an existing module that should be reused?
- Is this duplicating existing functionality?

If duplication is detected → reuse the existing implementation.

---

## Law 4 — Safe Implementation

### Code Quality
- Use descriptive variable names
- Avoid magic numbers
- Write readable functions
- Handle errors properly

```
Good: const paymentTimeoutMs = 5000
Bad:  const t = 5000
```

### Dependency Safety
Do not introduce new dependencies unless:
- Absolutely required
- Justified by the task

---

## Law 5 — Anti-Hallucination Rule
MUST NOT:
- Invent APIs
- Fabricate SDK functions
- Guess database schemas
- Assume environment variables

If required information is missing:
1. Search the repository
2. Inspect documentation
3. Request clarification

---

## Law 6 — Test Enforcement
Every code change must include **verification**.

Test types: Unit | Integration | Regression

Ensure:
- Code compiles
- Tests pass
- Lint passes
- Type checks pass

If tests cannot be written → provide **manual verification steps**.

---

## Law 7 — Security Review
Before submitting a PR, check for:
- Exposed secrets
- Injection vulnerabilities
- Unsafe input handling
- Insecure authentication logic

Ensure:
- No secrets committed
- Environment variables used correctly

---

## Law 8 — Stability Verification
Review code for:
- Race conditions
- Memory leaks
- Unhandled promises
- Infinite loops
- Blocking operations

---

## Law 9 — Documentation Integrity
If the change affects APIs, configuration, environment variables, or infrastructure — update:
- `README.md`
- API documentation
- `CHANGELOG.md`

---

## Law 10 — Pull Request Standard

### Title format:
```
type: short description

Examples:
feat: add escrow creation endpoint
fix: resolve websocket reconnect issue
refactor: simplify payment validation
```

### PR Description structure:
```
**Summary**
Short description of the change.

**Type**
[ ] Feature  [ ] Bug Fix  [ ] Refactor  [ ] Improvement

**Changes**
- Added new service
- Updated endpoint
- Added validation

**Testing**
Explain how it was tested.

**Risk Level**
Low / Medium / High

**Breaking Changes**
None or description
```

---

## Law 11 — Review Checklist
Before finalizing a PR verify:
- [ ] Architecture respected
- [ ] Minimal files changed
- [ ] Tests added or verified
- [ ] Documentation updated
- [ ] No secrets committed
- [ ] Lint passes
- [ ] Types compile
- [ ] CI pipeline expected to pass

---

## Law 12 — Git Discipline
Branch naming:
```
feature/<feature-name>
fix/<bug-name>
refactor/<module>

Examples:
feature/escrow-contract
fix/payment-timeout
refactor/websocket-service
```

---

## Law 13 — Commit Message Standard
Follow **Conventional Commits**:
```
feat: add webhook retry system
fix: prevent duplicate transaction processing
refactor: extract redis connection manager
test: add escrow validation tests
docs: update API documentation
```

---

## Law 14 — Merge Readiness
A PR is merge-ready only when:
- CI passes
- Tests pass
- No unresolved comments
- Branch is up to date with main

---

## Law 15 — Post-Merge Validation
After merge, verify:
- Deployment succeeds
- Logs show no errors
- Feature works as expected
- No regressions detected

---

## Enforcement
If any law is violated:
1. Halt implementation
2. Report the violation
3. Request correction

No code may be merged that violates this constitution.

---

## Guiding Philosophy

> Reliable software is not produced by speed.
> Reliable software is produced by discipline, verification, architectural respect, and testable changes.

This constitution ensures autonomous coding agents behave like **senior engineers, not autocomplete tools**.
