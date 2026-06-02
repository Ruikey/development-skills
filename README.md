# development-skills

This repository collects development-oriented Codex skills used during daily engineering work.
Each skill is kept as a small, self-contained directory so it can be reviewed, updated, and
installed independently.

## Repository layout

```text
development-skills/
├── README.md
├── java-project-standards/
│   ├── SKILL.md
│   ├── agents/
│   │   └── openai.yaml
│   └── references/
│       ├── alibaba-java-development-manual-v1.5.0.md
│       └── java-standards-checklist.md
└── .gitignore
```

## Skills

| Skill | Purpose |
| --- | --- |
| `java-project-standards` | Enforces Alibaba-style Java coding standards, layered architecture constraints, dependency rules, database conventions, logging, testing, and security checks for Java/JVM backend projects. |

## Skill conventions

Every skill directory should contain a required `SKILL.md` file with YAML frontmatter:

```yaml
---
name: skill-name
description: Clear trigger description for when Codex should use this skill.
---
```

Recommended optional directories:

- `agents/`: UI-facing metadata such as `openai.yaml`.
- `references/`: detailed documentation loaded only when needed.
- `scripts/`: deterministic helper scripts for repeatable workflows.
- `assets/`: templates, images, or other files used to produce outputs.

Keep `SKILL.md` concise. Put long checklists, manuals, examples, and domain references in
`references/` so the skill can use progressive disclosure instead of loading everything by
default.

## Maintenance workflow

1. Add or update one skill directory at a time.
2. Keep the skill `name` aligned with its directory name.
3. Make the `description` broad enough to trigger on real tasks, but specific enough to avoid
   unrelated work.
4. Update the skill list in this README when adding, renaming, or removing a skill.
5. Check that bundled references are linked from `SKILL.md` and have clear loading conditions.
6. Review `agents/openai.yaml` after changing `SKILL.md` so the display name, short description,
   and default prompt stay accurate.

## Local usage

To use a skill locally, sync or copy the skill directory into the Codex skills directory used by
your environment, for example:

```text
$CODEX_HOME/skills/java-project-standards/
```

Then start a new Codex session or reload skills if your environment supports it.

## Pre-commit checks

Before committing changes:

```sh
git status --short
find . -name .DS_Store -print
```

The repository ignores `.DS_Store`, but the second command is useful to catch local files before
sharing patches or packaging skills.
