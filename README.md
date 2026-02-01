# shirokuma-skills

Reusable AI skills and rules for Claude Code and 25+ AI tools following the [Agent Skills](https://agentskills.io) open standard.

## Prerequisites

Most skills in this collection require **[shirokuma-docs CLI](https://github.com/ShirokumaLibrary/shirokuma-docs)** for GitHub operations (Issues, Discussions, Projects) and code linting.

```bash
npm i -g @shirokuma-library/shirokuma-docs
```

Skills that work **without** shirokuma-docs: `managing-agents`, `managing-skills`, `managing-plugins`, `managing-output-styles`, `managing-rules`, `show-dashboard`, `show-handovers`, `show-specs`, `create-spec`, `add-issue-comment`, `manage-labels`

## Installation

### Claude Code Plugin

```bash
/plugin install shirokuma-skills
```

### Manual Copy

Copy the `skills/` and `rules/` directories to your project's `.claude/` directory.

### Via shirokuma-docs CLI

```bash
shirokuma-docs init --with-skills --with-rules --lang ja
```

## Structure

```
shirokuma-skills/
├── skills/                           # Agent Skills (23 skills)
│   ├── managing-*/                   # Meta skills (agents, skills, plugins, output-styles, rules)
│   ├── nextjs-vibe-coding/           # TDD implementation for Next.js
│   ├── code-reviewing/               # Code review with specialized roles
│   ├── frontend-designing/           # Memorable UI design
│   ├── codebase-rule-discovery/      # Pattern analysis for conventions
│   ├── project-config-generator/     # Project-specific config generation
│   ├── session-management/           # Session start/end with GitHub
│   ├── github-project-setup/         # GitHub Project initialization
│   ├── start-session/                # Session start
│   ├── end-session/                  # Session end with handover
│   ├── show-*/                       # Dashboard and display skills
│   ├── create-item/                  # Project item creation
│   ├── create-spec/                  # Spec discussion creation
│   ├── add-issue-comment/            # Issue/PR commenting
│   └── manage-labels/                # Label management
├── rules/                            # Claude Code rules (12 rules)
│   ├── skill-authoring.md            # Skill creation guidelines
│   ├── output-destinations.md        # Output destination conventions
│   ├── github/                       # GitHub conventions
│   │   ├── project-items.md
│   │   └── discussions-usage.md
│   ├── nextjs/                       # Next.js patterns (7 rules)
│   │   ├── tech-stack.md
│   │   ├── server-actions.md
│   │   ├── tailwind-v4.md
│   │   ├── radix-ui-hydration.md
│   │   ├── known-issues.md
│   │   ├── lib-structure.md
│   │   └── testing.md
│   └── shirokuma-docs/
│       └── shirokuma-annotations.md
└── .claude-plugin/                   # Claude Code plugin config
    ├── marketplace.json
    └── plugin.json
```

## Compatible Tools

This repository follows the Agent Skills standard, compatible with:

- **Claude Code** (Anthropic)
- **OpenAI Codex**
- **Cursor**
- **VS Code Copilot**
- **Gemini CLI**
- And 20+ more tools

## Skills

### Meta Skills

| Skill | Description |
|-------|-------------|
| `managing-agents` | Create and improve Claude Code agent files following Anthropic best practices |
| `managing-skills` | Create and improve Claude Code skill files |
| `managing-plugins` | Create and distribute Claude Code plugins |
| `managing-output-styles` | Manage output styles (default/explanatory/learning modes) |
| `managing-rules` | Create and organize Claude Code rules |

### Development Skills

| Skill | Description |
|-------|-------------|
| `nextjs-vibe-coding` | TDD implementation workflow for Next.js projects |
| `code-reviewing` | Code review workflow with specialized roles (code, security, testing, Next.js) |
| `frontend-designing` | Create distinctive, production-grade frontend interfaces |

### Analysis & Configuration Skills

| Skill | Description |
|-------|-------------|
| `codebase-rule-discovery` | Analyze codebases to discover patterns and propose conventions |
| `project-config-generator` | Generate project-specific configuration for skills |

### Session Management Skills

| Skill | Description |
|-------|-------------|
| `session-management` | GitHub-integrated session management |
| `start-session` | Start session showing project status and previous handovers |
| `end-session` | End session saving handover info and updating project items |

### GitHub Operation Skills

| Skill | Description |
|-------|-------------|
| `github-project-setup` | Automate GitHub Project setup with Status, Priority, Type, Size fields |
| `create-item` | Create project items (Issues or DraftIssues) |
| `create-spec` | Create spec Discussions for design documents |
| `add-issue-comment` | Add comments to Issues/PRs |
| `manage-labels` | Manage Issue/PR labels |

### Display Skills

| Skill | Description |
|-------|-------------|
| `show-dashboard` | Project dashboard aggregating GitHub data |
| `show-handovers` | Display past handover information |
| `show-issues` | List GitHub Issues with filtering |
| `show-project-items` | Display GitHub Project items with Status filter |
| `show-specs` | List spec Discussions |

## Rules

| Rule | Description |
|------|-------------|
| `skill-authoring.md` | Skill creation guidelines |
| `output-destinations.md` | Output destination conventions |
| `github/project-items.md` | GitHub Projects required fields and status workflow |
| `github/discussions-usage.md` | Discussion categories and usage patterns |
| `nextjs/tech-stack.md` | Tech stack reference |
| `nextjs/server-actions.md` | Server Actions patterns |
| `nextjs/tailwind-v4.md` | Tailwind CSS v4 patterns |
| `nextjs/radix-ui-hydration.md` | Radix UI hydration fix patterns |
| `nextjs/known-issues.md` | Known issues and CVEs |
| `nextjs/lib-structure.md` | Library structure conventions |
| `nextjs/testing.md` | Testing patterns |
| `shirokuma-docs/shirokuma-annotations.md` | Code annotation patterns |

## Usage Examples

### Install as Plugin

```bash
# Add marketplace
/plugin marketplace add https://raw.githubusercontent.com/ShirokumaLibrary/shirokuma-skills/main/.claude-plugin/marketplace.json

# Install plugin
/plugin install shirokuma-skills@shirokuma-library
```

### Copy Specific Skills

```bash
# Copy a single skill
cp -r shirokuma-skills/skills/nextjs-vibe-coding .claude/skills/

# Copy all rules
cp -r shirokuma-skills/rules/* .claude/rules/
```

### Initialize with shirokuma-docs

```bash
# Initialize project with Japanese language policy
shirokuma-docs init --with-skills --with-rules --lang ja
```

## License

MIT
