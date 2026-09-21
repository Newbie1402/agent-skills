# Source Documentation for Agent-Skills
This repo using for back-end and front-end skills
## Agent-Skills Directory Structure

```text
agent-skills/
├── README.md                     # 🎯 This file: architecture and rules
└── codex/                        # 📦 Codex rules and templates
    ├── skills/                   # Codex skill definitions
    │   └── SKILL.md                # Base skill template
    ├── HOUSE_STYLE.md            # Coding style and conventions
```

## How to using Code Graph to index code base

```bash
npx @colbymchenry/codegraph init -i

mkdir -p ~/.agents/skills

git clone https://github.com/agentic-ai/find-skills.git ~/.agents/skills/find-skills

npx skills add https://github.com/leonxlnx/taste-skill --skill full-output-enforcement
```
### With Front-end 


```bash
npx impeccable install
```
promt need to use with skill impeccable:
```bash
1. Primary users:
   Primarily internal staff and managers/admins. This is an internal business application used for day-to-day operations, so prioritize efficiency, clarity, information density, and low learning curve over flashy presentation.

2. Product position:
   A unified internal office suite for business operations. It may include travel/airline-related workflows, administration, accounting, reporting, and other internal operational modules, but it should feel like one consistent office/back-office product rather than a marketing website.

3. Future Impeccable build default:
   code-first.

This is an existing production application with an established React codebase. Prefer extending and reusing the existing design system, components, patterns, spacing, typography, and interaction conventions before introducing new visual concepts.

For new UI:

- prioritize practical UX and information density
- keep tables, forms, filters, dialogs, and workflows easy to scan
- avoid AI-slop aesthetics
- avoid excessive gradients, oversized typography, decorative cards, and unnecessary animations
- maintain visual consistency with existing screens
- prioritize responsive behavior and accessibility
- keep implementation maintainable and production-oriented
```

Old version not use
```bash
npx skills add https://github.com/Leonxlnx/taste-skill \
  --skill "gpt-taste"

npx skills add https://github.com/Leonxlnx/taste-skill \
  --skill "redesign-existing-projects"

npx skills add https://github.com/Leonxlnx/taste-skill \
  --skill "full-output-enforcement"
```

