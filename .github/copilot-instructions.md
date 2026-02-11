# Claude Code Plugin Marketplace - Development Guide

## What This Repository Is

This is a **Claude Code plugin marketplace** for Galaxy Project development tools - NOT a traditional software project. It contains:
- **Markdown-based skill definitions** (interactive command guides for Claude Code)
- **Agent definitions** (specialized sub-agents with specific capabilities)
- **JSON manifests** (marketplace and plugin metadata)

**No executable code, build systems, tests, or dependencies exist here.** The repository IS the deliverable.

## Repository Structure

```
galaxy-claude-marketplace/
├── .claude-plugin/
│   └── marketplace.json              # Root marketplace registry
├── plugins/
│   └── {plugin-name}/
│       ├── .claude-plugin/
│       │   └── plugin.json          # Plugin metadata
│       ├── skills/
│       │   └── {skill-name}/
│       │       ├── SKILL.md         # Skill definition (YAML + Markdown)
│       │       └── reference.md     # Optional supporting docs
│       ├── agents/
│       │   └── {agent-name}.md      # Agent definition (YAML + Markdown)
│       └── README.md                # Plugin documentation
└── README.md                         # User-facing marketplace docs
```

## Critical Conventions

### 1. Skills Are Markdown Files with YAML Frontmatter

Every skill follows this pattern (`skills/{name}/SKILL.md`):

```markdown
---
name: skill-name
description: >
  What this skill does.
  Use for: specific scenarios, keywords.
user-invocable: true
argument-hint: "[optional|args]"
---

Persona: You are an expert in...

Arguments:
- $ARGUMENTS - Description of what arguments mean

## Instructions

1. Step one...
2. Step two...
```

**Key frontmatter fields:**
- `user-invocable: true` - User invokes with `/skill-name`
- `user-invocable: false` - Auto-loads into context (like `galaxy-context`)
- `argument-hint` - Shown in help text, describes what arguments the skill accepts

### 2. Agents Are Read-Only Explorers with Tool Constraints

Agents live in `agents/{name}.md` with this pattern:

```markdown
---
name: agent-name
description: Agent purpose. Use when...
tools: Read, Glob, Grep, Bash
disallowedTools: Write, Edit
model: haiku
maxTurns: 15
---

You are a [role]...
```

**Pattern:** Use `haiku` model with read-only tools (`Read, Glob, Grep, Bash`) for fast exploration agents.

### 3. Two-Level Registration System

1. **Plugin manifest** (`plugins/{name}/.claude-plugin/plugin.json`):
   ```json
   {
     "name": "plugin-name",
     "description": "What this plugin does",
     "version": "0.1.0",
     "author": {"name": "Author Name"},
     "repository": "https://github.com/...",
     "license": "MIT",
     "keywords": ["galaxy", "..."]
   }
   ```

2. **Marketplace registry** (`.claude-plugin/marketplace.json`):
   ```json
   {
     "plugins": [{
       "name": "plugin-name",
       "source": "./plugins/plugin-name",
       "description": "...",
       "version": "0.1.0",
       ...
     }]
   }
   ```

**Both must be updated when adding/modifying plugins.**

### 4. No Relative Paths Across Plugins

Plugins are cached during installation. Use `${CLAUDE_PLUGIN_ROOT}` for file references in hooks/MCP configs:

```json
{
  "hooks": {
    "PostToolUse": [{
      "type": "command",
      "command": "${CLAUDE_PLUGIN_ROOT}/scripts/validate.sh"
    }]
  }
}
```

## Development Workflows

### Adding a New Skill

1. Create directory: `mkdir -p plugins/{plugin}/skills/{skill-name}`
2. Create `SKILL.md` with YAML frontmatter + Markdown body
3. Test locally: `claude --plugin-dir ./plugins/{plugin}`
4. Invoke with `/{skill-name}` to verify

### Adding a New Plugin

1. Create structure:
   ```bash
   mkdir -p plugins/my-plugin/{.claude-plugin,skills,agents}
   ```

2. Create `plugins/my-plugin/.claude-plugin/plugin.json`

3. Update `.claude-plugin/marketplace.json` to register the plugin

4. Validate: `claude plugin validate .`

### Modifying Existing Content

- **Skills/Agents:** Edit Markdown files directly
- **Metadata changes:** Update both `plugin.json` AND `marketplace.json`
- **Version policy:** Increment version in both files when making changes

## Testing and Validation

**The ONLY validation command for this repository is:**

```bash
claude plugin validate .
```

If asked to "run tests", "build", or "install dependencies" - this repo has none. Skills DESCRIBE Galaxy Project workflows but don't execute them.

**Local testing workflow:**
```bash
# Validate structure
claude plugin validate .

# Test plugin locally
claude --plugin-dir ./plugins/galaxy-dev

# Test skill invocation
/db-migration create

# Test agent
/agent galaxy-explorer
```

## Key Patterns from Existing Plugins

### galaxy-dev Plugin Architecture

**Skills:**
- `galaxy-context` (user-invocable: false) - Auto-loads to provide conventions and route to specialized skills
- `/db-migration` - Alembic database migration workflows with task-based routing
- `/api-endpoint` - FastAPI endpoint creation guide
- `/testing` - Galaxy test infrastructure guide

**Agent:**
- `galaxy-explorer` - Read-only exploration (haiku model, 15 turns max)

**Pattern:** One coordination skill (galaxy-context) routes to specialized task skills. Agent handles exploratory questions.

### Skill Argument Parsing Pattern

Many skills accept optional task arguments:

```markdown
Arguments:
- $ARGUMENTS - Optional: "create", "upgrade", "status", etc.

## If $ARGUMENTS is empty: Display Task Menu

Present menu of available tasks...

## If $ARGUMENTS is "create": Guide Through Creating...

Detailed step-by-step...
```

This creates multi-mode skills that can show a menu or jump to specific guidance.

## Common Pitfalls

1. **Don't create build/test infrastructure** - This repo contains documentation only
2. **Update both JSON files** - Plugin changes require updating `plugin.json` AND `marketplace.json`
3. **Avoid generic skill names** - Prefix with project context (e.g., `galaxy-`)
4. **No code execution** - Skills describe external tools, don't execute them
5. **Path resolution** - Use `${CLAUDE_PLUGIN_ROOT}` for any file references

## Supporting Documentation

Skills can reference additional Markdown files in their directory:
- `reference.md` - Extended reference material
- `examples.md` - Detailed examples

Reference from SKILL.md: `"For details, see reference.md in this skill directory."`

## When Working in This Repository

**Remember:**
- This repository IS the deliverable
- Modifications are Markdown + JSON only
- "Testing" means validating JSON structure and loading plugins into Claude Code
- Skills describe Galaxy Project commands, not commands for this repository
- The marketplace structure is the architecture - there's no runtime code to understand
