# FitExpert: A Return-Policy-Aware Agentic AI System for Reducing Sizing Failures in Luxury Fashion E-commerce

[![Research](https://img.shields.io/badge/Research-Agentic%20AI-blue)](https://arxiv.org)
[![Domain](https://img.shields.io/badge/Domain-Fashion%20E--commerce-purple)](https://github.com/lavankumargajula/fitexpert)
[![Status](https://img.shields.io/badge/Status-Active%20Research-green)](https://github.com/lavankumargajula/fitexpert)
[![License](https://img.shields.io/badge/License-MIT-yellow)](LICENSE)

> **Lavan Kumar Gajula** | Data Engineer & AI Systems Researcher | New York, NY  
> lavankumargajula9@gmail.com | [LinkedIn](https://linkedin.com/in/lavankumargajula) | [Google Scholar](https://scholar.google.com)

---

## Overview

FitExpert is a production agentic AI system that addresses **four documented, reproducible failure modes** in LLM-based conversational agents deployed in luxury fashion e-commerce. 

Built on a real-world business problem — where a New York luxury fashion retailer was losing significant revenue due to AI chatbot sizing failures — FitExpert introduces a novel **measurement-to-specification matching algorithm** and a **three-class uncertainty quantification framework** that incorporates return policy constraints directly into sizing recommendations.

This repository documents the research, architecture, algorithm, and evaluation methodology. A full research paper is in preparation for submission to **arxiv.org** and **ACM RecSys 2026**.

---

## The Problem

Online fashion retail suffers from return rates of **30–40%**, with sizing uncertainty as the primary driver. Current LLM-based conversational agents fail to address this problem reliably. Through analysis of **197 customer reviews** and systematic testing of a production chatbot deployed by a luxury fashion retailer (gown price range: $500–$700), we documented four critical failure modes:

### Failure Mode 1 — Incorrect Product Retrieval
**Observation:** A customer querying style #29454 (Lace Gown, $658) received information about style #29004 ($550 beaded gown) — a completely different product.  
**Root Cause:** Fuzzy text matching in the retrieval layer instead of exact SKU validation.  
**Business Impact:** Customer receives sizing advice for the wrong garment entirely.

### Failure Mode 2 — Cross-Style Sizing Contradiction
**Observation:** Identical customer measurements (bust 36", waist 29", hips 39") applied across two styles produced recommendations of **Size 6** and **Size 10–12** — a **four-size contradiction** on the same body.  
**Root Cause:** No consistent measurement engine; sizing interpreted independently per product query with no shared state.  
**Business Impact:** Complete loss of customer trust; customer abandons purchase.

### Failure Mode 3 — Escalation Failure at Decision Point
**Observation:** Customer asked: *"Does this dress have stretch? I'm between a 10 and 12 — which is safer given your return policy?"* Chatbot responded: *"Please leave your email and our team will get back to you."*  
**Root Cause:** No fabric metadata access; no return policy awareness; no uncertainty-aware decision framework.  
**Business Impact:** Customer left and purchased at a competitor retailer.

### Failure Mode 4 — Stateless Architecture (No Session Memory)
**Observation:** Returning customers must re-enter measurements on every visit. No customer profile persistence across sessions.  
**Root Cause:** Stateless agent design with no integration to customer account data.  
**Business Impact:** Friction on repeat visits; poor customer experience for loyal buyers.

---

## The FitExpert Solution

FitExpert addresses each failure mode through a structured six-step agentic pipeline:

```
Step 1: Customer enters style number
         ↓
Step 2: Exact SKU validation via Shopify Products API
         [Addresses Failure Mode 1 — eliminates fuzzy matching]
         ↓
Step 3: Garment specification retrieval from Shopify metafields
         (bust/waist/hip ranges per size, fabric_type, stretch_pct, runs_small flag)
         ↓
Step 4: Customer measurement input (bust, waist, hips)
         [Retrieved from saved profile if returning customer]
         [Addresses Failure Mode 4 — persistent customer memory]
         ↓
Step 5: Measurement-to-Specification Algorithm
         [Addresses Failure Mode 2 — consistent engine across all styles]
         [Addresses Failure Mode 3 — fabric-aware, return-policy-aware output]
         ↓
Step 6: Verdict output + measurement saved to customer Shopify account
```

---

## Core Algorithm — Measurement-to-Specification Matching

### Formula

```
Delta = |Customer_Measurement - Garment_Spec_Max|
Adjusted_Tolerance = Base_Tolerance × Fabric_Stretch_Factor
```

### Fabric Stretch Factors

| Fabric Type        | Stretch Factor | Effective Extra Tolerance |
|--------------------|---------------|--------------------------|
| Jersey / Knit      | High          | +1.5 inches              |
| Silk Charmeuse     | Medium-High   | +0.75 inches             |
| Structured Crepe   | Medium        | +0.5 inches              |
| Lace Overlay       | Medium        | +0.5 inches              |
| Beaded/Embellished | Low           | +0.25 inches             |

### Three-Class Uncertainty Quantification Framework

| Verdict       | Condition                                                      | Meaning                                      |
|---------------|----------------------------------------------------------------|----------------------------------------------|
| ✅ SAFE BUY   | All measurements fall within 0.15–0.85 of size range          | Customer measurements comfortably within spec |
| ⚠️ SIZE UP    | Any measurement falls within 0.85–1.0 of range (upper edge)   | Borderline fit — size up is safer            |
| 🚨 FIT RISK   | Any measurement exceeds range max after stretch adjustment     | This size will not fit — do not purchase     |

### Return Policy Integration

FitExpert incorporates the retailer's return policy directly into the decision boundary:

- **Return window:** 14 days
- **Return cost:** $30 restocking fee
- **Policy bias:** When a customer is borderline between SAFE BUY and SIZE UP, the system biases toward SIZE UP — it is always cheaper to size up than to return
- This is the first known implementation of **return-policy-aware uncertainty quantification** in a fashion sizing AI system

---

## System Architecture

```
┌─────────────────────────────────────────────────────┐
│                  CUSTOMER INTERFACE                  │
│              (Chat Widget / WhatsApp)                │
└──────────────────────┬──────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────┐
│                   n8n WORKFLOW ENGINE                 │
│                                                      │
│  Node 1: Style Number Validator                      │
│  Node 2: Shopify Products API (exact SKU lookup)     │
│  Node 3: Metafield Retriever (garment specs)         │
│  Node 4: Measurement Input Handler                   │
│  Node 5: Fit Algorithm Engine (core logic)           │
│  Node 6: Customer Profile Writer (Shopify)           │
└──────────────────────┬──────────────────────────────┘
                       │
         ┌─────────────┼─────────────┐
         │             │             │
┌────────▼───┐  ┌──────▼─────┐  ┌───▼──────────────┐
│  Shopify   │  │ Claude API │  │ Shopify Customer  │
│ Products   │  │ (Reasoning)│  │ Accounts (Memory) │
│    API     │  │            │  │                   │
└────────────┘  └────────────┘  └───────────────────┘
```

### Tech Stack

| Component         | Technology                        |
|-------------------|-----------------------------------|
| Workflow Engine   | n8n (self-hosted)                 |
| AI Reasoning      | Claude API (claude-sonnet) / GPT-4o |
| Product Data      | Shopify Admin API + Metafields    |
| Customer Memory   | Shopify Customer Metafields       |
| Chat Interface    | Tidio / WhatsApp Business API     |
| Deployment        | Railway / Vercel                  |

### Shopify Metafield Schema

```json
namespace: "fit_expert"

{
  "bust_ranges":   {"XS": [30,32], "S": [32,34], "M": [34,36], "L": [36,38], "XL": [38,40]},
  "waist_ranges":  {"XS": [24,26], "S": [26,28], "M": [28,30], "L": [30,32], "XL": [32,34]},
  "hip_ranges":    {"XS": [34,36], "S": [36,38], "M": [38,40], "L": [40,42], "XL": [42,44]},
  "fabric_type":   "silk_charmeuse",
  "stretch_pct":   0.75,
  "runs_small":    true,
  "fit_notes":     "Cut runs one size small. Size up if between sizes."
}
```

---

## Evaluation — QA Test Scenarios

FitExpert was evaluated against the production chatbot across five controlled test scenarios:

| Test | Customer Profile | Chatbot Result | FitExpert Result |
|------|-----------------|----------------|-----------------|
| T1 | Standard measurements, exact SKU | Wrong product info returned | Correct SKU validated, accurate spec retrieved |
| T2 | Same measurements, two styles | 4-size contradiction (S6 vs S10-12) | Consistent verdict across both styles |
| T3 | Borderline measurements, stretch fabric | "Leave your email" escalation failure | SIZE UP with fabric stretch reasoning explained |
| T4 | Returning customer, saved measurements | Re-entered measurements manually | Profile retrieved automatically, immediate verdict |
| T5 | Measurements exceed all sizes | No clear guidance | FIT RISK verdict, alternative styles suggested |

---

## Business Impact

| Metric                    | Value                              |
|---------------------------|------------------------------------|
| Annual returns prevented  | ~156 units (projected)             |
| Cost per return           | $100 (shipping + restocking)       |
| Annual savings            | ~$15,600                           |
| Operating cost            | $35–115/month                      |
| ROI                       | 11.3x                              |
| External build cost       | $17,000–$38,500 (agency estimate)  |

---

## Research Contributions

This work makes the following contributions to the literature on agentic AI systems:

1. **Failure Mode Taxonomy** — First documented taxonomy of LLM conversational agent failures specifically in luxury fashion e-commerce, grounded in 197 customer reviews and systematic chatbot testing

2. **Measurement-to-Specification Algorithm** — Novel fabric-aware matching algorithm that produces consistent sizing recommendations across product styles using structured garment metadata

3. **Return-Policy-Aware Uncertainty Quantification** — Three-class verdict system (SAFE BUY / SIZE UP / FIT RISK) that incorporates retailer return constraints directly into the decision boundary — first known implementation of this pattern

4. **Production Agent Architecture** — End-to-end structured agentic pipeline demonstrating how exact retrieval, external constraints, and persistent memory can be combined to address real-world LLM agent failures

---

## Research Paper

**Title:** FitExpert: A Return-Policy-Aware Structured Agent Architecture for Reducing Sizing Failures in Luxury Fashion E-commerce

**Status:** In preparation — arxiv submission target June/July 2026

**Target Venues:**
- arxiv.org cs.AI / cs.IR
- ACM RecSys 2026 Workshop on AI for Fashion
- WWW 2027 Industry Track

*Link will be added here upon publication.*

---

## Repository Structure

```
fitexpert/
├── README.md                  ← You are here
├── demo/
│   └── fitexpert_demo.html    ← Live side-by-side comparison demo
├── algorithm/
│   └── algorithm_spec.md      ← Full algorithm specification
├── architecture/
│   └── system_architecture.md ← Detailed system design
├── evaluation/
│   └── qa_test_scenarios.md   ← Test cases and results
└── research/
    └── paper_draft.md         ← Research paper (in progress)
```

---

## About the Author

**Lavan Kumar Gajula** is a Data Engineer with 4+ years of experience building large-scale production data systems, including pipelines processing 5TB+ daily across 200+ data sources. He holds an MS in Data Science and Analytics from New England College (GPA: 3.8) and is applying for funded PhD programs in Computer Science for Fall 2027, with a research focus on **Agentic AI Systems** — specifically how AI agents can be designed to make reliable, business-aware decisions in real-world production environments.

- **Email:** lavankumargajula9@gmail.com
- **LinkedIn:** linkedin.com/in/lavankumargajula
- **Location:** Jersey City, NJ / New York, NY

---

*© 2026 Lavan Kumar Gajula. This research is original work. All algorithm designs, failure mode documentation, and system architecture are the intellectual property of the author.*
