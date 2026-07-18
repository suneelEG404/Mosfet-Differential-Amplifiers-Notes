# Chapter 6 — Differential and Multistage Amplifiers
### Study Notes (based on SJTU Microelectronic Circuits slides, Yang Hua)

---

## 0. Why Differential Amplifiers Exist (Motivation)

Direct-coupled (DC) amplifier stages are needed in ICs because coupling/bypass capacitors can't be fabricated economically on-chip. But direct coupling between stages/signal source causes **temperature drift (zero drift)** — the output creeps even with zero input, because bias points shift with temperature and every stage's shift gets amplified by the next.

**Two key advantages of differential over single-ended amplifiers:**
1. Much less sensitive to noise/interference (common-mode signals picked up equally on both inputs cancel).
2. Enables biasing and inter-stage coupling *without* bypass/coupling capacitors — essential for IC fabrication.

> 💡 **My take:** This is really the foundational idea of essentially all analog IC design — the diff pair isn't just "a nice circuit," it's the *only* practical way to cascade DC-coupled gain stages without the offset of one stage blowing up the bias of the next. Once you see this, op-amp front ends, ADC comparators, and even ECL logic all look like variations on the same trick.

---

## 1. The BJT Differential Pair (Section 6.1)

**Topology:** Two matched BJTs (Q1, Q2) with coupled emitters, fed by a single tail current source *I* (ideal, infinite output resistance), each collector through $R_C$ to $V_{CC}$.

### Basic Operation — 4 regimes

| Case | Input | Result |
|---|---|---|
| (a) Common-mode | $v_{CM}$ on both bases | Current splits *I*/2, *I*/2 equally; $v_{C1}=v_{C2}$; **CM rejected** as long as both transistors stay active |
| (b) Large diff., +1V vs 0V | Q1 fully ON, Q2 fully OFF | All of *I* flows in Q1 |
| (c) Large diff., −1V vs 0V | Q1 OFF, Q2 ON | All of *I* flows in Q2 |
| (d) Small diff. signal $v_i$ | Linear region | Increment $\Delta I$ in Q1, decrement in Q2 — **linear amplification** |

> 💡 **My take:** This four-slide sequence (common-mode → hard switch each way → small-signal) is the cleanest way to *build intuition* before the math. It shows the diff pair is fundamentally a **current-steering switch** that happens to behave linearly near balance — this is exactly why the same topology reappears as an ECL logic gate (hard switching) and as a linear front-end (small-signal). Same circuit, different signal amplitude.

### Large-Signal Transfer Characteristic

$$i_{E1} = \frac{I_S}{\alpha}e^{(v_{B1}-v_E)/V_T}, \quad i_{E2} = \frac{I_S}{\alpha}e^{(v_{B2}-v_E)/V_T}$$

With $i_{E1}+i_{E2}=I$:

$$i_{E1} = \frac{I}{1+e^{(v_{B2}-v_{B1})/V_T}}, \qquad i_{E2} = \frac{I}{1+e^{(v_{B1}-v_{B2})/V_T}}$$

This gives the classic **tanh-shaped** normalized transfer curve of $i_C/I$ vs $v_{id}/V_T$, with:
- **Linear region** roughly $|v_{id}| \lesssim V_T/2$ (small-signal amp)
- **Full switching** at $|v_{id}| \approx 4V_T \approx 100\text{ mV}$ (fast current switch / logic)

**Linearizing trick:** insert equal emitter resistances $R_e$ in series with each emitter → trades gain for linear range (larger $IR_e/V_T$ → wider linear region, lower gain). Classic linearity-vs-gain tradeoff.

> 💡 **My take:** The $4V_T \approx 100\text{mV}$ threshold number is worth memorizing — it's the reason ECL logic swings are so small (~0.3–0.8V) and why BJT diff pairs make such fast comparators/switches.

---

## 2. Small-Signal Operation (Section 6.2)

For $v_{id} \ll 2V_T$:
$$g_m = \frac{\alpha I}{2V_T} \approx \frac{I}{2V_T}$$

**Half-circuit trick derivation (no $R_e$):**
$$r_e = \frac{V_T}{I/2}, \quad i_c = \alpha i_e = \frac{\alpha v_d}{2r_e} = g_m\frac{v_d}{2}$$

**With emitter resistors $R_e$:**
$$R_{id} = (1+\beta)(2r_e + 2R_e)$$

### Differential Gain
| Output taken | No $R_e$ | With $R_e$ |
|---|---|---|
| Single-ended | $A_{d1,2} = \mp\frac{1}{2}g_mR_C$ | $\mp\frac{1}{2}\frac{R_C}{r_e+R_e}$ |
| Differential | $A_d = g_mR_C$ | $\frac{R_C}{r_e+R_e}$ |

> Key rule of thumb (stated explicitly on the slides): **"Voltage gain = ratio of total resistance in collector circuit to total resistance in emitter circuit."** This is a very fast sanity-check formula worth memorizing.

### Differential Half-Circuit Analysis
Because the joint-emitter node sits at **0V (virtual ground)** for pure differential drive, each side is just a common-emitter amplifier biased at *I*/2. This "half-circuit" trick lets you reuse ordinary CE-amplifier formulas for gain, $R_{id}$, and frequency response — **huge simplification**.

Important subtlety (slide 21): the half-circuit equivalence also holds for **single-ended drive** (one input driven, other grounded) as long as $R_{EE} \gg r_e$ — because the emitter signal voltage still $\approx v_{id}/2$ and current into $R_{EE}$ is negligible.

> 💡 **My take:** The half-circuit method is *the* single most powerful analysis shortcut in this chapter. Once mastered, you never need to redo the "big" differential-pair small-signal analysis by hand again — just draw one CE stage biased at I/2 and apply standard formulas.

---

## 3. Common-Mode Gain & CMRR

With finite tail resistance $R_{EE}$ (not ideal current source):

$$A_{cm,\text{single-ended}} = -\frac{R_C}{2R_{EE}} \qquad A_{cm,\text{diff. output}} = 0 \text{ (if perfectly matched)}$$

$$CMRR_{\text{single-ended}} = \left|\frac{A_d}{A_{cm}}\right| \approx g_mR_{EE} \qquad CMRR_{\text{diff.}} = \infty \text{ (ideal, matched)}$$

**Mismatch effects** (real, non-ideal circuits):
$$CMRR = \frac{2g_mR_{EE}}{\Delta R_C/R_C} \qquad \text{or} \qquad CMRR = \frac{2g_mR_{EE}}{\Delta g_m/g_m}$$

**Input common-mode resistance:**
$$R_{icm} \approx (1+\beta)\left(R_{EE}\parallel \frac{r_o}{2}\right)$$
— very large, as desired (an ideal input shouldn't load down whatever's driving vCM).

> 💡 **My take:** This is where the "differential output = infinite CMRR" claim needs a grain of salt — it's a **first-order idealization**. In practice, differential output only helps CMRR to the extent the two halves are matched; component mismatch ($\Delta R_C$, $\Delta g_m$) is what ultimately limits real CMRR, often to 60–100 dB in practice, not infinity. This is exactly why datasheets specify CMRR as a number, not "infinite."

---

## 4. Other Nonideal Characteristics (Section 6.3)

- **Input offset voltage $V_{OS} = V_O/A_d$** — the equivalent input voltage needed to null the DC output offset caused by unavoidable mismatches (in real fabricated devices, $\beta$, $I_S$, $R_C$ never match perfectly).
- **Input bias/offset currents:** ideally $I_{B1}=I_{B2} = \frac{I/2}{\beta+1}$; mismatch gives $I_{OS}=|I_{B1}-I_{B2}|$.
- **Input common-mode range:** the $v_{CM}$ range over which both transistors stay in the active region (bounded by output swing and headroom for the tail current source).

---

## 5. MOS Differential Amplifiers (Section 6.4)

Same conceptual structure as BJT pair, using MOSFETs (Q1, Q2), tail current *I*, drain resistors $R_D$.

### Large-Signal Behavior
- Common-mode input: current splits equally, symmetric, CM rejected (same as BJT case).
- Differential input steers *I* from one MOS to the other over the range:
$$-\sqrt{2}V_{OV} \le v_{id} \le \sqrt{2}V_{OV}$$
(compare to BJT's $\pm 4V_T$ — the "linear window" is set by **overdrive voltage $V_{OV}$** here, not $V_T$.)

> 💡 **My take:** This is the crucial **design tradeoff you control in MOS but not (as directly) in BJT**: increasing $V_{OV}$ (by shrinking W/L or biasing at higher $I$) widens the linear input range but proportionally *reduces* $g_m = I/V_{OV}$, hence reduces gain. In BJT, $V_T$≈26mV is fixed by physics — you can only widen the linear range with emitter degeneration resistors $R_e$. So MOS diff pairs give you a much more direct "linearity vs. gain" knob via device sizing, at the cost of generally lower $g_m$ per unit current than BJTs (since $g_m^{BJT}=I/V_T$ always beats $g_m^{MOS}=I/(V_{OV}/2)$ for typical $V_{OV}>V_T$).

### Small-Signal Gain
$$g_m = \frac{I}{V_{OV}} \qquad A_d = g_mR_D \text{ (ideal)}, \qquad A_d = g_m(R_D\parallel r_o) \text{ (with } r_o\text{)}$$

### Common-Mode Gain & CMRR (with $R_{SS}$ finite)
$$A_{cm} \approx -\frac{R_D}{2R_{SS}}, \qquad CMRR = 2g_mR_{SS} / (\Delta R_D/R_D \text{ or } \Delta g_m/g_m)$$

Same structure as the BJT case — the math is essentially a direct MOS-transistor substitution ($g_m$ formula changes, everything else parallels the BJT derivation).

---

## 6. Biasing in Integrated Circuits (Section 6.5)

**IC design philosophy** (explicitly listed — very good to remember for exams):
- Use MOS/BJT transistors to realize functions wherever possible.
- **Avoid** large/moderate-value resistors (they eat chip area).
- Constant-current sources are cheap/available on-chip — use them liberally.
- No coupling/bypass capacitors on-chip (only externally, if at all).

### Basic Current Mirror (BJT)
$$\frac{I_O}{I_{REF}} = \frac{\beta}{\beta+2} = \frac{1}{1+2/\beta}$$
— error term from finite $\beta$ (base currents "stolen" from the mirror).

### Improved Mirrors
| Circuit | Key result | Improves |
|---|---|---|
| Base-current-compensated mirror | $I_O/I_{REF} \approx 1/(1+2/\beta^2)$ | Accuracy ($\beta$-error reduced from $O(1/\beta)$ to $O(1/\beta^2)$) |
| Wilson current mirror | Same accuracy as above + $R_o \approx \beta r_o/2$ | Both accuracy **and** output resistance |
| Widlar current source | $I_O = \frac{V_T}{R_E}\ln(I_{REF}/I_O)$ (transcendental!), $R_o \approx (1+g_mR_E')r_o$ | Lets you generate **small** currents with **small** resistors (saves chip area) + boosts $R_o$ |

> 💡 **My take:** The Widlar source is genuinely clever and worth understanding conceptually even if you never solve the transcendental equation by hand: it exploits the *logarithmic* $V_{BE}$–$I_C$ relationship so that a modest resistor can support a large voltage difference (hence small output current) — a linear resistor sized for a "normal" current would need to be huge to directly generate a µA-level current from a mA-level reference.

### MOSFET Analogs
$$\frac{I_O}{I_{REF}} = \frac{(W/L)_2}{(W/L)_1}$$
— set by **device geometry ratio**, not currents/betas! This is the crucial practical advantage of MOS current mirrors: precise, layout-controlled ratios instead of matching betas.

**Wilson MOS mirror:** $R_o \approx g_{m3}r_{o3}r_{o2}$ — much higher output resistance (cascode-like boosting).

### Comparison Table (BJT vs MOS mirrors, slide 88)
1. MOS mirror has **no finite-β error** at all (gate draws no DC current).
2. BJT allows output voltage down to $V_{CE,sat}$; MOS needs $V_{GS}-V_t > V_{OV}$ headroom — tighter as supply voltages shrink.
3. BJT ratio set by **relative emitter areas**; MOS ratio set by **relative W/L**.
4. Both have $r_o = V_A/I$, but $V_A$ tends to be **lower for MOS** devices (shorter channel lengths → more pronounced channel-length modulation) — so BJT current sources are generally "stiffer" (higher output resistance) for comparable bias current.

> 💡 **My take:** Point 2 above is *the* practical driver behind why low-voltage analog CMOS design is hard — as $V_{DD}$ shrinks toward ~1V, there's barely enough headroom to stack a cascode current source and still keep transistors in saturation. This is precisely why the slides note (p.57) "low-voltage operation... poses a host of challenges," and it's a big reason folding cascodes / gain-boosting techniques exist in modern op-amp design.

---

## 7. Differential Amplifier with Active Load (Section 6.6)

**Motivation:** Replace $R_D$ (or $R_C$) with a current-mirror active load:
1. Converts differential output → **single-ended** output (needed to drive most subsequent single-ended stages / usable op-amp output).
2. Achieves **much higher gain** than a resistor load could (since $r_o \gg$ any practical on-chip $R$), while also **saving chip area**.

### Key Quiescent Insight
If perfectly matched, **no DC current flows out the single-ended output node** — all the differential-mode signal current, however, *does* flow out (this is the "magic" of the active load: DC balance + AC current summing).

### Analysis Method: Short-Circuit Transconductance $G_m$ + Output Resistance $R_o$
Because the active-loaded circuit is **not symmetric**, the half-circuit trick from Section 6.2 **does not apply directly**. Instead:
$$A_d = G_mR_o$$
- $G_m = g_m$ (short-circuit transconductance, same magnitude as a simple diff pair)
- **BJT case:** $R_o = r_{o2}\parallel r_{o4}$ (parallel combination of NPN and PNP output resistances at the output node)
- **MOS case:** identical structure, $R_o = r_{o2}\parallel r_{o4}$

$$A_d = g_m(r_{o2}\parallel r_{o4}), \quad \text{if } r_{o2}=r_{o4}=r_o: \; A_d = \frac{1}{2}g_mr_o = \frac{A_0}{2}$$

where $A_0 = g_mr_o$ is the **intrinsic gain** of a single transistor — this connects diff-pair-with-active-load gain directly to the fundamental "gain limit" of the device technology.

**Input differential resistance:** $R_{id}=2r_\pi$ (BJT case) — unchanged from resistor-loaded case.

### Common-Mode Gain / CMRR (active load, BJT)
$$A_{cm} \approx -\frac{r_{o4}}{\beta_3 R_{EE}}, \qquad CMRR = (g_mr_{o2}\parallel r_{o4})\cdot\frac{\beta_3R_{EE}}{r_{o4}}$$

(MOS version): $A_{cm} \approx -\frac{1}{2g_{m3}R_{SS}}$, $CMRR = (g_mr_o)(g_mR_{SS})$

> 💡 **My take:** This section is arguably the most *practically important* one in the whole chapter — this exact topology (diff pair + current-mirror active load) **is** the classic 5-transistor OTA / first stage of essentially every two-stage CMOS op-amp (see the two-stage CMOS op-amp on slide 98 — Q1/Q2 diff pair with Q3/Q4 mirror load, feeding a second CE/CS gain stage with Miller compensation $C_C$). Understanding *why* $A_d = G_mR_o$ (and why the half-circuit trick breaks here) is the conceptual bridge between "toy" diff-pair analysis and real op-amp design.

---

## 8. Multistage Amplifiers (Section 6.7 / 6.9)

**Worked example structure** (4-stage op-amp-like amplifier):

| Stage | Transistors | Function | Gain (from worked numbers) |
|---|---|---|---|
| 1 | Q1, Q2 | Diff-in, diff-out (input stage) | $A_1 = 22.4$ |
| 2 | Q3(bias)/Q4, Q5 | Diff-in, single-ended-out | $A_2 = -59.2$ |
| 3 | Q7 (pnp) | CE stage — **also shifts DC level** | $A_3 = -6.42$ |
| 4 | Q8 | Emitter follower (output buffer) | $A_4 = 0.998$, $R_o = 152\,\Omega$ |

**Overall gain** $\approx A_1 \times A_2 \times A_3 \times A_4 \approx 22.4\times(-59.2)\times(-6.42)\times0.998 \approx 8480$ (≈78.6 dB) — I calculated this since the slides give per-stage but not overall gain explicitly.

Key design roles:
- **Stage 1 (diff pair):** sets input resistance, rejects common-mode/noise, provides initial gain.
- **Stage 2 (diff pair, single-ended out):** converts to single-ended, provides bulk of the voltage gain (loaded by high $R_{i3}$).
- **Stage 3 (pnp CE):** its main *job* is not gain, but **level-shifting** the DC voltage back down so the final stage's output can sit near 0V/ground — a very common multistage-amp trick since cascading NPN CE stages otherwise keeps ratcheting the DC level up toward $V_{CC}$.
- **Stage 4 (emitter follower):** low output resistance, near-unity gain, buffers to the load without loading down stage 3.
- Q9 + 28.6kΩ resistor: bias-current-generation branch (Widlar-type) feeding the whole IC's bias references.

> 💡 **My take:** This worked example is a great illustration of the "**gain stage vs. utility stage**" division-of-labor principle in analog design: not every stage is there to add gain. Stage 3 (Q7) trades away substantial gain (only 6.4×, actually *inverting*) purely to solve a DC level-shifting problem — a "cost of doing business" that a real designer must budget for. Recognizing which stage is "the gain stage" vs. "the plumbing" is a skill that transfers directly to reading real op-amp schematics (e.g., the 741).

### Two-Stage CMOS Op-Amp (slide 98, capstone circuit)
- **Stage 1:** Q1/Q2 diff pair with Q3/Q4 active (current-mirror) load — high gain, converts diff→single-ended (exactly Section 6.6's circuit!).
- **Stage 2:** Q6 (common-source, using node D2/D6) with Q7 as active load — second gain stage.
- **$C_C$ (Miller compensation capacitor):** bridges the two gain stages — for frequency stability (this connects forward to Chapter 8/9 material on op-amp frequency response, not covered in these slides but flagged by the presence of $C_C$).
- Q8, Q5 form the current-mirror bias reference for the whole amp, from $I_{REF}$.

> 💡 **My take:** This final circuit is essentially the "answer" to the whole chapter — it's a minimal but *real* two-stage CMOS op-amp topology (very close to the classic Miller op-amp used in countless real ICs). Everything before it (diff pairs, active loads, current mirrors, level shifting, output buffering) are the individual LEGO bricks; this schematic is where they snap together. If I were prioritizing study time, I'd make sure I can label every transistor's role in this one schematic from memory.

---

## 9. Worked Numeric Example (6.1) — What to Practice

Given: $R_C=10\text{k}\Omega$, $R_E=150\Omega$ (each), $R_{EE}=200\text{k}\Omega$, $I=1\text{mA}$, ±15V rails, 5kΩ series input resistors.

The slides pose (without giving numeric answers) — good practice targets:
1. Input differential resistance $R_{id}$
2. Overall differential voltage gain (excluding $r_o$)
3. Worst-case common-mode gain (±1% $R_C$ mismatch)
4. CMRR in dB
5. Input common-mode resistance (given Early voltage $V_A=100$V)

**Suggested approach / sanity check (not shown in slides — my addition):**
- $I_{CQ}=I/2=0.5\text{mA}$ per side ⟹ $r_e = V_T/I_E \approx 26\text{mV}/0.5\text{mA}=52\,\Omega$
- $R_{id}=(1+\beta)(2r_e+2R_E)$ — plug in your assumed $\beta$ (often 100 in these problems) to get a numeric $R_{id}$ in the tens-of-kΩ range, then add the external $2\times5\text{k}\Omega$ if the question wants total resistance the source sees.
- Gain rule-of-thumb: $A_d = R_C/(r_e+R_E) = 10000/(52+150) \approx 49.5$ (single-ended factor; ×2 if output taken differentially… but note here $R_E$ is *not* shared, so use the with-$R_E$ formula from Section 2 directly, not the bare $g_mR_C$ one).
- $A_{cm} \approx R_C/(2R_{EE}) = 10000/400000 = 0.025$ nominal; worst-case with 1% $R_C$ mismatch use the mismatch formula.
- $CMRR_{dB} = 20\log_{10}(A_d/A_{cm})$.

*(I'm providing the method rather than final numbers since the original slide leaves this as a homework problem — happy to work through exact numeric answers with you if you specify $\beta$ and want a full solution.)*

---

## 10. Overall Summary / Concept Map

```
Differential Pair (BJT or MOS)
   │
   ├── Large-signal: tanh-like transfer curve → switch (logic) or linear amp
   ├── Small-signal: half-circuit trick (symmetric case) → CE/CS analysis reused
   ├── Non-idealities: offset V/I, finite CMRR, finite Ricm
   │
   ├── + Resistive load (R_C/R_D) → basic gain stage, half-circuit works
   ├── + Active load (current mirror) → NO half-circuit, use G_m·R_o,
   │        higher gain (~A0/2), converts diff→single-ended
   │
   └── Biasing infrastructure: current mirrors (basic → β-compensated →
        Wilson → Widlar), IC design philosophy (no big R, no caps)
             │
             └── All combine into: Multistage Amplifier
                  (diff-in/diff-out → diff-in/SE-out → level-shift CE →
                   emitter-follower buffer)  ≈  real two-stage CMOS Op-Amp
```

---

## My Overall Assessment of the Deck

**Strengths:**
- Excellent pedagogical sequencing: qualitative operation (CM, hard-switch, small-signal) *before* equations — this is the right order to build intuition.
- The half-circuit method is introduced and then explicitly flagged as *breaking down* for active loads — a subtlety many textbooks gloss over, but it's called out clearly here (slide 80: "half-circuit is not available due to mismatch in circuit").
- Good bridge from "toy" diff pair → practical current mirrors → real multistage/op-amp circuit. By the end (slide 98), you can see exactly how each earlier concept becomes one piece of a real op-amp.

**Gaps / things I'd supplement if studying from this alone:**
- No frequency-response / pole-zero analysis is shown, even though $C_C$ (Miller compensation) appears in the final CMOS op-amp figure — that's Chapter 8/9 material, so keep in mind this deck is *not* self-contained for stability analysis.
- Numeric answers for Example 6.1 aren't included in the slides (they're posed as an in-class/homework exercise) — I've sketched the method above; let me know if you want me to carry it through with specific $\beta$/$V_A$ values.
- The mismatch/CMRR formulas are presented somewhat tersely; if this is for an exam, I'd recommend re-deriving $CMRR = 2g_mR_{ss}/(\Delta R_D/R_D)$ from KVL by hand once, rather than memorizing it, since the "$2g_mR_{ss}$" factor recurs everywhere in this chapter and is easy to mix up with the plain $g_mR_{ss}$ CMRR formula for the *ideal* matched case.

---

*Note: This document also references homework slides (pages 54, 89, 99) listing textbook problems (6.1, 6.18, 6.21, 6.27, 6.45, D6.60, 6.87, 6.121) — these appear to be assignment references specific to the SJTU course and are tied to a particular textbook edition (likely Sedra/Smith, *Microelectronic Circuits*), not concepts to summarize independently.*
