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

4. **Length floor: the output must be at least as long as the input.** Never shorten. If anything, the enhanced version may be slightly longer (added topic sentences, concrete framing, sharper transitions).

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
4. Never shorten the text.

Write each variant to a working file:
- `/tmp/persuade_A.md`
- `/tmp/persuade_B.md`
- `/tmp/persuade_C.md`

### Step 3 — Rank-order the three candidates

Read all three variants in a single comparison pass. Rank them from most to least persuasive using the **Scoring Rubric** (below), subject to the Preservation Contract — any variant that violates the contract is disqualified.

Record the winner. Only the winner moves to fact-checking.

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
- Output length ≥ input length? ✓/✗
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
   - Which variant (A/B/C) won and why.
   - Number of NEW claims fact-checked, how many accurate vs. replaced.
   - Each replacement made (original claim → replacement → source).
   - Confirmation that all Preservation Contract checks passed.
   - Any concerns about the author's original claims (flagged, not modified).

---

## Enhancement Principles

These are the techniques to apply *surgically* during Step 2. They modify prose in place; they do not license restructuring or compression.

### Apply where they strengthen

1. **Tighten hedging into confident framing.** "It seems possible that X may contribute to Y" → "X contributes to Y." (Only when the underlying claim is accurate and the author's evidence supports the stronger form.)

2. **Sharpen topic sentences.** If a paragraph opens with context or throat-clearing, add or strengthen a topic sentence that states the paragraph's point directly.

3. **Make abstract claims concrete.** "Certain cognitive activities are essential for health" → "Social connection, cultural transmission, and shared sensemaking are essential for health — social connection rivals smoking as a mortality risk factor (Holt-Lunstad, 2024)." Add specifics, numbers, named entities — drawing on claims already in the text or its citations.

4. **Strengthen transitions.** Replace "Also," "Furthermore," or implicit transitions with explicit logical connectors: "The reason this matters is…" or "This finding compounds a second problem…"

5. **Add a call-to-action where appropriate.** If the document ends weakly or without specifying next steps, add (do not replace) a concrete closing sentence about what the reader should do. Preserve any existing conclusion.

6. **Frame as collaborative problem-solving.** Where the text opposes a view, reframe opposition as joint problem-solving: not "X is wrong" but "X misses a piece of the picture — specifically…"

### Avoid

7. **No rhetorical aggression.** Do not attack, mock, or demolish opposing views. Make the positive case.

8. **No hedging that weakens the core claim.** (But DO preserve scholarly hedging that reflects genuine uncertainty — "evidence suggests," "the mechanism remains under study." Accuracy beats confidence.)

9. **No compression of scholarly apparatus.** Full references stay full. DOIs stay. PMIDs stay. Author lists stay. Context that frames a citation stays.

10. **No voice drift.** Don't convert academic prose into marketing copy, or a research outline into a narrative essay. Match the author's existing register sentence by sentence.

### Factual discipline

11. **Only ADD claims you are confident are factual.** If an enhancement would require a factual claim you can't verify, either omit the enhancement or phrase it as a question/hypothesis. The fact-check step will catch what slips through, but front-loading accuracy reduces the correction burden.

---

## Scoring Rubric

When comparing the three variants (Step 3), rank them relatively (1st, 2nd, 3rd) on each dimension, then pick an overall winner. Do not produce absolute numeric scores. **Disqualify any variant that violates the Preservation Contract.**

| Dimension | What to look for |
|---|---|
| **Persuasive sharpening** | Did hedging become confidence? Did topic sentences sharpen? Did abstractions become concrete? |
| **Call-to-action** | Did the ending gain a concrete next step (added, not replacing)? |
| **Logical flow** | Are transitions and within-section ordering stronger? |
| **Factual discipline** | Did enhancements avoid fabricating or over-reaching? |
| **Preservation** | Are all citations, sections, length, voice, and original claims intact? (If not → disqualify.) |

**Tiebreaker:** When two variants are close, prefer the one with higher preservation fidelity and higher factual discipline.
