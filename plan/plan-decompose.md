# Plan Decomposition Template

This template guides the decomposition of validated design documents into implementation and test tasks organized as a backlog.

---

## Purpose

Decomposes validated design documents into actionable work items at the appropriate granularity level, ensuring test tasks are created alongside implementation tasks.

---

## Input Requirements

Before starting decomposition, ensure you have:

- **Design document**: Validated and approved design document
- **Decomposition depth**: Target granularity level (epics/features/stories/tasks)
- **Test strategy configuration**: Ratio and thresholds for test task generation

---

## Output Structure

The decomposition produces a hierarchical backlog with **three-phase TDD ordering**:

```
Epic
├── Feature 1
│   ├── User Story 1.1
│   │   ├── UT1: Unit test task (RED)
│   │   ├── UT2: Unit test task (RED)
│   │   ├── T1: Implementation task [after: UT1] (GREEN)
│   │   ├── T2: Implementation task [after: UT2] (GREEN)
│   │   └── IT1: Integration test [after: T1, T2] (INTEGRATION)
│   └── User Story 1.2
│       ├── UT1: Unit test task (RED)
│       ├── T1: Implementation task [after: UT1] (GREEN)
│       └── IT1: Integration test [after: T1] (INTEGRATION)
└── Feature 2
    └── ...
```

**Phase Order:**
1. **RED** - Unit tests define behavior (UT tasks)
2. **GREEN** - Implementation makes tests pass (T tasks)
3. **INTEGRATION** - Integration tests verify assembled components (IT tasks)

---

## Sprint Directory Output (Recommended)

When decomposing for gigachad execution, produce a **sprint directory** instead of a flat TASK.md. Each task gets its own `.md` file with a full execution plan.

### Directory Structure

```
.gigachad/sprints/{sprint-name}/
├── sprint.md                  # Sprint overview + shared context + task index
├── UT1-1.md                   # Full plan for unit test task
├── T1-1.md                    # Full plan for implementation task
├── IT1-1.md                   # Full plan for integration test task
└── ...
```

### Sprint File (`sprint.md`)

Contains shared context that all agents need, plus a task index for dependency tracking:

```markdown
---
sprint: 5
name: Feature Name
status: pending
---

# Sprint 5: Feature Name

## Context
[Shared architectural context, key decisions, constraints.
This is injected into every agent's prompt so they don't need
to rediscover it.]

## Task Index
| ID | Description | Section | Depends On | Status |
|----|-------------|---------|------------|--------|
| UT1-1 | Test config loading | RED | - | pending |
| T1-1 | Implement config loader | GREEN | UT1-1 | pending |
| IT1-1 | Test full config flow | INTEGRATION | T1-1 | pending |
```

### Task Plan File (`{task_id}.md`)

Each task file has YAML frontmatter for machine-readable status + markdown body with the full execution plan:

```markdown
---
id: UT1-1
status: pending
section: RED
story: US-1
depends_on: []
---

# UT1-1: Test config loading from file

## Description
Test that the config loader reads YAML files and returns a validated config object.

## Plan
1. Create test in `tests/test_config.py`
2. Create a temporary YAML config file with known values
3. Call `load_config(path)` and assert returned object matches
4. Test missing file raises `ConfigNotFoundError`
5. Test invalid YAML raises `ConfigParseError`

## Reference Files
- `src/myapp/config/loader.py` -- target module
- `src/myapp/config/schema.py` -- config dataclass
- `tests/test_config.py` -- existing tests for patterns

## Acceptance Criteria
- [ ] load_config() returns validated config from YAML
- [ ] Missing file raises ConfigNotFoundError
- [ ] Invalid YAML raises ConfigParseError
```

### Frontmatter Fields

| Field | Required | Values | Description |
|-------|----------|--------|-------------|
| `id` | Yes | Task ID string | Must match filename (e.g., `UT1-1.md` has `id: UT1-1`) |
| `status` | Yes | `pending`, `in_progress`, `complete` | Updated by agent during execution |
| `section` | Yes | `RED`, `GREEN`, `INTEGRATION` | TDD phase |
| `story` | Yes | Story ID string | Parent user story (e.g., `US-1`) |
| `depends_on` | Yes | List of task IDs | `[]` for no deps, `[UT1-1]` for single dep |

### Plan Body Sections

| Section | Required | Purpose |
|---------|----------|---------|
| `## Description` | Yes | What this task accomplishes (1-3 sentences) |
| `## Plan` | Yes | Step-by-step execution plan (numbered list) |
| `## Reference Files` | No | File paths the agent should read first |
| `## Acceptance Criteria` | Yes | Checkbox criteria for completion |

### Key Principle: Plan Once, Execute Many

The execution plan is written **once** at decomposition time with the full design document in scope. Each agent receives a ready-to-execute plan instead of rediscovering context from a one-line description. This eliminates:
- Duplicated planning across UT → T → IT phases
- Context loss between agent invocations
- File contention in multi-agent execution

---

## Decomposition Rules

### 1. TDD Task Ordering

**Tests come FIRST, then implementation.** This enables RED→GREEN TDD workflow.

Default: **1:1 mapping** with explicit pairing using `[after: UTx]` syntax:

| Test Task (RED) | Implementation Task (GREEN) |
|-----------------|----------------------------|
| UT1: Test config loading from file | T1: Implement config loader [after: UT1] |
| UT2: Test CLI argument validation | T2: Add CLI argument parsing [after: UT2] |
| UT3: Test API client with mocked responses | T3: Implement API client [after: UT3] |

The `[after: UTx]` dependency tells the system:
- **Before T1**: Run UT1's test, expect FAIL (RED phase verified)
- **After T1**: Run UT1's test, expect PASS (GREEN phase verified)

### 2. Integration Test Threshold

Generate integration test tasks when a story touches **N or more components** (default N=2).

**Important:** Integration tests go in a SEPARATE section AFTER implementation, not with unit tests.

**Example** (three-phase TDD):
```markdown
## User Story: Generate commit message from staged changes

Components touched:
- CLI Interface
- Core Engine
- Claude API

Result: Story touches 3 components → Generate IT task in INTEGRATION phase

### Tests First (RED)
- UT1: Test CLI argument handling
- UT2: Test core engine logic

### Implementation (GREEN)
- T1: Wire CLI to core engine [after: UT1]
- T2: Connect core engine to API client [after: UT2]

### Integration Tests (INTEGRATION)
- IT1: Test end-to-end commit generation flow [after: T1, T2]
```

**Why IT comes after implementation:**
- Integration tests verify components work **together**
- You can't test integration until components **exist**
- IT tasks naturally depend on multiple T tasks completing first

### 3. Story Sizing

Each user story should be completable in **1-3 days** including tests.

If a story is larger, decompose further:
- Split by acceptance criteria
- Split by component boundaries
- Split by happy path vs error handling

### 4. Task Granularity

Tasks should be:
- **Atomic**: One logical unit of work
- **Estimable**: Clear enough to size (hours, not days)
- **Testable**: Has defined completion criteria

---

## Task Format

### Unit Test Task (RED Phase - Written First)

```markdown
### UT1: Test [component/behavior]

**Description**: [What is being tested]

**Test Cases**:
- [ ] [Test case 1: input → expected output]
- [ ] [Test case 2: edge case handling]
- [ ] [Test case 3: error scenario]

**Expected**: Test should FAIL until T1 is implemented
```

### Integration Test Task (RED Phase)

```markdown
### IT1: Test [workflow/integration]

**Description**: [What end-to-end flow is being tested]

**Test Scenarios**:
- [ ] [Scenario 1: Happy path through components]
- [ ] [Scenario 2: Error propagation across components]

**Components**: [List of components involved]
**Expected**: Test should FAIL until T1, T2 are implemented
```

### Implementation Task (GREEN Phase - Written After Tests)

```markdown
### T1: [Action verb] [component/feature] [after: UT1]

**Description**: [What needs to be implemented]

**Acceptance Criteria**:
- [ ] [Specific, verifiable criterion]
- [ ] [Specific, verifiable criterion]

**TDD Verification**:
- Before: UT1 must FAIL (RED)
- After: UT1 must PASS (GREEN)
```

### Dependency Syntax Reference

| Syntax | Meaning |
|--------|---------|
| `[after: UT1]` | Implements UT1; verifies RED→GREEN in strict-tdd mode |
| `[after: T1, T2]` | Depends on T1 and T2 completing first |
| `[after: UT1, T2]` | Implements UT1 and depends on T2 |
| (no suffix) | No explicit dependency, runs in listed order |

---

## User Story Template (TDD)

```markdown
## US-[ID]: [User story title]

**As a** [role]
**I want** [capability]
**So that** [benefit]

### Acceptance Criteria
- [ ] [Criterion 1 - specific, measurable]
- [ ] [Criterion 2 - specific, measurable]
- [ ] [Criterion 3 - specific, measurable]

### Tests First (RED)
- [ ] UT1: [Test description]
- [ ] UT2: [Test description]

### Implementation (GREEN)
- [ ] T1: [Task description] [after: UT1]
- [ ] T2: [Task description] [after: UT2]

### Integration Tests (INTEGRATION) - if 2+ components
- [ ] IT1: [Integration test description] [after: T1, T2]

### Definition of Done
- [ ] All unit tests written and initially failing (RED verified)
- [ ] All implementations complete and unit tests passing (GREEN verified)
- [ ] Integration tests written and passing (INTEGRATION verified)
- [ ] Code reviewed
- [ ] Documentation updated
```

### User Story Template (Sprint Directory)

For sprint directory output, each task in the story becomes its own file. The story-level structure appears in `sprint.md`:

**In `sprint.md`:**
```markdown
## US-1: Generate conventional format commits

## Context
[Shared context for all tasks in this story...]

## Task Index
| ID | Description | Section | Depends On | Status |
|----|-------------|---------|------------|--------|
| UT1-1 | Test format pattern matching | RED | - | pending |
| UT1-2 | Test scope extraction logic | RED | - | pending |
| T1-1 | Implement conventional format parser | GREEN | UT1-1 | pending |
| T1-2 | Add scope extraction from file paths | GREEN | UT1-2 | pending |
| IT1-1 | Test end-to-end format generation | INTEGRATION | T1-1, T1-2 | pending |
```

**Individual task files** (`UT1-1.md`, `T1-1.md`, etc.) each contain the full execution plan as shown above.

---

## Decomposition Checklist

Before finalizing decomposition:

### Coverage
- [ ] Every requirement from design has corresponding story/stories
- [ ] Every implementation task has at least one unit test task
- [ ] Stories touching 2+ components have integration test tasks
- [ ] Edge cases from design are captured in test tasks

### Three-Phase TDD Ordering
- [ ] Unit test tasks (UT) are in "Tests First (RED)" section
- [ ] Implementation tasks (T) are in "Implementation (GREEN)" section
- [ ] Integration test tasks (IT) are in "Integration Tests (INTEGRATION)" section
- [ ] Implementation tasks have `[after: UTx]` linking to their unit tests
- [ ] Integration tasks have `[after: T1, T2, ...]` linking to their implementations
- [ ] No IT tasks mixed with UT tasks in RED section

### Sizing & Quality
- [ ] All stories meet INVEST criteria (see plan-review.md)
- [ ] No story exceeds 3-day estimate including tests
- [ ] Task dependencies form a valid DAG (no cycles)

### Sprint Directory Checks (when producing sprint directory output)
- [ ] `sprint.md` has valid YAML frontmatter (sprint number, name, status)
- [ ] `sprint.md` Context section captures shared architectural decisions
- [ ] `sprint.md` Task Index matches individual task files (IDs, descriptions, deps)
- [ ] Every task has its own `{task_id}.md` file
- [ ] Task filenames match their `id` frontmatter field
- [ ] Every task file has required frontmatter (id, status, section, story, depends_on)
- [ ] Every task file has required body sections (Description, Plan, Acceptance Criteria)
- [ ] Plan sections contain numbered step-by-step execution plans
- [ ] Reference Files point to real files in the codebase
- [ ] Acceptance Criteria are specific checkbox items
- [ ] `depends_on` lists match `[after:]` semantics (UT before T, T before IT)

---

## Example Decomposition

**From Design**: "Support conventional, detailed, and brief commit message formats"

```markdown
## Epic: Commit Message Formatting

### Feature: Output Format Support

#### US-1: Generate conventional format commits

**As a** developer
**I want** commit messages in conventional format
**So that** my commits follow team standards

**Acceptance Criteria**:
- [ ] Output matches pattern: `type(scope): description`
- [ ] Type is one of: feat, fix, docs, style, refactor, test, chore
- [ ] Scope is optional and extracted from changed files
- [ ] Description is max 72 characters

**Tests First (RED)**:
- [ ] UT1: Test format pattern matching
- [ ] UT2: Test scope extraction logic
- [ ] UT3: Test 72-char truncation

**Implementation (GREEN)**:
- [ ] T1: Implement conventional format parser [after: UT1, UT3]
- [ ] T2: Add scope extraction from file paths [after: UT2]

---

#### US-2: Generate detailed format commits

**As a** developer
**I want** detailed commit messages with body
**So that** complex changes are well-documented

**Acceptance Criteria**:
- [ ] Output includes summary line + blank line + body
- [ ] Body includes bullet points of changes
- [ ] Each bullet corresponds to a file or logical change

**Tests First (RED)**:
- [ ] UT1: Test body formatting
- [ ] UT2: Test bullet generation

**Implementation (GREEN)**:
- [ ] T1: Implement detailed format generator [after: UT1]
- [ ] T2: Add change summarization logic [after: UT2]

**Integration Tests (INTEGRATION)**:
- [ ] IT1: Test end-to-end detailed format generation [after: T1, T2]

---

#### US-3: Generate brief format commits

**As a** developer
**I want** single-line commit messages
**So that** minor changes have minimal overhead

**Acceptance Criteria**:
- [ ] Output is single line, max 50 characters
- [ ] Captures primary change only
- [ ] No type prefix or scope

**Tests First (RED)**:
- [ ] UT1: Test single-line output
- [ ] UT2: Test 50-char truncation

**Implementation (GREEN)**:
- [ ] T1: Implement brief format generator [after: UT1, UT2]
```

---

## Anti-Patterns to Avoid

```markdown
# BAD - Implementation before tests
- [ ] T1: Implement validation
- [ ] UT1: Test validation [after: T1]  # Wrong order!

# BAD - No explicit pairing
- [ ] UT1: Test validation
- [ ] T1: Implement validation  # Which test does this relate to?

# BAD - Test and impl combined
- [ ] T1: Add user validation with tests  # Violates TDD phases

# BAD - Integration test in RED section (can't run before impl exists!)
### Tests First (RED)
- [ ] UT1: Test config loading
- [ ] IT1: Test full flow  # IT needs T tasks to exist first!

# BAD - Integration test without implementation dependencies
- [ ] IT1: Test full flow  # Missing [after: T1, T2] - how can it run?

# GOOD - Explicit three-phase TDD structure
### Tests First (RED)
- [ ] UT1: Test user validation rejects empty email

### Implementation (GREEN)
- [ ] T1: Add email validation to User model [after: UT1]

### Integration Tests (INTEGRATION)
- [ ] IT1: Test signup flow with validation [after: T1]
```

---

## Configuration Options

Adjust these defaults based on project needs:

| Setting | Default | Description |
|---------|---------|-------------|
| `test_ratio` | 1:1 | Test tasks per implementation task |
| `integration_threshold` | 2 | Components touched before IT required |
| `max_story_days` | 3 | Maximum story size in days |
| `include_e2e` | false | Generate E2E test tasks |
