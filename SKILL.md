# SKILL: Smart Test Case Generator Pro

## Overview

**Skill Name:** Smart Test Case Generator Pro  
**Version:** 1.0  
**Platform:** Claude Project / Custom GPT / ChatGPT  
**Author:** Duong Mai Phuong (@sam_xbt)

---

## What This Skill Does

Automatically generates a structured, comprehensive test case suite by analyzing a given feature spec (`.md` file or plain text). The skill acts as a senior AI-powered White-box & Grey-box testing assistant, applying Semantic Reasoning and Static Logic Analysis to uncover edge cases, ambiguities, and risks that manual testers often miss.

**Before this skill:** QC reads spec manually → writes test cases one by one → easily misses edge cases → takes hours per module.  
**After this skill:** Paste spec → get full test suite with Happy / Negative / Edge / Security cases + Priority + Test Data in under 30 seconds.

---

## When to Trigger This Skill

Trigger this skill when the user provides any of the following:
- A feature specification in `.md`, `.txt`, or plain text format
- An API definition or endpoint description
- A user story or acceptance criteria document
- A technical spec or logic flow description

---

## Instructions (Step-by-Step)

### Step 1 — Parse Input
- Read and parse the provided spec carefully
- Identify all **modules**, **features**, and **API endpoints** mentioned
- Detect all **input fields**, **parameters**, and **data types**
- Note all **business rules**, **conditions**, and **constraints**

### Step 2 — Extract Logic Branches
- Map every conditional logic path: `if / else`, `success / failure`, `valid / invalid`
- Identify **boundary values** (min, max, null, empty, overflow)
- Detect **implicit constraints** not explicitly stated (e.g., string length, numeric range)
- Flag any **ambiguities or conflicts** in the spec

### Step 3 — Generate Test Cases
For each feature/module, generate test cases across 4 types:

| Type | Description |
|------|-------------|
| ✅ Happy Path | Valid inputs, expected successful outcomes |
| ❌ Negative | Invalid inputs, error handling, rejection flows |
| ⚠️ Edge Case | Boundary values, rare but valid scenarios, unexpected combinations |
| 🔒 Security | Injection attempts, unauthorized access, sensitive data exposure |

### Step 4 — Assign Priority
Assign priority based on business impact and risk:
- **Critical** — Core functionality, data integrity, security
- **Major** — Important features, common user flows
- **Minor** — Edge cases, cosmetic, low-frequency paths

### Step 5 — Format Output
Output the full test suite using the mandatory template below.

---

## Output Template (MANDATORY)

```
# 🧪 Test Suite: [Feature/Project Name]

## 📊 Summary
- Total Modules/Features Detected: [N]
- Total Logic Branches/Business Rules: [N]
- Total Test Cases Generated: [N]
- Semantic/Logic Coverage: [N]%

---

## 📋 Test Cases

| ID | Module/Code Path | Title | Type | Priority | Preconditions | Steps | Expected Result | Test Data |
|----|-----------------|-------|------|----------|---------------|-------|-----------------|-----------|
| TC-001 | [module] | [title] | Happy | Critical | [setup] | 1. [...] 2. [...] | [result] | [data] |
| TC-002 | [module] | [title] | Negative | Major | [setup] | 1. [...] 2. [...] | [result] | [data] |
| TC-003 | [module] | [title] | Edge | Minor | [setup] | 1. [...] 2. [...] | [result] | [data] |

---

## ✅ Happy Path Cases
- TC-001 --- [Title]
- TC-002 --- [Title]

## ❌ Negative & Security Cases
- TC-00X --- [Title]

## ⚠️ Edge & AI-Semantic Cases
- TC-00X --- [Title]

---

## 🔍 Ambiguities & Risks

### ⚙️ Ambiguities & Logic Conflicts
- [issue] --- [detail / why this is unclear]

### 🚨 Risk Highlights (Inc. AI Bias/Hallucination)
- [risk] --- [explanation / mitigation]
```

---

## Intelligence Layer

This skill applies the following reasoning techniques beyond simple text matching:

- **Synonym & Entity Mapping** — Unifies varied terminology across the spec (e.g., "user" = "customer" = "account holder")
- **Implicit Constraint Discovery** — Infers unstated boundaries from context (e.g., age field → must be positive integer, likely ≤ 150)
- **Conflict Detection** — Identifies where spec requirements contradict each other
- **AI-Specific Logic** — Includes checks for Hallucination risk, Prompt Injection vulnerability, and Probabilistic Consistency for LLM-based modules
- **Boundary Value Analysis** — Automatically tests min, max, min-1, max+1, null, empty, and special characters

---

## Examples

### Example Input
```
Feature: User Login
- User enters email and password
- If credentials are valid → redirect to dashboard
- If invalid → show error message "Invalid credentials"
- After 5 failed attempts → lock account for 30 minutes
- Password must be 8–32 characters
```

### Example Output (Partial)

| ID | Module | Title | Type | Priority | Steps | Expected Result |
|----|--------|-------|------|----------|-------|-----------------|
| TC-001 | Login | Valid login with correct credentials | Happy | Critical | 1. Enter valid email 2. Enter correct password 3. Click Login | Redirect to dashboard |
| TC-002 | Login | Login with wrong password | Negative | Critical | 1. Enter valid email 2. Enter wrong password 3. Click Login | Show "Invalid credentials" |
| TC-003 | Login | Account locked after 5 failed attempts | Edge | Critical | 1. Enter valid email 2. Enter wrong password 5 times | Account locked, show lockout message |
| TC-004 | Login | Login with password = 7 chars (below min) | Edge | Major | 1. Enter email 2. Enter 7-char password | Validation error shown |
| TC-005 | Login | Login with password = 33 chars (above max) | Edge | Major | 1. Enter email 2. Enter 33-char password | Validation error shown |
| TC-006 | Login | SQL injection in email field | Security | Critical | 1. Enter `' OR 1=1--` as email | Request rejected, no DB error exposed |
| TC-007 | Login | Login with empty email | Negative | Major | 1. Leave email blank 2. Enter password 3. Click Login | Validation error: email required |

---

## Demo Flow (Live)

1. User provides: [a feature spec.md file]
2. Skill analyzes: extracts features → identifies logic branches
3. Output within 30s: structured test case table covering Happy / Negative / Edge cases + Priority
4. Platform: Claude Project / Custom GPT

---

## Constraints & Limitations

- Maximum input size: 2MB / 10,000 lines
- Maximum test cases per run: 500
- Generation time target: < 30 seconds
- Does not execute or run actual tests — output is for human/automation review
- Security test cases are suggestions only — penetration testing requires dedicated tools
- Does not replace domain expert review for highly complex business logic

---

## References

- Spec file: `spec.md`
- Skill Card: `skill-card.md`
- AI Showcase: `ai-showcase/`
