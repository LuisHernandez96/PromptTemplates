# Plan Review Template

This template provides review criteria for validating decomposed backlogs against INVEST principles and test coverage requirements.

---

## Purpose

Reviews decomposed backlog items (from plan-decompose.md) to ensure:
- Stories meet INVEST criteria
- Test coverage is complete
- **TDD ordering is correct** (tests listed before implementations)
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
|  | TDD Order       | Tests listed BEFORE impl?         |
|  | Check           | [after: UTx] syntax present?      |
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

### TDD Ordering Checks

#### Three-Phase Structure

Tasks must follow the three-phase TDD structure: RED → GREEN → INTEGRATION.

**Check**:
- [ ] "Tests First (RED)" section contains only UT tasks
- [ ] "Implementation (GREEN)" section contains only T tasks
- [ ] "Integration Tests (INTEGRATION)" section contains only IT tasks
- [ ] Sections appear in order: RED → GREEN → INTEGRATION

**Violation Example**:
```markdown
### Implementation (GREEN)
- [ ] T1: Implement config loader
- [ ] T2: Add validation

### Tests First (RED)
- [ ] UT1: Test config loading
- [ ] UT2: Test validation
```

**Issue**: Implementation listed before tests - wrong TDD order

**Fix**: Swap sections so Tests First (RED) comes before Implementation (GREEN)

---

#### Integration Tests in Correct Section

IT tasks must be in the INTEGRATION section, not mixed with UT tasks in RED.

**Check**:
- [ ] No IT tasks in "Tests First (RED)" section
- [ ] All IT tasks in "Integration Tests (INTEGRATION)" section
- [ ] IT tasks have `[after: T...]` dependencies

**Violation Example**:
```markdown
### Tests First (RED)
- [ ] UT1: Test config loading
- [ ] IT1: Test full flow  ← WRONG! IT can't run before T tasks exist
```

**Issue**: IT task in RED section - integration tests need implementations to exist first

**Fix**:
```markdown
### Tests First (RED)
- [ ] UT1: Test config loading

### Implementation (GREEN)
- [ ] T1: Implement config loader [after: UT1]

### Integration Tests (INTEGRATION)
- [ ] IT1: Test full flow [after: T1]
```

---

#### Explicit Test-Implementation Pairing

Implementation tasks must use `[after: UTx]` syntax to link to their tests.

**Check**:
- [ ] Every T-task has `[after: UTx]` suffix
- [ ] Referenced UT task exists
- [ ] No orphaned implementations without test links

**Violation Example**:
```markdown
### Tests First (RED)
- [ ] UT1: Test config loading
- [ ] UT2: Test validation

### Implementation (GREEN)
- [ ] T1: Implement config loader
- [ ] T2: Add validation logic
```

**Issue**: T1 and T2 missing `[after: UTx]` dependency links

**Fix**:
```markdown
### Implementation (GREEN)
- [ ] T1: Implement config loader [after: UT1]
- [ ] T2: Add validation logic [after: UT2]
```

---

#### No Combined Test-Implementation Tasks

Tests and implementations must be separate tasks, not combined.

**Check**:
- [ ] No task that "implements X with tests"
- [ ] Clear separation between RED (test) and GREEN (impl) phases

**Violation Example**:
```markdown
- [ ] T1: Add user validation with tests
```

**Issue**: Combines test writing and implementation in single task - violates TDD phases

**Fix**:
```markdown
- [ ] UT1: Test user validation rejects empty email
- [ ] T1: Add email validation to User model [after: UT1]
```

---

### Sprint Directory Structure Checks

When reviewing a sprint directory (instead of flat TASK.md), verify:

#### Frontmatter Consistency

**Check**:
- [ ] Every task file has valid YAML frontmatter with all required fields
- [ ] `id` field matches the filename (e.g., `UT1-1.md` has `id: UT1-1`)
- [ ] `status` is one of: `pending`, `in_progress`, `complete`
- [ ] `section` matches TDD phase: `RED` for UT, `GREEN` for T, `INTEGRATION` for IT
- [ ] `story` references a valid user story ID from `sprint.md`
- [ ] `depends_on` references only task IDs that exist as files

**Violation Example**:
```markdown
# File: UT1-1.md
---
id: UT-1-1          # Mismatch! Filename is UT1-1.md but id is UT-1-1
status: waiting     # Invalid! Must be pending/in_progress/complete
section: TEST       # Invalid! Must be RED/GREEN/INTEGRATION
story: US-1
depends_on: [UT99]  # UT99.md doesn't exist!
---
```

---

#### Sprint.md <-> Task File Consistency

**Check**:
- [ ] Every task in sprint.md Task Index has a corresponding `.md` file
- [ ] Every `.md` file (except sprint.md) appears in the Task Index
- [ ] Task descriptions in Task Index match `## Description` in task files
- [ ] Dependencies in Task Index match `depends_on` frontmatter

---

#### Plan Quality

**Check**:
- [ ] `## Plan` contains numbered steps (not vague prose)
- [ ] Steps reference specific files, functions, or modules
- [ ] Steps are ordered logically (read before write, test setup before assertions)
- [ ] Plan is executable without additional planning (agent shouldn't need `--plan-first`)

**Violation Example**:
```markdown
## Plan
Implement the config loader with proper error handling and tests.
```

**Issue**: Too vague -- a numbered step-by-step plan is required.

**Fix**:
```markdown
## Plan
1. Create `src/myapp/config/loader.py`
2. Define `load_config(path: Path) -> Config` function
3. Read YAML file with `yaml.safe_load()`
4. Validate against `Config` dataclass
5. Raise `ConfigNotFoundError` for missing files
6. Raise `ConfigParseError` for invalid YAML
```

---

#### Reference File Validity

**Check**:
- [ ] Reference files exist in the codebase
- [ ] `:symbol` references point to real functions/classes
- [ ] References are relevant to the task (not boilerplate)

---

### Test Coverage Checks

#### Implementation-Test Mapping

Every implementation task must have corresponding test task(s) with explicit `[after:]` links.

**Check**:
- [ ] Each T-task has at least one UT-task
- [ ] T-task uses `[after: UTx]` to reference its test
- [ ] No implementation task lacks a corresponding test

**Violation Example**:
```markdown
### Tests First (RED)
- [ ] UT1: Test config loading

### Implementation (GREEN)
- [ ] T1: Implement config loader [after: UT1]
- [ ] T2: Add validation logic
- [ ] T3: Create CLI parser
```

**Issue**: T2 and T3 have no test coverage and no `[after:]` links

---

#### Integration Test Threshold

Stories touching multiple components need integration tests in the INTEGRATION section.

**Check**:
- [ ] Stories with 2+ components have IT-task(s)
- [ ] IT-tasks are in "Integration Tests (INTEGRATION)" section
- [ ] IT-tasks have `[after: T...]` dependencies on implementations
- [ ] IT-tasks cover cross-component interactions

**Violation Example**:
```markdown
Components: CLI, Core Engine, API Client

### Tests First (RED)
- UT1: Test CLI
- UT2: Test core
- UT3: Test API client

### Implementation (GREEN)
- T1: Wire CLI to core [after: UT1]
- T2: Connect core to API [after: UT2, UT3]
```

**Issue**: No IT-task despite 3 components interacting

**Fix**: Add Integration Tests section:
```markdown
### Integration Tests (INTEGRATION)
- IT1: Test CLI → Core → API flow [after: T1, T2]
```

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

**Type**: INVEST-[I|N|V|E|S|T] | TDD-ORDER | TEST-COVERAGE | REQUIREMENTS | SCOPE-CREEP | DUPLICATE

**Location**: [Story ID or Task ID]

**Issue**: [Description of the problem]

**Impact**: [Why this matters]

**Suggested Fix**: [How to resolve]
```

### TDD-ORDER Finding Types

| Subtype | Description |
|---------|-------------|
| TDD-ORDER-SEQUENCE | Tests listed after implementation |
| TDD-ORDER-MISSING-LINK | Implementation lacks `[after: UTx]` |
| TDD-ORDER-COMBINED | Test and impl combined in single task |
| TDD-ORDER-ORPHAN | `[after:]` references non-existent test |
| TDD-ORDER-IT-MISPLACED | Integration test in RED section instead of INTEGRATION |
| TDD-ORDER-IT-NO-DEPS | Integration test missing `[after: T...]` dependencies |

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

### TDD Ordering
- Tasks with correct order (tests before impl): X/Y (Z%)
- Implementation tasks with `[after:]` links: X/Y (Z%)
- IT tasks in INTEGRATION section (not RED): X/Y (Z%)
- IT tasks with `[after: T...]` dependencies: X/Y (Z%)
- Combined test-impl tasks (violations): X

### Test Coverage
- Implementation tasks with tests: X/Y (Z%)
- Stories with integration tests: X/Y (Z%)
- Edge cases covered: X/Y (Z%)

### Requirements Coverage
- Requirements covered: X/Y (Z%)
- Missing requirements: [List]

### Scope Creep
- Stories without requirement link: [List]

### Sprint Directory Statistics (when reviewing sprint directory)
- Task files: [N] total ([N] UT, [N] T, [N] IT)
- Sprint.md Task Index entries: [N] (should match file count)
- Files with valid frontmatter: [N]/[N]
- Files with complete Plan sections: [N]/[N]
- Files with Acceptance Criteria: [N]/[N]
- Dependency references valid: [N]/[N]

### Findings
[List of findings with verdicts]

### Verdict
[ ] APPROVED - Ready for implementation
[ ] REVISE - Address findings first
```

---

## Review Checklist

Quick checklist for reviewers:

### INVEST
- [ ] All stories have user-facing value (V)
- [ ] No story larger than 3 days (S)
- [ ] All acceptance criteria are measurable (T)

### TDD Ordering (Three-Phase)
- [ ] Unit tests (UT) in "Tests First (RED)" section
- [ ] Implementations (T) in "Implementation (GREEN)" section
- [ ] Integration tests (IT) in "Integration Tests (INTEGRATION)" section
- [ ] Every T task has `[after: UTx]` dependency link
- [ ] Every IT task has `[after: T...]` dependency links
- [ ] No IT tasks mixed with UT tasks in RED section
- [ ] No combined "implement with tests" tasks

### Test Coverage
- [ ] Every implementation task has test task
- [ ] Multi-component stories have integration tests (IT threshold)
- [ ] Edge cases from design are captured in test tasks

### Requirements
- [ ] All design requirements have stories (Traceability)
- [ ] No stories outside design scope (Scope creep)
- [ ] No duplicate stories (Dedup)

### Sprint Directory (when applicable)
- [ ] All task files have valid YAML frontmatter
- [ ] Task IDs match filenames
- [ ] Sprint.md Task Index matches individual files
- [ ] Plan sections have numbered step-by-step instructions
- [ ] Reference files exist in codebase
- [ ] Dependencies form valid DAG (no cycles, all targets exist)
