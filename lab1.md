# Hands-On Workshop: Zero-Shot, Single-Shot, and Multi-Shot Prompting Mechanics

**Course Module:** 1.1 Foundations of LLMs & Gemini Architecture & 2.2 Few-Shot Prompting  
**Duration:** 90 Minutes  
**Format:** Interactive Lab Guide & Exercises  
**Target Audience:** Enterprise Engineers & AI Developers  

---

## Workshop Objectives
By the end of this workshop, participants will be able to:
1. Understand the operational mechanics and context window trade-offs between Zero-Shot, Single-Shot, and Multi-Shot prompting.
2. Select appropriate exemplar selection strategies to guide Gemini models without causing pattern overfitting.
3. Construct production-ready enterprise prompts that enforce strict JSON outputs and handle domain-specific edge cases.

---

## 1. Core Mechanics Overview

### 1.1 Zero-Shot Prompting
* **Definition:** Relying entirely on the model's pre-trained parametric knowledge without providing explicit input-output exemplars.
* **When to Use:** Standard NLP tasks (summarization, general translation, open-ended Q&A), fast prototyping, or low-latency high-throughput pipelines.
* **Trade-Off:** Lowest token context overhead; however, formatting compliance and accuracy on custom domain logic can be inconsistent.

### 1.2 Single-Shot Prompting
* **Definition:** Providing exactly **one** exemplar demonstrating the desired format, structure, or logical transformation before presenting the target task.
* **When to Use:** Enforcing specific output schemas (e.g., custom JSON keys, Markdown tables) or establishing tone and style boundaries.
* **Trade-Off:** Minor token overhead increase; significantly improves schema compliance compared to zero-shot.

### 1.3 Multi-Shot (Few-Shot) Prompting
* **Definition:** Supplying **two or more exemplars (typically 2–5)** to establish complex patterns, edge-case handling, and domain taxonomy.
* **When to Use:** Highly specialized classifications (e.g., Cyber Security severity scoring, log parsing), preventing edge-case failure modes, and domain-specific code/payload generation.
* **Trade-Off:** Highest context overhead and execution latency; risk of pattern overfitting if examples are biased or homogenous.

---

## 2. Exemplar Selection Strategy & Overfitting Prevention

When constructing Multi-Shot prompts for enterprise workloads, follow these rules:
1. **Diversity Over Quantity:** 3 distinct examples covering normal, edge, and error cases outperform 10 repetitive examples.
2. **Label Distribution Balance:** Ensure balanced representation across target classes (e.g., don't give 4 Critical examples and only 1 Low example).
3. **Format Consistency:** Keep delimiter syntax (`Input:`, `Output:`, `---`) consistent across all exemplars to avoid confusing the attention mechanism.
4. **Prevent Overfitting:** Avoid using identical keyword patterns in all exemplars so the model learns the *logic* rather than *string matching*.

---

## 3. Hands-On Lab Exercises

### Exercise 1: Support Ticket Severity Classification
**Scenario:** Classify incoming IT support tickets into severity tiers (`CRITICAL`, `HIGH`, `MEDIUM`, `LOW`).

#### Step 1: Zero-Shot Approach
```text
Role: You are an enterprise IT Service Management Classifier.

Task: Classify the severity of the following IT support ticket as CRITICAL, HIGH, MEDIUM, or LOW.
Output only the severity level.

Ticket: "The payment gateway API is returning HTTP 500 errors for 45% of checkout requests."
Severity:
```

#### Step 2: Single-Shot Approach
```text
Role: You are an enterprise IT Service Management Classifier.

Task: Classify the severity of the following IT support ticket as CRITICAL, HIGH, MEDIUM, or LOW.

Example:
Ticket: "User cannot connect to office printer."
Severity: LOW

Ticket: "The payment gateway API is returning HTTP 500 errors for 45% of checkout requests."
Severity:
```

#### Step 3: Multi-Shot Approach (Balanced Exemplars)
```text
Role: You are an enterprise IT Service Management Classifier.

Task: Classify the severity of the following IT support ticket as CRITICAL, HIGH, MEDIUM, or LOW.

Example 1:
Ticket: "User cannot connect to office printer."
Severity: LOW

Example 2:
Ticket: "Internal wiki loading slowly for overseas team members."
Severity: MEDIUM

Example 3:
Ticket: "Core database node failed over, secondary node running at 90% CPU."
Severity: HIGH

Example 4:
Ticket: "Global ERP application down across all regions during month-end closing."
Severity: CRITICAL

Ticket: "The payment gateway API is returning HTTP 500 errors for 45% of checkout requests."
Severity:
```

---

### Exercise 2: Cyber Security Alert Classification & Severity Scoring (VALEO Scenario)
**Scenario:** Transform raw security telemetry logs into structured JSON payloads with severity scoring.

#### Multi-Shot Enterprise Prompt Template
```text
Role: You are a Senior SOC Security Analyst.

Task: Analyze raw security alerts, assign a severity score (1-10), classify the attack category, and format output as JSON.

---
Example 1:
Input Log: "Failed SSH login attempt from IP 192.168.1.50 for user 'admin' (Attempt 1)."
Output JSON:
{
  "category": "Authentication Failure",
  "severity_score": 2,
  "action_required": "Log & Monitor",
  "target_asset": "192.168.1.50"
}

---
Example 2:
Input Log: "Multiple failed SSH root logins (500/min) from IP 185.220.101.5 followed by successful authentication."
Output JSON:
{
  "category": "Brute Force / Credential Compromise",
  "severity_score": 9,
  "action_required": "Isolate Host & Revoke Root Session",
  "target_asset": "185.220.101.5"
}

---
Example 3:
Input Log: "Outbound TCP connection on port 4444 to unknown external IP 45.33.32.156 carrying 4.2GB transferred data."
Output JSON:
{
  "category": "Data Exfiltration / C2 Communication",
  "severity_score": 10,
  "action_required": "Block IP on Firewall & Trigger Incident Response",
  "target_asset": "45.33.32.156"
}

---
Input Log: "Suspicious PowerShell execution with base64 encoded command string on endpoint HOST-US-EAST-102."
Output JSON:
```

---

## 4. Evaluation Matrix & Trade-Off Comparison

| Metric | Zero-Shot | Single-Shot | Multi-Shot |
| :--- | :--- | :--- | :--- |
| **Token Cost** | Lowest | Low-Medium | High |
| **Latency** | Fastest | Fast | Higher |
| **Schema Strictness** | Variable | High | Very High |
| **Edge-Case Handling** | Poor | Moderate | Excellent |
| **Best Production Use** | Simple text transforms | JSON formatting | Complex domain logic / Rules |

---

## 5. Participant Homework & Practical Assessment
1. Open Python environment with `google-genai` SDK installed.
2. Run the Cyber Security alert scenario using `gemini-2.5-flash` with Zero-Shot, Single-Shot, and Multi-Shot prompts.
3. Compare response structure, JSON key stability, and token consumption across all three runs.
