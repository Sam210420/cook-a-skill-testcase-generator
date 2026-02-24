# 🧪 Smart Test Case Generator Pro

> **AI Competition 2026 — Cook A Skill | Team: Cook Your Skill**  
> Author: Dương Mai Phương (@sam_xbt) | Background: Tester

---

## 💡 What is this?

**Smart Test Case Generator Pro** is an AI-powered skill that automatically generates a structured, comprehensive test case suite from a feature spec (`.md` file or plain text).

It acts as a **senior AI testing assistant**, applying Semantic Reasoning and Static Logic Analysis to uncover edge cases, ambiguities, and security risks that manual testers often miss.

---

## 🚀 Before vs After

| | BEFORE (Manual) | AFTER (With Skill) |
|--|--|--|
| ⏱ Time per feature | ~4 hours (half a day) | ~1 hour (review only) |
| 🔍 Edge case detection | Easy to miss | AI detects automatically |
| 📋 Format consistency | Varies per person | Structured, standardized |
| 🔒 Security cases | Often forgotten | Always included |

---

## 📦 Repo Structure

```
cook-a-skill-testcase-generator/
├── README.md         ← You are here
├── SKILL.md          ← Main skill instruction for AI
├── skill-card.md     ← 1-page skill summary
├── spec.md           ← Skill specification document
└── ai-showcase/      ← Screenshots of AI demo
```

---

## ⚙️ How to Use

### Step 1 — Open your AI tool
Claude Project, ChatGPT Custom GPT, or any AI chat interface.

### Step 2 — Load the skill
Copy the full content of `SKILL.md` → paste into the **system prompt / project instructions**.

### Step 3 — Feed your spec
Paste your feature spec into the chat:

```
Generate test cases for this feature spec:

[paste your spec.md content here]
```

### Step 4 — Get your test suite
Within 30 seconds, you'll receive a full test suite including:
- ✅ Happy Path cases
- ❌ Negative cases  
- ⚠️ Edge cases
- 🔒 Security cases
- 🔍 Ambiguities & Risk highlights

---

## 📊 Output Format

```
# 🧪 Test Suite: [Feature Name]

## 📊 Summary
- Total Modules Detected: N
- Total Test Cases Generated: N
- Semantic/Logic Coverage: N%

## 📋 Test Cases
| ID | Module | Title | Type | Priority | Steps | Expected Result |
|----|--------|-------|------|----------|-------|-----------------|
| TC-001 | ... | ... | Happy | Critical | ... | ... |
```

---

## 🎯 Tiêu chí chấm điểm (BGK)

| Tiêu chí | Mô tả |
|----------|-------|
| Coverage | Cover happy path, edge case, negative case đầy đủ |
| Chất lượng | Steps, expected result, precondition rõ ràng |
| Thông minh | Tự phát hiện edge case ẩn, chia priority |
| Format | Structured, dễ dùng, export được |
| Tốc độ | Feed spec → output nhanh < 30s |

---

## 🛠 Tech Stack

- **Platform:** Claude Project (claude.ai) + ChatGPT
- **Skill Type:** Prompt Engineering / AI Instruction Design
- **Input:** Feature spec `.md` file
- **Output:** Structured test case table in Markdown

---

## ⚠️ Limitations

- Does not execute actual tests — output is for human/automation review
- Maximum input: 2MB / 10,000 lines
- Security test cases are suggestions only — not a replacement for dedicated pen-testing tools

---

*Built with ❤️ for AI Competition 2026 — Lab3*
