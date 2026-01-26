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

The decomposition produces a hierarchical backlog:

```
Epic
├── Feature 1
│   ├── User Story 1.1
│   │   ├── T1: Implementation task
│   │   ├── T2: Implementation task
│   │   ├── UT1: Unit test task
│   │   └── UT2: Unit test task
│   └── User Story 1.2
│       ├── T1: Implementation task
│       ├── UT1: Unit test task
│       └── IT1: Integration test task
└── Feature 2
    └── ...
```

---

## Decomposition Rules

### 1. Test Task Ratio

Default: **1:1 mapping** between implementation tasks and test tasks.

For each implementation task, create at least one corresponding test task:

| Implementation Task | Test Task(s) |
|---------------------|--------------|
| T1: Implement config loader | UT1: Test config loading from file |
| T2: Add CLI argument parsing | UT2: Test CLI argument validation |
| T3: Implement API client | UT3: Test API client with mocked responses |

### 2. Integration Test Threshold

Generate integration test tasks when a story touches **N or more components** (default N=2).

**Example**:
```markdown
## User Story: Generate commit message from staged changes

Components touched:
- CLI Interface
- Core Engine
- Claude API

Result: Story touches 3 components → Generate IT task

Tasks:
- T1: Wire CLI to core engine
- T2: Connect core engine to API client
- UT1: Test CLI argument handling
- UT2: Test core engine logic
- IT1: Test end-to-end commit generation flow
```

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

### Implementation Task

```markdown
### T1: [Action verb] [component/feature]

**Description**: [What needs to be implemented]

**Acceptance Criteria**:
- [ ] [Specific, verifiable criterion]
- [ ] [Specific, verifiable criterion]

**Dependencies**: [T-IDs of blocking tasks, if any]
```

### Unit Test Task

```markdown
### UT1: Test [component/behavior]

**Description**: [What is being tested]

**Test Cases**:
- [ ] [Test case 1: input → expected output]
- [ ] [Test case 2: edge case handling]
- [ ] [Test case 3: error scenario]

**Covers**: T1, T2 (implementation tasks this tests)
```

### Integration Test Task

```markdown
### IT1: Test [workflow/integration]

**Description**: [What end-to-end flow is being tested]

**Test Scenarios**:
- [ ] [Scenario 1: Happy path through components]
- [ ] [Scenario 2: Error propagation across components]

**Components**: [List of components involved]
**Covers**: T1, T2, T3 (implementation tasks this validates)
```

---

## User Story Template

```markdown
## US-[ID]: [User story title]

**As a** [role]
**I want** [capability]
**So that** [benefit]

### Acceptance Criteria
- [ ] [Criterion 1 - specific, measurable]
- [ ] [Criterion 2 - specific, measurable]
- [ ] [Criterion 3 - specific, measurable]

### Tasks

#### Implementation
- T1: [Task description]
- T2: [Task description]

#### Unit Tests
- UT1: [Test description] (covers T1)
- UT2: [Test description] (covers T2)

#### Integration Tests
- IT1: [Test description] (covers T1, T2) - if applicable

### Definition of Done
- [ ] All implementation tasks complete
- [ ] All unit tests passing
- [ ] All integration tests passing (if applicable)
- [ ] Code reviewed
- [ ] Documentation updated
```

---

## Decomposition Checklist

Before finalizing decomposition:

- [ ] Every requirement from design has corresponding story/stories
- [ ] Every implementation task has at least one test task
- [ ] Stories touching 2+ components have integration test tasks
- [ ] All stories meet INVEST criteria (see plan-review.md)
- [ ] No story exceeds 3-day estimate including tests
- [ ] Task dependencies are identified and documented
- [ ] Edge cases from design are captured in test tasks

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

**Tasks**:
- T1: Implement conventional format parser
- T2: Add scope extraction from file paths
- UT1: Test format pattern matching (covers T1)
- UT2: Test scope extraction logic (covers T2)
- UT3: Test 72-char truncation (covers T1)

---

#### US-2: Generate detailed format commits

**As a** developer
**I want** detailed commit messages with body
**So that** complex changes are well-documented

**Acceptance Criteria**:
- [ ] Output includes summary line + blank line + body
- [ ] Body includes bullet points of changes
- [ ] Each bullet corresponds to a file or logical change

**Tasks**:
- T1: Implement detailed format generator
- T2: Add change summarization logic
- UT1: Test body formatting (covers T1)
- UT2: Test bullet generation (covers T2)
- IT1: Test end-to-end detailed format generation (covers T1, T2)

---

#### US-3: Generate brief format commits

**As a** developer
**I want** single-line commit messages
**So that** minor changes have minimal overhead

**Acceptance Criteria**:
- [ ] Output is single line, max 50 characters
- [ ] Captures primary change only
- [ ] No type prefix or scope

**Tasks**:
- T1: Implement brief format generator
- UT1: Test single-line output (covers T1)
- UT2: Test 50-char truncation (covers T1)
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
