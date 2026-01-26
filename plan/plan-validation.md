# Plan Validation Template

This template guides the validation of findings from plan review (plan-review.md) against actual backlog items, confirming or rejecting each finding.

---

## Purpose

Validates plan review findings to:
- Confirm issues are genuine problems
- Identify false positives from misreading
- Catch scope creep in suggested fixes
- Consolidate duplicate findings
- Ensure findings align with design intent

---

## Validation Process

```
                 PLAN VALIDATION WORKFLOW
+--------------------------------------------------------+
|                                                        |
|  +-----------------+                                   |
|  | Load Review     | Get findings from plan-review     |
|  | Findings        |                                   |
|  +--------+--------+                                   |
|           |                                            |
|           v                                            |
|  +-----------------+                                   |
|  | For Each        |                                   |
|  | Finding         |------+                            |
|  +--------+--------+      |                            |
|           |               |                            |
|           v               |                            |
|  +-----------------+      |                            |
|  | Read Actual     | Check backlog item directly       |
|  | Backlog Item    |                                   |
|  +--------+--------+      |                            |
|           |               |                            |
|           v               |                            |
|  +-----------------+      |                            |
|  | Assign          | VALID, FALSE_POSITIVE,            |
|  | Verdict         | SCOPE_CREEP, ALREADY_OK, MERGED   |
|  +--------+--------+      |                            |
|           |               |                            |
|           +---------------+                            |
|           |                                            |
|           v                                            |
|  +-----------------+                                   |
|  | Generate        | VALID findings need action        |
|  | Action Items    |                                   |
|  +-----------------+                                   |
|                                                        |
+--------------------------------------------------------+
```

---

## Validation Statuses

### VALID

The finding is confirmed accurate and requires action.

**Criteria**:
- Issue exists as described in the backlog item
- Impact assessment is accurate
- Fix is necessary for backlog quality

**Action**: Apply suggested fix or generate alternative options

---

### FALSE_POSITIVE

The finding is incorrect; the backlog item is actually fine.

**Criteria**:
- Reviewer misread or misunderstood the content
- Issue based on incorrect assumptions about requirements
- Backlog item correctly implements design intent

**Action**: No change needed; document why finding was rejected

**Example**:
```markdown
Finding: US-3 missing acceptance criteria for error handling

Validation: FALSE_POSITIVE
Reason: Error handling is explicitly scoped to US-7 per design section 4.2.
US-3 correctly focuses on happy path only.
```

---

### SCOPE_CREEP

The finding suggests additions beyond design scope.

**Criteria**:
- Suggested fix adds functionality not in design
- Finding introduces new requirements
- Enhancement disguised as defect

**Action**: Reject or defer to future iteration

**Example**:
```markdown
Finding: US-2 should include caching for performance

Validation: SCOPE_CREEP
Reason: Caching is listed in "Future Considerations" section of design.
Not in scope for current implementation.
```

---

### ALREADY_OK

The issue was already addressed or content already exists.

**Criteria**:
- Issue was fixed in a previous revision
- Content exists but reviewer missed it
- Different story already covers this concern

**Action**: No change needed; reference where it's handled

**Example**:
```markdown
Finding: No integration test for CLI-to-API flow

Validation: ALREADY_OK
Reason: IT-3 in US-5 covers this exact flow.
Reviewer checked US-2 but missed US-5.
```

---

### MERGED

Finding consolidated with another finding.

**Criteria**:
- Same underlying issue as another finding
- Single fix addresses both concerns
- Findings are duplicates or near-duplicates

**Action**: Reference primary finding; apply fix once

**Example**:
```markdown
Finding #7: US-3 has vague acceptance criterion
Finding #12: US-3 criterion "fast response" is unmeasurable

Validation for #12: MERGED with #7
Reason: Both point to same criterion in US-3. Fix once.
```

---

## Validation Checks

### INVEST Violation Verification

For INVEST findings, verify against actual story content:

```markdown
## Finding: US-4 violates Small (S) - too large

### Verification Steps:
1. Read US-4 in full
2. Count implementation tasks: [N]
3. Estimate total effort: [X days]
4. Check if naturally splits into smaller stories

### Verdict: [VALID | FALSE_POSITIVE]

### Evidence:
- Task count: 8 implementation tasks
- Estimated effort: 5 days
- Natural split point: Separate "create" from "update" operations
```

---

### Coverage Gap Verification

For coverage findings, cross-reference design document:

```markdown
## Finding: REQ-5 (rate limiting) has no story

### Verification Steps:
1. Locate REQ-5 in design document
2. Search backlog for related stories
3. Check if covered implicitly by another story

### Verdict: [VALID | FALSE_POSITIVE | ALREADY_OK]

### Evidence:
- REQ-5 in design: "Handle API rate limiting with exponential backoff"
- Stories searched: US-1 through US-12
- Related story found: None / US-7 acceptance criterion 3
```

---

### Test Task Verification

For test coverage findings, verify test task existence:

```markdown
## Finding: T-4 (validation logic) has no unit test

### Verification Steps:
1. Locate T-4 in backlog
2. Search for UT tasks referencing T-4
3. Check if tested as part of integration test

### Verdict: [VALID | FALSE_POSITIVE | ALREADY_OK]

### Evidence:
- T-4 description: "Add input validation for user data"
- UT tasks referencing T-4: None
- IT tasks covering T-4: IT-2 includes validation in flow but not isolated
```

---

## Validation Record Format

Document each validation:

```markdown
### Validation [N]: Finding #[X] - [Brief Title]

**Original Finding**:
- Type: [INVEST-X | TEST-COVERAGE | REQUIREMENTS | etc.]
- Issue: [Original issue description]
- Suggested Fix: [Original suggestion]

**Validation**:
- Status: VALID | FALSE_POSITIVE | SCOPE_CREEP | ALREADY_OK | MERGED
- Reason: [Explanation for verdict]
- Evidence: [What you checked to reach this conclusion]

**Action** (if VALID):
- [ ] [Specific action to take]
```

---

## Validation Summary Template

```markdown
## Plan Validation Summary

**Validated**: [Date]
**Review Reference**: [Link to plan review]
**Validator**: [Name/Agent]

### Findings by Status

| Status | Count | Finding IDs |
|--------|-------|-------------|
| VALID | X | #1, #4, #7 |
| FALSE_POSITIVE | X | #2, #5 |
| SCOPE_CREEP | X | #3 |
| ALREADY_OK | X | #6 |
| MERGED | X | #8 → #1 |

### VALID Findings - Action Required

| Finding | Issue | Action |
|---------|-------|--------|
| #1 | US-2 too large | Split into US-2a, US-2b |
| #4 | T-5 missing test | Add UT-5 covering validation |
| #7 | REQ-3 not covered | Create US-13 for retry logic |

### Rejected Findings - No Action

| Finding | Status | Reason |
|---------|--------|--------|
| #2 | FALSE_POSITIVE | Story correctly scoped per design 3.1 |
| #3 | SCOPE_CREEP | Caching is future enhancement |
| #5 | FALSE_POSITIVE | Reviewer misread task description |
| #6 | ALREADY_OK | Covered by IT-3 in US-5 |

### Next Steps

1. [ ] Apply fixes for VALID findings
2. [ ] Update backlog with new/modified items
3. [ ] Re-run plan review on updated backlog
```

---

## Common Validation Patterns

### Pattern 1: Implicit Coverage

Finding claims something is missing, but it's implicitly handled:

```markdown
Finding: No error handling for network timeout
Validation: Check if error handling is:
- Part of a shared utility story
- Covered by framework/library defaults
- Explicitly deferred in design
```

### Pattern 2: Different Granularity

Finding claims story is too large/small, but scope is intentional:

```markdown
Finding: US-5 should be split
Validation: Check design for:
- Stated atomic operations
- Dependencies requiring single story
- Explicit sizing guidance
```

### Pattern 3: Test Coverage Dispute

Finding claims missing test, but coverage exists differently:

```markdown
Finding: T-3 has no unit test
Validation: Check for:
- Integration test covering same logic
- Test task using different naming
- Acceptance test at story level
```

---

## Validation Checklist

Before finalizing validation:

- [ ] Each finding has been verified against actual backlog content
- [ ] Status assignment includes supporting evidence
- [ ] MERGED findings reference their primary finding
- [ ] FALSE_POSITIVE findings explain the misunderstanding
- [ ] SCOPE_CREEP findings reference design scope limitations
- [ ] VALID findings have clear, actionable next steps
- [ ] Summary accurately counts findings by status
