# Unified AI Engineering System V1 (Revised)

## Canonical Skills

### Knowledge Skills
- /research
- /explain
- /teach

### Planning Skills
- /discover
- /design
- /ui-design

### Delivery Skills
- /implement
- /testing

### Review Skills
- /review-change
- /review
- /security-review
- /performance-review
- /docs-review

### Improvement Skills
- /investigate
- /refactor
- /retro

### Communication Skills
- /commit
- /pr
- /jira-update
- /release-notes

### Governance Skills
- /document
- /ship

## Agent Model
ECC Orchestrator
- Product Analyst
- Solution Architect
- Senior Engineer
- Security Engineer
- Data Architect
- UI Architect
- Design Critic
- Goldfish Reviewer

## Critical Changes

### /explain
Explains repository architecture, dependencies, features, and decisions.

### /teach
Teaches concepts using examples, exercises and reasoning.

### /testing
Owns coverage analysis, test planning and readiness scoring.

### /docs-review
Validates documentation, code comments, ADRs, onboarding quality and feature documentation.

### /retro
Produces readiness, debt analysis and next-step recommendations.

### /commit
Produces conventional commit, evidence, impacted files/functions, risk assessment and validation results.

### /pr
Consumes outputs from testing, review-change and docs-review to build reviewer-ready PR packages.

### /review-change
Matt Pocock-style implementation review focused on maintainability, correctness and engineering quality.

## Workflow
/discover → /research → /design → /ui-design → /implement → /testing → /review-change → /review → /docs-review → /commit → /pr → /jira-update → /release-notes → /retro → /ship
