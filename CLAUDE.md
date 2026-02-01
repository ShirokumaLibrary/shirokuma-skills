# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Project Overview

**shirokuma-skills** - Reusable AI skills and rules for Claude Code following the Agent Skills open standard.

### Purpose

Skills and rules collection distributed to projects via:
- `shirokuma-docs init --with-skills --with-rules`
- Claude Code plugin system
- Manual copy

### Repository Organization

| Repo | Purpose |
|------|---------|
| [ShirokumaDevelopment/shirokuma-skills](https://github.com/ShirokumaDevelopment/shirokuma-skills) | Private (development) |
| [ShirokumaLibrary/shirokuma-skills](https://github.com/ShirokumaLibrary/shirokuma-skills) | Public (release) |

Release via: `shirokuma-docs repo-pairs release shirokuma-skills --tag <version>`

## Project Structure

```
shirokuma-skills/
├── skills/                    # Agent Skills (16 skills)
│   ├── managing-*/            # Meta skills (agents, skills, plugins, output-styles, rules)
│   ├── managing-github-items/ # Create items/specs, comments, labels
│   ├── nextjs-vibe-coding/    # TDD implementation for Next.js
│   ├── code-reviewing/        # Code review with specialized roles
│   ├── frontend-designing/    # Memorable UI design
│   ├── codebase-rule-discovery/ # Pattern analysis
│   ├── github-project-setup/  # GitHub Project initialization
│   ├── project-config-generator/ # Project-specific config generation
│   ├── publishing/            # Public repo release workflow
│   ├── starting-session/      # Session start with context display
│   ├── ending-session/        # Session end with handover save
│   └── showing-github/        # Dashboard, items, issues, handovers, specs
├── rules/                     # Claude Code rules (12 rules)
│   ├── skill-authoring.md
│   ├── output-destinations.md
│   ├── github/                # GitHub conventions
│   ├── nextjs/                # Next.js patterns
│   └── shirokuma-docs/        # Annotation patterns
├── .claude-plugin/            # Plugin manifest
│   ├── plugin.json
│   └── marketplace.json
└── CLAUDE.md
```

## Skill Structure Convention

Each skill follows this structure:

```
skill-name/
├── SKILL.md              # Main skill definition (required)
├── common/               # Shared/reusable content
│   ├── patterns/         # Best practice patterns
│   ├── reference/        # Reference documentation
│   └── templates/        # Code templates
└── project/              # Project-specific (generated per-project)
    ├── reference/
    ├── patterns/
    ├── issues/
    ├── setup/
    └── workflows/
```

**Important**: The `project/` directory is NOT included in this repo. It is generated per-project by `project-config-generator` skill.

## Development Commands

```bash
# No build step required (pure markdown/YAML)

# Validate skill structure
python3 .claude/skills/managing-skills/scripts/quick_validate.py skills/<skill-name>

# Release to public
cd /home/squeeze.linux/projects/shirokuma-skills
shirokuma-docs repo-pairs release shirokuma-skills --tag <version>
```

## Key Rules

1. **SKILL.md format**: Follow Agent Skills standard (see `rules/skill-authoring.md`)
2. **Language**: All skill/rule content in English, UI-facing content supports i18n
3. **No project-specific content**: `common/` only - project-specific goes in `project/` (generated)
4. **Templates use `.template` extension**: e.g., `component.test.tsx.template`
5. **Keep skills self-contained**: Each skill directory should work independently

## Adding a New Skill

1. Create `skills/<skill-name>/SKILL.md`
2. Add `common/` directory if needed (patterns, reference, templates)
3. Update `shirokuma-docs init` command's `AVAILABLE_SKILLS` list
4. Update `README.md` skills table
5. Release: `shirokuma-docs repo-pairs release shirokuma-skills --tag <version>`

## Adding a New Rule

1. Create `rules/<category>/<rule-name>.md` or `rules/<rule-name>.md`
2. Update `shirokuma-docs init` command's `AVAILABLE_RULES` list
3. Release with skills

## AI Behavior Rules

1. **Tool priority**: Use `shirokuma-docs` CLI over `gh` commands. Fallback to `gh` only when shirokuma-docs fails (state the reason).
2. **Scope awareness**: This repo is Markdown only. Do not review source code, dependencies, or infrastructure of other projects unless explicitly asked.
3. **Error recovery**: Never say "I'll be careful next time." Always propose a config file change (CLAUDE.md, skills, rules, settings) to prevent recurrence.
4. **Confirmation required**: Ask before deviating from the user's request scope.
