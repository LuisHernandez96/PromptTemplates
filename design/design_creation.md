# Design Document Creation Guide

This guide provides a template and best practices for creating design documents that are internally consistent, complete, and easy to review.

---

## Template Structure

```markdown
# [Project Name]: [Brief Description]

## Executive Summary
- 2-3 sentence overview of what the project does
- Key value proposition

## Requirements
- Runtime requirements (Python version, dependencies, etc.)
- External tool requirements

## Problem Statement
### Current Issues
- Numbered list of problems being solved
### Goals
- Numbered list of goals

## Architecture Overview
- ASCII diagram showing component relationships
- Brief explanation of each major component

## CLI/API Reference
- For each command/endpoint:
  - Description
  - Usage examples (bash code blocks)
  - Flags/parameters table (Flag | Default | Description)

## Configuration System
- Config file location and format
- Complete example config with comments
- Explanation of each setting

## Use Cases
- Numbered real-world scenarios
- Each with code examples

## Package/Project Structure
- Directory tree showing file organization
- Brief description of each major file/folder

## Future Considerations
- Features explicitly out of scope for v1
- Potential enhancements

## Testing Strategy
- Testable acceptance criteria for each requirement
- Unit test considerations
- Integration test considerations (when applicable)
- Test data requirements
- Edge cases and error scenarios
```

---

## Design Document Types

Design documents typically fall into two categories based on their level of detail:

### High-Level Design (HLD)

Describes **what** the system does - the big picture.

**Contents**:
- System architecture and major components
- Technology stack and platform choices
- External system integrations
- Data flow between components
- Key design decisions and trade-offs

**Audience**: Project stakeholders, architects, senior developers, operations teams

**When to use**: Initial design phase, architecture reviews, stakeholder alignment

### Low-Level Design (LLD)

Describes **how** to build it - implementation specifics.

**Contents**:
- Detailed module specifications
- Class diagrams and data structures
- Algorithm pseudocode
- Database schemas and queries
- API contracts with request/response examples
- Error handling strategies

**Audience**: Developers, code reviewers, QA engineers

**When to use**: Before implementation, during code reviews, for onboarding

### Choosing the Right Level

| Scenario | Recommended |
|----------|-------------|
| New system with multiple stakeholders | Both HLD and LLD |
| Small feature in existing system | LLD only |
| External team handoff | Both HLD and LLD |
| Internal prototype | HLD only |
| Complex algorithm implementation | LLD with algorithm details |

---

## Stakeholders and Concerns

Identify who will read and use the design document, and what each stakeholder cares about.

### Stakeholder Template

```markdown
## Stakeholders

| Stakeholder | Role | Primary Concerns |
|-------------|------|------------------|
| Product Manager | Decision maker | Does it meet requirements? Timeline? |
| Tech Lead | Approver | Architecture quality? Maintainability? |
| Developers | Implementers | Clear specs? Feasible? Testable? |
| QA Engineers | Validators | How to test? Edge cases? |
| DevOps | Operators | Deployment? Monitoring? Scaling? |
| Security | Reviewers | Vulnerabilities? Compliance? |
```

### Addressing Stakeholder Concerns

Each major section should map to stakeholder concerns:

| Section | Addresses Concerns For |
|---------|------------------------|
| Executive Summary | Product Manager, Leadership |
| Architecture Overview | Tech Lead, Developers |
| API Reference | Developers, QA |
| Security Design | Security, Compliance |
| Deployment | DevOps, Operations |
| Testing Strategy | QA, Developers |

---

## IEEE 1016 Design Viewpoints

IEEE 1016 defines standard viewpoints for comprehensive design documentation. Use this checklist to ensure complete coverage:

### Required Viewpoints

| Viewpoint | Purpose | Key Questions |
|-----------|---------|---------------|
| **Context** | System boundaries | What's inside vs outside the system? Who/what interacts with it? |
| **Composition** | Component decomposition | What are the major parts? How is the system divided? |
| **Logical** | Functional decomposition | What functions does each component perform? |
| **Dependency** | Component relationships | What depends on what? What's the build/deploy order? |

### Data-Oriented Viewpoints

| Viewpoint | Purpose | Key Questions |
|-----------|---------|---------------|
| **Information** | Data entities | What data is stored? What are the relationships? |
| **Interface** | External contracts | What APIs are exposed? What formats are used? |

### Behavioral Viewpoints

| Viewpoint | Purpose | Key Questions |
|-----------|---------|---------------|
| **Interaction** | Component communication | How do components talk to each other? Sync or async? |
| **State Dynamics** | State machines | What states exist? What triggers transitions? |
| **Algorithm** | Key algorithms | What are the core algorithms? Complexity? |

### Structural Viewpoints

| Viewpoint | Purpose | Key Questions |
|-----------|---------|---------------|
| **Structure** | Internal organization | How is code organized? Modules? Layers? |
| **Patterns Use** | Design patterns | What patterns are applied? Why? |
| **Resource** | Resource allocation | What resources are needed? Constraints? |

### Viewpoint Coverage Template

```markdown
## Design Viewpoint Coverage

| Viewpoint | Covered In | Notes |
|-----------|------------|-------|
| Context | Architecture Overview | See system diagram |
| Composition | Package Structure | Component breakdown |
| Logical | CLI Reference, Core sections | Per-component docs |
| Dependency | Architecture Overview | Arrows in diagram |
| Information | Database Design | ER diagram included |
| Interface | API Reference | Full endpoint docs |
| Interaction | Sequence diagrams | In Architecture section |
| State Dynamics | N/A | No complex state machines |
| Algorithm | Core Engine section | Search algorithm detailed |
| Structure | Package Structure | Directory layout |
| Patterns Use | Architecture Overview | Factory, Strategy noted |
| Resource | Non-Functional Requirements | Memory, CPU limits |
```

---

## Design Rationale

Document key decisions to help future maintainers understand why choices were made.

### Decision Record Template

```markdown
## Design Decisions

### Decision 1: [Decision Title]

**Context**: [What situation prompted this decision?]

**Options Considered**:
1. **Option A**: [Description]
   - Pros: [Benefits]
   - Cons: [Drawbacks]
2. **Option B**: [Description]
   - Pros: [Benefits]
   - Cons: [Drawbacks]
3. **Option C**: [Description]
   - Pros: [Benefits]
   - Cons: [Drawbacks]

**Decision**: Option [X]

**Rationale**: [Why this option was chosen over others]

**Trade-offs Accepted**: [What downsides were consciously accepted]

**Revisit Conditions**: [When should this decision be reconsidered]
```

### Example Decision Record

```markdown
### Decision 1: Message Queue vs Direct API Calls

**Context**: Need to handle async processing of large batch jobs

**Options Considered**:
1. **Direct API calls with polling**
   - Pros: Simple, no additional infrastructure
   - Cons: Client must manage retries, not scalable
2. **RabbitMQ message queue**
   - Pros: Mature, good tooling, durable
   - Cons: Additional infrastructure, learning curve
3. **Redis Streams**
   - Pros: Already using Redis for cache, fast
   - Cons: Less durable, fewer features

**Decision**: Option 2 (RabbitMQ)

**Rationale**: Durability is critical for batch jobs. Team has prior
RabbitMQ experience. The additional infrastructure cost is justified
by reliability requirements.

**Trade-offs Accepted**: Higher operational complexity, requires
RabbitMQ expertise for debugging.

**Revisit Conditions**: If message volume drops below 100/hour,
reconsider simpler polling approach.
```

---

## Section Details

### Executive Summary

Keep this brief (2-3 sentences). Answer:
- What does this project do?
- Who is it for?
- What's the key benefit?

```markdown
## Executive Summary

Gigachad is a CLI tool that generates AI-powered commit messages from staged git changes.
It targets developers who want consistent, descriptive commit messages without manual effort.
The tool integrates with existing git workflows and supports customizable templates.
```

### Requirements

List all dependencies with specific versions where it matters:

```markdown
## Requirements

### Runtime
- Python 3.10+
- `anthropic` SDK >= 0.25.0

### External Tools
- Git 2.0+
- Claude API key (set via `ANTHROPIC_API_KEY` environment variable)
```

### Problem Statement

Separate the problems from the goals:

```markdown
## Problem Statement

### Current Issues
1. Manual commit messages are inconsistent across team members
2. Developers spend time crafting messages instead of coding
3. Large commits lack structured summaries

### Goals
1. Generate consistent commit messages automatically
2. Support multiple output formats (conventional, detailed, brief)
3. Integrate seamlessly with existing git workflows
```

### Architecture Overview

Always include visual diagrams. Consider multiple diagram types:

#### Component Diagram

Show high-level system components and their relationships:

```markdown
## Architecture Overview

### System Components

┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│   CLI       │────▶│   Core       │────▶│   Claude    │
│  Interface  │     │   Engine     │     │     API     │
└─────────────┘     └──────────────┘     └─────────────┘
       │                   │
       │                   ▼
       │            ┌──────────────┐
       └───────────▶│   Config     │
                    │   Manager    │
                    └──────────────┘

- **CLI Interface**: Parses commands, validates input, formats output
- **Core Engine**: Orchestrates diff extraction, prompt building, response parsing
- **Claude API**: Handles API communication, retries, rate limiting
- **Config Manager**: Loads/merges config from files and CLI flags
```

#### Data Flow Diagram

Show how data moves through the system:

```markdown
### Data Flow

User Input → CLI Parser → Validator → Core Engine → API Client → External API
                                           ↓
                                      Response Parser → Formatter → Output
```

#### Deployment Diagram

For distributed systems, show deployment topology:

```markdown
### Deployment Architecture

┌─────────────────────────────────────────────────────────┐
│                     Production Environment               │
├─────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │
│  │  Web Server │  │  App Server │  │  Database   │     │
│  │  (nginx)    │──│  (Node.js)  │──│  (Postgres) │     │
│  └─────────────┘  └─────────────┘  └─────────────┘     │
│         │                │                              │
│         ▼                ▼                              │
│  ┌─────────────┐  ┌─────────────┐                      │
│  │    CDN      │  │   Cache     │                      │
│  │(CloudFlare) │  │  (Redis)    │                      │
│  └─────────────┘  └─────────────┘                      │
└─────────────────────────────────────────────────────────┘
```

#### Technology Stack Justification

Document why each technology was chosen:

```markdown
### Technology Stack

| Layer | Technology | Justification |
|-------|------------|---------------|
| Runtime | Python 3.10+ | Team expertise, rich ecosystem |
| HTTP Client | httpx | Async support, modern API |
| CLI Framework | Click | Declarative, well-documented |
| Configuration | Pydantic | Type safety, validation |
| Testing | pytest | Industry standard, good plugins |
```

### CLI/API Reference

For each command, provide:
1. Description
2. Usage example
3. Flags table with defaults

```markdown
## CLI Reference

### `generate`

Generates a commit message from staged changes.

**Usage**:
```bash
gigachad generate --format conventional
```

**Flags**:

| Flag | Default | Description |
|------|---------|-------------|
| `--format` | `conventional` | Output format: conventional, detailed, brief |
| `--model` | `claude-sonnet` | Claude model to use |
| `--dry-run` | `false` | Preview without committing |
```

### Configuration System

Show the complete config with all options:

```markdown
## Configuration System

Config file location: `~/.config/gigachad/config.yaml`

```yaml
# Model settings
model: claude-sonnet      # Options: claude-sonnet, claude-opus
max_tokens: 500           # Maximum tokens in response

# Output settings
format: conventional      # Options: conventional, detailed, brief
include_body: true        # Include detailed body after summary

# Git settings
auto_stage: false         # Auto-stage all changes before generating
```
```

### Use Cases

Number each scenario and include working examples:

```markdown
## Use Cases

### 1. Quick commit message

Generate and commit in one step:

```bash
git add .
gigachad generate --commit
```

### 2. Preview before committing

Review the generated message first:

```bash
gigachad generate --dry-run
# Review output, then:
gigachad generate --commit
```
```

### Package/Project Structure

Show the directory tree:

```markdown
## Package Structure

```
gigachad/
├── src/
│   ├── cli.py          # CLI entry point and argument parsing
│   ├── core.py         # Core generation logic
│   ├── config.py       # Configuration management
│   └── api.py          # Claude API wrapper
├── tests/
│   ├── test_cli.py
│   └── test_core.py
├── pyproject.toml
└── README.md
```
```

### Future Considerations

List what's explicitly out of scope:

```markdown
## Future Considerations

The following are out of scope for v1 but may be added later:

- **Multi-language support**: Currently English only
- **IDE plugins**: VS Code, JetBrains integrations
- **Team templates**: Shared template repositories
- **Offline mode**: Local model support
```

### Testing Strategy

Define how the implementation will be verified:

```markdown
## Testing Strategy

### Acceptance Criteria
Each requirement should have testable acceptance criteria:

| Requirement | Acceptance Criteria | Test Type |
|-------------|---------------------|-----------|
| Generate commit message | Given staged changes, returns message in <2s | Integration |
| Support conventional format | Output matches `type(scope): description` pattern | Unit |
| Handle empty diff | Returns user-friendly error message | Unit |

### Unit Test Coverage
- Core logic functions (parsing, formatting, validation)
- Configuration loading and merging
- Error handling paths

### Integration Test Coverage
- End-to-end CLI workflows
- API communication (with mocks)
- File system interactions

### Test Data Requirements
- Sample git diffs (small, medium, large)
- Various commit types (feat, fix, refactor, etc.)
- Edge cases: empty diff, binary files, large files

### Edge Cases and Error Scenarios
| Scenario | Expected Behavior |
|----------|-------------------|
| No staged changes | Error: "No changes staged for commit" |
| API rate limit | Retry with exponential backoff, max 3 attempts |
| Invalid API key | Error: "Invalid API key. Check ANTHROPIC_API_KEY" |
| Network timeout | Error: "Connection timeout. Check network." |
```

---

## Non-Functional Requirements

Define measurable quality attributes for the system.

### Performance Requirements

```markdown
## Non-Functional Requirements

### Performance

| Metric | Requirement | Measurement Method |
|--------|-------------|-------------------|
| Response Time | < 2 seconds for 95th percentile | Load testing with k6 |
| Throughput | 100 requests/second | Stress testing |
| Startup Time | < 500ms cold start | Benchmark script |
| Memory Usage | < 256MB peak | Profiling with memory_profiler |
```

### Scalability Requirements

```markdown
### Scalability

| Dimension | Current Target | Future Target | Scaling Strategy |
|-----------|----------------|---------------|------------------|
| Users | 1,000 concurrent | 10,000 concurrent | Horizontal pod scaling |
| Data Volume | 100GB | 1TB | Database sharding |
| Request Rate | 100 req/s | 1,000 req/s | Load balancer + caching |
```

### Reliability Requirements

```markdown
### Reliability

| Metric | Requirement | Implementation |
|--------|-------------|----------------|
| Uptime | 99.9% (8.76 hours downtime/year) | Multi-AZ deployment |
| Recovery Time (RTO) | < 1 hour | Automated failover |
| Recovery Point (RPO) | < 5 minutes | Continuous replication |
| Error Rate | < 0.1% of requests | Circuit breakers, retries |
```

### Other Quality Attributes

```markdown
### Maintainability
- Code coverage target: 80%
- Cyclomatic complexity: < 10 per function
- Documentation coverage: All public APIs

### Observability
- Structured logging (JSON format)
- Distributed tracing (OpenTelemetry)
- Metrics dashboard (Grafana)
- Alerting thresholds defined

### Compatibility
- Backward compatible with API v1 for 6 months
- Supported platforms: Linux, macOS, Windows
- Supported Python versions: 3.10, 3.11, 3.12
```

---

## Security Design

Address security concerns from the start of the design process.

### Threat Model

```markdown
## Security Design

### Threat Model

| Threat | Impact | Likelihood | Mitigation |
|--------|--------|------------|------------|
| API key exposure | High | Medium | Environment variables, secret manager |
| Injection attacks | High | Low | Input validation, parameterized queries |
| Data breach | High | Low | Encryption at rest and in transit |
| Unauthorized access | Medium | Medium | Authentication, authorization |
| Denial of service | Medium | Medium | Rate limiting, WAF |
```

### Authentication and Authorization

```markdown
### Authentication

| Mechanism | Use Case | Implementation |
|-----------|----------|----------------|
| API Keys | Service-to-service | Header: `X-API-Key` |
| JWT Tokens | User sessions | Bearer token, 1-hour expiry |
| OAuth 2.0 | Third-party integrations | Authorization code flow |

### Authorization

| Resource | Read | Write | Admin |
|----------|------|-------|-------|
| User Profile | Owner, Admin | Owner | Admin |
| Projects | Team members | Owner, Admin | Admin |
| System Config | Admin | Admin | Admin |
```

### Data Protection

```markdown
### Data Protection

| Data Type | Classification | Protection |
|-----------|----------------|------------|
| API Keys | Secret | Encrypted at rest, masked in logs |
| User PII | Confidential | Encrypted, access audited |
| Usage Metrics | Internal | Access controlled |
| Public Docs | Public | None required |

### Encryption
- In Transit: TLS 1.3 required
- At Rest: AES-256 for sensitive data
- Key Management: AWS KMS / HashiCorp Vault
```

### Compliance Considerations

```markdown
### Compliance

| Requirement | Applicability | Implementation |
|-------------|---------------|----------------|
| GDPR | EU users | Data deletion API, consent management |
| SOC 2 | Enterprise customers | Audit logging, access controls |
| HIPAA | Healthcare | N/A - not handling PHI |
```

---

## Risk Analysis

Identify and plan for technical risks.

### Risk Register Template

```markdown
## Risk Analysis

| Risk | Probability | Impact | Mitigation | Contingency |
|------|-------------|--------|------------|-------------|
| API provider outage | Low | High | Multi-provider fallback | Manual processing queue |
| Database corruption | Low | High | Daily backups, replication | Point-in-time recovery |
| Performance degradation | Medium | Medium | Monitoring, auto-scaling | Feature flags to disable expensive ops |
| Integration failure | Medium | Medium | Contract testing, mocks | Graceful degradation |
| Security vulnerability | Low | High | Dependency scanning, audits | Incident response plan |
```

### Risk Categories

```markdown
### Technical Risks
- Third-party API changes or deprecation
- Scaling limitations in current architecture
- Technical debt accumulation

### Integration Risks
- External system compatibility
- Data format mismatches
- Network reliability

### Performance Risks
- Unexpected load patterns
- Resource contention
- Cold start latency

### Security Risks
- Vulnerability in dependencies
- Credential exposure
- Insufficient access controls
```

---

## Deliverables Checklist

Use this checklist as exit criteria for the design phase.

```markdown
## Design Phase Deliverables

### Core Documents
- [ ] High-Level Design (HLD) document complete
- [ ] Low-Level Design (LLD) document complete (if applicable)
- [ ] Design decisions documented with rationale

### Architecture Artifacts
- [ ] System architecture diagram
- [ ] Component interaction diagrams
- [ ] Data flow diagrams
- [ ] Deployment architecture (if applicable)

### Technical Specifications
- [ ] API specifications (endpoints, request/response formats)
- [ ] Database design (schemas, ER diagrams)
- [ ] Configuration specifications
- [ ] Error handling specifications

### Quality Artifacts
- [ ] Non-functional requirements defined (performance, scalability)
- [ ] Security design documented
- [ ] Test strategy defined
- [ ] Risk analysis completed

### Stakeholder Alignment
- [ ] Stakeholder concerns addressed
- [ ] Design review completed
- [ ] Sign-off obtained from Tech Lead
- [ ] Sign-off obtained from Security (if applicable)

### Traceability
- [ ] Requirements mapped to design elements
- [ ] Design elements mapped to test cases
```

---

## Requirement Traceability

Ensure every requirement maps to design elements and tests.

### Traceability Matrix Template

```markdown
## Requirement Traceability Matrix

| Req ID | Requirement | Design Element | Test Cases | Status |
|--------|-------------|----------------|------------|--------|
| FR-001 | Generate commit message | Core Engine, API Client | TC-001, TC-002 | Designed |
| FR-002 | Support multiple formats | Formatter module | TC-003, TC-004 | Designed |
| FR-003 | Read config from file | Config Manager | TC-005 | Designed |
| NFR-001 | Response time < 2s | Caching layer | TC-010 | Designed |
| NFR-002 | 99.9% uptime | Multi-AZ deployment | TC-011 | Planned |
```

### Coverage Analysis

```markdown
### Coverage Summary

| Category | Total | Covered | Coverage |
|----------|-------|---------|----------|
| Functional Requirements | 15 | 15 | 100% |
| Non-Functional Requirements | 8 | 6 | 75% |
| Security Requirements | 5 | 5 | 100% |

### Gaps
- NFR-003 (Scalability): Design pending load test results
- NFR-004 (Recovery): DR plan in progress
```

### Orphan Detection

Design elements without requirements may indicate scope creep:

```markdown
### Orphan Analysis

| Design Element | Has Requirement | Notes |
|----------------|-----------------|-------|
| Admin Dashboard | No | Added for operational convenience |
| Metrics Export | No | Requested by DevOps, needs formal requirement |
```

---

## Consistency Rules

Follow these rules to avoid common review issues:

### 1. Single Source of Truth for Defaults

If a default value appears in multiple places, they must match.

**Bad**:
```markdown
## CLI Reference
| Flag | Default |
| `--format` | `conventional` |

## Configuration
format: detailed  # Default format
```

**Good**:
```markdown
## CLI Reference
| Flag | Default |
| `--format` | `conventional` |

## Configuration
format: conventional  # Default format (matches CLI default)
```

### 2. Complete Flag Coverage

Every flag used in examples must appear in the flags table.

**Bad**:
```bash
gigachad generate --format conventional --verbose
```
(But `--verbose` isn't in the flags table)

**Good**: Add all flags to the table, or only use documented flags in examples.

### 3. Consistent Syntax in Examples

Global options placement and flag ordering must be consistent throughout.

**Bad**:
```bash
# Example 1
gigachad --verbose generate --format conventional

# Example 2
gigachad generate --format conventional --verbose
```

**Good**: Pick one style and use it everywhere.

### 4. Schema Alignment

If output schemas appear in multiple sections, they must match exactly.

**Bad**:
```markdown
## API Reference
Returns: { "message": string, "tokens": number }

## Use Cases
Output: { "message": string, "token_count": number }
```

**Good**: Use identical field names everywhere.

### 5. Complete Component Coverage

Every component in the architecture diagram must have a documentation section.

**Bad**: Architecture shows 4 components, but only 3 are documented.

**Good**: Each component has a corresponding section explaining its role.

### 6. Example/Documentation Parity

If a section says "supports X, Y, Z", provide examples for all of them (or state which are shown).

**Bad**:
```markdown
Supports formats: conventional, detailed, brief

Example:
```bash
gigachad generate --format conventional
```
```

**Good**:
```markdown
Supports formats: conventional, detailed, brief

Examples:
```bash
# Conventional format (default)
gigachad generate --format conventional

# Detailed format with full explanation
gigachad generate --format detailed

# Brief one-liner
gigachad generate --format brief
```
```

### 7. Testable Acceptance Criteria

Every requirement must have specific, measurable acceptance criteria that can be verified with automated tests.

**Bad**:
```markdown
## Requirements
- The CLI should be fast
- Error messages should be helpful
```

**Good**:
```markdown
## Requirements
- Generate commit message in <2 seconds for diffs under 1000 lines
- Error messages include: error type, cause, and suggested fix
```

### 8. Test Coverage Completeness

Every documented feature or behavior should have corresponding test scenarios defined.

**Bad**:
```markdown
## Features
- Supports conventional, detailed, brief formats
- Handles rate limiting with retries

## Testing Strategy
- Test conventional format output
```

**Good**:
```markdown
## Features
- Supports conventional, detailed, brief formats
- Handles rate limiting with retries

## Testing Strategy
- Test conventional format output matches pattern
- Test detailed format includes body section
- Test brief format is single line
- Test retry logic triggers on 429 response
- Test max retries exceeded returns error
```

---

## Pre-Review Checklist

Before submitting for review, verify:

### Consistency
- [ ] All defaults are consistent across CLI docs, config examples, and prose
- [ ] Every flag in examples appears in the flags table
- [ ] All architecture components have documentation sections
- [ ] Output schemas match wherever they appear
- [ ] Examples use consistent syntax patterns
- [ ] Terminology is consistent throughout (see Common Pitfalls)

### Completeness
- [ ] HLD and LLD clearly distinguished (or single document scope stated)
- [ ] All IEEE viewpoints addressed or explicitly scoped out
- [ ] Stakeholders and their concerns identified
- [ ] Future considerations lists what's explicitly out of scope
- [ ] Design rationale documented for key decisions

### Quality Attributes
- [ ] Non-functional requirements are specific and measurable
- [ ] Performance targets defined with measurement methods
- [ ] Scalability limits and strategies documented
- [ ] Reliability requirements specified (uptime, RTO, RPO)

### Security
- [ ] Threat model documented
- [ ] Authentication and authorization approach defined
- [ ] Data protection strategy specified
- [ ] Compliance requirements addressed (if applicable)

### Testability
- [ ] Code examples are syntactically correct and runnable
- [ ] Acceptance criteria are specific and measurable (not vague like "fast" or "user-friendly")
- [ ] Edge cases and error scenarios are defined with expected behaviors
- [ ] Test data requirements are specified (if applicable)
- [ ] Every documented feature has corresponding test scenarios

### Traceability
- [ ] Requirements mapped to design elements
- [ ] Design elements mapped to test cases
- [ ] No orphan design elements (elements without requirements)

### Deliverables
- [ ] All required artifacts from Deliverables Checklist are complete
- [ ] Stakeholder sign-off obtained

---

## Common Pitfalls

### Pitfall 1: Documenting aspirations as features

Don't document features that aren't implemented. If something is planned but not built, put it in "Future Considerations".

### Pitfall 2: Inconsistent terminology

Pick one term and stick with it:
- "config file" vs "configuration file" vs "settings file"
- "flag" vs "option" vs "argument"
- "generate" vs "create" vs "produce"

### Pitfall 3: Orphaned references

If you mention "see Section X" or "as described above", verify that section exists and says what you claim.

### Pitfall 4: Version drift

When updating one section, search the document for related content that may need updating too.

### Pitfall 5: Missing error cases

Document what happens when things go wrong, not just the happy path.
