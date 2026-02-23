# SKILL SPEC
## Smart Test Case Generator Pro

---

## 1. Context

In modern software development, QC/Testers spend significant time manually analyzing product specifications and writing test cases.  
Manual writing often leads to:

- Missed edge cases
- Inconsistent format
- Lack of priority classification
- Poor coverage visibility
- Manual reporting effort after testing

This skill aims to encode professional QA thinking into a structured AI workflow.

---

## 2. Objective

Design an AI Skill that:

Input: A structured product specification file (spec.md)  
Output: A complete, structured, production-ready test case suite with coverage analysis and report-ready format.

The skill must simulate how a senior QA analyzes a feature.

---

## 3. Target Users

- QC / Tester
- QA Engineer
- QA Manager
- Product Owner (for spec review)

---

## 4. Input Specification

The skill accepts:

- Markdown spec (.md)
- May include:
  - Feature descriptions
  - User stories
  - Acceptance criteria
  - Business rules
  - API documentation
  - Validation rules

Input may be incomplete or ambiguous.

---

## 5. Output Requirements (Structured & Standardized)

The output MUST be a structured table with the following columns:

| ID | Feature | Title | Type | Priority | Preconditions | Steps | Expected Result | Test Data | Notes |

### Type must include:
- Happy Path
- Negative Case
- Edge Case

### Priority levels:
- Critical
- Major
- Minor

---

## 6. Workflow (AI Instruction Layer)

The skill must execute the following logical steps:

### Step 1: Parse Feature Scope
- Extract distinct features/modules
- Identify user flows
- Detect validation rules

### Step 2: Happy Path Generation
- Generate normal success scenarios
- Cover main user journey

### Step 3: Negative Case Generation
- Invalid input
- Missing required fields
- Wrong format
- Unauthorized access
- Business rule violation

### Step 4: Edge Case Detection
- Boundary values
- Null / Empty
- Max length / Min length
- Large data input
- Concurrent scenarios

### Step 5: Ambiguity Detection
- Highlight unclear logic
- Identify missing acceptance criteria
- Suggest clarification questions

### Step 6: Priority Assignment
Assign priority based on:
- Business impact
- Risk level
- Frequency of usage
- Security relevance

### Step 7: Coverage Validation
- Ensure every feature has:
  - At least 1 happy path
  - At least 1 negative case
  - At least 1 edge case
- Calculate coverage summary %

### Step 8: Format Standardization
Ensure:
- Clear, numbered steps
- Measurable expected result
- Structured and consistent output

---

## 7. Intelligence Layer (What Makes It "Smart")

The skill must:

- Detect hidden edge cases
- Identify logic gaps
- Suggest missing validations
- Highlight inconsistent requirements
- Suggest additional test data scenarios
- Generate QA review notes

---

## 8. Coverage Rules

For each feature:

Minimum requirement:
- 1 Happy path
- 2 Negative cases
- 2 Edge cases

If business-critical:
- Additional stress and abnormal flow cases

---

## 9. Example

### Example Input:

"User can register with email and password.
Password must be at least 8 characters."

### Example Output:

TC-001 | Registration success | Happy Path | Major | ...
TC-002 | Invalid email format | Negative | Major | ...
TC-003 | Password length = 7 | Edge Case | Major | ...
TC-004 | Empty email | Negative | Critical | ...

---

## 10. Export Format

The skill must support:

- Markdown table
- CSV-ready format
- Automation-ready JSON structure
- Cypress / Playwright compatible structure

---

## 11. Business Value

Before:
- 4–6 hours per complex feature
- Manual coverage checking
- Inconsistent structure

After:
- Generate test suite in minutes
- Structured & standardized output
- Coverage visible
- Reduced manual QA effort

---

## 12. Evaluation Alignment (Based on Guidebook)

This skill is optimized for BGK scoring criteria:

- Coverage → full scenario generation
- Quality → detailed steps & expected results
- Intelligence → edge detection & ambiguity analysis
- Format → structured & export-ready
- Speed → automated generation

---

## 13. MVP Scope (Day 2 Target)

Input:
- 1 spec.md file

Output:
- Full structured test case suite
- Edge case detection
- Coverage summary
- Ambiguity notes

---

## 14. Limitation

- Cannot fully validate real system behavior
- Relies on spec completeness
- Business logic validation requires domain knowledge

---

## 15. Future Roadmap

- Risk scoring model
- Integration with Jira
- Auto-generate execution report
- Test summary report generation
- Regression impact analysis
## 16. Strict Output Contract (Mandatory)

The AI MUST:

- Output test cases in consistent ID format: TC-001, TC-002...
- Ensure steps are numbered (1,2,3...)
- Ensure Expected Result is measurable and verifiable
- No vague statements such as "System works correctly"
- No missing columns
- No duplicated cases
- Do not mix multiple scenarios in one test case
## 17. Coverage Calculation Rule

Coverage must be calculated as:

Coverage % = 
(Number of generated test cases covering distinct logic branches)
/ 
(Number of distinct business rules extracted from spec)

Output must include:

- Total features detected
- Total business rules detected
- Total test cases generated
- Coverage percentage
## 18. Ambiguity & Risk Report

The skill must output a separate section:

### Ambiguity Report
- Missing validation rules
- Undefined error messages
- Conflicting business rules
- Unclear edge boundaries

### Risk Highlight
- Security sensitive areas
- Financial transaction logic
- Data integrity risk
## 19. Performance Expectation

The skill should:

- Generate output under 30 seconds for a 3-page spec
- Maintain structured output consistency
- Avoid repetition
- Avoid hallucinated features not present in spec
- Do not assume unspecified behavior; instead, list it under Ambiguity Report
## 20. Data Dictionary – Test Case Structure Definition

Each generated test case must strictly follow the schema below:

| Field | Type | Required | Description |
|-------|------|----------|------------|
| ID | String | Yes | Unique identifier (TC-001 format) |
| Feature | String | Yes | Feature or module name |
| Title | String | Yes | Clear and concise test case name |
| Type | Enum | Yes | Happy Path / Negative / Edge |
| Priority | Enum | Yes | Critical / Major / Minor |
| Preconditions | String | Optional | System state before execution |
| Steps | List<String> | Yes | Numbered execution steps |
| Expected Result | String | Yes | Measurable, verifiable outcome |
| Test Data | String | Optional | Specific input data used |
| Notes | String | Optional | Additional QA notes |

Constraints:
- Steps must be atomic and sequential.
- Expected Result must be observable and testable.
- One scenario per test case.
## 21. Error Matrix – Generator Failure Handling

The skill must handle the following error scenarios:

| Error Type | Cause | Tool Behavior | User Message |
|------------|-------|--------------|--------------|
| Invalid file format | Not .md or unsupported format | Reject input | "Unsupported file format. Please upload a Markdown spec file." |
| Empty spec | No content | Stop processing | "Spec file is empty." |
| Extremely large spec | Exceeds defined boundary | Partial parse or reject | "Spec exceeds maximum supported size." |
| Ambiguous spec | Missing business rules | Generate ambiguity report | "Spec contains unclear requirements." |
| Internal parsing failure | AI cannot extract structure | Safe fallback | "Unable to parse spec structure." |

The skill must never:
- Crash silently
- Output incomplete tables without warning
## 22. Tool Boundaries

Input Constraints:
- Maximum file size: 10,000 words
- Maximum features supported per run: 20
- Supported language: English (MVP phase)

Output Constraints:
- Maximum 500 test cases per generation
- Generation time target: < 30 seconds

Out of Scope:
- Real system execution validation
- Integration with external APIs (MVP)
- UI testing automation execution
## 23. Security & Data Protection

Since this generator may process confidential product specifications, the following principles apply:

- No persistent storage of input data
- No training on user-provided spec
- No external API transmission (MVP phase)
- Sensitive data masking (if detected)
- Input is processed in-session only

If sensitive information such as:
- API keys
- Database credentials
- Financial logic
- Authentication flows

is detected, the skill must:
- Flag under Risk Highlight
- Avoid echoing secrets in output

