# SKILL: Smart Test Case Generator Pro

## Overview

**Skill Name:** Smart Test Case Generator Pro  
**Version:** 2.0  
**Platform:** Claude Project / Custom GPT / ChatGPT  
**Author:** Duong Mai Phuong (@sam_xbt)

---

## What This Skill Does

Automatically generates a structured, comprehensive test case suite by analyzing a given feature spec (`.md` file or plain text). The skill acts as a senior AI-powered White-box & Grey-box testing assistant, applying Semantic Reasoning and Static Logic Analysis to uncover edge cases, ambiguities, and risks that manual testers often miss.

**Before this skill:** QC reads spec manually → writes test cases one by one → easily misses edge cases → takes hours per module.  
**After this skill:** Paste spec → get full test suite with Happy / Negative / Edge / Security cases + Gherkin format + Traceability + Coverage Matrix in under 30 seconds.

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
- Assign a **Requirement ID (REQ-XXX)** to each business rule found — used for Traceability

### Step 2 — Extract Logic Branches
- Map every conditional logic path: `if / else`, `success / failure`, `valid / invalid`
- Identify **boundary values** (min, max, null, empty, overflow)
- Detect **implicit constraints** not explicitly stated (e.g., string length, numeric range)
- Flag any **ambiguities or conflicts** in the spec

### Step 3 — Generate Test Cases (4 Types)

| Type | Description |
|------|-------------|
| ✅ Happy Path | Valid inputs, expected successful outcomes |
| ❌ Negative | Invalid inputs, error handling, rejection flows |
| ⚠️ Edge Case | Boundary values, rare but valid scenarios, unexpected combinations |
| 🔒 Security | Injection attempts, unauthorized access, sensitive data exposure |

### Step 4 — Write Gherkin Format
For every test case, include a **Given/When/Then** block:
```
Given [precondition / system state]
When [action performed]
Then [expected outcome]
And [additional assertion if needed]
```

### Step 5 — Add Traceability
Each test case must reference the **Requirement ID (REQ-XXX)** it covers, so reviewers can trace back to the original spec requirement.

### Step 6 — Build Coverage Matrix
After generating all test cases, produce a **Coverage Matrix** showing which requirements are covered by which test cases, and flag any uncovered requirements.

### Step 7 — Assign Priority
- **Critical** — Core functionality, data integrity, security
- **Major** — Important features, common user flows
- **Minor** — Edge cases, cosmetic, low-frequency paths

### Step 8 — Format Output
Output using the mandatory template below.

---

## Output Template (MANDATORY)

```
# 🧪 Test Suite: [Feature/Project Name]

## 📊 Summary
- Total Modules/Features Detected: [N]
- Total Requirements (REQ) Identified: [N]
- Total Test Cases Generated: [N]
- Semantic/Logic Coverage: [N]%

---

## 📋 Test Cases

| ID | REQ | Module | Title | Type | Priority | Preconditions | Steps | Expected Result | Test Data |
|----|-----|--------|-------|------|----------|---------------|-------|-----------------|-----------|
| TC-001 | REQ-001 | [module] | [title] | Happy | Critical | [setup] | 1.[...] 2.[...] | [result] | [data] |

---

## 🥒 Gherkin Scenarios

### TC-001 — [Title]
Given [precondition]
When [action]
Then [expected result]

### TC-002 — [Title]
Given [precondition]
When [action]
Then [expected result]
And [additional assertion]

---

## 🗺️ Coverage Matrix

| Requirement | Description | TC IDs | Status |
|-------------|-------------|--------|--------|
| REQ-001 | [description] | TC-001, TC-002 | ✅ Covered |
| REQ-002 | [description] | TC-003 | ✅ Covered |
| REQ-003 | [description] | — | ❌ Not Covered |

---

## ✅ Happy Path Cases
- TC-001 (REQ-001) --- [Title]

## ❌ Negative & Security Cases
- TC-00X (REQ-00X) --- [Title]

## ⚠️ Edge & AI-Semantic Cases
- TC-00X (REQ-00X) --- [Title]

---

## 🔍 Ambiguities & Risks

### ⚙️ Ambiguities & Logic Conflicts
- [issue] --- [detail]

### 🚨 Risk Highlights (Inc. AI Bias/Hallucination)
- [risk] --- [explanation / mitigation]
```

---

## Intelligence Layer

- **Synonym & Entity Mapping** — Unifies varied terminology across the spec
- **Implicit Constraint Discovery** — Infers unstated boundaries from context
- **Conflict Detection** — Identifies where spec requirements contradict each other
- **AI-Specific Logic** — Checks for Hallucination risk, Prompt Injection, Probabilistic Consistency
- **Boundary Value Analysis** — Tests min, max, min-1, max+1, null, empty, special characters
- **Traceability Engine** — Auto-assigns REQ IDs to every business rule for full trace coverage

---

## Example

### Input
```
Feature: User Login
- User enters email and password
- If credentials are valid → redirect to dashboard
- If invalid → show error "Invalid credentials"
- After 5 failed attempts → lock account for 30 minutes
- Password must be 8–32 characters
```

### Output (Partial)

**Requirements Extracted:**
- REQ-001: Valid credentials → redirect to dashboard
- REQ-002: Invalid credentials → show error message
- REQ-003: 5 failed attempts → lock account 30 minutes
- REQ-004: Password length must be 8–32 characters

**Test Cases:**

| ID | REQ | Module | Title | Type | Priority | Expected Result |
|----|-----|--------|-------|------|----------|-----------------|
| TC-001 | REQ-001 | Login/Auth | Valid login with correct credentials | Happy | Critical | Redirect to dashboard |
| TC-002 | REQ-002 | Login/Auth | Login with wrong password | Negative | Critical | Show "Invalid credentials" |
| TC-003 | REQ-003 | Login/Auth | Account locked after 5 failed attempts | Edge | Critical | Account locked, lockout message shown |
| TC-004 | REQ-004 | Login/Auth | Password = 7 chars (below min) | Edge | Major | Validation error shown |
| TC-005 | REQ-004 | Login/Auth | Password = 33 chars (above max) | Edge | Major | Validation error shown |
| TC-006 | REQ-001 | Login/Auth | SQL injection in email field | Security | Critical | Request rejected, no DB error exposed |

**Gherkin:**
```gherkin
# TC-001
Given an active user account exists
When the user enters a valid email and correct password (8-32 chars)
And clicks the Login button
Then the user is redirected to the dashboard

# TC-003
Given an active user account exists
When the user enters wrong password 5 consecutive times
Then the account is locked
And a lockout message is displayed for 30 minutes
```

**Coverage Matrix:**

| Requirement | Description | TC IDs | Status |
|-------------|-------------|--------|--------|
| REQ-001 | Valid login → dashboard | TC-001, TC-006 | ✅ Covered |
| REQ-002 | Invalid login → error message | TC-002 | ✅ Covered |
| REQ-003 | 5 failed attempts → lock 30min | TC-003 | ✅ Covered |
| REQ-004 | Password 8–32 chars | TC-004, TC-005 | ✅ Covered |

---

## Demo Flow (Live)

1. User provides: [a feature spec.md file]
2. Skill analyzes: extracts features → identifies logic branches → assigns REQ IDs
3. Output within 30s: test cases + Gherkin scenarios + Coverage Matrix
4. Platform: Claude Project / Custom GPT

---

## Constraints & Limitations

- Maximum input size: 2MB / 10,000 lines
- Maximum test cases per run: 500
- Generation time target: < 30 seconds
- Does not execute or run actual tests — output is for human/automation review
- Security test cases are suggestions only
- Does not replace domain expert review for complex business logic

---

## References

- Spec file: `spec.md`
- Skill Card: `skill-card.md`
- AI Showcase: `ai-showcase/`
