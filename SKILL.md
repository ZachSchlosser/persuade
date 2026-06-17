---
name: persuade
description: Rewrite any text to be maximally persuasive while keeping every claim factually accurate. Generates three persuasive variants, rank-orders them head-to-head, fact-checks the winner (Consensus MCP + web search), and replaces any inaccurate claim with the strongest accurate replacement available. Use when the user wants to make an essay, op-ed, grant proposal, or any prose more persuasive without sacrificing factual accuracy.
user-invocable: true
allowed-tools: Read, Write, Edit, WebSearch, Agent, mcp__consensus__search
argument-hint: <file-path-or-text>
---

# Persuade

Rewrite text to be maximally persuasive while keeping every claim factually accurate.

## Input

`$ARGUMENTS` is either:
- A path to a text/markdown file (read it with the Read tool), or
- Inline text to rewrite directly.

If no argument is provided, ask the user to supply the text or a file path.

---

## Workflow

### Step 1 — Read the input

Locate and read the source text. Store it as `SOURCE`.

### Step 2 — Generate three persuasive rewrites

Produce **three independent rewrites** (A, B, C) of `SOURCE`. Each rewrite must:

1. Preserve the original argument, stance, and intent.
2. Be a complete, standalone piece (not a diff).
3. Apply the **Combined Persuasiveness Prompt** (below).
4. Differ meaningfully from the other two — vary structure, examples, and framing while keeping the same thesis.

Write each rewrite to a working file:
- `/tmp/persuade_A.md`
- `/tmp/persuade_B.md`
- `/tmp/persuade_C.md`

### Step 3 — Rank-order the three candidates

Read all three candidates in a single comparison pass. Rank them from most to least persuasive using the **Scoring Rubric** (below). This head-to-head relative ranking is more reliable than independent absolute scores.

Record the winner. Do NOT yet fact-check the losers — only the winner moves forward.

### Step 4 — Extract and fact-check the winner's claims

From the winning rewrite:

1. **Extract every factual claim** — any statement that could be verified or falsified (statistics, dates, names, attributions, causal assertions, study findings, quotes).
2. **Fact-check each claim** using BOTH:
   - **Consensus MCP** (`mcp__consensus__search`) — for research-backed claims, statistics, study findings.
   - **WebSearch** — for current events, dates, names, quotes, and anything Consensus doesn't cover.
3. For each claim, record: the claim, the verdict (ACCURATE / INACCURATE / UNVERIFIABLE), and the source URL.

### Step 5 — Replace inaccurate claims

For each claim marked INACCURATE or UNVERIFIABLE:

1. **Find the strongest accurate replacement** — a fact that serves the same rhetorical function in the argument and is as specific, concrete, and persuasive as possible. Search via Consensus + WebSearch to confirm the replacement is accurate.
2. **Rewrite the sentence** containing the bad claim to use the replacement, keeping the surrounding flow intact.
3. If genuinely no accurate replacement exists that serves the argument, **remove the sentence** rather than leaving an unsupported claim. Note the removal in the report.

Do not touch claims marked ACCURATE — leave them exactly as written.

### Step 6 — Re-fact-check the corrected text

Run one confirmation pass over the corrected winner. If new inaccuracies were introduced by the replacements (rare but possible), repeat Step 5 once. **Cap at one additional correction round.** If errors remain after that, flag them in the report rather than continuing.

### Step 7 — Write the final output

Write the final, fact-checked version to:
- The same directory as the input file, with `_persuasive` suffix (e.g., `essay.md` → `essay_persuasive.md`), OR
- `/tmp/persuade_output.md` if the input was inline text.

### Step 8 — Report

Print to the user:
1. **The final version** (full text).
2. A brief **change report**:
   - Which candidate (A/B/C) won and why.
   - Number of claims fact-checked, how many were accurate vs. replaced.
   - Each replacement made (original claim → replacement → source).
   - Any remaining flagged claims or removals.

---

## Combined Persuasiveness Prompt

When generating rewrites (Step 2), apply ALL of the following principles.

### Use

1. **Facts and evidence.** Pack the argument with substantive, verifiable factual claims. Every paragraph should carry real informational content — statistics, findings, dates, named entities. Density of real claims is the primary driver of persuasiveness.

2. **Call-to-action framing.** Where appropriate, suggest concrete actions the reader can take. Even in an essay, end with a specific "what to do" rather than a vague conclusion.

3. **Concreteness.** Use specific nouns, active verbs, numbers, units, and named entities rather than abstractions. "A 2023 Pew study of 5,000 Americans" beats "research shows." Concrete phrasing is both more persuasive and more credible.

4. **Logical structure.** Build the argument as a clear chain: claim → evidence → implication.

5. **Dispassionate tone.** Avoid anger, outrage, and hyperbole. A calm, confident, reasoned voice persuades better than an emotional one.

6. **Rapport and affirmation.** Acknowledge the reader's perspective and valid concerns before making the counter-case.

7. **Collaborative framing.** Frame the argument as joint problem-solving ("here's what we're facing and what we can do") rather than as a battle to be won.

### Avoid

8. **No hedging that weakens the core claim.** State the argument confidently. (Note: this applies to rhetorical hedging, NOT to factual accuracy — claims must still be true.)

9. **No argumentative aggression.** Do not challenge, rebut, or attack the opposing view directly. Make the positive case; don't tear down the other side.

10. **No filler, padding, or meta-commentary.** Every sentence should carry semantic weight. Remove polite preambles, restatements, and transitional throat-clearing.

### Factual discipline

11. **Only include claims you are confident are factual.** If you're not confident a claim is true, either omit it or phrase it as a question or hypothesis rather than asserting it. Fewer claims, all accurate, beats many claims with some fabricated.

---

## Scoring Rubric

When comparing the three candidates (Step 3), rank them relatively (1st, 2nd, 3rd) on each dimension, then pick an overall winner. Do not produce absolute numeric scores.

| Dimension | What to look for |
|---|---|
| **Information density** | Volume of substantive, verifiable factual claims per paragraph. More real content = better. |
| **Call-to-action strength** | Does it include a concrete, specific action for the reader? |
| **Concreteness** | Specifics, numbers, named entities, dates vs. vague abstractions. |
| **Logical flow** | Clear claim → evidence → implication structure. |
| **Tone** | Calm, confident, dispassionate. No outrage or aggression. |
| **Absence of anti-patterns** | No hedging, no argumentative attacks, no filler/padding. |
| **Factual discipline** | Does it avoid fabricating or over-reaching? A candidate that sounds persuasive but makes dubious claims should be penalized — it will lose claims in fact-checking. |

**Tiebreaker:** When two candidates are close, prefer the one with higher factual discipline — it will survive fact-checking more intact.
