
# Unified AI Engineering System V7
## Repository Specification & Bootstrap Architecture

Version: 7.0
Status: Implementation Repository Blueprint

# Purpose

Define the exact repository structure, configuration standards, agent contracts, skill templates, MCP integration architecture, CI/CD integration, memory layout, and bootstrap process required to create a deployable AI Engineering Operating System.

---
# Repository Layout

/agents
  /product-analyst
  /solution-architect
  /staff-engineer
  /senior-engineer
  /security-engineer
  /data-architect
  /ui-architect
  /design-critic
  /goldfish-reviewer
  /principal-reviewer

/skills
  /research
  /explain
  /teach
  /discover
  /design
  /ui-design
  /implement
  /testing
  /review-change
  /review
  /security-review
  /performance-review
  /docs-review
  /investigate
  /refactor
  /retro
  /commit
  /pr
  /jira-update
  /release-notes
  /document
  /ship

/docs
  /adr
  /features
  /architecture
  /runbooks
  /troubleshooting

/memory
  /session
  /project
  /organization

/mcp
/config
/templates
/tests
/ci
/standards

---
# Skill File Standard

Each skill contains:

SKILL.md
prompt.md
examples/
checklists/
workflows/

Frontmatter:

name:
description:
owner_agent:
required_inputs:
artifacts_produced:
quality_gates:
mcp_dependencies:
review_requirements:

---
# Agent Definition Standard

Each agent folder contains:

agent.md
persona.md
handoffs.md
review-rules.md

Every agent defines:
- Responsibilities
- Non-responsibilities
- Inputs
- Outputs
- Escalations
- Approval authority

---
# Agent Handoff Contracts

Product Analyst -> Solution Architect
Artifacts:
- requirements.md
- assumptions.md
- constraints.md
- risk-analysis.md

Solution Architect -> Senior Engineer
Artifacts:
- design-spec.md
- ADRs
- testing strategy

Senior Engineer -> Review Layer
Artifacts:
- code
- tests
- implementation notes

Review Layer -> Release Layer
Artifacts:
- review reports
- approval reports
- readiness scores

---
# MCP Integration Layer

Registry Format:

id:
name:
purpose:
capabilities:
required_skills:

Required MCPs:
- Context7
- Filesystem
- Git
- Browser
- Playwright
- Database

Optional MCPs:
- OpenAPI
- Terraform
- Kubernetes
- Azure
- AWS
- Observability

---
# Memory Storage Schema

Project Memory:
- glossary
- architecture summary
- ADR references
- standards
- reusable patterns

Organization Memory:
- postmortems
- lessons learned
- migration guides
- approved patterns

Memory Promotion Workflow:
Capture -> Review -> Approve -> Publish

---
# Workflow Definitions

Feature Workflow:
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

Bug Workflow:
/investigate
/implement
/testing
/review-change
/ship

---
# Review Engine

Scoring:
Engineering Score
Documentation Score
Security Score
Testing Score
UX Score

Thresholds:
90-100 Excellent
80-89 Acceptable
70-79 Conditional
<70 Fails Gate

---
# Commit Template

Title:
<type>(scope): summary

Body:
- executive summary
- files changed
- functions changed
- validation evidence
- risk assessment

---
# Pull Request Template

Summary
Business Value
Architecture Impact
Files Changed
Testing Evidence
Documentation Changes
Deployment Plan
Rollback Plan
Known Risks
Reviewer Focus Areas

---
# ADR Template

Title
Problem
Context
Alternatives
Decision
Consequences
Rollback
Approval

---
# Documentation Requirements

README
Architecture Overview
Developer Setup
Feature Guides
Runbooks
Troubleshooting
Glossary
ADR Index

/docs-review validates compliance.

---
# CI/CD Integration

Required Checks:
- build
- lint
- unit tests
- integration tests
- security scan
- docs-review
- review-change

PR merge blocked on failures.

---
# VS Code Integration

Commands registered:
/research
/explain
/teach
/discover
/design
/ui-design
/implement
/testing
/review-change
/review
/docs-review
/commit
/pr
/ship

---
# Organization Governance

Required Human Approval:
- auth changes
- schema changes
- production releases
- security exceptions

---
# Bootstrap Procedure

1. Create repository structure.
2. Install MCP servers.
3. Register agents.
4. Register skills.
5. Configure memory stores.
6. Configure CI rules.
7. Configure approval gates.
8. Run validation workflow.
9. Execute sample feature workflow.
10. Publish standards.

---
# Definition of Done

A repository is compliant when:
- all skills exist
- all agents exist
- MCP registry configured
- memory configured
- review gates active
- CI validation active
- documentation complete
- sample workflow passes end-to-end
