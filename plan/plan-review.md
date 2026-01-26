# Plan Review Template

This template provides review criteria for validating decomposed backlogs against INVEST principles and test coverage requirements.

---

## Purpose

Reviews decomposed backlog items (from plan-decompose.md) to ensure:
- Stories meet INVEST criteria
- Test coverage is complete
- All design requirements are covered
- No scope creep or duplicates

---

## Review Process

```
                    PLAN REVIEW WORKFLOW
+--------------------------------------------------------+
|                                                        |
|  +-----------------+                                   |
|  | Load Backlog    |                                   |
|  | & Design Doc    |                                   |
|  +--------+--------+                                   |
|           |                                            |
|           v                                            |
|  +-----------------+                                   |
|  | INVEST Check    | Check each story against          |
|  | (per story)     | I-N-V-E-S-T criteria              |
|  +--------+--------+                                   |
|           |                                            |
|           v                                            |
|  +-----------------+                                   |
|  | Test Coverage   | Verify test tasks exist           |
|  | Check           | for all implementation            |
|  +--------+--------+                                   |
|           |                                            |
|           v                                            |
|  +-----------------+                                   |
|  | Coverage Check  | All requirements covered?         |
|  | (vs Design)     | Any scope creep?                  |
|  +--------+--------+                                   |
|           |                                            |
|           v                                            |
|  +-----------------+                                   |
|  | Generate        | List issues with verdicts         |
|  | Findings        |                                   |
|  +-----------------+                                   |
|                                                        |
+--------------------------------------------------------+
```

---

## Review Criteria

### INVEST Criteria

Each user story must satisfy all INVEST criteria:

#### I - Independent

The story can be developed and delivered independently.

**Check**:
- [ ] No hard dependencies on other stories in same sprint
- [ ] Can be tested in isolation
- [ ] Delivers value without waiting for other stories

**Violation Example**:
```markdown
US-1: Create user table in database
US-2: Implement user registration

Issue: US-2 cannot be tested without US-1 complete
```

**Fix**: Combine into single story or explicitly sequence

---

#### N - Negotiable

Implementation details are flexible; the story focuses on the "what", not "how".

**Check**:
- [ ] Story describes outcome, not implementation steps
- [ ] Multiple technical approaches could satisfy criteria
- [ ] No specific technology mandated (unless required)

**Violation Example**:
```markdown
US-1: Use Redis to cache API responses

Issue: Specifies implementation (Redis) rather than goal (caching)
```

**Fix**: "Cache API responses to reduce latency below 100ms"

---

#### V - Valuable

The story delivers clear value to users or the business.

**Check**:
- [ ] "So that" clause articulates real benefit
- [ ] Not purely technical/infrastructure work
- [ ] Stakeholder can understand the value

**Violation Example**:
```markdown
US-1: Refactor database module

Issue: Technical task with no user-visible value stated
```

**Fix**: "Reduce query response time from 500ms to 50ms so users experience faster page loads"

---

#### E - Estimable

The team can reasonably estimate the effort required.

**Check**:
- [ ] Scope is clear and bounded
- [ ] No significant unknowns or research needed
- [ ] Acceptance criteria are specific enough to size

**Violation Example**:
```markdown
US-1: Integrate with external payment provider

Issue: Too vague - which provider? What integration depth?
```

**Fix**: "Integrate Stripe checkout for one-time payments with webhook handling"

---

#### S - Small

The story can be completed in 1-3 days including tests.

**Check**:
- [ ] Estimate is 1-3 days (or team's sprint fraction)
- [ ] Can demo completed work within iteration
- [ ] Not an epic disguised as a story

**Violation Example**:
```markdown
US-1: Implement authentication system

Issue: Too large - includes login, logout, password reset, sessions, etc.
```

**Fix**: Split into: "Implement login flow", "Implement logout", "Implement password reset"

---

#### T - Testable

The story has clear, verifiable acceptance criteria.

**Check**:
- [ ] Each acceptance criterion is specific and measurable
- [ ] Can write automated tests against criteria
- [ ] Pass/fail is unambiguous

**Violation Example**:
```markdown
Acceptance Criteria:
- [ ] System is responsive
- [ ] Error messages are helpful
```

**Fix**:
```markdown
Acceptance Criteria:
- [ ] Page loads in <2 seconds on 3G connection
- [ ] Error messages include: error code, description, suggested action
```

---

### Test Coverage Checks

#### Implementation-Test Mapping

Every implementation task must have corresponding test task(s).

**Check**:
- [ ] Each T-task has at least one UT-task
- [ ] Test task explicitly references which T-task(s) it covers

**Violation Example**:
```markdown
Tasks:
- T1: Implement config loader
- T2: Add validation logic
- T3: Create CLI parser
- UT1: Test config loading (covers T1)
```

**Issue**: T2 and T3 have no test coverage

---

#### Integration Test Threshold

Stories touching multiple components need integration tests.

**Check**:
- [ ] Stories with 2+ components have IT-task(s)
- [ ] IT-tasks cover cross-component interactions

**Violation Example**:
```markdown
Components: CLI, Core Engine, API Client

Tasks:
- T1: Wire CLI to core
- T2: Connect core to API
- UT1: Test CLI
- UT2: Test core
- UT3: Test API client
```

**Issue**: No IT-task despite 3 components interacting

---

#### Edge Case Coverage

Test tasks must include edge cases and error scenarios from design.

**Check**:
- [ ] Test tasks include error scenarios
- [ ] Edge cases from design doc are represented
- [ ] Boundary conditions are tested

**Violation Example**:
```markdown
Design specifies:
- Handle rate limiting with retry
- Handle network timeout
- Handle invalid API key

Test tasks:
- UT1: Test successful API call
```

**Issue**: Error scenarios not covered in test tasks

---

### Coverage Checks

#### Requirements Coverage

All design requirements must map to stories.

**Check**:
- [ ] Every requirement has at least one story
- [ ] Create traceability matrix: Requirement → Story(s)

**Format**:
```markdown
| Requirement | Story | Status |
|-------------|-------|--------|
| REQ-1: Support conventional format | US-1 | Covered |
| REQ-2: Support detailed format | US-2 | Covered |
| REQ-3: Support brief format | - | MISSING |
```

---

#### Scope Creep Detection

Stories must trace back to design requirements.

**Check**:
- [ ] Every story links to a design requirement
- [ ] No stories that add undocumented features

**Violation Example**:
```markdown
US-5: Add dark mode support

Issue: Not in design document - scope creep
```

**Action**: Remove or document as enhancement for future

---

#### Duplicate Detection

No redundant stories covering the same functionality.

**Check**:
- [ ] No two stories have overlapping acceptance criteria
- [ ] Similar stories have clear differentiation

---

## Findings Format

Document each issue found during review:

```markdown
### Finding [N]: [Brief Title]

**Type**: INVEST-[I|N|V|E|S|T] | TEST-COVERAGE | REQUIREMENTS | SCOPE-CREEP | DUPLICATE

**Location**: [Story ID or Task ID]

**Issue**: [Description of the problem]

**Impact**: [Why this matters]

**Suggested Fix**: [How to resolve]
```

---

## Review Summary Template

```markdown
## Plan Review Summary

**Reviewed**: [Date]
**Backlog Version**: [Version/commit]
**Design Document**: [Reference]

### Statistics
- Total Stories: [N]
- Total Tasks: [N implementation, N test]
- Test-to-Implementation Ratio: [N:N]

### INVEST Compliance
| Criterion | Pass | Fail | Issues |
|-----------|------|------|--------|
| Independent | X | Y | #1, #5 |
| Negotiable | X | Y | - |
| Valuable | X | Y | #3 |
| Estimable | X | Y | - |
| Small | X | Y | #2, #4 |
| Testable | X | Y | #6 |

### Test Coverage
- Implementation tasks with tests: X/Y (Z%)
- Stories with integration tests: X/Y (Z%)
- Edge cases covered: X/Y (Z%)

### Requirements Coverage
- Requirements covered: X/Y (Z%)
- Missing requirements: [List]

### Scope Creep
- Stories without requirement link: [List]

### Findings
[List of findings with verdicts]

### Verdict
[ ] APPROVED - Ready for implementation
[ ] REVISE - Address findings first
```

---

## Review Checklist

Quick checklist for reviewers:

- [ ] All stories have user-facing value (V)
- [ ] No story larger than 3 days (S)
- [ ] All acceptance criteria are measurable (T)
- [ ] Every implementation task has test task (Coverage)
- [ ] Multi-component stories have integration tests (IT threshold)
- [ ] All design requirements have stories (Traceability)
- [ ] No stories outside design scope (Scope creep)
- [ ] No duplicate stories (Dedup)
