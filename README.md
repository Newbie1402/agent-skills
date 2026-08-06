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
npx skills add https://github.com/Leonxlnx/taste-skill \
  --skill "gpt-taste"

npx skills add https://github.com/Leonxlnx/taste-skill \
  --skill "redesign-existing-projects"

npx skills add https://github.com/Leonxlnx/taste-skill \
  --skill "full-output-enforcement"
```

