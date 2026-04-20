# Contributing to ai-skillforge

Thank you for your interest in contributing! This guide explains how to add or improve content in this repository.

## Table of Contents

- [Getting Started](#getting-started)
- [Repository Structure](#repository-structure)
- [Contributing Claude Agents](#contributing-claude-agents)
- [Contributing Claude Rules](#contributing-claude-rules)
- [Contributing Claude Skills](#contributing-claude-skills)
- [Contributing Cursor Rules](#contributing-cursor-rules)
- [Pull Request Guidelines](#pull-request-guidelines)
- [Quality Checklist](#quality-checklist)

---

## Getting Started

1. **Fork** the repository and create a feature branch from `main`.
2. Make your changes following the relevant guidelines below.
3. Open a pull request using the [PR template](.github/pull_request_template.md).

---

## Repository Structure

```
.
├── Claude/
│   ├── agents/      # Claude sub-agent definitions
│   ├── rules/       # Coding standards (common + language-specific)
│   └── skills/      # Agent skill libraries (SKILL.md format)
└── Cursor/
    ├── backend/
    ├── code-as-infra/
    └── frontend/
```

---

## Contributing Claude Agents

Agents live in `Claude/agents/`. Each file is a standalone Markdown document.

### File format

```markdown
---
name: <agent-name>
description: >-
  One or two sentences describing what the agent does and when it should be used.
  Include trigger phrases (e.g. "Use PROACTIVELY when...").
tools: ["Read", "Grep", "Glob", "Bash", "Write"]
model: sonnet   # or opus / haiku
---

You are a ... (system prompt)

## Section
...
```

### Guidelines

- Use **kebab-case** for the filename and the `name` field (e.g., `code-reviewer.md`).
- Choose the smallest model that can do the job (`haiku` → `sonnet` → `opus`).
- List only the tools the agent actually needs.
- Write the `description` so Claude can decide when to invoke the agent automatically — include specific trigger phrases.
- Keep agent files focused; delegate subtasks to other agents rather than creating a monolithic agent.

---

## Contributing Claude Rules

Rules live in `Claude/rules/` and follow a two-layer structure:

- `common/` — language-agnostic principles that apply to all projects.
- `<language>/` — language-specific rules that extend or override `common/`.

### Adding rules to an existing language

1. Open the relevant file in `Claude/rules/<language>/`.
2. Add your content, keeping the file's existing structure and tone.
3. Reference the corresponding `common/` file at the top if the file extends it:
   ```
   > This file extends [common/testing.md](../common/testing.md) with Python-specific content.
   ```

### Adding a new language

1. Create a `Claude/rules/<language>/` directory.
2. Add files that mirror the `common/` layer as needed:
   - `coding-style.md`
   - `testing.md`
   - `patterns.md`
   - `hooks.md`
   - `security.md`
3. Each file must start with a reference to its `common/` counterpart.
4. Update [`Claude/rules/README.md`](Claude/rules/README.md) to include the new language.

### Guidelines

- Keep rules **prescriptive and concise** — tell the agent what to do, not how to implement it.
- Avoid duplicating content already in `common/`; reference it instead.
- Rules tell you *what* to do; skills (see below) tell you *how* to do it.

---

## Contributing Claude Skills

Skills live in `Claude/skills/`. Each skill is a directory containing a `SKILL.md` file.

### Directory structure

```
Claude/skills/<skill-name>/
└── SKILL.md              # Required
└── <reference>.md        # Optional supporting files
```

### SKILL.md format

```markdown
---
name: <skill-name>
description: >-
  One sentence describing what this skill does and when to use it.
---

# Skill Title

Brief description.

## Prerequisites

- ...

## Workflow

### Step 1: ...
...
```

### Guidelines

- Use **kebab-case** for the directory name and the `name` field.
- Keep `SKILL.md` **under 1100 lines**. Move detailed reference material to separate files in the same directory.
- Include concrete, executable steps with code examples where helpful.
- Make the skill self-contained — a reader should be able to follow it without outside context.

---

## Contributing Cursor Rules

Cursor rules live in `Cursor/` as `.mdc` files, organized by technology stack.

### Where to place files

| Category | Path |
|----------|------|
| Backend (Python) | `Cursor/backend/python/<technology>.mdc` |
| Backend (Database) | `Cursor/backend/db/<technology>.mdc` |
| Code as Infrastructure | `Cursor/code-as-infra/<technology>.mdc` |
| Frontend | `Cursor/frontend/<technology>.mdc` |

### Guidelines

- Use the technology name as the filename in **kebab-case** (e.g., `django-rest-framework.mdc`).
- Keep rules focused and technology-specific.
- Follow the conventions described in **Cursor/Guidelines for Applying Cursor Rules.pdf**.

---

## Pull Request Guidelines

- **One logical change per PR** — separate unrelated additions into separate PRs.
- **Fill in the PR template** — describe what you added, why it's useful, and how you tested it.
- **Update the README** if you add a new agent, language rule set, or notable skill.
- **Review your own diff** before requesting review — check for typos, formatting issues, and broken links.

---

## Quality Checklist

Before opening a PR, confirm:

- [ ] File is placed in the correct directory.
- [ ] Filename follows the naming convention for its type.
- [ ] YAML frontmatter is present and valid (for agents and skills).
- [ ] Content is accurate, concise, and free of typos.
- [ ] No sensitive information (credentials, internal URLs, proprietary data) is included.
- [ ] README updated if the addition warrants it.
