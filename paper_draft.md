# FitExpert Research Paper — Draft

**Title:** FitExpert: A Return-Policy-Aware Structured Agent Architecture for Reducing Sizing Failures in Luxury Fashion E-commerce

**Author:** Lavan Kumar Gajula  
**Affiliation:** Independent Researcher, New York, NY  
**Contact:** lavankumargajula9@gmail.com  
**Status:** Draft — Target submission arxiv.org June/July 2026

---

## Abstract

Online fashion retail suffers from return rates of 30–40%, with sizing uncertainty as the primary driver of customer dissatisfaction and revenue loss. Current large language model (LLM)-based conversational agents deployed in fashion e-commerce fail to address this problem reliably. Through systematic analysis of 197 customer reviews and controlled testing of a production chatbot deployed by a luxury fashion retailer processing $500–$700 gowns, we document four reproducible architectural failure modes: incorrect product retrieval via fuzzy matching, cross-style sizing contradictions exceeding four sizes on the same body, escalation failure at critical decision points, and complete loss of customer state between sessions. These are not edge cases — they are structural limitations of current generative agent architectures applied to structured decision problems. We present FitExpert, a production agentic AI system that addresses each failure through a six-node structured pipeline incorporating exact SKU validation, fabric-aware measurement-to-specification matching, and a novel three-class uncertainty quantification framework that integrates return policy constraints directly into sizing decisions. FitExpert achieves correct verdicts across all five controlled evaluation scenarios where the baseline system failed in all five. Our work contributes a failure mode taxonomy for production fashion AI agents, a return-policy-aware decision framework applicable to high-stakes e-commerce AI, and an architecture pattern demonstrating how deterministic reasoning engines can be combined with generative models to produce reliable, auditable agent behavior.

**Keywords:** agentic AI, conversational agents, LLM failures, fashion e-commerce, recommendation systems, uncertainty quantification, retrieval-augmented generation

---

## 1. Introduction

When a customer pays $658 for an evening gown online, the sizing decision is consequential. A wrong recommendation means a return — a logistical burden for the customer, a financial cost for the retailer, and a measurable loss of trust in the AI system that gave the advice. Despite this, the LLM-based conversational agents currently deployed in luxury fashion e-commerce treat sizing as a language problem: interpret the question, generate an answer, move on.

This framing is wrong. Sizing is a structured data problem — a comparison between the customer's body measurements and the garment's physical specifications, modified by fabric elasticity, construction details, and the customer's risk tolerance given the retailer's return policy. Language models are not well-suited to this problem by default, and deploying them without the appropriate structural scaffolding produces exactly the failures we document in this paper.

We came to this research through direct observation. Working with LARA New York, a luxury fashion retailer operating on Shopify, we analyzed 197 customer reviews and conducted systematic testing of their production AI chatbot. We documented four failure modes that were not random or rare — they were reproducible, architectural, and directly traceable to design choices in the underlying system. These failures were causing customers to abandon purchases and return items, with measurable revenue impact.

FitExpert was built to address these failures. It is not a research prototype disconnected from production reality — it is designed for deployment on Shopify, integrated with real product data through the Shopify metafields API, and tested against real customer scenarios. Its core contribution is not a new model architecture but a new systems architecture: one that separates language understanding from decision-making, integrates structured domain data into the reasoning pipeline, and builds business constraints into the decision boundary rather than appending them as post-hoc caveats.

The rest of this paper is organized as follows. Section 2 reviews related work on LLM agent failures, fashion recommendation systems, and uncertainty quantification in conversational AI. Section 3 documents the four failure modes in detail. Section 4 presents the FitExpert architecture and core algorithm. Section 5 describes our evaluation methodology and results. Section 6 discusses limitations and future research directions.

---

## 2. Related Work

*(In progress — to be completed March 2026)*

Key areas to cite:
- RAG failure modes in production systems (Lewis et al. 2020 + recent critiques)
- Fashion recommendation systems (survey papers 2022–2024)
- Uncertainty quantification in conversational AI
- Agentic AI systems and tool use (ReAct, Toolformer)
- E-commerce conversational agents (industry papers)
- Return policy optimization in retail AI

---

## 3. Failure Mode Taxonomy

*(Full documentation in /evaluation/qa_test_scenarios.md)*

### 3.1 Failure Mode 1 — Incorrect Product Retrieval
*(See QA Test Scenario 1)*

### 3.2 Failure Mode 2 — Cross-Style Sizing Contradiction
*(See QA Test Scenario 2)*

### 3.3 Failure Mode 3 — Escalation Failure at Decision Point
*(See QA Test Scenario 3)*

### 3.4 Failure Mode 4 — Stateless Architecture
*(See QA Test Scenario 4)*

---

## 4. The FitExpert System

*(Full documentation in /architecture/system_architecture.md and /algorithm/algorithm_spec.md)*

### 4.1 Design Principles

### 4.2 Six-Node Pipeline Architecture

### 4.3 Measurement-to-Specification Algorithm

### 4.4 Return-Policy-Aware Uncertainty Quantification

---

## 5. Evaluation

*(Full results in /evaluation/qa_test_scenarios.md)*

---

## 6. Discussion and Future Work

The failures documented in this paper are not unique to luxury fashion e-commerce. Any domain where an AI agent must make structured decisions — insurance underwriting, medical triage, financial planning, legal document review — faces the same fundamental tension: language models are good at language, but decisions often require structured reasoning over domain-specific data.

The architectural pattern demonstrated in FitExpert — deterministic algorithm at the decision core, LLM at the language periphery — is generalizable. The return-policy-aware decision bias is one instance of a broader pattern: business constraint integration at the decision layer. Future work will explore this pattern across other high-stakes commercial domains.

Specific future directions:
1. Computer vision-based measurement extraction from customer photos
2. Multi-item cart sizing with cross-garment consistency
3. Longitudinal evaluation across full production deployment
4. Generalization of the failure mode taxonomy to other fashion categories (menswear, footwear, athletic apparel)
5. Extension of the three-verdict framework to a continuous confidence score

---

## References

*(To be completed)*

---

*Draft — Lavan Kumar Gajula, February 2026. Do not distribute.*
