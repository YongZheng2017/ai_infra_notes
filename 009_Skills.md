# Skills

## 创建Skill

安装node, npx, Python

mkdir -p my-first-skill

Create my-first-skill/SKILL.md

```
---
name: my-first-skill
description: Turn rough meeting notes into a compact decision record when the user asks to capture a technical decision.
---

# Decision record

Extract the decision, context, alternatives, owner, and next review date.
If the notes do not contain a decision, ask one clarifying question instead
of inventing one.
```

&nbsp;

## Install the complete reviewer bundle

npx skills add rohitg00/ai-engineering-from-scratch --skill skill-contract-reviewer --full-depth


