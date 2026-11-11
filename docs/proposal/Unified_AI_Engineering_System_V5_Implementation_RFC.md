
# RFC-0001: Unified AI Engineering Operating System

Status: Draft
Version: 5.0
Supersedes: V4 Master Specification
Audience: Platform Engineering, Developer Experience, AI Engineering, Architecture Review Board

# 1. Purpose

This RFC defines the implementation architecture for a production-ready AI Engineering Operating System.

Goals:
- Deterministic workflows
- Consistent engineering quality
- Repeatable architecture decisions
- Strong documentation culture
- Human-governed approvals
- Organization-wide learning

# 2. System Components

## Orchestration Layer
ECC Conductor

Responsibilities:
- State management
- Workflow execution
- Agent routing
- Context packaging
- Memory access
- Approval coordination

## Skill Layer

Knowledge:
- /research
- /explain
- /teach

Planning:
- /discover
- /design
- /ui-design

Delivery:
- /implement
- /testing

Review:
- /review-change
- /review
- /security-review
- /performance-review
- /docs-review

Improvement:
- /investigate
- /refactor
- /retro

Communication:
- /commit
- /pr
- /jira-update
- /release-notes

Governance:
- /document
- /ship

# 3. Agent Contracts

## Product Analyst
Inputs:
- business request
- repository context

Outputs:
- requirements.md
- assumptions.md
- constraints.md
- risk-analysis.md

Exit Criteria:
- measurable requirements exist

## Solution Architect
Inputs:
- requirements
- research artifacts

Outputs:
- design-spec.md
- ADR draft

Exit Criteria:
- architecture review passed

## Senior Engineer
Inputs:
- approved design

Outputs:
- code
- tests
- implementation notes

Exit Criteria:
- tests green

## Goldfish Reviewer
Inputs:
- specification or change set

Outputs:
- ambiguity report
- missing assumptions

Exit Criteria:
- no critical ambiguities

# 4. Context Engineering Strategy

Priority Order:
1. User request
2. Active workflow artifact
3. Project standards
4. ADRs
5. Repository code
6. Organizational memory
7. External research

Context Budget Allocation:
- 40% active task
- 25% design artifacts
- 20% codebase context
- 10% memory
- 5% external references

# 5. Memory Architecture

## Session Memory
TTL: workflow duration

## Project Memory
Contains:
- glossary
- ADRs
- coding standards
- architecture patterns

## Organizational Memory
Contains:
- validated reusable patterns
- postmortems
- migration lessons

Promotion Requirements:
- reviewed
- validated
- reusable across projects

# 6. MCP Invocation Policy

Filesystem MCP:
Mandatory before recommendations involving existing code.

Git MCP:
Mandatory for commit, PR, review-change, and investigate workflows.

Context7:
Mandatory when discussing libraries/frameworks.

Playwright:
Mandatory for UX review and release validation.

# 7. Workflow State Machines

Feature:
Idea -> Discover -> Research -> Design -> Goldfish Review -> Implement -> Testing -> Review Change -> Review -> Docs Review -> PR -> Ship -> Production

Bug:
Issue -> Investigate -> Root Cause -> Fix -> Testing -> Review Change -> Ship

Refactor:
Analyze -> Design -> Refactor -> Testing -> Review -> Ship

# 8. Skill Specifications

## /review-change
Owner: Principal Engineer Persona

Checks:
- correctness
- complexity
- maintainability
- design adherence

Severity:
- Critical
- High
- Medium
- Low

## /docs-review
Scores:
- onboarding readiness
- feature documentation
- architecture coverage
- glossary completeness

Fail Conditions:
- undocumented major feature
- missing setup path
- undocumented operational dependencies

## /testing
Outputs:
- coverage delta
- risk score
- missing scenarios
- readiness score

# 9. Commit Specification

Required Output:
- conventional commit title
- executive summary
- file count
- functions changed
- evidence section
- risk assessment

Evidence Section:
- unit tests
- integration tests
- e2e tests
- coverage delta
- static analysis
- code smell delta
- security status

# 10. Pull Request Specification

Sections:
- executive summary
- business impact
- architecture impact
- files changed
- reviewer focus areas
- testing evidence
- documentation changes
- deployment plan
- rollback plan
- known risks

# 11. Documentation Standards

Repository Minimum:
- README
- CONTRIBUTING
- ARCHITECTURE
- ADR INDEX
- FEATURE DOCS
- RUNBOOKS
- TROUBLESHOOTING
- GLOSSARY

Code Comment Rules:
Comment:
- intent
- rationale
- constraints

Do Not Comment:
- obvious language constructs

# 12. Governance

Human Approval Required:
- auth changes
- authorization changes
- schema changes
- production deployment
- security exceptions

# 13. Model Routing

GPT-5:
- orchestration
- architecture
- reasoning

Claude:
- document generation
- reviews

Gemini:
- large repository analysis

Local Models:
- private codebases
- regulated environments

# 14. Repository Bootstrap

/agents
/skills
/docs
/docs/adr
/docs/features
/docs/runbooks
/memory/project
/memory/org
/standards
/tests

# 15. Metrics

Engineering:
- lead time
- deployment frequency
- MTTR
- change failure rate

AI:
- acceptance rate
- review rejection rate
- hallucination rate

Documentation:
- onboarding score
- docs coverage

# 16. Success Criteria

A new engineer should be able to:
- understand the system
- run the project
- trace major features
- deploy safely
- troubleshoot common failures

using documentation and skills with minimal tribal knowledge.
