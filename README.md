# ai-skillforge

A curated collection of AI coding rules, guidelines, and agent skills for use with [Cursor](https://cursor.sh/) and other AI-assisted development tools.

## Overview

This repository organizes reusable `.mdc` rule files and agent skills by category, helping AI assistants generate consistent, high-quality code and perform specialized workflows across different projects.

## Structure

```
.
├── Cursor/                          # .mdc rules by technology stack
│   ├── backend/
│   │   ├── db/
│   │   │   └── postgresql.mdc
│   │   └── python/
│   │       ├── boto3.mdc
│   │       ├── django.mdc
│   │       ├── django-rest-framework.mdc
│   │       ├── fastapi.mdc
│   │       ├── flask.mdc
│   │       ├── flask-restful.mdc
│   │       ├── pydantic.mdc
│   │       └── sqlalchemy.mdc
│   ├── code-as-infra/
│   │   ├── docker.mdc
│   │   └── terraform.mdc
│   └── frontend/
│       ├── next-js.mdc
│       ├── react.mdc
│       └── three-js.mdc
└── skills/                          # Agent skills (SKILL.md format)
    ├── create-pr/
    │   └── SKILL.md
    └── pr-review/
        └── SKILL.md
```

## Usage

### Rules

Copy the relevant `.mdc` rule files into your project's `.cursor/rules/` directory. Refer to the included **Guidelines for Applying Cursor Rules.pdf** for best practices on structuring and applying rules effectively.

### Skills

Copy the skill directories you need into your project's `.cursor/skills/` or personal `~/.cursor/skills/` directory. Each skill contains a `SKILL.md` file that teaches the agent a specialized workflow.

| Skill | Description |
|-------|-------------|
| [create-pr](skills/create-pr/SKILL.md) | Create a GitHub PR from the current branch using the `gh` CLI. |
| [pr-review](skills/pr-review/SKILL.md) | Review GitHub PRs and post inline code review comments using the `gh` CLI. |

## Contributing

Pull requests are welcome.

**Adding rules:**
- Place them in the appropriate technology subdirectory under `Cursor/`.
- Follow the existing naming convention (`<technology>.mdc`).
- Keep rules focused, concise, and technology-specific.

**Adding skills:**
- Create a new directory under `skills/` named after the skill.
- Include a `SKILL.md` with YAML frontmatter (`name` and `description`) and clear instructions.
- Keep the `SKILL.md` under 500 lines. Use separate reference files for detailed docs.
- Update the skills table in this README.
