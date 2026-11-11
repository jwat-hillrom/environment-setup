# V9 Specification: Prototype & Replicate-UI Skills

## Overview

This document defines two Tier-1 design workflow skills:

- `/prototype`
- `/replicate-ui`

These skills exist between design and implementation.

Workflow:

```text
/discover
/research
/design
/ui-design

/prototype
/prototype-review

/replicate-ui
/replication-review

/implement
```

---

# /prototype

## Purpose

Generate, iterate, validate, and approve user experiences before implementation.

The objective is validating UX and workflows before engineering effort is invested.

## Agent

### Prototype Architect

Responsibilities:

- Concept generation
- UX exploration
- Flow modeling
- Interactive prototypes
- Design-system enforcement
- Handoff specifications

Non-Responsibilities:

- Production code
- API design
- Database design
- Backend implementation

## Inputs

Required:

- requirements.md
- design-spec.md
- ui-design-spec.md

Optional:

- DESIGN.md
- Figma references
- screenshots
- branding guidelines

## Dependencies

Agents:
- UI Architect
- Design Critic
- Goldfish Reviewer

Systems:
- Hallmark
- Better Icons
- Impeccable
- Anti-Slop

MCP:
- Browser MCP
- Playwright MCP
- Filesystem MCP

## Workflow

### Phase 1: Concept Generation

Generate three unique approaches:

- Option A
- Option B
- Option C

Each includes:

- Visual mockup
- UX philosophy
- Strengths
- Weaknesses
- Estimated implementation effort

### Phase 2: Iteration Loop

Users may provide unlimited feedback.

Examples:

- Move filters to sidebar
- Increase information density
- Simplify workflow
- Add approvals section

### Phase 3: Interactive Prototype

Generate:

- prototype.html

Requirements:

- Clickable
- Navigable
- Mock data
- No backend dependency
- No authentication dependency

### Phase 4: Prototype Review

Execute:

- Impeccable Review
- Anti-Slop Review
- Goldfish Review

Generate:

- prototype-review.md

### Phase 5: Prototype Approval

Human approval required.

Once approved, generate implementation-ready handoff artifacts.

## Artifacts

- prototype-options.md
- prototype.html
- prototype-review.md
- prototype-spec.md
- implementation-handoff.md

## Quality Gates

Must pass:

- WCAG 2.2 AA
- Design.md compliance
- Goldfish review
- Impeccable review
- Anti-Slop review

---

# /prototype-review

## Purpose

Perform structured design validation before implementation.

## Outputs

- Accessibility findings
- UX findings
- Design consistency findings
- Implementation concerns
- Approval recommendation

---

# /replicate-ui

## Purpose

Convert approved designs, Figma artifacts, screenshots, prototypes, or other reference imagery into application-specific implementation plans.

Focus:

- Fidelity
- Component reuse
- Design-system adherence
- Production feasibility

## Agent

### UI Replication Architect

Responsibilities:

- Image analysis
- Component mapping
- Design token mapping
- UI implementation planning
- Reuse optimization

Non-Responsibilities:

- Direct production deployment
- Architecture ownership

## Inputs

Required:

- Reference image(s)

Optional:

- Figma exports
- Prototype artifacts
- Existing application screenshots
- DESIGN.md
- Component library

## Dependencies

Agents:

- UI Architect
- Design Critic
- Solution Architect

Systems:

- Better Icons
- Hallmark
- Impeccable
- Anti-Slop
- imagegen-skills

MCP:

- Browser MCP
- Filesystem MCP
- Playwright MCP

## Workflow

### Phase 1: Visual Analysis

Identify:

- Layout hierarchy
- Navigation structure
- Forms
- Grids
- Tables
- Modals
- Actions
- Visual patterns

### Phase 2: Application Analysis

Scan:

- Existing components
- Existing pages
- Design tokens
- Styling systems

### Phase 3: Mapping

Produce:

- Reusable components
- New components required
- Missing design tokens
- Icon substitutions

### Phase 4: Approach Generation

Generate three paths:

#### Option A

Maximum reuse.

#### Option B

Balanced reuse and fidelity.

#### Option C

Maximum visual fidelity.

### Phase 5: Iteration

User feedback refines the selected option.

### Phase 6: Replication Specification

Generate implementation package.

## Artifacts

- ui-analysis.md
- component-mapping.md
- token-mapping.md
- ui-replication-spec.md
- implementation-plan.md

## Quality Gates

Must satisfy:

- Design system consistency
- Accessibility standards
- Component reuse goals
- Review approval

---

# /replication-review

## Purpose

Validate replication accuracy before implementation.

## Outputs

- Fidelity score
- Reuse score
- Accessibility findings
- Consistency findings
- Approval recommendation

---

# /visual-regression

## Purpose

Compare implemented UI against approved prototype or replicated design.

## Inputs

- Approved prototype
- Reference design
- Implemented UI

## Outputs

- Similarity score
- Layout differences
- Typography differences
- Spacing differences
- Accessibility regressions
- Recommended fixes

## Tooling

- Playwright MCP
- Browser MCP
- imagegen-skills

---

# Updated Design Lifecycle

```text
/discover
/research
/design
/ui-design

/prototype
/prototype-review

/replicate-ui
/replication-review

/implement
/testing
/review-change
/review
/docs-review
/commit
/pr
/release-notes
/retro
/ship
```
