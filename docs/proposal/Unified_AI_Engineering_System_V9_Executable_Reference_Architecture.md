
# Unified AI Engineering System V9
## Executable Reference Architecture

Version: 9.0
Status: Executable Reference Architecture

# Objective

This document defines the final reference architecture capable of being translated directly into:

- ECC workflows
- Agent definitions
- Skill implementations
- MCP manifests
- Memory stores
- CI/CD pipelines
- VS Code integrations
- Repository generators

The system is designed such that every architectural component has an executable counterpart.

---
# Reference Repository Layout

/platform
  /agents
  /skills
  /workflows
  /memory
  /mcp
  /governance
  /templates
  /pipelines
  /scoring
  /examples

---
# Agent Runtime Contract

Every agent MUST provide:

agent.yaml
prompt.md
outputs.json
handoffs.yaml

Required Fields:
- name
- version
- objective
- responsibilities
- constraints
- escalation_rules
- required_artifacts
- output_artifacts

---
# Executable Agent Definitions

## Product Analyst
Inputs:
- business request
- existing documentation

Outputs:
- requirements.md
- assumptions.md
- constraints.md
- stakeholder-map.md
- risk-analysis.md

## Solution Architect
Outputs:
- design-spec.md
- architecture-diagram.md
- integration-plan.md
- adr.md

## Senior Engineer
Outputs:
- implementation-plan.md
- source code
- tests

## Security Engineer
Outputs:
- threat-model.md
- security-findings.md

## Goldfish Reviewer
Outputs:
- ambiguity-report.md
- assumption-report.md

## Design Critic
Outputs:
- ux-report.md
- accessibility-report.md
- anti-slop-report.md

---
# Skill Manifest Standard

Each skill contains:

skill.yaml
prompt.md
workflow.yaml
checks.yaml
examples/

Example:

name: explain
category: knowledge
owner: product-analyst
inputs:
  - question
outputs:
  - explanation.md
quality_gates:
  - architecture_verified

---
# Reference Skill Library

Knowledge:
- research
- explain
- teach

Planning:
- discover
- design
- ui-design

Delivery:
- implement
- testing

Review:
- review-change
- review
- security-review
- performance-review
- docs-review

Improvement:
- investigate
- refactor
- retro

Communication:
- commit
- pr
- jira-update
- release-notes

Governance:
- document
- ship

---
# Workflow DSL

workflow:
  id: feature
  stages:
    - discover
    - research
    - design
    - ui-design
    - implement
    - testing
    - review-change
    - review
    - docs-review
    - commit
    - pr
    - retro
    - ship

Gate failures halt execution.

---
# Artifact Registry

requirements.md
research-report.md
design-spec.md
adr.md
ui-spec.md
implementation-plan.md
test-plan.md
review-report.md
docs-report.md
commit-report.md
pr-package.md
retro-report.md
release-notes.md

Each artifact includes:
- owner
- version
- status
- reviewers

---
# Context Assembly Engine

Sources:
1. User request
2. Active workflow artifacts
3. Standards
4. ADRs
5. Repository context
6. Project memory
7. Organizational memory
8. External references

Context budget managed automatically.

---
# Memory Schemas

Session Memory
- active task
- temporary findings

Project Memory
- glossary
- architecture
- ADR references
- standards

Organization Memory
- patterns
- lessons learned
- migration guidance

---
# MCP Manifest Format

mcp:
  id:
  capabilities:
  authentication:
  routing_rules:
  required_by:

Required MCPs:
- Context7
- Filesystem
- Git
- Browser
- Playwright
- Database

---
# Review & Scoring Engine

Scores:
- Engineering
- Testing
- Documentation
- Security
- UX
- Readiness

Scale:
0-100

Release Threshold:
85+

---
# Commit Generator Specification

Produces:
- conventional commit
- executive summary
- files changed
- functions changed
- evidence
- risk assessment

Mandatory Evidence:
- tests
- coverage
- static analysis
- security status

---
# Pull Request Generator Specification

Produces:
- summary
- business outcome
- architecture impact
- testing evidence
- reviewer guidance
- deployment plan
- rollback plan

---
# Governance Engine

Human approvals required:
- auth
- authorization
- schema changes
- production deployment
- security exceptions

---
# CI/CD Pipeline Definitions

Pipeline Stages:
1. Build
2. Lint
3. Tests
4. Security
5. Docs Review
6. Review Validation
7. Release Readiness

---
# VS Code Integration

Generated Assets:
- commands
- prompt entries
- tasks
- snippets

---
# Generator Contract

Input:
system.yaml

Output:
Fully generated repository.

Validation:
- workflow integrity
- skill integrity
- handoff integrity
- MCP integrity
- governance integrity

---
# Example Execution

Feature Request
→ Discover
→ Research
→ Design
→ Goldfish Review
→ Implement
→ Testing
→ Review
→ Docs Review
→ PR
→ Retro
→ Ship

All outputs become governed artifacts.

---
# Exit Criteria

A generated repository must:
- execute every workflow
- pass all validation checks
- provide auditable artifacts
- support onboarding without tribal knowledge
- support continuous organizational learning
