# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Commands

### Development Workflow
- **Git operations**: Use standard git commands for commits, pushes, and pull requests. Follow the PR template at `.github/pull_request_template.md` for pull requests.
- **Contributing**: See CONTRIBUTING.md for detailed guidelines. Fork the repo, create a feature branch from main, make changes, and open a PR.
- **Installing rules locally**: Copy rule directories from `Claude/rules/` to `~/.claude/rules/` (e.g., `cp -r Claude/rules/common ~/.claude/rules/common`). Use `./install.sh <language>` for quick installation (when available).
- **Installing agents**: Copy agent files from `Claude/agents/` to `~/.claude/agents/` or `.claude/agents/` (project-level).
- **Installing skills**: Copy skill directories from `Claude/skills/` to `~/.claude/skills/` or `.claude/skills/` (project-level).

### Repository Maintenance
- No build/lint/test scripts exist - this is a documentation repository.
- Use `git log` to see recent changes, often involving additions of new agents (47 total), skills (183 total), and rules.

## Architecture Overview

This repository is a curated collection of AI coding rules, guidelines, agent definitions, and skills for Claude and Cursor. It's organized into two main sections:

### Claude/ Directory
- **agents/**: 47 specialized sub-agent definitions as .md files with YAML frontmatter (name, description, tools, model). Examples: architect, code-reviewer, security-reviewer, python-reviewer, tdd-guide.
- **rules/**: Coding standards organized in layered structure:
  - common/: Language-agnostic principles (coding-style.md, testing.md, security.md, patterns.md, hooks.md, agents.md)
  - Language directories (typescript/, python/, golang/, web/, swift/, php/, cpp/, csharp/, dart/, java/, kotlin/, perl/, rust/): Extend common rules with technology-specific guidance
  - Rules define standards; skills provide actionable workflows.
- **skills/**: 183 agent skills as directories containing SKILL.md files with specialized workflows. Examples: create-pr, pr-review, django-patterns, python-testing, security-review, tdd-workflow, git-workflow, api-design.

### Cursor/ Directory
- Contains .mdc rule files for Cursor IDE, organized by technology stack:
  - backend/python/: django.mdc, django-rest-framework.mdc, fastapi.mdc, flask.mdc, flask-restful.mdc, boto3.mdc, pydantic.mdc, sqlalchemy.mdc
  - backend/db/: postgresql.mdc
  - code-as-infra/: docker.mdc, terraform.mdc
  - frontend/: next-js.mdc, react.mdc, three-js.mdc
- Each .mdc file provides best practices and conventions for specific frameworks.

The repository follows a layered approach where common rules provide defaults, and language-specific rules override for idiomatic patterns. Contributing guidelines in CONTRIBUTING.md ensure quality additions.