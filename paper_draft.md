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

### 2.1 Production Agentic AI Systems and the Role of Verifiers

Pan et al. (2025) studied production-grade agentic AI systems across 12 enterprise case studies spanning business operations, software development, and scientific discovery. Their central finding is that useful agentic AI relies on strong verifiers — deterministic mechanisms that confirm whether an output is correct — rather than advanced AI generation capabilities alone. They conclude that the lack of strong verifiers contributes more to the gap between academic prototypes and production systems than any lack of AI capability. Critically, Stoica et al. (2024), cited by Pan et al., identify specifications as "the missing link" in making LLM system development an engineering discipline. FitExpert directly addresses both findings: the Shopify metafield specification schema provides the structured domain specifications that the baseline chatbot lacked, and the three-verdict measurement algorithm functions as a strong domain-specific verifier for sizing decisions — deterministic, auditable, and consistent across all product styles.

### 2.2 Retrieval-Augmented Generation and Its Limitations

Lewis et al. (2020) introduced Retrieval-Augmented Generation (RAG) as a framework for combining parametric and non-parametric memory in language models, demonstrating improved performance on knowledge-intensive NLP tasks. However, production deployments of RAG-based systems have revealed systematic failure modes when retrieval quality is compromised. The baseline chatbot evaluated in this paper uses a RAG-like architecture for product information retrieval — and Failure Mode 1 (incorrect product retrieval) is a direct consequence of fuzzy matching in the retrieval layer. FitExpert replaces probabilistic retrieval with exact SKU lookup via the Shopify Admin API, eliminating this failure mode entirely.

### 2.3 LLM Agent Architectures

Yao et al. (2023) introduced ReAct, a framework that synergizes reasoning and acting in language models through interleaved thought-action-observation traces. Pan et al. (2025) observe that simple fixed-loop ReAct-based and RAG-based agents are common in production to an extreme, with clear benefits in software maintainability and reliability. FitExpert adopts this principle of deliberate simplicity: a six-node fixed pipeline where each node has a single, well-defined responsibility. The LLM is restricted to natural language understanding and explanation generation — it does not make sizing decisions. This separation of concerns addresses the root cause of Failure Mode 2 (cross-style sizing contradictions), where the baseline system's generative architecture produced inconsistent outputs across product queries.

### 2.4 Fashion Recommendation and Sizing Systems

*(To be completed April 2026 — search: "fashion size recommendation deep learning 2023 2024")*

Key papers to find and cite:
- Survey of AI-based fashion recommendation systems
- Size and fit prediction using body measurements
- Return rate reduction through personalized sizing

### 2.5 Uncertainty Quantification in Conversational AI

*(To be completed April 2026 — search: "uncertainty quantification conversational AI e-commerce")*

Key papers to find and cite:
- Confidence estimation in recommendation systems
- Escalation strategies in task-oriented dialogue systems
- Human-in-the-loop AI for high-stakes decisions

### 2.6 E-commerce Conversational Agents

*(To be completed April 2026 — search: "conversational agent e-commerce customer service LLM 2024")*

Key papers to find and cite:
- State-of-the-art chatbot deployments in retail
- Customer abandonment and AI agent failures
- Personalization in e-commerce AI

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

Lewis, P., Perez, E., Piktus, A., Petroni, F., Karpukhin, V., Goyal, N., ... & Riedel, S. (2020). Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks. *Advances in Neural Information Processing Systems (NeurIPS)*. https://arxiv.org/abs/2005.11401

Pan, M., Zhu, Y., Davis, J.Q., Cogo, R., Agrawal, L.A., Arabzadeh, N., ... & Zaharia, M. (2025). Useful Agentic AI: A Systems Outlook. UC Berkeley / Stanford University / UIUC. https://saa2025.github.io/papers/Useful%20Agentic%20AI%20-%20A%20Systems%20Outlook.pdf

Stoica, I., Zaharia, M., Gonzalez, J., Goldberg, K., Zhang, H., Angelopoulos, A., ... & Davis, J.Q. (2024). Specifications: The missing link to making the development of LLM systems an engineering discipline. *arXiv preprint arXiv:2412.05299*. https://arxiv.org/abs/2412.05299

Yao, S., Zhao, J., Yu, D., Du, N., Shafran, I., Narasimhan, K., & Cao, Y. (2023). ReAct: Synergizing Reasoning and Acting in Language Models. *International Conference on Learning Representations (ICLR)*. https://arxiv.org/abs/2210.03629

*(Additional references to be added April–June 2026)*

---

*Draft — Lavan Kumar Gajula, March 2026. Do not distribute.*
