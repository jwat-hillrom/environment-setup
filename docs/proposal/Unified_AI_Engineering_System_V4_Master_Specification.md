
# Unified AI Engineering System V4 Master Specification

Version: 4.0
Status: Authoritative Source of Truth

## Executive Summary
A unified AI Engineering Operating System combining ECC, Matt Pocock Skills, Elephant-Goldfish Model, Hallmark, Impeccable, Anti-Slop, Better Icons, MCP infrastructure, VS Code, and GitHub Copilot.

Goals:
- High-quality engineering decisions
- Design-first development
- TDD-first implementation
- Strong documentation culture
- Organizational learning
- Multi-agent review
- Human approval for consequential changes

---
# Architecture Layers

L1 User Skills
L2 Workflow Orchestration (ECC)
L3 Specialist Agents
L4 MCP Infrastructure
L5 Repositories/Runtime/Services
L6 Memory Systems
L7 Governance & Metrics

---
# Canonical Skill Catalog

## Knowledge Skills
### /research
Investigates codebase, docs, dependencies, architecture, and external technologies before recommendations.
Outputs: viability, alternatives, risks, recommendation.

### /explain
Explains features, dependencies, architecture, protocols, workflows, services and decisions.
Must answer: What, Why, How, Where.

### /teach
Creates learning plans, exercises, examples, and comprehension checks.
Optimized for understanding, not task completion.

---
# Planning Skills

### /discover
Generates requirements, stakeholders, assumptions, constraints and risks.

### /design
Produces implementation-ready architecture specifications.
Includes ADR validation.

### /ui-design
Pipeline:
DESIGN.md -> Hallmark -> Better Icons -> Impeccable -> Anti-Slop

Outputs:
- UX architecture
- component hierarchy
- accessibility requirements
- icon mappings

---
# Delivery Skills

### /implement
Mandatory process:
1. Tests
2. Fail verification
3. Implementation
4. Refactor
5. Validation
6. Documentation update

### /testing
Evaluates:
- Unit tests
- Integration tests
- E2E tests
- Security tests
- Performance tests

Outputs:
Coverage, gaps, confidence score, missing scenarios.

---
# Review Skills

### /review-change
Matt-Pocock-style engineering review of a code change.
Focus:
- correctness
- maintainability
- hidden assumptions
- complexity
- design adherence

### /review
System-level review.
Focus:
- architecture
- reliability
- security
- scalability
- operational readiness

### /security-review
Threat modeling, auth review, dependency review, secrets review.

### /performance-review
Latency, scale, memory, rendering, query, and caching analysis.

### /docs-review
Mandatory before ship.
Checks:
- README
- onboarding guide
- architecture docs
- ADRs
- feature docs
- runbooks
- glossary
- code comments

Rule:
Comment non-obvious WHY, not obvious WHAT.

---
# Improvement Skills

### /investigate
Root cause analysis and evidence collection.

### /refactor
Architecture and maintainability improvements without behavior changes.

### /retro
Produces:
- wins
- lessons learned
- technical debt
- readiness score
- next recommended actions

---
# Communication Skills

### /commit
Generates:
- Conventional Commit title
- executive summary
- files changed count
- functions/classes changed
- validation evidence
- risk assessment

Mandatory Evidence:
- tests passed
- coverage delta
- static analysis status
- code smell delta
- security scan status

### /pr
Consumes:
- /testing
- /review-change
- /docs-review
- /commit

Produces reviewer-ready pull request package.

Contains:
- summary
- business value
- files changed
- review focus areas
- risk assessment
- testing evidence
- deployment plan
- rollback plan

### /jira-update
Creates non-technical stakeholder updates.
Audience:
PMs, Managers, Directors.

### /release-notes
Creates customer-facing and support-facing release notes.

---
# Governance Skills

### /document
Creates ADRs, runbooks, setup guides, architecture docs.

### /ship
Final approval gate.
Requires success from:
- testing
- docs-review
- review
- security-review
- ux-review

---
# Agent Catalog

## ECC Conductor
Owns orchestration, routing, memory, approvals.

## Product Analyst
Discovery, requirements, research.

## Solution Architect
System design and tradeoffs.

## Staff Engineer
Implementation planning.

## Senior Engineer
Coding and testing.

## UI Architect
UX architecture and component planning.

## Design Critic
Impeccable + Anti-Slop review.

## Security Engineer
Security and compliance.

## Data Architect
Schema and migration review.

## Goldfish Reviewer
Reviews with intentionally limited context.
Identifies ambiguity and assumptions.

## Principal Reviewer
Final engineering quality review.

---
# MCP Routing Matrix

Context7:
research, design, implement, review, teach

Filesystem:
all skills

Git:
research, review-change, investigate, commit, pr

Browser:
ui-design, ux-review, ship

Playwright:
testing, ux-review, ship

Database:
design, investigate, review

Future:
OpenAPI, Kubernetes, Terraform, Azure, AWS, Observability

---
# Memory Architecture

## Session Memory
Task-specific.

## Project Memory
Architectural decisions, standards, glossary, patterns.

## Organizational Memory
Cross-project reusable knowledge.

Promotion Rules:
Only validated and reusable knowledge enters Organizational Memory.

---
# ADR Lifecycle

Draft -> Review -> Approved -> Superseded -> Archived

Every significant architectural decision requires an ADR.

---
# Documentation Standards

Required Documents:
- README
- Architecture Overview
- Setup Guide
- Feature Documentation
- Runbook
- ADR Index
- Glossary
- Troubleshooting Guide

Success Criteria:
A new engineer can understand and run the project from docs alone.

---
# Human Approval Gates

Required Human Approval:
- Production deployment
- Authentication changes
- Authorization changes
- Database migrations
- Architecture changes
- Security exceptions

AI recommendations are never sufficient alone.

---
# Model Routing Strategy

GPT-5:
Orchestration, reasoning, architecture.

Claude:
Long-form reviews and documents.

Gemini:
Large-context repository analysis.

Local Models:
Privacy-sensitive workloads.

---
# Workflow State Machine

Idea
-> Discover
-> Research
-> Design
-> Goldfish Review
-> UI Design
-> Implement
-> Testing
-> Review-Change
-> Review
-> Docs Review
-> Commit
-> PR
-> Jira Update
-> Release Notes
-> Retro
-> Ship
-> Production

---
# Quality Gates

Discovery Gate
Design Gate
Implementation Gate
Testing Gate
Review Gate
Documentation Gate
Release Gate

Failure at any gate blocks promotion.

---
# Metrics

Engineering:
- Lead Time
- Deployment Frequency
- Change Failure Rate
- MTTR

AI:
- Acceptance Rate
- Rework Rate
- Review Findings
- Hallucination Rate

Design:
- Accessibility Score
- UX Score
- Consistency Score

Documentation:
- Documentation Coverage
- Onboarding Readiness

---
# Repository Structure

/docs
/adr
/designs
/requirements
/agents
/skills
/memory
/standards
/tests
/runbooks

Required Standards:
DESIGN.md
ENGINEERING.md
SECURITY.md
TESTING.md
DOCUMENTATION.md

---
# End-State Vision

Developers interact with a small, stable set of commands.
All complexity is hidden behind orchestration.
Knowledge is retained.
Quality is enforced through layered review.
Documentation stays current.
Design and engineering quality are both first-class concerns.
