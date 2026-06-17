# persuade

A Claude skill that enhances any text's persuasiveness while preserving its structure, citations, voice, and length — and keeping every claim factually accurate.

## What it does

Give it an essay, op-ed, grant proposal, or any argumentative prose. It will:

1. **Generate three enhanced variants** that apply persuasion techniques *surgically* — tightening hedging, sharpening topic sentences, making abstract claims concrete, strengthening transitions, adding call-to-action. The document's structure, citations, voice, and length are preserved.
2. **Rank-order them head-to-head** to pick the most persuasive candidate (disqualifying any that violate the preservation contract).
3. **Fact-check only the NEW claims** the skill added (your original claims are yours and aren't modified or re-verified) using web search + the Consensus academic database.
4. **Replace any inaccurate new claim** with the strongest accurate replacement available, or remove the addition if no accurate replacement exists.
5. **Verify preservation** — explicit checks that all citations, sections, length, voice, and original claims are intact before output.

The result: your text, recognizably yours, with the rhetoric sharpened rather than replaced.

## Why it works

The skill's prompt design is grounded in recent empirical research on what makes LLM-generated text persuasive. Key findings it integrates:

- **Information density is the strongest lever.** Packing arguments with verifiable factual claims drives persuasion more than personalization, model scale, or rhetorical style. — [Hackenburg, Tappin, et al. (2025), *The Levers of Political Persuasion with Conversational AI*, Science](https://www.science.org/doi/10.1126/science.aea3884) · [arXiv](https://arxiv.org/pdf/2507.13919) · [Supplementary materials](https://github.com/kobihackenburg/scaling-conversational-AI/blob/main/Supplementary%20Materials.pdf)

- **Call-to-action framing has the strongest covariation with attitude shift** (+0.379), outperforming bare fact-citation. Argumentative aggression (−0.334) and hedging (−0.117) *reduce* persuasion. — [Chen, Kalla & Le (2026), *Benchmarking Political Persuasion Risks Across Frontier LLMs*](https://arxiv.org/pdf/2603.09884v1)

- **Semantic density (information per token) improves accuracy by +8.4 pp** with zero added tokens or latency. Concrete nouns, active verbs, and quantification beat verbose abstractions. — [Ahmed (2026), *Maximizing Information Per Token Improves LLM Accuracy*](https://arxiv.org/pdf/2604.17659)

- **Logical reasoning and a dispassionate voice mediate persuasiveness.** Perceptions of "more facts and evidence" and "logical reasoning" were significant mediators of LLM persuasiveness; anger reduced it. — [Bai, Voelkel, Muldowney, Eichstaedt & Willer (2025), *LLM-generated messages can persuade humans on policy issues*, Nature Communications](https://www.nature.com/articles/s41467-025-61345-5)

- **The persuasion-accuracy trade-off is real and must be actively managed.** Every lever that increases persuasiveness also tends to degrade factual accuracy. This skill front-loads accuracy during generation and back-ends it with search-augmented fact-checking (validated at r = 0.84 against human fact-checkers in the Hackenburg et al. pipeline).

### Architecture choice: enhance, don't rewrite

The skill is framed as **enhancement, not rewriting**. Earlier versions treated the input as raw material to be restructured for maximum persuasion — which destroyed citations, shortened the text, and drifted from the author's voice. The current design enforces a hard **Preservation Contract** (citations verbatim, structure intact, length floor, voice preserved, original claims untouchable) and scopes persuasion techniques to *surgical, in-place edits* plus within-section paragraph reorganization and added strengthening sentences.

This matters because the persuasion research the skill draws on studied *conversational* AI — short dialogues where density and concision win. Applying those heuristics to cited academic prose is a category error: in scholarly work, the citation apparatus and qualifying context *are* the persuasive content. The skill now detects and respects that.

### Architecture choice: explore-then-commit

Within the enhancement scope, the skill still separates generation from verification. It produces three enhanced variants, picks the winner, *then* fact-checks only the newly added claims. This prevents the persuasion-accuracy tension from oscillating inside a loop.

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
