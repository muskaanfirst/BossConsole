# Automated Repository Security & Code Quality Audit Workflow

This workflow provides a standardized instruction set for BossConsole AI agents to perform automated security, code-quality, and vulnerability scans across project codebases.

## Overview

This is a reusable prompt for an agent session, not an installed scanner or scheduled GitHub Actions workflow. Give the agent this document and the repository root to review. Follow the repository's `AGENTS.md` instructions and the user's authorized scope. Review source without running the application, changing code, or contacting live services; propose fixes in the report.

By executing this workflow, the AI agent autonomously navigates the project directory, flags security vulnerabilities or unhandled runtime exceptions, and formats the findings into a clean, actionable markdown report.

## Workflow Execution Stages

### Stage 1: File Discovery & Environment Isolation
- **Scope Target**: Enumerate source files within the selected repository, respecting ignore rules. Include relevant languages and configuration (for example `.kt`, `.kts`, `.java`, `.py`, `.ts`, `.json`, `.sql`, and `.yml`); record the actual scope and any omitted areas. Do not traverse other repositories or follow symlinks outside the selected root.
- **Boundary Rules**: Read `.gitignore` to understand exclusions. Exclude ignored, generated, and sensitive local files, including `.env` and its variants, private keys, `secrets.json`, `local.properties`, `.git/`, `build/`, `.gradle/`, and `node_modules/`. Do not bypass exclusions to search for secrets.
- **Verification Check**: Verify that file read permissions are granted before proceeding to execution.

### Stage 2: Security & Quality Analysis
The agent evaluates the codebase against three core audit rules:

1. **Credential & Secret Exposure Check**
   - Identify suspected hardcoded tokens, secret keys, and passwords in in-scope source. Distinguish public identifiers and test placeholders from credentials; a private endpoint alone is not proof of a vulnerability.
   - Never copy credential values into tool output, reports, patch snippets, or public issues. Use `[REDACTED]` with a file location and describe the exposure without testing the credential against a service.
2. **Exception Handling & Fault Tolerance**
   - Flag empty or swallowed `catch` blocks, missing null checks, and unhandled async/promise rejections.
3. **Performance & Anti-Pattern Detection**
   - Highlight redundant resource allocations, inefficient loop nesting, and unclosed stream connections.

### Stage 3: Structured Report Generation
Validate each candidate against its call sites and existing safeguards before reporting it. Give evidence and an affected scenario; distinguish confirmed findings from hypotheses and code-quality suggestions. Classify severity by demonstrated impact and likelihood, rather than by a pattern match alone.

Compile the findings into a new local file named `AUDIT_REPORT.md` in the selected project root using the structure below. If it already exists, use a unique suffixed filename instead of overwriting it. Keep the report local and review it for sensitive content before sharing or committing. State that this is a source review with limited coverage, not a guarantee that the repository is secure.

````markdown
# Repository Audit Summary

## Executive Summary
- **Repository / Revision**: [Root and commit]
- **Scope / Exclusions**: [Reviewed areas, skipped files, and limitations]
- **Validation Performed**: [Checks actually run, or source inspection only]
- **Total Files Scanned**: [Count]
- **Critical Vulnerabilities**: [Count]
- **Warnings / Code Smells**: [Count]

## Findings Breakdown

### 1. [Issue Title]
- **Severity**: [CRITICAL / HIGH / MEDIUM / LOW]
- **File Location**: `path/to/file.ext` (Line: [Line Number])
- **Confidence**: [Confirmed / Needs validation]
- **Description**: Brief explanation of the risk or code smell.
- **Evidence / Affected Scenario**: Explain the reachable path and existing safeguards, with all secrets redacted.
- **Recommended Remediation**:
  ```text
  // Proposed code fix or patched implementation
  ```
````
