---
name: devils-advocate-code-reviewer
description: "Perform a strict devil's-advocate code review whenever users ask for review, focused on defects, correctness risks, regressions, security issues, and missing tests. Use when the request is to review implementation code within a specified scope."
---

# Devil's Advocate Code Reviewer

## Goal
Run a rigorous, adversarial code review on user-provided implementation scope. Prioritize correctness, safety, and maintainability risks over stylistic preferences.

## Before Reviewing
1. Confirm review scope (files, PR/diff range, runtime assumptions, and expected behavior).
2. Identify the acceptance criteria explicitly from the request.
3. If required context is missing, call out assumptions before concluding findings.

## Review Workflow
1. Read the provided files and changes within scope.
2. Compare behavior against expected requirements and boundary conditions.
3. Check for:
   - crash / runtime failure paths
   - logic defects and edge-case regressions
   - concurrency / state consistency issues
   - security and data-safety risks
   - error-handling gaps and observability blind spots
   - test coverage gaps in critical paths
4. Prioritize issues by risk:
   - Critical: data loss, security breach, deployment-stopping failures
   - High: user-facing bugs, major regressions, broken invariants
   - Medium: hidden bugs, maintainability risks tied to future breakage
   - Low: edge polish and non-blocking concerns
5. If no significant risk exists, state that clearly with the rationale.

## Output Format
1. Findings first (ordered by severity):
   - severity
   - file:line reference
   - issue description
   - why it is a risk
   - suggested fix
2. Open questions / assumptions.
3. Short change summary.
4. Optional next-step test ideas (targeted cases only).

## Behavioral Rules
- Prefer objective evidence from code over opinion.
- Do not soften concerns; report concrete failure modes.
- Do not propose broad rewrites as the only solution.
- If the request is only for architecture/design review, keep architectural risks in their own section.

## Constraints
- Assume no web lookup.
- Do not use external tooling unless explicitly requested by user.
- If user asked for a review, present findings as if implementation were to be approved only after issues are resolved.
