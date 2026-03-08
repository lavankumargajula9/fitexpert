# FitExpert System Architecture

**Author:** Lavan Kumar Gajula  
**Version:** 1.0  
**Date:** February 2026

---

## 1. Architecture Overview

FitExpert is a six-node agentic pipeline built on n8n workflow automation, integrating the Shopify Admin API for product and customer data, a large language model for natural language reasoning, and a deterministic algorithm engine for sizing decisions.

The architecture is deliberately **structured** rather than purely generative. The sizing verdict is produced by the algorithm engine — not the LLM. The LLM is responsible only for natural language understanding, explanation generation, and conversation management. This separation ensures that sizing decisions are deterministic, auditable, and consistent across all customer interactions.

---

## 2. Full System Diagram

```
╔══════════════════════════════════════════════════════════════════════╗
║                        CUSTOMER LAYER                               ║
║                                                                      ║
║   Customer types in chat widget (Tidio / WhatsApp Business API)     ║
║   "I'm interested in style 29454. Can you help me find my size?"    ║
╚══════════════════════════════╦═══════════════════════════════════════╝
                               ║
                               ▼
╔══════════════════════════════════════════════════════════════════════╗
║                      n8n WORKFLOW ENGINE                             ║
║                                                                      ║
║  ┌─────────────────────────────────────────────────────────────┐    ║
║  │  NODE 1 — Style Number Extractor                            │    ║
║  │  • LLM extracts style number from customer message          │    ║
║  │  • Validates format (numeric, 5-digit)                      │    ║
║  │  • If invalid → ask customer to confirm style number        │    ║
║  └──────────────────────────┬──────────────────────────────────┘    ║
║                             │                                        ║
║  ┌──────────────────────────▼──────────────────────────────────┐    ║
║  │  NODE 2 — Shopify SKU Validator                             │    ║
║  │  • Calls Shopify Products API with exact style number       │    ║
║  │  • Confirms product exists in catalog                       │    ║
║  │  • Returns: product name, price, available sizes            │    ║
║  │  • If not found → "I couldn't find that style. Here are     │    ║
║  │    similar styles: [alternatives]"                          │    ║
║  └──────────────────────────┬──────────────────────────────────┘    ║
║                             │                                        ║
║  ┌──────────────────────────▼──────────────────────────────────┐    ║
║  │  NODE 3 — Garment Spec Retriever                            │    ║
║  │  • Calls Shopify Metafields API for fit_expert namespace    │    ║
║  │  • Retrieves: bust/waist/hip ranges per size                │    ║
║  │  • Retrieves: fabric_type, stretch_pct, runs_small flag     │    ║
║  │  • Retrieves: fit_notes for customer display                │    ║
║  └──────────────────────────┬──────────────────────────────────┘    ║
║                             │                                        ║
║  ┌──────────────────────────▼──────────────────────────────────┐    ║
║  │  NODE 4 — Customer Measurement Handler                      │    ║
║  │  • Checks Shopify Customer Account for saved measurements   │    ║
║  │  • If found → "I have your measurements on file. Shall I    │    ║
║  │    use bust 36", waist 29", hips 39"?"                      │    ║
║  │  • If not found → ask customer to enter measurements        │    ║
║  │  • Validates measurement format and reasonable ranges       │    ║
║  └──────────────────────────┬──────────────────────────────────┘    ║
║                             │                                        ║
║  ┌──────────────────────────▼──────────────────────────────────┐    ║
║  │  NODE 5 — Fit Algorithm Engine  ← CORE RESEARCH NODE       │    ║
║  │                                                             │    ║
║  │  INPUT:  customer measurements + garment spec + fabric      │    ║
║  │                                                             │    ║
║  │  PROCESS:                                                   │    ║
║  │  1. Apply runs-small correction if flagged                  │    ║
║  │  2. Look up fabric stretch factor                           │    ║
║  │  3. Find candidate size from measurement ranges             │    ║
║  │  4. Calculate position within size range (0.0 → 1.0)       │    ║
║  │  5. Apply return-policy bias threshold                      │    ║
║  │  6. Output verdict: SAFE BUY / SIZE UP / FIT RISK           │    ║
║  │                                                             │    ║
║  │  OUTPUT: verdict + recommended size + explanation text      │    ║
║  └──────────────────────────┬──────────────────────────────────┘    ║
║                             │                                        ║
║  ┌──────────────────────────▼──────────────────────────────────┐    ║
║  │  NODE 6 — Response Generator + Profile Writer               │    ║
║  │  • LLM formats algorithm output into natural language       │    ║
║  │  • Saves customer measurements to Shopify Customer Account  │    ║
║  │  • Returns final response to customer in chat               │    ║
╚══╧═════════════════════════════════════════════════════════════╧════╝
                             │
                             ▼
╔══════════════════════════════════════════════════════════════════════╗
║                       DATA LAYER                                     ║
║                                                                      ║
║  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐  ║
║  │  Shopify Products│  │   Claude API /   │  │ Shopify Customer │  ║
║  │  Admin API       │  │   GPT-4o         │  │ Accounts         │  ║
║  │                  │  │                  │  │                  │  ║
║  │  • Product data  │  │  • NLU           │  │  • Saved         │  ║
║  │  • SKU catalog   │  │  • Explanation   │  │    measurements  │  ║
║  │  • Metafields    │  │    generation    │  │  • Order history │  ║
║  │  • Pricing       │  │  • Conversation  │  │  • Preferences   │  ║
║  │  • Availability  │  │    management    │  │                  │  ║
║  └──────────────────┘  └──────────────────┘  └──────────────────┘  ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 3. Data Flow — Step by Step

### 3.1 New Customer, First Visit

```
Customer: "Hi, I'm looking at style 29454"
    ↓
Node 1: Extract "29454" → validate format ✓
    ↓
Node 2: GET /products?handle=29454 → returns "Silk Evening Gown, $658, sizes XS-XL" ✓
    ↓
Node 3: GET /metafields?namespace=fit_expert → returns spec data ✓
    ↓
Node 4: GET /customers/{id}/metafields → no measurements found
         → "To find your perfect size, I'll need your measurements. 
            Could you share your bust, waist, and hip measurements in inches?"
    ↓
Customer: "Bust 36, waist 29, hips 39"
    ↓
Node 5: Algorithm runs → SIZE UP verdict
    ↓
Node 6: Format response → save measurements → return to customer
         → "Based on your measurements and this gown's silk charmeuse fabric,
            I recommend Size L. Your measurements are at the upper edge of 
            Size M, and sizing up ensures a comfortable fit. With our $30 
            return fee, L is the safer choice. ⚠️ SIZE UP → Size L"
```

### 3.2 Returning Customer

```
Customer: "I'm back! Looking at style 31205 this time"
    ↓
Node 1-3: Style validated, specs retrieved
    ↓
Node 4: GET /customers/{id}/metafields → measurements found: bust 36, waist 29, hips 39
         → "Welcome back! I have your measurements on file (bust 36", 
            waist 29", hips 39"). Shall I use these?"
    ↓
Customer: "Yes please"
    ↓
Node 5: Algorithm runs with saved measurements
    ↓
Node 6: Verdict returned instantly — no re-entry required
```

---

## 4. API Integrations

### 4.1 Shopify Products API

```
GET https://{store}.myshopify.com/admin/api/2024-01/products.json?handle={style_number}

Response fields used:
- id
- title
- variants[].title (size names)
- variants[].price
- variants[].inventory_quantity
- metafields (via separate call)
```

### 4.2 Shopify Metafields API

```
GET https://{store}.myshopify.com/admin/api/2024-01/products/{product_id}/metafields.json

Namespace: fit_expert
Keys retrieved:
- bust_ranges    (JSON string)
- waist_ranges   (JSON string)
- hip_ranges     (JSON string)
- fabric_type    (string)
- stretch_pct    (decimal)
- runs_small     (boolean)
- fit_notes      (string)
```

### 4.3 Shopify Customer Metafields API

```
GET  https://{store}.myshopify.com/admin/api/2024-01/customers/{customer_id}/metafields.json
POST https://{store}.myshopify.com/admin/api/2024-01/customers/{customer_id}/metafields.json

Namespace: fit_expert
Keys:
- saved_bust    (decimal)
- saved_waist   (decimal)
- saved_hips    (decimal)
- last_updated  (datetime)
```

### 4.4 Claude API (LLM Layer)

```
POST https://api.anthropic.com/v1/messages

Model: claude-sonnet
Max tokens: 500
Temperature: 0.3 (low temperature for consistent, factual responses)

System prompt responsibilities:
- Extract style numbers from natural language
- Format algorithm verdicts into customer-friendly explanations
- Handle edge cases (customer unsure of measurements, requests alternatives)
- Maintain conversational tone while staying factual

The LLM does NOT make sizing decisions. 
All sizing decisions are made by the algorithm engine in Node 5.
```

---

## 5. Design Principles

### 5.1 Deterministic Core, Generative Periphery

The most important architectural decision in FitExpert is the separation of concerns between the algorithm engine and the LLM. The LLM handles language — understanding what the customer is asking, and explaining what the algorithm decided. The algorithm handles decisions — producing a consistent, auditable sizing verdict.

This prevents the LLM from "hallucinating" sizing recommendations, which is the root cause of Failure Mode 2 (cross-style sizing contradictions) in the baseline chatbot.

### 5.2 Exact Retrieval Over Fuzzy Matching

Every product lookup in FitExpert uses exact SKU matching via the Shopify Products API. There is no similarity search, no fuzzy matching, no vector embedding retrieval. If a customer enters an invalid style number, the system says so and asks for clarification — it does not guess.

This directly addresses Failure Mode 1 (incorrect product retrieval).

### 5.3 State Persistence as First-Class Requirement

Customer measurements are saved to their Shopify account at the end of every interaction. This is not optional — it is a core function of Node 6. Every future interaction begins by checking for saved measurements.

This directly addresses Failure Mode 4 (stateless architecture).

### 5.4 Business Context Awareness

The algorithm knows the return policy. It knows the return fee. It uses this information to make recommendations that serve the customer's real interests, not just their immediate measurement math. This is a novel pattern in fashion AI — business constraint integration at the decision layer, not the presentation layer.

---

## 6. Deployment

| Environment | Technology | Notes |
|-------------|-----------|-------|
| Workflow Engine | n8n (self-hosted) | $20/month on Railway |
| LLM API | Anthropic Claude API | $10–50/month depending on volume |
| Chat Widget | Tidio | Free tier sufficient for POC |
| Store Platform | Shopify | Client's existing infrastructure |
| Total operating cost | $35–115/month | vs $17,000–38,500 agency build |

---

*Architecture designed by Lavan Kumar Gajula, February 2026.*  
*Original intellectual property of the author. All rights reserved.*
