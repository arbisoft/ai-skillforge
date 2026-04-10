# ai-guide-rules

A curated collection of AI coding rules and guidelines for use with [Cursor](https://cursor.sh/) and other AI-assisted development tools.

## Overview

This repository organizes reusable `.mdc` rule files by technology stack, helping AI assistants generate consistent, high-quality code across different projects.

## Structure

```
.
└── Cursor/
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

## Usage

Copy the relevant `.mdc` rule files into your project's `.cursor/rules/` directory. Refer to the included **Guidelines for Applying Cursor Rules.pdf** for best practices on structuring and applying rules effectively.

## Contributing

Pull requests are welcome. When adding new rules:
- Place them in the appropriate technology subdirectory.
- Follow the existing naming convention (`<technology>.mdc`).
- Keep rules focused, concise, and technology-specific.
