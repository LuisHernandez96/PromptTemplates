# Design Review Validation Loop Workflow

This document describes the iterative design review workflow used to validate and improve design documents.

---

## Overview

Design review validation is a multi-round process that systematically identifies and addresses issues in design documents. Each round follows the same structured approach until no new issues are found.

```
                      DESIGN REVIEW LOOP (Batch Processing)
+-------------------------------------------------------------+
|                                                             |
|  +----------------+                                         |
|  |  Read Design   |                                         |
|  |   Document     |                                         |
|  +-------+--------+                                         |
|          |                                                  |
|          v                                                  |
|  +----------------+                                         |
|  |  Identify ALL  |  Scan entire document                   |
|  |    Issues      |  Document all potential issues          |
|  |                |  with locations & descriptions          |
|  +-------+--------+                                         |
|          |                                                  |
|          v                                                  |
|  +----------------+                                         |
|  |  Assign ALL    |  VALID -> needs fix                     |
|  |   Verdicts     |  FALSE POSITIVE -> no change            |
|  |                |  SCOPE CREEP -> defer                   |
|  |                |  ALREADY OK -> resolved                 |
|  +-------+--------+  MERGED -> combined                     |
|          |                                                  |
|          v                                                  |
|  +----------------+                                         |
|  |  Batch Options |  Present 2-3 options per                |
|  |  for ALL VALID |  VALID issue upfront                    |
|  |                |  Get user choices at once               |
|  +-------+--------+                                         |
|          |                                                  |
|          v                                                  |
|  +----------------+                                         |
|  |  Apply Fixes   |  Fix issues in sequence                 |
|  |  Sequentially  |  using selected options                 |
|  +-------+--------+                                         |
|          |                                                  |
|          v                                                  |
|  +----------------+                                         |
|  |  Batch Verify  |  Verify all fixes at end                |
|  |   All Fixes    |  of round                               |
|  +-------+--------+                                         |
|          |                                                  |
|          v                                                  |
|  +----------------+   No issues   +----------------+        |
|  |   More         | ------------> |     DONE       |        |
|  |   Issues?      |               +----------------+        |
|  +-------+--------+                                         |
|          | Yes                                              |
|          |                                                  |
|          +---------------------------> Next Round           |
|                                                             |
+-------------------------------------------------------------+
```

---

## Batch Processing Approach

The workflow uses batch processing to improve efficiency and reduce context switching:

### 1. Upfront Issue Presentation
Present all issues for a round at once with their verdicts assigned:

```markdown
## Round N: Issue Summary

| # | Issue | Verdict | Reason |
|---|-------|---------|--------|
| 1 | Missing error handling | VALID | Required for robustness |
| 2 | Typo in variable name | VALID | Inconsistent naming |
| 3 | Add caching layer | SCOPE CREEP | Enhancement, not fix |
| 4 | Unclear param description | ALREADY OK | Fixed in previous round |
| 5 | Same as Issue 2 | MERGED | Combined with Issue 2 |
```

### 2. Batched Option Selection
Use multi-question UI to gather user choices for all VALID issues at once:

```markdown
## Decisions Needed

**Issue 1: Missing error handling**
- [ ] A: Add try-catch wrapper (Recommended)
- [ ] B: Return error codes
- [ ] C: Skip

**Issue 2: Typo in variable name**
- [ ] A: Rename to 'userData' (Recommended)
- [ ] B: Rename to 'userInfo'
- [ ] C: Skip
```

### 3. Sequential Fix Application
After user selects all options, apply fixes in sequence to avoid conflicts.

### 4. Batch Verification
Verify all changes at the end of the round rather than after each individual fix:

```markdown
## Round N Verification

| Issue | Fix Applied | Verification | Result |
|-------|-------------|--------------|--------|
| 1 | Added try-catch | grep "try {" | PASS |
| 2 | Renamed variable | grep "userData" | PASS |
```

---

## Issue Format

When documenting a potential issue, use this structured format:

```markdown
### Issue N: [Brief Title]

**Location**: Lines X-Y (Section Name)

**Problem**: [Description of what's wrong or unclear]

**Impact**: [Why this matters - confusion, inconsistency, missing functionality, etc.]

**Verdict**: VALID | FALSE POSITIVE | SCOPE CREEP | ALREADY OK | MERGED

**Reasoning**: [Why you assigned this verdict]
```

### Example Issue

```markdown
### Issue 3: Inconsistent CLI Flag in Examples

**Location**: Lines 76-97 (CLI Subcommands section)

**Problem**: The `--working-dir` flag is shown as `-C` in some examples but
the actual CLI invocation uses a different placement pattern.

**Impact**: Users may get confused about the correct syntax for specifying
the working directory.

**Verdict**: VALID

**Reasoning**: This is a real inconsistency that could cause user confusion
and failed commands.
```

### Testability Issue Examples

```markdown
### Issue 7: Vague Acceptance Criteria

**Location**: Lines 45-50 (Requirements section)

**Problem**: The requirement states "CLI should respond quickly" without
defining a specific threshold or measurement method.

**Impact**: Cannot write automated tests to verify this requirement.
Implementation and testing teams have no clear target.

**Verdict**: VALID

**Reasoning**: Acceptance criteria must be specific and measurable.
Suggest: "CLI returns response within 2 seconds for diffs under 1000 lines."
```

```markdown
### Issue 8: Missing Error Scenario

**Location**: Lines 120-140 (API Integration section)

**Problem**: Document describes API rate limiting behavior but doesn't
specify what happens when max retries are exhausted.

**Impact**: Edge case behavior is undefined. Tests cannot verify error
handling for this scenario.

**Verdict**: VALID

**Reasoning**: Error scenarios must have defined expected behaviors to
enable proper test coverage and consistent implementation.
```

---

## Verdict Categories

### VALID

The issue is real and requires a fix.

**Criteria**:

*Consistency Issues*
- Factual error or inconsistency
- Contradicts other parts of the document
- Defaults don't match across sections
- Terminology is inconsistent

*Completeness Issues*
- Missing required information
- IEEE viewpoint not addressed (and not scoped out)
- Key design decision lacks rationale
- Stakeholder concern not addressed

*Quality Attribute Issues*
- Non-functional requirements are vague or unmeasurable
- Performance/scalability targets missing or unclear
- Reliability requirements undefined
- Security considerations missing

*Usability Issues*
- Would cause user confusion or errors
- Breaks expected functionality
- Examples don't match documentation

*Testability Issues*
- Acceptance criteria are vague or untestable
- Edge cases or error scenarios are undefined
- Test data requirements are missing (when applicable)

*Traceability Issues*
- Requirement has no design element
- Design element has no requirement (scope creep indicator)

**Action**: Fix in current round

### FALSE POSITIVE

The issue appears problematic but is actually correct upon closer inspection.

**Criteria**:
- Reviewer misread or misunderstood
- Issue based on incorrect assumptions
- Edge case that's actually handled elsewhere
- Behavior is intentional and documented

**Action**: No change needed

### SCOPE CREEP

The suggestion is valid but out of scope for the current document/iteration.

**Criteria**:
- Nice-to-have improvement
- Feature request disguised as issue
- Would require significant redesign
- Better addressed in a future version
- Enhancement rather than correction
- Design element that doesn't trace to any requirement
- IEEE viewpoint that's not relevant to this system
- Security hardening beyond stated threat model

**Action**: Document in "Future Considerations" or defer

**Note**: Use Requirement Traceability Review to identify design elements
without requirements—these are often scope creep indicators.

### ALREADY OK

The issue was already resolved or the content is already present in the document.

**Criteria**:
- Issue was fixed in a previous round
- Content already exists but reviewer missed it
- Behavior is already documented elsewhere
- Previous fix addressed this concern

**Action**: No change needed, note where it's already handled

### MERGED

The issue is combined with another issue because they share the same fix.

**Criteria**:
- Same underlying cause as another issue
- Single fix addresses multiple concerns
- Issues are duplicates or near-duplicates
- Fixing one automatically resolves the other

**Action**: Reference the primary issue number, apply fix once

---

## Review Categories

Organize your review by category to ensure comprehensive coverage.

### IEEE Viewpoint Review

Check that design viewpoints are properly addressed:

| Viewpoint | Review Questions |
|-----------|------------------|
| **Context** | Are system boundaries clearly defined? Are external actors identified? |
| **Composition** | Is the component decomposition logical and complete? |
| **Logical** | Are component responsibilities clearly defined? No overlapping duties? |
| **Dependency** | Are dependencies explicit? Any circular dependencies? |
| **Information** | Are data entities and relationships documented? |
| **Interface** | Are all APIs fully specified? Request/response formats clear? |
| **Interaction** | Is component communication documented? Sync vs async clear? |
| **State Dynamics** | Are state machines documented where applicable? |
| **Algorithm** | Are key algorithms described? Complexity noted? |
| **Structure** | Is code organization clear? Consistent with patterns? |
| **Patterns Use** | Are design patterns identified and justified? |
| **Resource** | Are resource requirements documented? |

**Issue Template**:
```markdown
### Issue N: Missing [Viewpoint] Coverage

**Location**: [Section or entire document]

**Problem**: The [viewpoint] viewpoint is not addressed. [Specific gap]

**Impact**: [What stakeholder concern is unaddressed]

**Verdict**: VALID / SCOPE CREEP

**Reasoning**: [Is this viewpoint relevant to this design?]
```

---

### HLD/LLD Review Criteria

Verify the appropriate level of detail for the document type:

#### High-Level Design (HLD) Review

| Criterion | Check |
|-----------|-------|
| Architecture completeness | All major components shown? |
| Technology justification | Stack choices explained? |
| Integration points | External systems identified? |
| Scalability approach | Growth strategy documented? |
| Trade-off documentation | Key decisions explained? |

**Common HLD Issues**:
- Too much implementation detail (belongs in LLD)
- Missing component interactions
- Undocumented technology choices
- Vague scalability statements

#### Low-Level Design (LLD) Review

| Criterion | Check |
|-----------|-------|
| Algorithm detail | Pseudocode or flowcharts provided? |
| Data structures | Classes/schemas fully specified? |
| Error handling | All error paths documented? |
| API contracts | Request/response examples complete? |
| Implementation feasibility | Can a developer implement from this? |

**Common LLD Issues**:
- Missing edge case handling
- Incomplete API specifications
- Vague algorithm descriptions
- Missing data validation rules

---

### Non-Functional Requirements Review

Verify quality attributes are specific and measurable:

| Category | Review Questions |
|----------|------------------|
| **Performance** | Are response times specific? (e.g., "< 200ms" not "fast") |
| **Scalability** | Are limits defined? (e.g., "10,000 users" not "scalable") |
| **Reliability** | Is uptime specified? RTO/RPO defined? |
| **Availability** | Are SLA targets stated? Failover documented? |
| **Maintainability** | Are code quality targets set? |
| **Observability** | Are logging, metrics, tracing specified? |

**Issue Template**:
```markdown
### Issue N: Vague [Category] Requirement

**Location**: Lines X-Y (Non-Functional Requirements section)

**Problem**: The requirement "[quote]" is not measurable.

**Impact**: Cannot write automated tests. No clear target for implementation.

**Verdict**: VALID

**Reasoning**: NFRs must be specific. Suggest: "[specific alternative]"
```

---

### Security Review Criteria

Verify security is addressed comprehensively:

| Aspect | Review Questions |
|--------|------------------|
| **Threat Model** | Are relevant threats identified? Mitigations documented? |
| **Authentication** | Is auth mechanism specified? Token handling clear? |
| **Authorization** | Are access controls defined? Role-based? Resource-based? |
| **Data Protection** | Is encryption specified? (in transit, at rest) |
| **Input Validation** | Are validation rules documented? Injection prevention? |
| **Secrets Management** | How are credentials stored and accessed? |
| **Audit Logging** | Are security events logged? |
| **Compliance** | Are regulatory requirements addressed? |

**Common Security Issues**:
- Missing threat model
- Vague authentication description ("uses OAuth" without details)
- No encryption specification
- Missing input validation rules
- Credentials in config examples

**Issue Template**:
```markdown
### Issue N: Incomplete Security Design

**Location**: [Section or N/A if missing]

**Problem**: [Specific security aspect] is not documented.

**Impact**: Security review cannot be completed. Implementation may have vulnerabilities.

**Verdict**: VALID

**Reasoning**: Security design is required for [type of system/data handled].
```

---

### Design Rationale Review

Verify decisions are documented and justified:

| Criterion | Check |
|-----------|-------|
| Key decisions identified | Are major architectural choices documented? |
| Alternatives listed | Were other options considered? |
| Rationale provided | Is "why" explained, not just "what"? |
| Trade-offs acknowledged | Are downsides of chosen approach noted? |
| Revisit conditions | When should decisions be reconsidered? |

**Common Rationale Issues**:
- Decisions stated without alternatives
- Missing "why" explanation
- No acknowledgment of trade-offs
- Outdated decisions not revisited

**Issue Template**:
```markdown
### Issue N: Undocumented Decision

**Location**: [Section where decision is implied]

**Problem**: The decision to use [technology/approach] is not explained.

**Impact**: Future maintainers won't understand why this choice was made.
Changes may inadvertently break design intent.

**Verdict**: VALID

**Reasoning**: Key decisions should include alternatives and rationale.
```

---

### Requirement Traceability Review

Verify requirements are mapped to design elements:

| Check | Question |
|-------|----------|
| Forward traceability | Does every requirement map to a design element? |
| Backward traceability | Does every design element trace to a requirement? |
| Test coverage | Are test cases linked to requirements? |
| Gap analysis | Are missing mappings identified? |
| Orphan detection | Are there design elements without requirements (scope creep)? |

**Common Traceability Issues**:
- Requirements with no corresponding design
- Design elements that don't trace to requirements (scope creep)
- Missing test case mappings
- Incomplete traceability matrix

**Issue Template**:
```markdown
### Issue N: Untraced [Requirement/Design Element]

**Location**: [Traceability matrix or relevant section]

**Problem**: [Requirement X] has no corresponding design element.
OR: [Design element Y] does not trace to any requirement.

**Impact**: Cannot verify requirement is addressed. OR: Potential scope creep.

**Verdict**: VALID / SCOPE CREEP

**Reasoning**: [Why this gap matters]
```

---

## Quality Bar: When to Report No Issues

A well-written design document may have no issues to report. This is a valid and desirable outcome.

**Do NOT feel obligated to find issues.** The goal is correctness, not quantity. If after a thorough read you find nothing significant, report:

```markdown
## Round N: No Issues Found

After reviewing the document, no issues warrant flagging.

**Areas Reviewed**:
- [List sections/aspects checked]

**Conclusion**: Document is internally consistent and ready for implementation.
```

**Signs of a healthy "no issues" verdict:**

*Consistency*
- All defaults are consistent across sections
- Examples match their documentation
- Schemas are aligned throughout
- All components mentioned are documented
- Code examples use correct syntax

*Completeness*
- All relevant IEEE viewpoints are addressed (or explicitly scoped out)
- HLD and LLD are clearly distinguished
- Stakeholders and concerns are identified
- Design rationale is provided for key decisions

*Quality Attributes*
- Non-functional requirements are specific and measurable
- Performance targets include measurement methods
- Scalability limits are defined
- Reliability requirements specify uptime/RTO/RPO

*Security*
- Threat model is documented
- Authentication/authorization is specified
- Data protection approach is defined
- Compliance requirements are addressed (if applicable)

*Testability*
- Acceptance criteria are specific and measurable
- Edge cases and error scenarios are defined
- Test data requirements are specified where needed

*Traceability*
- Requirements map to design elements
- Design elements trace to requirements
- No orphan elements (unexplained scope)

**Do NOT flag:**
- Stylistic preferences
- Minor wording that could be "slightly better"
- Suggestions that are enhancements rather than corrections
- Issues that require speculation about intent

---

## Decision Format

For each VALID issue, present **minimum 2-3 substantive options** before applying a fix. Avoid offering just "recommended fix" and "skip" - provide real alternatives.

```markdown
## Decision: [Issue Title]

### Options

**Option A: [Name]** (Recommended)
- Description: [What this approach does]
- Pros: [Benefits]
- Cons: [Drawbacks]

**Option B: [Name]**
- Description: [What this approach does]
- Pros: [Benefits]
- Cons: [Drawbacks]

**Option C: [Name]**
- Description: [What this approach does]
- Pros: [Benefits]
- Cons: [Drawbacks]

### Selected: Option [X]

**Rationale**: [Why this option was chosen]
```

### Batched Option Presentation

When presenting options for multiple issues, use a compact format:

```markdown
## Round N Decisions (5 VALID issues)

### Issue 1: Missing validation
- **A**: Add Zod schema validation (Recommended)
- **B**: Add manual type checks
- **C**: Add runtime assertions

### Issue 2: Inconsistent naming
- **A**: Use camelCase everywhere (Recommended)
- **B**: Use snake_case everywhere
- **C**: Keep mixed (document convention)

### Issue 3: Unclear error message
- **A**: Add context with variable values (Recommended)
- **B**: Add error codes for lookup
- **C**: Link to documentation

[User selects: 1-A, 2-A, 3-B]
```

### Example Decision

```markdown
## Decision: Clarify Template Override Syntax

### Options

**Option A: Add inline syntax description**
- Description: Add a note after the `--template` flag explaining `key=path` format
- Pros: Minimal change, keeps info close to usage
- Cons: Table cells become cluttered

**Option B: Add dedicated subsection**
- Description: Create "Template Override Syntax" section with examples
- Pros: Comprehensive, room for examples
- Cons: Adds verbosity, may be overkill

### Selected: Option A

**Rationale**: The `key=path` format is simple enough to explain inline,
and the resolution chain section already provides additional context.
```

---

## Summary Table Format

Track all issues and their disposition in a summary table:

```markdown
## Round N Summary

| Issue | Description | Verdict | Action |
|-------|-------------|---------|--------|
| 1 | Missing X in section Y | VALID | Fixed: added X |
| 2 | Unclear Z behavior | FALSE POSITIVE | Explained in section W |
| 3 | Add feature F | SCOPE CREEP | Defer to v2 |
| 4 | Inconsistent naming | VALID | Fixed: renamed to Y |
```

### Verification Section

After applying fixes, verify each change:

```markdown
## Verification

| Fix | Verification Method | Result |
|-----|---------------------|--------|
| Issue 1 | grep "X" in section Y | PASS |
| Issue 4 | Search for old name | PASS (0 occurrences) |
```

---

## Example: Gigachad Design Review

The gigachad standalone design document underwent multiple review rounds:

### Round 1
- **Issues Found**: 5
- **Valid**: 4 (CLI flag inconsistencies, missing schema details)
- **False Positive**: 1
- **All valid issues fixed**

### Round 2
- **Issues Found**: 5
- **Valid**: 5 (Remaining `-w` vs `-C` inconsistencies, schema clarifications)
- **All valid issues fixed**

### Round 3
- **Issues Found**: 8
- **Valid**: 6 (Template variables, config file consistency)
- **Scope Creep**: 2
- **All valid issues fixed**

### Round 4
- **Issues Found**: 20
- **Valid**: 9 (Documentation gaps, interface clarifications)
- **False Positive**: 5
- **Scope Creep**: 4
- **Already OK**: 2
- **All valid issues fixed using batch processing**

### Round 5
- **Issues Found**: 10
- **Valid**: 6 (Final polish items, edge cases)
- **False Positive**: 2
- **Already OK**: 1
- **Merged**: 1
- **All valid issues fixed**
- **Review complete - no new issues found in subsequent scan**

---

## Best Practices

1. **Read the entire document** before identifying issues - context matters

2. **Be specific with locations** - include line numbers when possible

3. **Distinguish between errors and preferences** - only VALID issues require fixes

4. **Verify fixes** - after applying changes, confirm they work as expected

5. **Document decisions** - future reviewers benefit from understanding rationale

6. **Limit scope per round** - focus on a specific number of issues to avoid fatigue

7. **Fresh eyes between rounds** - taking breaks improves issue detection

8. **Don't force issues** - If the document is solid, say so. Zero issues is success.

9. **Distinguish value from noise** - Ask: "Would fixing this meaningfully improve the document for its readers?" If not, skip it.

---

## Efficiency Tips

1. **Batch similar fixes together** - Group related changes (e.g., all naming fixes, all documentation additions) to reduce file read/write cycles

2. **Use grep to verify multiple changes at once** - Instead of verifying each fix individually, run a single verification pass at round end:
   ```bash
   grep -n "oldPattern\|newPattern" file.md
   ```

3. **Group related issues in summary** - When presenting issues, organize by category (consistency, completeness, clarity) for easier decision-making

4. **Front-load verdict assignment** - Assign all verdicts before presenting options; this prevents re-reading the document for each issue

5. **Use multi-select UI** - Present all VALID issue options at once using multi-question format instead of sequential prompts

6. **Track merged issues** - When two issues share a fix, mark one as MERGED to avoid duplicate work

7. **Skip verification for trivial fixes** - Typo corrections and simple word changes don't need grep verification

8. **Document as you go** - Update the summary table immediately after each fix while context is fresh

---

## When to Stop

The review loop terminates when:

- A round produces zero VALID issues (this is a success, not a failure to find issues)
- The reviewer explicitly determines the document meets quality standards
- All remaining issues are classified as SCOPE CREEP
- Time/iteration budget exhausted

**Important**: Zero issues is the goal state, not a problem. A well-written document should eventually reach this point. If you find yourself stretching to find issues, stop. The document is likely ready.

After completion, the document should be:
- Internally consistent
- Complete for its stated scope
- Clear and unambiguous
- Ready for implementation
