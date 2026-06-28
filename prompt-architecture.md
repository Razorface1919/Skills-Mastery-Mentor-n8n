# 🧠 Architecture Spec: Template-Driven Prompting

## Overview

Unlike standard LLM implementations that rely on conversational agents or chat interfaces, this project utilizes **Template-Driven Prompting**. 

In this architecture, Google Gemini 2.5 Flash is not treated as a conversational bot. Instead, it acts as a hybrid calculation engine and backend rendering engine. It ingests a structured JSON payload, performs conditional logic and mathematical calculations, and injects the output into a strictly enforced HTML skeleton.

---

## 1. The Data Pipeline (Ingestion)

The prompt begins by ingesting a payload of 5 variables dynamically passed from the n8n Webhook trigger using expression syntax (`{{ $json['field'] }}`).

### Input Vectors:
*   **User Name:** String *(Used for personalized UI headers)*
*   **Target Skill:** String *(The core subject matter)*
*   **Current Level:** Enum *(Beginner / Basics / Intermediate)*
*   **Time Commitment:** Enum *(3-5 hrs / 6-10 hrs / 10-20+ hrs)*
*   **Core Motivation:** Enum *(Career / Side Income / Hobby / Fun)*

---

## 2. Conditional Logic Routing (The "Brain")

To ensure the output doesn't feel like generic AI text, the prompt acts as a routing switch based on the **Core Motivation** variable. The LLM is instructed to shift its internal weighting before generating content:

*   **If Career Guidance:** Weights output toward ATS-optimization, portfolio-worthy milestones, and hiring manager expectations.
*   **If Side Income:** Shifts focus to freelance marketplaces, pricing strategies, and minimizing time-to-first-gig.
*   **If Hobby / For Fun:** Completely disables monetization framing. Weights heavily toward sustainable enjoyment, novelty, and avoiding burnout.

---

## 3. The LLM Calculation Engine

Before writing any copy, the LLM is forced to compute a set of relative metrics based on the user's **Target Skill** and **Current Level**. It calculates these specific data points to be used as UI rendering values:

*   **Stage Timelines:** Estimates realistic practice hours to reach Beginner, Intermediate, Advanced, and Master stages.
*   **CSS Progress Variables:** Calculates relative percentages (e.g., converting the user's current progress into `[currentLevelPercent]` and stage thresholds into `[barBeginner]`, `[barMaster]`). These are directly injected into inline CSS `width:` properties to render dynamic bar charts.
*   **Market Metrics:** Estimates quantitative values for Learning Difficulty (%) and Market Demand (%).

---

## 4. The Presentation Layer (HTML Skeleton)

The second half of the system prompt is a pre-designed, heavily styled, inline-CSS HTML skeleton. 

Instead of asking the AI to "write an email," the prompt provides the exact HTML DOM structure filled with bracketed placeholders (e.g., `[Skill]`, `[Sub-skill 1]`, `[incBarPro]`).

---

## Strict Output Enforcement

To guarantee a 0% failure rate in the n8n workflow, the prompt applies extreme output constraints:

*   **Zero Markdown:** The LLM is strictly forbidden from using markdown fences (e.g., ```html).
*   **Zero Conversational Filler:** The LLM cannot output introductory or concluding text (e.g., "Here is your output:").
*   **Absolute Interpolation:** The output must begin with `<div style...>` and end with `</div>`. Every single placeholder must be replaced with calculated data or generated copy, ensuring no raw brackets hit the user's inbox.

---

## 🎯 Architectural Benefits

By restricting the LLM to a strict data-injection role within a hardcoded template, this pipeline achieves:

*   **100% Reliable CSS Rendering:** The layout never breaks because the LLM cannot alter the DOM structure.
*   **Zero-Code Parsing:** Because the output is pure HTML, the n8n Gmail node requires no regex, code-splitting, or data parsing.
*   **Ultra-Low Latency:** Processing raw data into a fixed template reduces token generation time, executing end-to-end in < 30 seconds.