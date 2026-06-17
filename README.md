# persuade

A [Claude Code](https://docs.claude.com/en/docs/claude-code) skill that rewrites any text to be maximally persuasive while keeping every claim factually accurate.

## What it does

Give it an essay, op-ed, grant proposal, or any argumentative prose. It will:

1. **Generate three independent persuasive rewrites** of your text.
2. **Rank-order them head-to-head** to pick the most persuasive candidate.
3. **Fact-check every factual claim** in the winner using web search + the Consensus academic database.
4. **Replace any inaccurate claim** with the strongest accurate replacement available — no hedging, no flags, no silent removals. The output is always fully accurate.
5. **Re-verify** the corrected text and output the final version with a change report.

The result: text that hits as hard as possible, where every claim survives scrutiny.

## Why it works

The skill's prompt design is grounded in recent empirical research on what makes LLM-generated text persuasive. Key findings it integrates:

- **Information density is the strongest lever.** Packing arguments with verifiable factual claims drives persuasion more than personalization, model scale, or rhetorical style. — [Hackenburg, Tappin, et al. (2025), *The Levers of Political Persuasion with Conversational AI*, Science](https://www.science.org/doi/10.1126/science.aea3884) · [arXiv](https://arxiv.org/pdf/2507.13919) · [Supplementary materials](https://github.com/kobihackenburg/scaling-conversational-AI/blob/main/Supplementary%20Materials.pdf)

- **Call-to-action framing has the strongest covariation with attitude shift** (+0.379), outperforming bare fact-citation. Argumentative aggression (−0.334) and hedging (−0.117) *reduce* persuasion. — [Chen, Kalla & Le (2026), *Benchmarking Political Persuasion Risks Across Frontier LLMs*](https://arxiv.org/pdf/2603.09884v1)

- **Semantic density (information per token) improves accuracy by +8.4 pp** with zero added tokens or latency. Concrete nouns, active verbs, and quantification beat verbose abstractions. — [Ahmed (2026), *Maximizing Information Per Token Improves LLM Accuracy*](https://arxiv.org/pdf/2604.17659)

- **Logical reasoning and a dispassionate voice mediate persuasiveness.** Perceptions of "more facts and evidence" and "logical reasoning" were significant mediators of LLM persuasiveness; anger reduced it. — [Bai, Voelkel, Muldowney, Eichstaedt & Willer (2025), *LLM-generated messages can persuade humans on policy issues*, Nature Communications](https://www.nature.com/articles/s41467-025-61345-5)

- **The persuasion-accuracy trade-off is real and must be actively managed.** Every lever that increases persuasiveness also tends to degrade factual accuracy. This skill front-loads accuracy during generation and back-ends it with search-augmented fact-checking (validated at r = 0.84 against human fact-checkers in the Hackenburg et al. pipeline).

### Architecture choice: explore-then-commit

The skill deliberately separates generation from verification. It produces three candidates optimized purely for persuasion, picks the winner, *then* fact-checks. This prevents the two objectives from fighting each other inside a loop — a pattern motivated by the documented persuasion-accuracy tension.

## Install

### Option A: Direct copy

```bash
mkdir -p ~/.claude/skills/persuade
curl -o ~/.claude/skills/persuade/SKILL.md https://raw.githubusercontent.com/ZachSchlosser/persuade/main/SKILL.md
```

### Option B: Clone

```bash
git clone https://github.com/ZachSchlosser/persuade.git
cp -r persuade/SKILL.md ~/.claude/skills/persuade/SKILL.md
```

Then restart Claude Code (or run `/reload`) so the skill is discovered.

### Requirements

- Claude Desktop or [Claude Code](https://docs.claude.com/en/docs/claude-code) CLI
- **Web search** access (built into Claude Code)
- **[Consensus MCP server](https://github.com/ConsensusApp/consensus-mcp)** configured in your Claude Code MCP settings — used for research-claim fact-checking. The skill also falls back to web search if Consensus is unavailable.

## Usage

```
/persuade path/to/your-essay.md
```

or inline:

```
/persuade "Paste your text here..."
```

The skill writes the final version to `<filename>_persuasive.md` (or `/tmp/persuade_output.md` for inline input) and prints a change report.

## How fact-checking works

1. **Claim extraction** — every falsifiable statement is isolated (stats, dates, attributions, causal claims, quotes).
2. **Dual-source verification** — research claims go to the Consensus academic search; everything else goes to web search.
3. **Strongest-replacement policy** — when a claim fails, the skill finds the most persuasive *accurate* fact that serves the same rhetorical function. If none exists, the sentence is removed.
4. **One re-verification pass** with a single correction round cap to bound runtime.

## Limitations

- **LLM persuasion judges are reliable for ranking, not absolute scores.** The skill uses head-to-head comparison rather than numeric scoring to match this reliability profile.
- **The Information-prompt finding is strongest for Claude and Grok.** It can reduce persuasiveness on GPT models ([Chen et al. 2026](https://arxiv.org/pdf/2603.09884v1)). Since the skill runs on Claude, the prompt is tuned accordingly.
- **Personalization is omitted.** It had negligible effect (+0.43 pp) in the Hackenburg et al. study — not worth the complexity or the privacy implications.

## Research sources

| Paper | Link | Key contribution |
|---|---|---|
| Hackenburg, Tappin, et al. (2025) — *The Levers of Political Persuasion with Conversational AI* | [Science](https://www.science.org/doi/10.1126/science.aea3884) · [arXiv](https://arxiv.org/pdf/2507.13919) · [Supplements](https://github.com/kobihackenburg/scaling-conversational-AI/blob/main/Supplementary%20Materials.pdf) | Information density as primary persuasion lever; persuasion-accuracy trade-off |
| Chen, Kalla & Le (2026) — *Benchmarking Political Persuasion Risks Across Frontier LLMs* | [arXiv](https://arxiv.org/pdf/2603.09884v1) | Call-to-action as strongest covariate; model-dependence of information prompts |
| Ahmed (2026) — *Maximizing Information Per Token Improves LLM Accuracy* | [arXiv](https://arxiv.org/pdf/2604.17659) | Semantic Density Effect (SDE); concreteness heuristics |
| Bai, Voelkel, Muldowney, Eichstaedt & Willer (2025) — *LLM-generated messages can persuade humans on policy issues* | [Nature Communications](https://www.nature.com/articles/s41467-025-61345-5) | Logical reasoning and dispassionate voice as mediators |
| Jaipersaud, Krueger & Lubana (2025) — *How Do LLMs Persuade?* | [arXiv](https://arxiv.org/abs/2508.05625) | Linear-probe analysis of persuasion dynamics |
| Poungpeth, Clark & Mitra (2026) — *Spontaneous Persuasion: An Audit of Model Persuasiveness in Everyday Conversations* | [arXiv](https://arxiv.org/html/2604.22109v2) | LLMs default to informationally dense strategies |

## License

MIT — see [LICENSE](LICENSE).
