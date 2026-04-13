# ai-skillforge

A curated collection of AI coding rules, guidelines, agent definitions, and skills for use with [Claude](https://claude.ai/) and [Cursor](https://cursor.sh/).

## Overview

This repository organizes reusable rule files, agent definitions, and skills by category, helping AI assistants generate consistent, high-quality code and perform specialized workflows across different projects.

## Structure

```
.
├── Claude/                          # Claude-specific resources
│   ├── agents/                      # Sub-agent definitions (.md with YAML frontmatter)
│   ├── rules/                       # Coding standards and guidelines
│   │   ├── common/                  # Language-agnostic principles (always install)
│   │   ├── typescript/
│   │   ├── python/
│   │   ├── golang/
│   │   ├── web/
│   │   ├── swift/
│   │   ├── php/
│   │   └── ...                      # Other language-specific rule sets
│   └── skills/                      # Agent skills (SKILL.md format)
│       ├── create-pr/
│       ├── pr-review/
│       └── ...                      # 180+ skills across domains
└── Cursor/                          # Cursor .mdc rules by technology stack
    ├── backend/
    │   ├── db/
    │   │   └── postgresql.mdc
    │   └── python/
    │       ├── boto3.mdc
    │       ├── django.mdc
    │       ├── django-rest-framework.mdc
    │       ├── fastapi.mdc
    │       ├── flask.mdc
    │       ├── flask-restful.mdc
    │       ├── pydantic.mdc
    │       └── sqlalchemy.mdc
    ├── code-as-infra/
    │   ├── docker.mdc
    │   └── terraform.mdc
    └── frontend/
        ├── next-js.mdc
        ├── react.mdc
        └── three-js.mdc
```

---

## Claude

### Agents

Agent definitions live in `Claude/agents/`. Each file is a Markdown document with YAML frontmatter specifying the agent's `name`, `description`, available `tools`, and preferred `model`.

Copy the agent files you need into `~/.claude/agents/` (global) or `.claude/agents/` (project-level).

**Example agents:**

| Agent | Description |
|-------|-------------|
| `architect` | Software architecture specialist for system design and technical decisions |
| `code-reviewer` | Expert code review for quality, security, and maintainability |
| `security-reviewer` | Security-focused review for vulnerabilities and compliance |
| `python-reviewer` | Python-specific code quality and idiom checks |
| `tdd-guide` | Test-driven development guidance and workflow |
| `performance-optimizer` | Performance analysis and optimization recommendations |

> 47 agents are available. Browse [`Claude/agents/`](Claude/agents/) for the full list.

### Rules

Rules live in `Claude/rules/` and are organized into a **common** layer plus **language-specific** directories. See [`Claude/rules/README.md`](Claude/rules/README.md) for full installation instructions and details on rule priority.

**Quick install (recommended):**

```bash
# Install common rules + one or more language-specific rule sets
./install.sh typescript
./install.sh python
./install.sh golang
```

**Manual install:**

```bash
# Common rules are required for all projects
cp -r Claude/rules/common ~/.claude/rules/common

# Add language-specific rules as needed
cp -r Claude/rules/python ~/.claude/rules/python
cp -r Claude/rules/typescript ~/.claude/rules/typescript
cp -r Claude/rules/golang ~/.claude/rules/golang
```

**Available rule sets:** `common`, `typescript`, `python`, `golang`, `web`, `swift`, `php`, `cpp`, `csharp`, `dart`, `java`, `kotlin`, `perl`, `rust`

### Skills

Skills live in `Claude/skills/`. Each skill is a directory containing a `SKILL.md` file that teaches the agent a specialized workflow.

Copy the skill directories you need into `~/.claude/skills/` (global) or `.claude/skills/` (project-level).

**Selected skills:**

| Skill | Description |
|-------|-------------|
| [create-pr](Claude/skills/create-pr/SKILL.md) | Create a GitHub PR from the current branch using the `gh` CLI |
| [pr-review](Claude/skills/pr-review/SKILL.md) | Review GitHub PRs and post inline code review comments |
| [django-patterns](Claude/skills/django-patterns/SKILL.md) | Django best practices and patterns |
| [python-testing](Claude/skills/python-testing/SKILL.md) | Python testing workflows and patterns |
| [security-review](Claude/skills/security-review/SKILL.md) | Security review checklist and workflow |
| [tdd-workflow](Claude/skills/tdd-workflow/SKILL.md) | Test-driven development workflow |
| [git-workflow](Claude/skills/git-workflow/SKILL.md) | Git branching, committing, and collaboration workflow |
| [api-design](Claude/skills/api-design/SKILL.md) | REST and API design guidelines |

> 183 skills are available across domains including backend, frontend, DevOps, AI/ML, and more. Browse [`Claude/skills/`](Claude/skills/) for the full list.

---

## Cursor

### Rules

Copy the relevant `.mdc` rule files into your project's `.cursor/rules/` directory. Refer to the included **[Guidelines for Applying Cursor Rules.pdf](Cursor/Guidelines%20for%20Appying%20Cursor%20Rules.pdf)** for best practices on structuring and applying rules effectively.

**Available rule files:**

| Category | Rules |
|----------|-------|
| Backend / Python | `django`, `django-rest-framework`, `fastapi`, `flask`, `flask-restful`, `boto3`, `pydantic`, `sqlalchemy` |
| Backend / Database | `postgresql` |
| Code as Infrastructure | `docker`, `terraform` |
| Frontend | `next-js`, `react`, `three-js` |

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for full guidelines. Pull requests are welcome!
