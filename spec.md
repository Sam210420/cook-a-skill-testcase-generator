# SKILL SPEC: Smart Test Case Generator

## 1. Context
QC/Testers spend significant time manually writing test cases from product specs.
Edge cases and negative cases are often missed.

## 2. Objective
Design an AI Skill that converts a spec.md file into a structured, production-ready test case list.

---

## 3. Input

Input format:
- Product specification in markdown (.md)
- May contain:
  - User stories
  - Feature descriptions
  - Acceptance criteria
  - API definition
  - Business rules

---

## 4. Output Requirements

Output must be structured as a table with:

| ID | Title | Type | Priority | Preconditions | Steps | Expected Result | Test Data |

Types must include:
- Happy path
- Negative case
- Edge case

Priority levels:
- Critical
- Major
- Minor

---

## 5. Workflow (Instruction for AI)

Step 1: Parse feature list from spec  
Step 2: Identify user flows  
Step 3: Generate happy path test cases  
Step 4: Generate negative cases  
Step 5: Detect edge cases (boundary values, null input, invalid format, etc.)  
Step 6: Identify ambiguities in spec and highlight them  
Step 7: Assign priority  
Step 8: Format into structured output table  

---

## 6. Intelligence Requirements

AI must:
- Detect missing acceptance criteria
- Highlight ambiguous logic
- Suggest additional test data
- Ensure coverage completeness

---

## 7. Example

### Example Input:
"User can register with email and password..."

### Example Output:
TC-001 | Register successfully | Happy | Major | ...
TC-002 | Invalid email format | Negative | Major | ...
TC-003 | Password boundary length 8 | Edge | Minor | ...

---

## 8. Export Format

Must support:
- Markdown table
- CSV-ready format
- Automation-ready format (Cypress / Playwright)

---

## 9. Limitations

- Cannot fully validate business logic without real system
- Requires reasonably structured spec

---

## 10. MVP Scope (Day 2)

Input: 1 spec.md  
Output: Full structured test case table + edge case detection

