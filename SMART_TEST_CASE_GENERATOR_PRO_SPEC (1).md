# SMART TEST CASE GENERATOR PRO --- OPENCLAW SKILL SPEC

## 1. Project Overview

**Name:** Smart Test Case Generator Pro (OpenClaw Skill)

**Purpose:**\
Automatically generate a structured test case suite from a Markdown
product specification.\
Output includes: - Full test cases (Happy / Negative / Edge) - Priority
classification - Coverage calculation - Ambiguity detection - Risk
highlights - Markdown-ready export format

The skill simulates senior-level QA analytical thinking.

------------------------------------------------------------------------

## 2. Target Users

-   QC / Test Engineer\
-   QA Manager\
-   Product Owner\
-   Automation Engineer

------------------------------------------------------------------------

## 3. Input Specification

**Input Type:** Markdown file (`.md`)

May include: - Feature description\
- User stories\
- Acceptance criteria\
- Business rules\
- Validation rules\
- API documentation\
- Domain constraints

Spec may be incomplete or ambiguous.\
The skill must detect and report such cases.

------------------------------------------------------------------------

## 4. Output Specification

The skill MUST generate structured Markdown output using the mandatory
template below.

Output must: - Be readable and structured - Contain full test case
table - Include coverage summary - Include ambiguity & risk sections -
Be automation-friendly

No structural deviation allowed.

------------------------------------------------------------------------

## 5. Output Markdown Template (MANDATORY)

### 🧪 Test Suite: {{Project/Feature Name}}

#### 📌 Summary

-   **Total Features Detected:** {{total_features}}\
-   **Total Business Rules:** {{total_business_rules}}\
-   **Total Test Cases Generated:** {{total_test_cases}}\
-   **Coverage %:** {{coverage_percentage}}%

------------------------------------------------------------------------

### 📂 Test Cases

  ---------------------------------------------------------------------------------------------
  ID   Feature   Title   Type   Priority   Preconditions   Steps   Expected     Test    Notes
                                                                   Result       Data    
  ---- --------- ------- ------ ---------- --------------- ------- ------------ ------- -------

  ---------------------------------------------------------------------------------------------

{{#each testCases}} \| {{id}} \| {{feature}} \| {{title}} \| {{type}} \|
{{priority}} \| {{preconditions}} \| {{steps}} \| {{expectedResult}} \|
{{testData}} \| {{notes}} \| {{/each}}

------------------------------------------------------------------------

### 📊 Coverage Breakdown

### 🎯 Happy Path Cases

{{#each happyCases}} - **{{id}}** --- {{title}} {{/each}}

### ❌ Negative Cases

{{#each negativeCases}} - **{{id}}** --- {{title}} {{/each}}

### ⚡ Edge Cases

{{#each edgeCases}} - **{{id}}** --- {{title}} {{/each}}

------------------------------------------------------------------------

### ⚠ Ambiguities & Risks

#### 🔎 Ambiguities Detected

{{#each ambiguityList}} - **{{issue}}** --- {{detail}} {{/each}}

#### 🚨 Risk Highlights

{{#each riskList}} - **{{risk}}** --- {{explanation}} {{/each}}

------------------------------------------------------------------------

## 6. Workflow & Logical Rules

### Step 1 --- Parse Spec

-   Read `.md` file
-   Extract features, stories, acceptance criteria, business rules
-   Identify validation constraints

### Step 2 --- Extract Business Logic

-   Convert text into atomic business rules
-   Tag rules by feature/module

### Step 3 --- Generate Test Cases

Generate: - Happy Path cases - Negative cases - Edge cases

### Step 4 --- Assign Priority

Priority levels: - Critical - Major - Minor

Based on: - Business impact - Risk severity - Usage frequency - Security
sensitivity

### Step 5 --- Compute Coverage

Coverage % =\
(number of distinct logic branches covered) /\
(number of distinct business rules extracted)

Must display coverage in summary.

### Step 6 --- Detect Ambiguity

Detect: - Missing validation - Conflicting criteria - Unclear boundary
definitions - Undefined error messages

### Step 7 --- Format Output

-   Strictly follow template
-   No missing columns
-   Steps must be numbered
-   Expected results must be measurable

------------------------------------------------------------------------

## 7. Intelligence Layer

The skill must: - Detect hidden edge cases - Identify logic gaps -
Suggest missing validations - Highlight inconsistent requirements -
Suggest additional test data - Generate QA review notes

Must interpret semantic meaning of spec, not pattern-match text.

------------------------------------------------------------------------

## 8. Coverage Rules

For each feature:

Minimum: - 1 Happy path - 2 Negative cases - 2 Edge cases

If business-critical: - Add abnormal flow scenarios - Add stress-related
cases

------------------------------------------------------------------------

## 9. Error Matrix --- Generator Failure Handling

  ---------------------------------------------------------------------------
  Error Type         Cause        Tool Behavior         User Message
  ------------------ ------------ --------------------- ---------------------
  Invalid file       Not `.md`    Reject input          Unsupported file
  format                                                format. Please upload
                                                        a Markdown spec file.

  Empty spec         No content   Stop processing       Spec file is empty.

  Extremely large    Exceeds      Partial parse or      Spec exceeds maximum
  spec               defined      reject                supported size.
                     boundary                           

  Ambiguous spec     Missing      Generate ambiguity    Spec contains unclear
                     business     report                requirements.
                     rules                              

  Internal parsing   Structure    Safe fallback         Unable to parse spec
  failure            unreadable                         structure.
  ---------------------------------------------------------------------------

The tool must never: - Crash silently - Output incomplete tables

------------------------------------------------------------------------

## 10. Technical System Boundaries

### Input Constraints

-   Maximum file size: 2MB
-   Maximum word count: 10,000 words
-   Maximum features per run: 20
-   Supported language (MVP): English only
-   Unsupported formats: PDF, DOCX, images

### Output Constraints

-   Maximum 500 test cases per generation
-   Generation time target: under 30 seconds
-   Hard timeout: 45 seconds

### System Limits

-   Maximum concurrent requests: 5
-   Memory ceiling: 512MB per execution

------------------------------------------------------------------------

## 11. Security & Data Lifecycle Policy

This generator may process confidential product specifications.

### Data Handling Principles

-   No persistent storage of input data
-   No training on user-provided spec
-   No external API transmission (MVP phase)
-   No plaintext logging of spec content
-   Session memory cleared after generation
-   No analytics tracking of spec content

### Encryption Policy

-   All input must be transmitted via HTTPS (TLS 1.2+)
-   Data encrypted in transit
-   No database storage of raw spec content
-   Enterprise mode may support client-side encryption

### Sensitive Data Handling

If spec contains: - API keys - Database credentials - Financial logic -
Authentication flows

The skill must: - Flag under Risk Highlights - Avoid echoing secrets in
output

------------------------------------------------------------------------

## 12. Data Dictionary --- Test Case Schema

  Field             Type     Required   Description
  ----------------- -------- ---------- -----------------------------------
  ID                String   Yes        Unique identifier (TC-001 format)
  Feature           String   Yes        Module name
  Title             String   Yes        Clear test case title
  Type              Enum     Yes        Happy / Negative / Edge
  Priority          Enum     Yes        Critical / Major / Minor
  Preconditions     String   Optional   System state before execution
  Steps             List     Yes        Numbered steps
  Expected Result   String   Yes        Measurable outcome
  Test Data         String   Optional   Input values
  Notes             String   Optional   QA comments

Constraints: - Steps must be atomic - Expected result must be
observable - One scenario per test case

------------------------------------------------------------------------

## 13. Performance Expectation

The skill must: - Generate output under 30 seconds for 3-page spec -
Maintain structured consistency - Avoid repetition - Avoid hallucinated
features - Not assume unspecified behavior

------------------------------------------------------------------------

## 14. MVP Scope

Day 1: - Parse spec - Extract business rules

Day 2: - Generate Happy / Negative / Edge cases - Compute coverage -
Detect ambiguity - Output Markdown

------------------------------------------------------------------------

## 15. Evaluation Alignment

This skill is optimized for BGK scoring: - Coverage --- full scenario
coverage - Quality --- clear steps + measurable expected results -
Intelligence --- ambiguity & edge detection - Format --- structured
Markdown - Speed --- under 30 seconds
