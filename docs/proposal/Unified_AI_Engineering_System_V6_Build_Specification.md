
# Unified AI Engineering System V6
## Build Specification & Reference Implementation

Version: 6.0
Status: Build Ready

# Purpose

This document defines the implementation details required to construct the AI Engineering Operating System.

It extends the V5 RFC by defining:
- Skill schemas
- Agent contracts
- Workflow definitions
- Context engineering
- Memory architecture
- MCP routing
- Review scoring
- Governance
- Repository layout
- Example implementations

# Skill Schema Standard

Every skill SHALL contain:

```yaml
name:
description:
owner_agent:
required_inputs:
optional_inputs:
required_artifacts:
quality_gates:
mcp_dependencies:
output_artifacts:
```

# Standard Skill Lifecycle

Request
→ Context Collection
→ MCP Discovery
→ Analysis
→ Artifact Generation
→ Review
→ Quality Gate
→ Delivery

# Agent Prompt Contract

Every agent must define:

## Identity
Purpose and responsibilities.

## Inputs
Artifacts and context.

## Constraints
Boundaries.

## Output Format
Structured outputs.

## Escalation Rules
When human review is required.

# Canonical Agents

## Product Analyst
Produces:
- requirements
- assumptions
- stakeholders
- risks

## Solution Architect
Produces:
- architecture
- ADRs
- integration plans

## Senior Engineer
Produces:
- code
- tests
- implementation notes

## Security Engineer
Produces:
- threat model
- security findings

## UI Architect
Produces:
- UX architecture
- component trees
- icon mappings

## Goldfish Reviewer
Produces:
- ambiguity reports
- assumption reports

# Workflow Definitions

## Feature Workflow

/discover
/research
/design
/ui-design
/implement
/testing
/review-change
/review
/docs-review
/commit
/pr
/jira-update
/release-notes
/retro
/ship

Blocking Gate Failure Stops Workflow.

## Bug Workflow

/investigate
/implement
/testing
/review-change
/ship

## Refactor Workflow

/research
/design
/refactor
/testing
/review
/docs-review
/ship

# Context Engineering Specification

Priority:
1. User Request
2. Active Artifacts
3. Standards
4. ADRs
5. Repository
6. Project Memory
7. Organizational Memory
8. External Research

Injected Context Must Include:
- architecture summary
- glossary
- standards
- relevant ADRs
- active feature artifacts

# Memory Specification

## Session Memory
TTL: Current workflow.

## Project Memory
Stores:
- ADRs
- glossary
- architecture decisions
- standards

## Organizational Memory
Stores:
- reusable solutions
- postmortems
- migration lessons

Promotion Policy:
Reviewed + validated + reusable.

# MCP Capability Registry

## Context7
Framework research.

## Filesystem
Repository understanding.

## Git
Change analysis.

## Browser
Runtime validation.

## Playwright
Testing and screenshots.

## Database
Schema analysis.

Future:
- OpenAPI
- Kubernetes
- Terraform
- Azure
- AWS
- Observability

# Review Scoring Model

## Engineering Score
- correctness
- maintainability
- complexity
- testing

Range: 0-100

## Documentation Score
- onboarding
- architecture docs
- feature docs
- runbooks
- glossary

Range: 0-100

## UX Score
- accessibility
- consistency
- usability

Range: 0-100

# Commit Specification

Required Sections:
- conventional commit
- executive summary
- file count
- functions changed
- evidence
- risk assessment

Evidence:
- tests passed
- coverage delta
- static analysis
- code smell delta
- security status

# PR Specification

Required Sections:
- executive summary
- business outcome
- architecture impact
- files changed
- reviewer focus areas
- testing evidence
- documentation changes
- deployment plan
- rollback plan
- risks

# Documentation Standard

Required:
- README
- CONTRIBUTING
- ARCHITECTURE
- ADR Index
- Feature Docs
- Runbooks
- Troubleshooting
- Glossary

Comment Rule:
Explain intent, rationale, constraints.
Never explain obvious code.

# ADR Template

Problem
Context
Options Considered
Decision
Consequences
Rollback Strategy

# Repository Template

/agents
/skills
/docs
/docs/adr
/docs/features
/docs/runbooks
/docs/troubleshooting
/memory/project
/memory/org
/tests
/standards

# Governance

Human Approval Required:
- auth changes
- authorization changes
- schema changes
- production deploys
- security exceptions

# Model Routing Engine

GPT-5
- orchestration
- architecture
- reasoning

Claude
- reviews
- long documentation

Gemini
- massive repository analysis

Local Models
- regulated environments

# Skill Dependency Graph

/review-change depends on:
- implement
- testing

/pr depends on:
- commit
- docs-review
- review-change
- testing

/ship depends on:
- review
- docs-review
- testing
- security-review

# Bootstrap Checklist

1. Install MCP servers
2. Configure ECC
3. Register agents
4. Register skills
5. Configure memory stores
6. Configure review rules
7. Configure approval gates
8. Validate workflow execution

# End-to-End Success Definition

A new engineer can:
- understand the system
- locate architecture decisions
- implement features
- understand code rationale
- safely deploy
- contribute documentation

without requiring tribal knowledge.
