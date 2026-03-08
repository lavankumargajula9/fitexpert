# FitExpert Algorithm Specification

**Version:** 1.0  
**Author:** Lavan Kumar Gajula  
**Date:** February 2026  
**Status:** Production Design

---

## 1. Overview

The FitExpert Measurement-to-Specification Matching Algorithm is the core computational engine of the FitExpert AI system. It takes three inputs — customer measurements, garment specification data, and fabric type — and produces a single deterministic sizing verdict with an explanation.

The algorithm is designed around two principles that do not exist in any known prior fashion sizing AI system:

1. **Fabric-aware tolerance** — the acceptable deviation from garment spec varies based on how much the fabric stretches
2. **Return-policy-aware decision bias** — when a customer is borderline, the algorithm biases toward the safer size, accounting for the financial and logistical cost of a return

---

## 2. Inputs

### 2.1 Customer Measurements

```
customer_bust    (float)  — measured in inches, at fullest part of chest
customer_waist   (float)  — measured in inches, at natural waist
customer_hips    (float)  — measured in inches, at fullest part of hips
```

### 2.2 Garment Specification (from Shopify Metafields)

```
bust_ranges   (dict)  — {size: [min, max]} in inches per size
waist_ranges  (dict)  — {size: [min, max]} in inches per size  
hip_ranges    (dict)  — {size: [min, max]} in inches per size
fabric_type   (str)   — one of: jersey_knit, silk_charmeuse, structured_crepe, lace_overlay, beaded_embellished
stretch_pct   (float) — additional inches of stretch beyond spec max
runs_small    (bool)  — if true, apply +0.5 inch correction to customer measurements before calculation
fit_notes     (str)   — free text notes for customer display
```

### 2.3 Return Policy Context

```
return_window_days  (int)   — number of days for return (LARA: 14 days)
return_fee          (float) — restocking/shipping cost (LARA: $30.00)
policy_bias         (bool)  — if true, apply conservative bias at borderline decisions
```

---

## 3. Fabric Stretch Factors

The stretch factor represents additional tolerance in inches that a garment can accommodate beyond its stated spec maximum. This is a property of the fabric, not the size.

| Fabric Type          | Stretch Factor (inches) | Notes                                      |
|----------------------|------------------------|--------------------------------------------|
| jersey_knit          | +1.50"                 | High elasticity. Most forgiving fabric.    |
| silk_charmeuse       | +0.75"                 | Moderate drape. Some natural give.         |
| structured_crepe     | +0.50"                 | Light structure. Minimal stretch.          |
| lace_overlay         | +0.50"                 | Overlay stretches slightly. Lining does not.|
| beaded_embellished   | +0.25"                 | Beading restricts movement. Near zero give.|

---

## 4. Algorithm Steps

### Step 1 — Runs-Small Correction

```python
if runs_small == True:
    adjusted_bust  = customer_bust  - 0.5
    adjusted_waist = customer_waist - 0.5
    adjusted_hips  = customer_hips  - 0.5
else:
    adjusted_bust  = customer_bust
    adjusted_waist = customer_waist
    adjusted_hips  = customer_hips
```

### Step 2 — Retrieve Fabric Stretch Factor

```python
stretch_factors = {
    "jersey_knit":          1.50,
    "silk_charmeuse":       0.75,
    "structured_crepe":     0.50,
    "lace_overlay":         0.50,
    "beaded_embellished":   0.25
}

fabric_stretch = stretch_factors[fabric_type]
```

### Step 3 — Find Candidate Size

Identify which size the customer falls into based on raw measurements:

```python
candidate_size = None

for size in ["XS", "S", "M", "L", "XL", "XXL"]:
    if (bust_ranges[size][0]  <= adjusted_bust  <= bust_ranges[size][1] and
        waist_ranges[size][0] <= adjusted_waist <= waist_ranges[size][1] and
        hip_ranges[size][0]   <= adjusted_hips  <= hip_ranges[size][1]):
        candidate_size = size
        break
```

### Step 4 — Calculate Delta for Each Measurement

```python
bust_delta  = adjusted_bust  - bust_ranges[candidate_size][1]
waist_delta = adjusted_waist - waist_ranges[candidate_size][1]
hip_delta   = adjusted_hips  - hip_ranges[candidate_size][1]
```

Delta is positive when customer measurement EXCEEDS the garment spec max for their candidate size. Delta is negative when they are comfortably within range.

### Step 5 — Apply Uncertainty Thresholds

```python
# Position within size range (0.0 = at minimum, 1.0 = at maximum)
bust_position  = (adjusted_bust  - bust_ranges[candidate_size][0])  / (bust_ranges[candidate_size][1]  - bust_ranges[candidate_size][0])
waist_position = (adjusted_waist - waist_ranges[candidate_size][0]) / (waist_ranges[candidate_size][1] - waist_ranges[candidate_size][0])
hip_position   = (adjusted_hips  - hip_ranges[candidate_size][0])   / (hip_ranges[candidate_size][1]   - hip_ranges[candidate_size][0])

max_position = max(bust_position, waist_position, hip_position)
```

### Step 6 — Apply Return Policy Bias

```python
# Standard thresholds
SAFE_THRESHOLD    = 0.85   # Below this = comfortable fit
BORDERLINE_UPPER  = 1.00   # Above this = exceeds spec

# If return policy bias is active, lower the safe threshold
# This makes the system more conservative — biases toward SIZE UP
if policy_bias == True:
    SAFE_THRESHOLD = 0.80   # Tighter safe zone when returns are costly
```

### Step 7 — Verdict Output

```python
if max_position <= SAFE_THRESHOLD:
    verdict = "SAFE BUY"
    explanation = f"Your measurements fall comfortably within size {candidate_size}. This dress should fit well."

elif max_position <= BORDERLINE_UPPER:
    next_size = get_next_size(candidate_size)
    verdict = "SIZE UP"
    explanation = f"Your measurements are at the upper edge of size {candidate_size}. We recommend size {next_size} for a comfortable fit, especially given this fabric's limited stretch. With a ${return_fee} return fee, sizing up is the safer choice."

else:
    # Measurements exceed spec even after stretch factor
    if any_measurement_within_stretch(adjusted_bust, adjusted_waist, adjusted_hips, bust_ranges, waist_ranges, hip_ranges, candidate_size, fabric_stretch):
        next_size = get_next_size(candidate_size)
        verdict = "SIZE UP"
        explanation = f"Your measurements slightly exceed size {candidate_size}'s spec, but this {fabric_type.replace('_',' ')} fabric provides {fabric_stretch}\" of additional stretch. We recommend size {next_size} for the best fit."
    else:
        verdict = "FIT RISK"
        explanation = f"Your measurements exceed size {candidate_size} beyond what this fabric can accommodate. Please contact us to discuss alternative styles."
```

---

## 5. Complete Verdict Definitions

| Verdict     | Condition                                         | Customer Action                  |
|-------------|--------------------------------------------------|----------------------------------|
| SAFE BUY ✅  | max_position ≤ 0.80 (with bias) or ≤ 0.85       | Purchase with confidence         |
| SIZE UP ⚠️  | 0.80–1.00 range, or slightly exceeds with stretch | Order one size larger            |
| FIT RISK 🚨 | Exceeds spec max even after fabric stretch factor | Do not purchase this style/size  |

---

## 6. Example Calculation

**Customer:** bust 36", waist 29", hips 39"  
**Style:** Silk Charmeuse Evening Gown  
**Size M spec:** bust [34–36"], waist [27–29"], hips [37–39"]  
**Fabric:** silk_charmeuse (+0.75" stretch)  
**Return fee:** $30 (bias active)

```
Candidate size: M (measurements within spec range)

bust_position  = (36 - 34) / (36 - 34) = 1.00  ← AT MAXIMUM
waist_position = (29 - 27) / (29 - 27) = 1.00  ← AT MAXIMUM  
hip_position   = (39 - 37) / (39 - 37) = 1.00  ← AT MAXIMUM

max_position = 1.00

With policy_bias = True → SAFE_THRESHOLD = 0.80
1.00 > 0.80 → SIZE UP

Fabric check: measurements AT spec max, not exceeding it
→ Within stretch tolerance of +0.75"

VERDICT: SIZE UP ⚠️
EXPLANATION: "Your measurements are at the maximum of Size M for each dimension. 
Given silk charmeuse's modest stretch (+0.75"), we recommend Size L for a 
comfortable fit. With a $30 return fee, sizing up is the safer choice."
```

---

## 7. Design Decisions and Research Rationale

### Why three verdicts instead of two?

Binary fit/no-fit systems force customers into a purchase or no-purchase decision. The borderline case — where a customer could fit in this size but would be more comfortable in the next — is where the most returns happen. The SIZE UP verdict captures this explicitly.

### Why fabric stretch factors instead of a fixed tolerance?

A jersey knit dress and a beaded gown with identical size specs behave completely differently on a body. Applying a fixed tolerance across all fabrics produces incorrect recommendations. The stretch factor table was derived from material science properties of common fashion fabrics.

### Why incorporate return policy into the algorithm?

The agent exists inside a commercial environment. A recommendation that ignores the cost and friction of a return is an incomplete recommendation. By building the return policy into the decision boundary, FitExpert aligns with the real incentives of both the customer and the retailer.

---

*Algorithm designed and documented by Lavan Kumar Gajula, February 2026.*  
*All rights reserved. Original intellectual property of the author.*
