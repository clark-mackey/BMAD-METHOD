# The 9 Skill Categories

From Anthropic's internal taxonomy (Thariq Shihipar, "Lessons from Building Claude Code: How We Use Skills"). Every skill should map to exactly one category. If it spans multiple, consider splitting it.

## 1. Library & API Reference
**Purpose:** Explain correct library, CLI, or SDK usage with reference code and gotchas.
**Examples:** "How to use our internal GraphQL client", "Supabase RLS patterns for this project"
**Key trait:** Heavy on code examples and edge cases. Light on workflow.

## 2. Product Verification
**Purpose:** Describe how to test and verify code, often paired with external tools like Playwright or pytest.
**Examples:** "Run E2E tests for checkout flow", "Visual regression testing skill"
**Key trait:** Includes specific test commands, expected outputs, and failure interpretation.

## 3. Data Fetching & Analysis
**Purpose:** Connect to data and monitoring stacks. Often includes credential management.
**Examples:** "Query our analytics dashboard", "Pull metrics from Grafana"
**Key trait:** Handles authentication, pagination, rate limits.

## 4. Business Process Automation
**Purpose:** Automate repetitive workflows into single commands.
**Examples:** "Generate weekly client report", "Onboard new team member"
**Key trait:** Multi-step orchestration. Often combines multiple tools.

## 5. Code Scaffolding & Templates
**Purpose:** Generate framework boilerplate for specific codebase patterns.
**Examples:** "Create new API endpoint with our standard structure", "Scaffold React component"
**Key trait:** Produces files that follow project conventions. Templates in templates/.

## 6. Code Quality & Review
**Purpose:** Enforce code quality standards and assist with code review.
**Examples:** "Review PR against team style guide", "Audit for accessibility violations"
**Key trait:** Has a checklist or rubric. Produces structured findings.

## 7. CI/CD & Deployment
**Purpose:** Handle building, pushing, and deploying code.
**Examples:** "Deploy to staging", "Run the release process"
**Key trait:** Often has destructive actions — needs safety guardrails.

## 8. Runbooks
**Purpose:** Take symptoms and produce structured investigation reports.
**Examples:** "Investigate slow API response times", "Debug failed email delivery"
**Key trait:** Diagnostic flow — starts with symptoms, works toward root cause.

## 9. Infrastructure Operations
**Purpose:** Perform routine maintenance with guardrails for destructive actions.
**Examples:** "Rotate API keys", "Scale database resources", "Clean up old deployments"
**Key trait:** High-risk operations. Must have confirmation steps and rollback plans.

## How to Classify

Ask: "What is the **primary value** this skill delivers?"

- If it **teaches** Claude about a tool → Library & API Reference
- If it **verifies** something works → Product Verification
- If it **retrieves** data → Data Fetching & Analysis
- If it **automates** a business workflow → Business Process Automation
- If it **generates** project-specific code → Code Scaffolding & Templates
- If it **evaluates** code quality → Code Quality & Review
- If it **ships** code → CI/CD & Deployment
- If it **diagnoses** problems → Runbooks
- If it **maintains** infrastructure → Infrastructure Operations

## Red Flag: Mixed Categories

If a skill does code scaffolding AND code review AND deployment, it's three skills wearing a trench coat. Split it. Each skill should have a single clear purpose that maps to one category.
