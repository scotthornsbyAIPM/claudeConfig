---
name: shared-code-review
description: Structured code review skill. Use this when the user asks for a code review, wants feedback on their changes, or runs /shared-code-review. Reviews staged changes, a specific file, or a PR for correctness, security, readability, and improvements.
---

# Code Review Skill

You are performing a structured code review. Follow the steps below precisely.

## Step 1: Identify the scope

Determine what is being reviewed:
- If the user provided a file path or code snippet, review that.
- If the user referenced a PR number, use `gh pr diff <number>` to get the diff.
- If no scope was given, run `git diff HEAD` to review all uncommitted changes.

## Step 2: Analyse the code

Review the code across these four dimensions:

### Correctness
- Does the logic achieve the intended outcome?
- Are there off-by-one errors, null/undefined risks, or edge cases not handled?
- Are function inputs validated where necessary?

### Security
- Any SQL injection, XSS, or OWASP Top 10 vulnerabilities?
- Are secrets or credentials hardcoded?
- Are user inputs sanitised before use?

### Readability & Maintainability
- Are variable and function names clear and consistent?
- Is the code DRY (no unnecessary duplication)?
- Are there any functions doing too many things?

### Suggested Improvements
- Performance gains (unnecessary loops, redundant queries, etc.)
- Simplifications or cleaner patterns
- Missing tests for critical paths

## Step 3: Output format

Structure your review as follows:

## Code Review

### Summary
1-2 sentence overview of the changes and overall quality

### Correctness
- PASS / WARN / FAIL finding

### Security
- PASS / WARN / FAIL finding

### Readability
- PASS / WARN / FAIL finding

### Suggested Improvements
- specific, actionable suggestion with file:line reference where possible

### Verdict
LGTM / Needs minor changes / Needs significant changes
