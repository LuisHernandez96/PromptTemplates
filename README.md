# Prompt Templates

A collection of structured templates for AI-assisted software development workflows, covering the full lifecycle from design through implementation and testing.

## Structure

```
templates/
├── coding/
│   └── coding.md               # Implementation quality guidelines
├── design/
│   ├── design_creation.md      # Guide for creating design documents
│   └── design-review-workflow.md # Iterative design review process
├── plan/
│   ├── plan-decompose.md       # Decompose designs into backlog items
│   ├── plan-review.md          # Review backlog against INVEST criteria
│   └── plan-validation.md      # Validate review findings
└── testing/
    └── testing.md              # Test quality + TDD workflow guidance
```

## Templates

### Coding Templates

#### coding.md
Implementation quality guidelines including:
- Core principles (minimal implementation, YAGNI, single responsibility)
- Anti-patterns with BAD/GOOD examples (over-engineering, ignoring test boundaries)
- Security guidance (OWASP Top 10 checklist)
- Code quality guidelines (naming, function size, error handling)
- Pre-completion checklist

### Design Templates

#### design_creation.md
Comprehensive guide for creating design documents including:
- Template structure (Executive Summary, Requirements, Architecture, etc.)
- HLD vs LLD guidance
- IEEE 1016 viewpoint coverage
- Design rationale documentation
- Consistency rules and pre-review checklist

#### design-review-workflow.md
Iterative validation workflow for design documents:
- Batch processing approach for issue identification
- Verdict categories: VALID, FALSE POSITIVE, SCOPE CREEP, ALREADY OK, MERGED
- Review categories (IEEE viewpoints, NFRs, security, traceability)
- Decision format with 2-3 options per issue

### Plan Templates

#### plan-decompose.md
Decomposes validated designs into implementation tasks:
- Hierarchical backlog structure (Epic > Feature > Story > Task)
- 1:1 test-to-implementation task ratio
- Integration test threshold (2+ components)
- Story sizing guidelines (1-3 days)

#### plan-review.md
Reviews decomposed backlogs for quality:
- INVEST criteria validation (Independent, Negotiable, Valuable, Estimable, Small, Testable)
- Test coverage checks
- Requirements traceability
- Scope creep and duplicate detection

#### plan-validation.md
Validates findings from plan review:
- Confirms or rejects each finding
- Same verdict categories as design review
- Verification patterns for common scenarios

### Testing Templates

#### testing.md
Test quality guidelines and TDD workflow guidance:
- TDD phase discipline (RED/GREEN/REFACTOR)
- Prompting patterns for AI-assisted TDD
- Premature pass detection and investigation
- Anti-patterns (trivial assertions, over-mocking, weak comparisons)
- Best practices (test behavior not implementation, specific assertions)
- Mock gotchas and platform-specific considerations
- Test audit patterns and checklist

## Usage

These templates are designed to be used with AI assistants (like Claude) to guide structured software development workflows:

1. **Design Phase**: Use `design_creation.md` to create a design document, then `design-review-workflow.md` to validate it iteratively
2. **Planning Phase**: Use `plan-decompose.md` to break down the design into tasks, `plan-review.md` to validate the backlog, and `plan-validation.md` to confirm findings
3. **Implementation Phase**: Use `coding.md` for implementation quality guidelines and `testing.md` for TDD workflow and test quality

## License

MIT
