# **Size a Saudi Data Centre \-  ( TEAM 2 )**

*Team Case Study — HUMAIN's First Building*

AI Data Center Bootcamp · Week 5 · Day 3

**SITE**

| 50 MW power connection 18,000 NVIDIA GB300 GPUs Source: NVIDIA, 13 May 2025; DCD, 28 May 2025 Not confirmed in service as of 31 July 2026 |
| :---- |

# **Question 1 — How many racks and GPUs does the site's power buy?**

**FORMULA**

| Compute power \= (Gate power ÷ PUE) × (1 − other IT %) × (1 − headroom %) Racks \= Compute power ÷ power per rack GPUs  \= Racks × GPUs per rack |
| :---- |

**CALCULATION**

| IT power        \= 50 MW ÷ 1.25 \= 40 MW After other IT  \= 40 MW × (1 − 0.10) \= 36 MW After headroom  \= 36 MW × (1 − 0.20) \= 28.8 MW   ← power for compute servers Racks \= 28.8 MW ÷ 0.12 MW/rack \= 240 racks GPUs  \= 240 × 72 \= 17,280 GPUs |
| :---- |

**Result: 240 racks, 17,280 GPUs — within \~4% of the announced 18,000, so the announcement checks out against the arithmetic.**

# **Question 2 — Largest open model it can serve, and how many copies?**

**FORMULA**

| Memory per copy \= (parameters × bytes per parameter) \+ context memory Copies \= Total site memory ÷ Memory per copy |
| :---- |

**CALCULATION**

| Model: Kimi K2, 1 trillion parameters, fp8 (1 byte/param) Weights \= 1,000,000,000,000 × 1 byte \= 1,000 GB Context \= 300 GB   (32 long conversations, reusing the morning slide's figure for this model) Memory per copy \= 1,000 \+ 300 \= 1,300 GB Total site memory \= 240 racks × 20 TB/rack \= 4,800 TB Copies \= 4,800 TB ÷ 1.3 TB \= 3,692 copies |
| :---- |

**Result: Kimi K2 (1T params), \~3,690 copies fit by memory alone — a sign that at this scale, memory is not the real bottleneck.**

# **Question 3 — Largest model trainable in six months?**

**FORMULA**

| Sustained compute per GPU \= peak FLOPS × sustained % Total fleet compute       \= GPUs × sustained compute per GPU × seconds in six months Compute needed to train   \= 6 × N × (20 × N) \= 120 × N² N \= √(Total fleet compute ÷ 120\) |
| :---- |

**CALCULATION**

| Sustained per GPU \= 989 TFLOPS × 0.40 \= 395.6 TFLOPS \= 3.956×10¹⁴ FLOPS/s Total fleet compute (per second) \= 17,280 × 3.956×10¹⁴ ≈ 6.84×10¹⁸ FLOPS/s Six months ≈ 15,768,000 seconds Total ops over 6 months \= 6.84×10¹⁸ × 1.5768×10⁷ ≈ 1.08×10²⁶ operations N² \= 1.08×10²⁶ ÷ 120 ≈ 9.0×10²³ N  \= √(9.0×10²³) ≈ 9.5×10¹¹ |
| :---- |

**Result: ≈ 950 billion parameters — just under Kimi K2's 1 trillion.**

# **Question 4 — Electricity bill for a month?**

**FORMULA**

| Average power  \= Gate power × average draw % Monthly energy \= Average power × hours in a month Bill           \= Monthly energy × rate per kWh |
| :---- |

**CALCULATION**

| Average power  \= 50 MW × 0.65 \= 32.5 MW Hours in month \= 30 × 24 \= 720 h Monthly energy \= 32.5 MW × 720 h \= 23,400 MWh \= 23,400,000 kWh Bill \= 23,400,000 kWh × $0.048/kWh (industrial rate assumed)      \= $1,123,200 |
| :---- |

**Result: ≈ $1,123,200 per month (≈ $1,872,000 if billed at the commercial rate instead — flagged as an assumption).**

# **Question 5 — Cost per million tokens, at 30% and at 80% sold?**

**FORMULA**

| Total monthly cost \= (Non-electricity rate × MW) \+ Electricity bill Max monthly capacity (tokens) \= (tokens/sec/GPU × GPUs) × seconds in a month Tokens sold \= Max monthly capacity × % sold Cost per million tokens \= Total monthly cost ÷ (Tokens sold ÷ 1,000,000) |
| :---- |

**CALCULATION**

| Non-electricity cost \= $450,000/MW × 50 MW \= $22,500,000 Total monthly cost   \= $22,500,000 \+ $1,123,200 \= $23,623,200 Tokens/sec at full tilt \= 125 × 17,280 \= 2,160,000 tokens/sec Seconds in a month      \= 720 × 3,600 \= 2,592,000 s Max monthly capacity    \= 2,160,000 × 2,592,000 ≈ 5.60 trillion tokens At 30%: tokens sold \= 5.60T × 0.30 \= 1.68 trillion → 1,680,000 million tokens Cost/million \= $23,623,200 ÷ 1,680,000 ≈ $14.06 At 80%: tokens sold \= 5.60T × 0.80 \= 4.48 trillion → 4,480,000 million tokens Cost/million \= $23,623,200 ÷ 4,480,000 ≈ $5.27 |
| :---- |

**Result: $14.06/million tokens at 30% sold, $5.27/million tokens at 80% sold — confirmed by the invariant ratio 80÷30 ≈ 2.67 matching 14.06÷5.27 ≈ 2.67.**

# **Summary**

| \# | Question | Formula | Result |
| :---- | :---- | :---- | :---- |
| 1 | Racks & GPUs | (Gate/PUE)×0.9×0.8 ÷ kW/rack | 240 racks, 17,280 GPUs |
| 2 | Largest model served | weights×bytes \+ context, ÷ into total memory | Kimi K2, \~3,690 copies |
| 3 | Largest model trained (6mo) | N \= √(fleet FLOPs ÷ 120\) | \~950 billion params |
| 4 | Monthly bill | Gate × draw% × hours × rate | \~$1,123,200 |
| 5 | Cost / million tokens | Total cost ÷ tokens sold | $14.06 (30%) / $5.27 (80%) |

# **What the Announcement Doesn't Tell You**

There is no public confirmation this site is actually operational as of 31 July 2026 — every number above assumes it is running at full announced power and 65% average draw, but real demand (and therefore whether it lands closer to the $14 or $5 end of the cost range) is entirely unknown from the announcement itself.