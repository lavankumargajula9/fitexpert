# FitExpert Evaluation — QA Test Scenarios

**Author:** Lavan Kumar Gajula  
**Date:** February 2026  
**Purpose:** Comparative evaluation of FitExpert vs production baseline chatbot

---

## Evaluation Methodology

Five controlled test scenarios were designed to directly target the four documented failure modes. Each scenario uses a fixed customer profile and a fixed product, tested against both the baseline production chatbot and FitExpert under identical conditions.

**Evaluation criteria per scenario:**
- Correct product retrieved? (Y/N)
- Consistent verdict across styles? (Y/N)
- Definitive recommendation given? (Y/N)
- Customer state preserved? (Y/N)
- Return policy considered? (Y/N)

---

## Test Scenario 1 — Incorrect Product Retrieval

**Target Failure Mode:** Failure Mode 1  
**Customer query:** "Can you help me find my size for style 29454?"  
**Correct product:** Style 29454 — Silk Lace Evening Gown, $658

| Metric | Baseline Chatbot | FitExpert |
|--------|-----------------|-----------|
| Product retrieved | Style 29004 ($550 beaded gown) ❌ | Style 29454 ($658 lace gown) ✅ |
| Correct specs used | No — wrong garment | Yes — exact SKU match |
| Customer impact | Receives sizing advice for wrong dress | Receives correct advice |
| Verdict | FAIL | PASS |

**Root cause of baseline failure:** Fuzzy text matching returned the nearest string match rather than exact style number lookup.

---

## Test Scenario 2 — Cross-Style Sizing Contradiction

**Target Failure Mode:** Failure Mode 2  
**Customer profile:** bust 36", waist 29", hips 39"  
**Style A:** 29454 — Silk Lace Gown (structured crepe lining)  
**Style B:** 31205 — Jersey Wrap Dress (jersey knit)

| Metric | Baseline Chatbot | FitExpert |
|--------|-----------------|-----------|
| Verdict for Style A | Size 6 | SIZE UP → Size L |
| Verdict for Style B | Size 10–12 | SAFE BUY → Size M |
| Size contradiction | 4-size gap (S6 vs S10-12) ❌ | Consistent — fabric difference explained ✅ |
| Explanation provided | None | "Jersey knit provides 1.5" more stretch than structured crepe, which is why Size M works here but we recommended L for the lace gown" |
| Customer impact | Complete confusion, abandoned purchase | Understands why sizes differ across styles |
| Verdict | FAIL | PASS |

**Note:** FitExpert's verdicts differ across styles, but for a correct reason — the fabric stretch factor. The baseline chatbot's contradiction has no logical basis.

---

## Test Scenario 3 — Escalation Failure at Decision Point

**Target Failure Mode:** Failure Mode 3  
**Customer profile:** bust 37.5", waist 30.5", hips 40" (borderline between M and L)  
**Style:** 29454 — Silk Charmeuse Evening Gown  
**Customer question:** "Does this dress have stretch? I'm between a 10 and 12 — which is safer given your return policy?"

| Metric | Baseline Chatbot | FitExpert |
|--------|-----------------|-----------|
| Fabric information provided | None ❌ | "Silk charmeuse provides approximately 0.75 inches of natural give" ✅ |
| Return policy referenced | None ❌ | "With our $30 return fee, sizing up is the safer financial choice" ✅ |
| Definitive recommendation | "Please leave your email and our team will get back to you" ❌ | "SIZE UP → Size L — your measurements are at the upper edge of M, and given the limited stretch of silk charmeuse and our return fee, L is the safer choice" ✅ |
| Customer outcome | Left and purchased at Nordstrom | Received actionable recommendation, able to purchase |
| Verdict | FAIL | PASS |

---

## Test Scenario 4 — Stateless Architecture / No Memory

**Target Failure Mode:** Failure Mode 4  
**Customer:** Returning customer, previously measured: bust 36", waist 29", hips 39"  
**Interaction:** Customer returns one week later asking about a new style

| Metric | Baseline Chatbot | FitExpert |
|--------|-----------------|-----------|
| Customer recognized | No ❌ | Yes — Shopify account checked ✅ |
| Measurements retrieved | Not available — must re-enter ❌ | Retrieved: "I have your measurements on file: bust 36", waist 29", hips 39"" ✅ |
| Time to recommendation | Customer re-enters measurements (~2–3 minutes) | Immediate — confirmed with one click |
| Customer experience | Friction and frustration | Seamless repeat experience |
| Verdict | FAIL | PASS |

---

## Test Scenario 5 — Measurements Exceed All Available Sizes

**Target Failure Mode:** Edge case — no suitable size exists  
**Customer profile:** bust 48", waist 42", hips 50" (exceeds XL in all measurements)  
**Style:** 29454 — Silk Lace Gown (available sizes XS–XL)

| Metric | Baseline Chatbot | FitExpert |
|--------|-----------------|-----------|
| Handles out-of-range gracefully | Recommends "XL" without flagging concern ❌ | FIT RISK verdict with clear explanation ✅ |
| Alternative suggestions | None ❌ | "This style does not accommodate your measurements. We recommend styles 31456 and 31789 which are available in extended sizing" ✅ |
| Customer impact | Customer purchases XL, dress does not fit, return required | Customer redirected to appropriate alternatives |
| Verdict | FAIL | PASS |

---

## Summary Results

| Test Scenario | Failure Mode Targeted | Baseline | FitExpert |
|--------------|----------------------|----------|-----------|
| T1 — Wrong product retrieval | Failure Mode 1 | ❌ FAIL | ✅ PASS |
| T2 — Cross-style contradiction | Failure Mode 2 | ❌ FAIL | ✅ PASS |
| T3 — Escalation failure | Failure Mode 3 | ❌ FAIL | ✅ PASS |
| T4 — No session memory | Failure Mode 4 | ❌ FAIL | ✅ PASS |
| T5 — Out-of-range measurements | Edge case | ❌ FAIL | ✅ PASS |

**Baseline chatbot: 0/5 scenarios passed**  
**FitExpert: 5/5 scenarios passed**

---

## Limitations and Future Work

1. **Sample size:** Evaluation conducted on 5 controlled scenarios. Full production evaluation requires real customer interaction data across all 20 SKUs in scope.

2. **Measurement accuracy:** The algorithm assumes accurate self-reported measurements. A future direction is computer vision-based measurement from photos.

3. **Fabric database coverage:** Current stretch factors cover 5 fabric types. Expansion to 20+ fabric types requires materials science validation.

4. **Multi-garment sessions:** Current architecture handles one style per session. Future work: multi-item cart sizing with cross-item consistency checking.

5. **Cultural measurement conventions:** Customers from different regions may report measurements in different conventions (e.g., EU sizing, CM vs inches). Localization layer needed for international deployment.

---

*Evaluation conducted by Lavan Kumar Gajula, February 2026.*
