# 💰 Agent Cost Strategy

## **"Use the cheapest model that can reliably do the job."**

The **Agent Cost Strategy** is a framework for managing the economic footprint of multi-agent AI workflows. In an era of massive context windows and high-token throughput, unmanaged model selection can lead to exponential cost increases. This document provides the architectural rules for tiered model selection, sub-agent delegation, and cache optimization to ensure high-performance AI workflows remain economically sustainable.

***

## 🏗 The Three-Tier Model Architecture

To optimize spend, every task must be mapped to one of three distinct tiers.

| Tier | Capability | Best For... | Example Models |
| :--- | :--- | :--- | :--- |
| ⚡ **Fast/Cheap** | High speed, low reasoning. | Sub-agents, cron jobs, lookups, simple replies, monitoring. | Claude Haiku, GPT-4o-mini, Gemini Flash |
| ⚖️ **Mid-tier** | Balanced reasoning/cost. | Main session dialogue, multi-step tasks, PR reviews, feature building. | Claude Sonnet, GPT-4o, Gemini Pro |
| 🧠 **Powerful** | Deep reasoning, high complexity. | Architecture, complex debugging, deep audits, "Plan B" when cheaper models fail. | Claude Opus, GPT-4.5, Gemini Ultra |

***

## 🚦 Task-to-Tier Routing Matrix

Use this matrix to determine which model should be assigned to a specific workload.

| If the task is... | Use Tier... |
| :--- | :--- |
| **Automated/Background** (Cron, Heartbeats, Monitoring) | ⚡ **Fast/Cheap** |
| **Simple/Short** (Replies, "Hi", "OK", "Yes", "No") | ⚡ **Fast/CR** |
| **Information Retrieval** (Research, Search, Lookups) | ⚡ **Fast/Cheap** |
| **Code Maintenance** (Writing boilerplate, fixing tests) | ⚡ **Fast/Cheap** |
| **Feature Development** (Building new components) | ⚖️ **Mid-tier** |
| **Logic Verification** (Reviewing a Pull Request) | ⚖️ **Mid-tier** |
| **Complex Problem Solving** (Architecture, Debugging) | 🧠 **Powerful** |
| **The "Escalation" Rule** | ⬆️ **Move Up One Tier** (If a cheaper model fails twice) |

***

## 🚨 The Critical "Sub-Agent" Rule

### **The single largest source of cost leakage is the "Inheritance Leak."**

By default, most orchestration frameworks spawn sub-agents using the **parent session's model**. If your main agent is running on **Claude Sonnet (Mid-tier)**, every sub-agent it spawns will also run on Sonnet unless explicitly overridden.

> [!CAUTION]
> **Always explicitly set the model when spawning sub-agents.**
>
> **Bad:** `spawn_agent(task="fix_test")` $\rightarrow$ (Inherits Sonnet $\rightarrow$ Expensive)
> **Good:** `spawn_agent(task="fix_test", model="claude-haiku")` $\rightarrow$ (Uses Haiku $\rightarrow$ Cheap)

**Goal:** Aim for an **80/20 split**: 80% of your workload should run on **Fast/Cheap** models, and only 20% should utilize **Mid-tier** or **Powerful** models.

***

## 📉 Advanced Optimization Techniques

### 1. Prompt Caching & Session Management

The most effective way to reduce costs is to maximize **Cache Hits**.

* **The Strategy:** Keep sessions alive as long as possible. Every time you start a new session (`/new`), you trigger a "Cold Start" where the model must re-process the entire context at a premium rate.
* **The Math:** A "Cold" 600k token load is expensive. Once the cache is warm, subsequent messages cost **~90% less**.
* **The Rule:** Only end a session when the context is genuinely full (>80%) or for privacy/data-retention reasons.

### 2. The Batch API (The 50% Discount)

For any task that does not require a real-time response (e.g., overnight analysis, scheduled report generation, data mining), **always use the Batch API**.

* **Benefit:** Up to a **50% discount** on token costs.
* **Trade-off:** Results are delivered asynchronously (up to 24-hour window).

### 3. Heartbeat & Cron Optimization

For background tasks that run on a schedule:

* **Model Selection:** Always use the **Fast/Cheap** tier.
* **Cache Warmth:** Set heartbeat intervals to just under your provider's **Cache TTL** (Time-to-Live). This ensures you are paying for **cache-reads** rather than full **input-processing** rates.

***

## 🔍 Audit Checklist: Are you over-spending?

If you can answer **"Yes"** to any of these, your architecture needs optimization:

* [ ] Are you using a Mid-tier model for "Hi", "Thanks", or "OK" messages?
* [ ] Are your sub-agents inheriting the parent's expensive model?
* [ ] Are your background/cron jobs using anything other than the Fast/Cheap tier?
* [ ] Are you starting new sessions too frequently, causing constant "Cold Start" costs?
* [ ] Are you using real-time APIs for tasks that could wait 24 hours for a Batch API response?
