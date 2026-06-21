---
name: persuade
description: Enhance any text's persuasiveness while keeping every claim factually accurate and preserving the document's structure, citations, voice, and length. Generates three enhanced variants, rank-orders them head-to-head, fact-checks the winner (Consensus MCP + web search), and replaces any inaccurate claim with the strongest accurate replacement available. Use when the user wants to make an essay, op-ed, grant proposal, or any prose more persuasive without sacrificing factual accuracy or scholarly apparatus.
user-invocable: true
allowed-tools: Read, Write, Edit, WebSearch, Agent, mcp__consensus__search
argument-hint: <file-path-or-text>
---

# Persuade

Enhance a text's persuasiveness while preserving its structure, citations, voice, and length — and keeping every claim factually accurate.

## Input

`$ARGUMENTS` is either:
- A path to a text/markdown file (read it with the Read tool), or
- Inline text to enhance directly.

If no argument is provided, ask the user to supply the text or a file path.

---

## PRESERVATION CONTRACT (read first, never violate)

These constraints apply to every step. They are non-negotiable.

1. **Preserve all citations and references verbatim.** If the source contains footnotes, endnotes, inline citations, DOIs, PMIDs, URLs, or bibliographic entries, they must appear in the output in their original format and position. Never compress a full reference into a parenthetical. Never convert formats. Citations are load-bearing, not removable scaffolding.

2. **Preserve document structure.** Do not invent, merge, split, rename, or reorder sections. Do not change headers. Do not convert a numbered outline into narrative prose, or vice versa. The document's overall arc must be recognizable.

3. **Within a section, light paragraph reorganization is permitted** — you may reorder paragraphs inside an existing section, add new sentences, or tighten sentences in place. You may not move content *between* sections or delete a paragraph's substance.

4. **Length parity within ±5%.** The output should be approximately the same length as the input. Minor variance from in-place editing (a few words either way) is expected and fine. Substantial shortening (more than 5% below input length) indicates content loss and is a preservation violation; substantial lengthening (more than 5% above) suggests unnecessary padding.

5. **Preserve author voice and register.** If the input is formal academic prose, the output stays formal academic. If it's conversational, it stays conversational. Do not "polish" the author's voice into a different register.

6. **Preserve all original factual claims and examples.** You may tighten their framing, but you may not remove a claim or example the author included. New supporting claims may be ADDED (and will be fact-checked); original ones are preserved.

If any enhancement would require violating these constraints, do not make that enhancement. It is better to leave a passage unchanged than to violate the contract.

---

## Workflow

### Step 1 — Read the input

Locate and read the source text. Store it as `SOURCE`. Note its structure (sections, headers, citation format), voice, and length — these will be checked against the output.

### Step 2 — Generate three enhanced variants

Produce **three independent enhancements** (A, B, C) of `SOURCE`. Each variant must:

1. Start from `SOURCE` and apply the **Enhancement Principles** (below) surgically — modify sentences in place, add strengthening sentences, lightly reorder paragraphs within their section.
2. Obey the **Preservation Contract** in full.
3. Differ from the other two variants only in *which enhancements they apply and how* — not in base content, structure, or voice. All three should be recognizably versions of `SOURCE`, not three different essays.
4. Keep length within ±5% of the source (see Preservation Contract item 4).

Write each variant to a working file:
- `/tmp/persuade_A.md`
- `/tmp/persuade_B.md`
- `/tmp/persuade_C.md`

### Step 3 — Score the three candidates and pick a winner

Score each variant quantitatively. No file checkpoints, no table-writing machinery — just score, sum, and declare.

**3a. Preservation check.** For each variant, verify the Preservation Contract items. Mark PASS or FAIL on each. A variant with any FAIL on citations, sections, voice, or original-claims is DISQUALIFIED and receives no further scoring. Length failures (outside ±5%) get penalized in the scoring step rather than disqualified, since they may reflect legitimate editing.

**3b. Quantitative scoring.** For each non-disqualified variant, assign a score from **1–10** on each dimension (10 = best). Use the full range — don't cluster all scores at 7-8. The dimensions:

| Dimension | What 10 looks like |
|---|---|
| **Persuasive sharpening** | Hedging consistently tightened into confident framing; topic sentences sharpened; abstractions made concrete throughout. |
| **Information density** | Every substantive paragraph carries concrete, verifiable claims; sparse paragraphs received Tier 2 additions where appropriate. |
| **Call-to-action** | Ends with a specific, concrete, time-bound next step for the reader (added, not replacing existing conclusion). |
| **Logical flow** | Transitions explicitly link paragraphs; within-section ordering builds the argument; the chain claim→evidence→implication is clear. |
| **Factual discipline** | No fabricated claims; Tier 2 additions are verifiable; scholarly hedging preserved where uncertainty is genuine. |
| **Preservation fidelity** | All citations, sections, voice, and original claims intact; length within ±5%; reads as the author's text enhanced, not rewritten. |

**3c. Sum and declare.** Sum each variant's six scores (max 60). The highest total wins. On a tie, preservation fidelity is the tiebreaker.

**3d. Record the scores.** Note each variant's per-dimension scores and total. These numbers appear in the final report (Step 9).

### Step 4 — Extract and fact-check the winner's NEW claims

From the winning variant:

1. **Identify claims that are NEW or MODIFIED relative to `SOURCE`.** Original claims from the input are presumed to be the author's and are not re-verified (they were the user's to begin with). Only newly added or rewritten factual claims need checking.
2. **Extract each new/modified claim** — statistics, dates, names, attributions, causal assertions, study findings, quotes.
3. **Fact-check each** using BOTH:
   - **Consensus MCP** (`mcp__consensus__search`) — for research-backed claims, statistics, study findings.
   - **WebSearch** — for current events, dates, names, quotes, and anything Consensus doesn't cover.
4. For each claim, record: the claim, the verdict (ACCURATE / INACCURATE / UNVERIFIABLE), and the source URL.

### Step 5 — Replace inaccurate NEW claims

For each NEW claim marked INACCURATE or UNVERIFIABLE:

1. **Find the strongest accurate replacement** — a fact that serves the same rhetorical function and is as specific and persuasive as possible. Confirm via Consensus + WebSearch.
2. **Edit the sentence** to use the replacement, keeping surrounding flow intact.
3. If no accurate replacement exists, **remove the added sentence** (since it was an addition, removing it returns to the author's original — which is always safe).

Do not touch the author's original claims, even if you suspect they're inaccurate — those are the user's responsibility, not the skill's. Flag any concerns in the report only.

### Step 6 — Re-fact-check the corrected text

Run one confirmation pass over the corrected winner. If replacements introduced new inaccuracies, repeat Step 5 once. **Cap at one additional correction round.** If errors remain, flag them.

### Step 7 — Verify preservation

Before outputting, explicitly verify each item in the **Preservation Contract**:
- All original citations present and in original format? ✓/✗
- All sections present, in original order, with original headers? ✓/✗
- Output length within ±5% of input? ✓/✗
- Author voice preserved? ✓/✗
- No original claims or examples removed? ✓/✗

If any check fails, fix the violation before outputting.

### Step 8 — Write the final output

Write the final, verified version to:
- The same directory as the input file, with `_persuasive` suffix (e.g., `essay.md` → `essay_persuasive.md`), OR
- `/tmp/persuade_output.md` if the input was inline text.

### Step 9 — Report

Print to the user:
1. **The final version** (full text).
2. A brief **change report**:
   - **Quantitative scores per variant.** For each variant (A/B/C), list the 1–10 score on each dimension (Persuasive sharpening, Information density, Call-to-action, Logical flow, Factual discipline, Preservation fidelity) and the total out of 60.
   - **Winner declaration:** which variant won and its total score.
   - **Source density classification** (Sparse/Moderate/Dense) from Step 2 and which enhancement tiers were applied.
   - Number of NEW claims fact-checked, how many accurate vs. replaced.
   - Each replacement made (original claim → replacement → source).
   - Confirmation that all Preservation Contract checks passed (Step 7).
   - Any concerns about the author's original claims (flagged, not modified).

---

## Enhancement Principles

These are the techniques to apply *surgically* during Step 2. They modify prose in place; they do not license restructuring or compression.

### Step 2 prelude: Assess baseline density

Before writing the three variants, estimate how information-dense the source already is. Scan each substantive paragraph and count (roughly) how many concrete, verifiable factual claims it contains — statistics, dates, named studies, quantified findings, cited assertions. Then classify the source:

- **Sparse:** many paragraphs make arguments without supporting claims, or rest on assertion alone. Average fewer than ~1 concrete claim per paragraph.
- **Moderate:** most paragraphs have at least one supporting claim, but some arguments are underspecified. Average ~1–2 claims per paragraph.
- **Dense:** most paragraphs carry multiple cited claims. Average 2+ claims per paragraph.

Record this classification. It governs how aggressively principles 3 and 4 (below) are applied:
- **Sparse → Tier 2 (add new claims) is the primary lever.** Add specific, cited supporting claims to underspecified paragraphs. This is where the largest persuasion gains are available.
- **Moderate → apply both Tier 1 (reframe existing) and Tier 2 (add new) selectively.**
- **Dense → Tier 1 (reframe existing) is the primary lever.** Adding more claims has diminishing returns at high baseline density; focus on making existing claims land harder through concreteness and framing.

### Apply where they strengthen

1. **Tighten hedging into confident framing.** "It seems possible that X may contribute to Y" → "X contributes to Y." (Only when the underlying claim is accurate and the author's evidence supports the stronger form.)

2. **Sharpen topic sentences.** If a paragraph opens with context or throat-clearing, add or strengthen a topic sentence that states the paragraph's point directly.

3. **Make abstract claims concrete — Tier 1 (always apply).** "Certain cognitive activities are essential for health" → "Social connection, cultural transmission, and shared sensemaking are essential for health — social connection rivals smoking as a mortality risk factor (Holt-Lunstad, 2024)." Add specifics, numbers, named entities — drawing on claims already in the text or its citations. This tightens existing material; it does not add new claims.

4. **Add new supporting claims where the argument is thin — Tier 2 (apply when source is Sparse or Moderate).** Where a paragraph asserts something important without a concrete supporting fact, ADD a specific, verifiable claim with a citation. Example: if the source says "AI is already changing how people communicate" without a study attached, add "Hohenstein et al. (2023) found AI-mediated communication measurably alters language complexity and social relationships within weeks." These additions will be fact-checked in Step 4 — only add claims you are confident are accurate. Target: bring sparse paragraphs up to at least one concrete cited claim each.

5. **Strengthen transitions.** Replace "Also," "Furthermore," or implicit transitions with explicit logical connectors: "The reason this matters is…" or "This finding compounds a second problem…"

6. **Add a call-to-action.** This is mandatory, not optional. The document must end with (or gain, if absent) a concrete, specific next step for the reader — what to fund, what to read, what to decide, what to do. Add this as a closing sentence or short paragraph; do not replace the author's existing conclusion. If the source already has a call-to-action, sharpen it to be more specific and time-bound.

7. **Frame as collaborative problem-solving.** Where the text opposes a view, reframe opposition as joint problem-solving: not "X is wrong" but "X misses a piece of the picture — specifically…"

### Avoid

8. **No rhetorical aggression.** Do not attack, mock, or demolish opposing views. Make the positive case.

9. **No hedging that weakens the core claim.** (But DO preserve scholarly hedging that reflects genuine uncertainty — "evidence suggests," "the mechanism remains under study." Accuracy beats confidence.)

10. **No compression of scholarly apparatus.** Full references stay full. DOIs stay. PMIDs stay. Author lists stay. Context that frames a citation stays.

11. **No voice drift.** Don't convert academic prose into marketing copy, or a research outline into a narrative essay. Match the author's existing register sentence by sentence.

### Factual discipline

12. **Only ADD claims you are confident are factual.** If an enhancement would require a factual claim you can't verify, either omit the enhancement or phrase it as a question/hypothesis. The fact-check step will catch what slips through, but front-loading accuracy reduces the correction burden. Tier 2 additions (principle 4) are the highest-risk for fabrication — apply extra scrutiny to your own new claims.

---

## Scoring Rubric (dimension definitions for Step 3)

Step 3 requires quantitative scoring (1–10 per dimension). Use the full range — avoid clustering all variants at 7-8. The dimensions:

| Dimension | What 10 looks like |
|---|---|
| **Persuasive sharpening** | Hedging consistently tightened into confident framing; topic sentences sharpened; abstractions made concrete throughout. |
| **Information density** | Every substantive paragraph carries concrete, verifiable claims; sparse paragraphs received Tier 2 additions where appropriate. |
| **Call-to-action** | Ends with a specific, concrete, time-bound next step for the reader (added, not replacing existing conclusion). |
| **Logical flow** | Transitions explicitly link paragraphs; within-section ordering builds the argument; the chain claim→evidence→implication is clear. |
| **Factual discipline** | No fabricated claims; Tier 2 additions are verifiable; scholarly hedging preserved where uncertainty is genuine. |
| **Preservation fidelity** | All citations, sections, voice, and original claims intact; length within ±5%; reads as the author's text enhanced, not rewritten. |

**Aggregate rule:** Sum the six scores per variant (max 60). Highest total wins. Tiebreaker: highest preservation fidelity score.
