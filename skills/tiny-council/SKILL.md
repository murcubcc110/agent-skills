---
name: tiny-council
description: Convene a lightweight internal council of diverse reasoning perspectives to evaluate a meaningful decision, plan, design, bug, tradeoff, code change, or high-stakes question. Use when the user asks for a council, panel, debate, multiple viewpoints, devil's advocate, pressure-test, stress-test, architecture review, risk review, decision support, or stronger reasoning before acting. Do not use for simple factual lookups, summaries, writing-only tasks, or questions with one clear answer; answer those directly instead.
---

# Council

Run a structured internal review from several independent reasoning methods, then synthesize a decision-ready answer.

## Process

1. Check whether the request actually needs a council.
   - If it is a simple factual lookup, summary, translation, or writing-only task, say that it does not need a council and answer directly.
   - If the user explicitly wants a council anyway, give a compact council answer but preserve factual uncertainty.
2. Restate the user's question or decision in one sentence.
3. If code, files, URLs, or project context are relevant, inspect the referenced material before framing the council.
4. Select 3 to 5 roles, then show the selected roles before the council views. Prefer the standard 5 reasoning roles for decisions and tradeoffs:
   - 反証役: inversion; assume it failed and trace why.
   - 原理分解役: decomposition; identify and challenge load-bearing assumptions.
   - 拡張探索役: analogy and upside; find larger or adjacent opportunities.
   - 素朴質問役: naive questioning; flag missing context, jargon, and unclear claims.
   - 実行計画役: dependency graphing and outside view; identify the practical first step and likely real-world failure points.
5. For domain-heavy factual questions where the user still wants a council, choose domain roles instead, but include one 検証役 focused on uncertainty and source quality.
6. Have each role reason independently. Keep each role's section concise and focused on:
   - **Vote status** (Choose from: [Approve], [Reject], or [Abstain])
   - Main recommendation
   - Key evidence or assumptions
   - Biggest risk
7. Make the roles challenge each other. Identify conflicts, weak assumptions, missing information, and what all roles may have missed.
8. Add a 悪魔の代弁者 pass against the emerging consensus:
   - State the emerging consensus in one sentence.
   - Give the strongest case that this consensus is wrong.
   - Say whether the council can rebut that challenge.
9. Synthesize under the title "合議結果":
   - **Final vote tally** (Count the votes from all roles)
   - Consensus
   - Disagreements
   - Recommended path
   - What the user gives up by following it
   - Next actions

## Output

Use this structure unless the user asks for another format:

```markdown
**Question**
<one-sentence restatement>

**Council Fit**
<why this does or does not need a council; answer directly if not>

**Roles**
- <Role>
- <Role>
- <Role>

**Council Views**
- **<Role> [<Vote: Approve/Reject/Abstain>]**: <recommendation, evidence, risk>

**Challenge Round**
- <conflict, concern, assumption, or missing point>
- <conflict, concern, or assumption>

**悪魔の代弁者**
- **暫定合意**: <one-sentence consensus>
- **最強の反論**: <best argument that the consensus is wrong>
- **合議側の応答**: <rebut, concede, or revise>

**合議結果**
- **Vote Tally**: [Approve: X, Reject: Y, Abstain: Z]
<clear recommendation and why>

**失うもの**
<cost, risk, or strongest dissent the recommendation asks the user to accept>

**Next Actions**
1. <action>
2. <action>
3. <action>
```

## Guidelines

- Do not invent certainty. Mark assumptions clearly.
- If code or files are relevant, inspect them before giving final recommendations.
- If the decision is risky or irreversible, include a lower-risk fallback.
- For factual or historical questions, prioritize accuracy over theatrical disagreement. Use sources when current, contested, or high-stakes facts matter.
- Do not smooth over disagreement. Classify it as either a value tension, an error catch, or missing evidence when useful.
- The final synthesis may side with a minority view if its reasoning is strongest.
- If the council cannot decide without missing information, say what is missing and give the best provisional answer.
- Keep the answer compact unless the user asks for deep analysis.

<!-- References consulted when shaping this lightweight council workflow:
- https://github.com/ngmeyer/council-review
- https://github.com/aiwithremy/claude-skills-llm-council
- https://platform.claude.com/docs/en/agents-and-tools/tool-use/advisor-tool
-->
