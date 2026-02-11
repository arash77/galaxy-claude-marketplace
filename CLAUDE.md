# CLAUDE.md - galaxy-claude-marketplace

**Quick Links:**
- [Official Marketplace Documentation](https://code.claude.com/docs/en/plugin-marketplaces)
- [Official Plugin Discovery Guide](https://code.claude.com/docs/en/discover-plugins)
- [Official Plugins Reference](https://code.claude.com/docs/en/plugins-reference)

## Table of Contents

1. [Project Overview](#project-overview)
2. [Architecture](#architecture)
3. [Plugin Development Workflow](#plugin-development-workflow)
   - [Adding a New Plugin](#adding-a-new-plugin)
   - [Creating a Skill](#creating-a-skill)
   - [Creating an Agent](#creating-an-agent)
4. [Key Conventions](#key-conventions)
   - [Skill YAML Frontmatter](#skill-yaml-frontmatter-fields)
   - [Agent Definition Format](#agent-definition-format)
   - [marketplace.json Structure](#marketplacejson-structure)
   - [plugin.json Structure](#pluginjson-structure)
5. [Current Plugins](#current-plugins)
6. [Plugin Caching and File Resolution](#plugin-caching-and-file-resolution)
7. [Development Best Practices](#development-best-practices)
8. [Validation and Testing Workflow](#validation-and-testing-workflow)
9. [Common Tasks](#common-tasks)
10. [Installation Methods](#installation-methods)
11. [For Claude Code Instances](#for-claude-code-instances)
12. [Common Pitfalls and Troubleshooting](#common-pitfalls-and-troubleshooting)
13. [Useful Commands](#useful-commands)
14. [Future Plugin Ideas](#future-plugin-ideas)

## Project Overview

This repository is a **Claude Code plugin marketplace** specifically designed for Galaxy Project development. Unlike traditional code repositories, this is a monorepo containing:

- **Markdown-based skill definitions** (interactive command guides)
- **Agent definitions** (specialized sub-agents for exploration)
- **JSON plugin manifests** (metadata and registry)

**No traditional source code, build systems, tests, or dependencies exist here.** This repository IS the deliverable - a collection of Claude Code plugins that enhance the development experience when working with the Galaxy Project codebase.

### Purpose

Enable Galaxy developers to work more efficiently with Claude Code by providing:
- Domain-specific skills for common Galaxy tasks (database migrations, API endpoints, testing)
- Architecture-aware agents that understand Galaxy's patterns and conventions
- Automatic context loading for Galaxy development best practices

## Architecture

### Hierarchy: Marketplace → Plugins → Commands/Agents

```
galaxy-claude-marketplace/
├── .claude-plugin/
│   └── marketplace.json          # Root marketplace manifest
├── plugins/
│   └── {plugin-name}/
│       ├── .claude-plugin/
│       │   └── plugin.json       # Individual plugin manifest
│       ├── skills/
│       │   └── {skill-name}/
│       │       └── SKILL.md      # Command/skill definition (Markdown + YAML frontmatter)
│       ├── agents/
│       │   └── {agent-name}.md   # Agent definition (Markdown + YAML frontmatter)
│       └── README.md             # Plugin documentation
├── README.md                     # User-facing marketplace documentation
└── CLAUDE.md                     # This file (for Claude Code instances)
```

**Terminology note:** In Claude Code, "skills" are also called "commands" - both terms refer to the same thing. This marketplace uses "skills" in directory names, but the official docs often use "commands".

### How Files Relate

1. **marketplace.json** (root `.claude-plugin/marketplace.json`)
   - Registry of all plugins in this marketplace
   - Points to plugin directories via `source` field
   - Users add this marketplace to Claude Code with `/plugin marketplace add`

2. **plugin.json** (each plugin's `.claude-plugin/plugin.json`)
   - Metadata for individual plugin
   - Claude Code uses this to display plugin info and validate installation

3. **SKILL.md** files (in `skills/{skill-name}/SKILL.md`)
   - YAML frontmatter defines skill metadata
   - Markdown body contains instructions for Claude Code when skill is invoked
   - Users invoke with `/{skill-name}` or skills auto-load based on `user-invocable` setting

4. **Agent .md** files (in `agents/{agent-name}.md`)
   - YAML frontmatter defines agent capabilities, tools, and constraints
   - Markdown body provides agent personality and instructions
   - Users spawn with `/agent {agent-name}` or Claude Code spawns automatically

## Plugin Development Workflow

### Adding a New Plugin

1. **Create plugin directory structure:**
   ```bash
   mkdir -p plugins/my-plugin/{.claude-plugin,skills,agents}
   ```

2. **Create plugin manifest:** `plugins/my-plugin/.claude-plugin/plugin.json`
   ```json
   {
     "name": "my-plugin",
     "description": "Brief description of what this plugin does",
     "version": "0.1.0",
     "author": {
       "name": "Your Name"
     },
     "repository": "https://github.com/arash77/galaxy-claude-marketplace",
     "license": "MIT",
     "keywords": ["galaxy", "your", "keywords"]
   }
   ```

3. **Register in marketplace:** Update `.claude-plugin/marketplace.json`
   ```json
   {
     "plugins": [
       {
         "name": "my-plugin",
         "source": "./plugins/my-plugin",
         "description": "Brief description",
         "version": "0.1.0",
         "author": {
           "name": "Your Name"
         },
         "homepage": "https://github.com/arash77/galaxy-claude-marketplace",
         "repository": "https://github.com/arash77/galaxy-claude-marketplace",
         "license": "MIT",
         "keywords": ["galaxy", "your", "keywords"]
       }
     ]
   }
   ```

4. **Add skills and agents** (see sections below)

5. **Test locally:**
   ```bash
   claude --plugin-dir ./plugins/my-plugin
   ```

6. **Submit pull request** with updated marketplace.json

### Creating a Skill

Skills are interactive guides that Claude Code follows when invoked by the user.

1. **Create skill directory and file:**
   ```bash
   mkdir -p plugins/my-plugin/skills/my-skill
   touch plugins/my-plugin/skills/my-skill/SKILL.md
   ```

2. **Write SKILL.md with YAML frontmatter:**
   ```markdown
   ---
   name: my-skill
   description: >
     Brief description of what this skill does.
     Use for: specific scenarios, keywords, use cases.
   user-invocable: true
   argument-hint: "[optional|arguments|here]"
   ---

   Persona: You are an expert in...

   Arguments:
   - $ARGUMENTS - Description of arguments if skill accepts them

   ## Instructions

   When this skill is invoked:
   1. First step...
   2. Second step...
   3. ...

   ## Examples

   Provide examples of what this skill helps with...
   ```

3. **Key frontmatter fields:**
   - `name` (required) - Skill identifier, used for invocation (`/name`)
   - `description` (required) - What the skill does, when to use it
   - `user-invocable` (optional, default: true) - If `false`, skill auto-loads in context
   - `argument-hint` (optional) - Shown in help text for skill arguments

4. **Automatic skills** (user-invocable: false)
   - Auto-load when Claude Code detects relevant context
   - Example: `galaxy-context` loads automatically in Galaxy repositories
   - Use for conventions, routing logic, or persistent context

### Creating an Agent

Agents are specialized sub-agents with specific capabilities and constraints.

1. **Create agent file:**
   ```bash
   touch plugins/my-plugin/agents/my-agent.md
   ```

2. **Write agent definition with YAML frontmatter:**
   ```markdown
   ---
   name: my-agent
   description: Brief description of agent's purpose. Use when...
   tools: Read, Glob, Grep, Bash
   disallowedTools: Write, Edit
   model: haiku
   maxTurns: 15
   ---

   You are a [role description]...

   ## Your Capabilities

   You can:
   - List what agent can do...

   You CANNOT:
   - List what agent cannot do...

   ## Instructions

   When invoked:
   1. ...
   2. ...
   ```

3. **Key frontmatter fields:**
   - `name` (required) - Agent identifier
   - `description` (required) - Agent's purpose and when to use
   - `tools` (optional) - Allowed tools (default: all)
   - `disallowedTools` (optional) - Tools agent cannot use
   - `model` (optional) - Claude model (haiku, sonnet, opus)
   - `maxTurns` (optional) - Maximum agent iterations

4. **Tool constraints:**
   - `tools: Read, Glob, Grep` - Only allow reading/searching
   - `disallowedTools: Write, Edit` - Prevent modifications
   - Use `haiku` model for fast, read-only exploration agents

## Key Conventions

### Skill YAML Frontmatter Fields

```yaml
---
name: skill-name                    # Required - used for /skill-name invocation
description: >                      # Required - multi-line description
  What this skill does.
  Use for: scenarios, keywords.
user-invocable: true                # Optional - false = auto-load, true = explicit /skill-name
argument-hint: "[args]"             # Optional - shown in help text
disable-model-invocation: false     # Optional - if true, skill loads without model inference
---
```

**user-invocable behavior:**
- `true` (default): User explicitly invokes with `/skill-name [args]`
- `false`: Skill auto-loads into context when relevant (like `galaxy-context`)

**disable-model-invocation:**
- `false` (default): Skill instructions are processed by Claude (normal behavior)
- `true`: Skill loads directly without model inference (faster, for simple prompts)

**description field:**
- First sentence: what skill does
- Second part: when/why to use it
- Include keywords that help Claude Code decide when to invoke

### Agent Definition Format

```yaml
---
name: agent-name             # Required - agent identifier
description: Brief purpose   # Required - when to use this agent
tools: Read, Glob, Grep      # Optional - whitelist allowed tools
disallowedTools: Write, Edit # Optional - blacklist forbidden tools
model: haiku                 # Optional - haiku/sonnet/opus (default: sonnet)
maxTurns: 15                 # Optional - max iterations (default: 10)
---
```

**Model selection:**
- `haiku`: Fast, cost-effective, good for exploration/reading
- `sonnet`: Balanced, good for complex analysis
- `opus`: Most capable, use sparingly for complex reasoning

**Tool constraints pattern:**
- Read-only agents: `tools: Read, Glob, Grep, Bash` + `disallowedTools: Write, Edit`
- General purpose: omit both fields (gets all tools)
- Restricted: explicitly list allowed tools

### marketplace.json Structure

```json
{
  "name": "marketplace-name",
  "owner": {
    "name": "Owner Name",
    "email": "email@example.com"
  },
  "metadata": {
    "description": "Marketplace description",
    "version": "1.0.0",
    "pluginRoot": "./plugins"
  },
  "plugins": [
    {
      "name": "plugin-name",
      "source": "./plugins/plugin-name",
      "description": "Plugin description",
      "version": "0.1.0",
      "author": {
        "name": "Author Name"
      },
      "homepage": "https://...",
      "repository": "https://...",
      "license": "MIT",
      "keywords": ["keyword1", "keyword2"]
    }
  ]
}
```

**Important:**
- `name` must be kebab-case, no spaces (users see this when installing: `plugin-name@marketplace-name`)
- `source` must be relative path from marketplace root (or a source object for GitHub/Git URLs)
- `version` should follow semver (major.minor.patch)
- `keywords` help users discover plugins with `/plugin search`
- `metadata.pluginRoot` (optional): base directory prepended to relative plugin source paths
- `strict` field (optional, default: true) controls how marketplace entry merges with plugin.json:
  - `true` (default): marketplace fields merge with plugin.json (plugin.json can define components)
  - `false`: marketplace entry defines everything (plugin.json must not declare components)

**Reserved marketplace names** (cannot be used):
- `claude-code-marketplace`, `claude-code-plugins`, `claude-plugins-official`
- `anthropic-marketplace`, `anthropic-plugins`
- `agent-skills`, `life-sciences`
- Names that impersonate official marketplaces (like `official-claude-plugins`)

### plugin.json Structure

```json
{
  "name": "plugin-name",
  "description": "Plugin description",
  "version": "0.1.0",
  "author": {
    "name": "Author Name"
  },
  "repository": "https://github.com/...",
  "license": "MIT",
  "keywords": ["keyword1", "keyword2"]
}
```

**Must match marketplace.json entry** for consistency.

## Current Plugins

### galaxy-dev (v0.2.0)

**Location:** `plugins/galaxy-dev/`

**Purpose:** Galaxy Project development tools for database migrations, API development, testing, linting, and codebase exploration.

#### Skills

1. **galaxy-context** (automatic)
   - Auto-loads in Galaxy repositories
   - Provides conventions, patterns, and skill routing logic
   - Guides Claude Code to invoke appropriate skills proactively
   - File: `skills/galaxy-context/SKILL.md`

2. **/galaxy-db-migration** (user-invocable)
   - Database migration workflows with Alembic
   - Arguments: `[create|upgrade|downgrade|status|troubleshoot]`
   - Use for: schema changes, migrations, SQLAlchemy model modifications
   - File: `skills/galaxy-db-migration/SKILL.md`

3. **/galaxy-api-endpoint** (user-invocable)
   - Guide for creating new REST API endpoints
   - Arguments: `[resource-name]`
   - Use for: FastAPI routers, new API routes
   - File: `skills/galaxy-api-endpoint/SKILL.md`

4. **/galaxy-testing** (user-invocable)
   - Test running and writing guide
   - Arguments: `[run|write|unit|api|integration]`
   - Use for: pytest commands, test creation patterns
   - File: `skills/galaxy-testing/SKILL.md`

5. **/galaxy-linting** (user-invocable)
   - Code linting, formatting, and type checking workflows
   - Arguments: `[check|fix|python|client|mypy|full]`
   - Use for: ruff, black, isort, flake8, ESLint, Prettier, mypy, code style enforcement, CI compliance
   - File: `skills/galaxy-linting/SKILL.md`

#### Agents

1. **galaxy-explorer**
   - Read-only exploration agent
   - Model: haiku (fast)
   - Tools: Read, Glob, Grep, Bash (no Write/Edit)
   - MaxTurns: 15
   - Use for: architecture questions, pattern discovery, code location
   - File: `agents/galaxy-explorer.md`

## Plugin Caching and File Resolution

**Important:** When users install a plugin, Claude Code copies the plugin directory to a cache location (typically `~/.claude/plugins/cache/`). This means:

1. **Plugins cannot reference files outside their directory** using paths like `../shared-utils`
2. **Use `${CLAUDE_PLUGIN_ROOT}` in hooks and MCP server configs** to reference files within the plugin

### ${CLAUDE_PLUGIN_ROOT} Variable

When defining hooks or MCP servers in your plugin, use the `${CLAUDE_PLUGIN_ROOT}` variable to reference files:

```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Write|Edit",
      "hooks": [{
        "type": "command",
        "command": "${CLAUDE_PLUGIN_ROOT}/scripts/validate.sh"
      }]
    }]
  },
  "mcpServers": {
    "my-server": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/server-binary",
      "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.json"]
    }
  }
}
```

This variable expands to the absolute path of the installed plugin directory.

### Sharing Files Across Plugins

If you need to share files across multiple plugins:
- **Option 1:** Use symlinks (followed during copying)
- **Option 2:** Restructure so shared directory is inside each plugin's source path
- **Option 3:** Duplicate shared files in each plugin (simplest for small files)

## Development Best Practices

### Do's
- Keep skills focused on a single task or workflow
- Use clear, imperative language in skill instructions
- Include examples and common scenarios
- Test skills locally before submitting PR
- Follow existing naming conventions (kebab-case)
- Document when/why to use each skill in description
- Use `${CLAUDE_PLUGIN_ROOT}` for any file paths in hooks or MCP servers
- Validate your plugin with `claude plugin validate .` before submitting

### Don'ts
- Don't add build systems or dependencies to this repo
- Don't include executable code (this is documentation only)
- Don't create overly broad skills (split into focused skills instead)
- Don't forget to update both plugin.json and marketplace.json
- Don't use generic names (prefix with project context like "galaxy-")
- Don't reference files outside the plugin directory with relative paths like `../`

### Validation and Testing Workflow

```bash
# Validate marketplace JSON structure
claude plugin validate .

# Or from within Claude Code
/plugin validate .

# Test individual plugin locally
claude --plugin-dir ./plugins/galaxy-dev

# Add marketplace for testing
/plugin marketplace add ./

# Install plugin from local marketplace
/plugin install galaxy-dev@galaxy-claude-marketplace

# Test skill invocation
/galaxy-db-migration create

# Test agent spawning
/agent galaxy-explorer

# Verify marketplace structure
cat .claude-plugin/marketplace.json | jq .

# Check all plugin manifests
find plugins -name plugin.json -exec cat {} \;

# Clear plugin cache if needed (after major changes)
rm -rf ~/.claude/plugins/cache
```

**Important validation checks:**
- No duplicate plugin names in marketplace
- All required fields present (name, source for plugins; name, owner for marketplace)
- Valid JSON syntax (no trailing commas, missing quotes, etc.)
- Source paths don't contain path traversal (`..`)
- Plugin directories contain required `.claude-plugin/plugin.json` file

## Common Tasks

### Adding a New Skill to galaxy-dev

1. Create skill directory:
   ```bash
   mkdir -p plugins/galaxy-dev/skills/new-skill
   ```

2. Create `plugins/galaxy-dev/skills/new-skill/SKILL.md` with frontmatter

3. Test:
   ```bash
   claude --plugin-dir ./plugins/galaxy-dev
   /new-skill
   ```

4. Update plugin README if needed

5. Submit PR

### Updating an Existing Skill

1. Edit the SKILL.md file directly

2. Increment version in `plugins/galaxy-dev/.claude-plugin/plugin.json`

3. Update corresponding entry in `.claude-plugin/marketplace.json`

4. Test locally

5. Submit PR

### Adding Supporting Documentation

Skills can reference additional markdown files:

```
skills/
└── my-skill/
    ├── SKILL.md          # Main skill definition
    ├── reference.md      # Additional reference material
    └── examples.md       # Extended examples
```

Reference from SKILL.md:
```markdown
For detailed reference, see `reference.md` in this skill directory.
```

## Repository Metadata

- **Purpose:** Claude Code plugin marketplace for Galaxy Project
- **Content:** Markdown skill definitions + JSON manifests
- **No code builds, tests, or dependencies**
- **Skills describe Galaxy commands, not commands for this repo**
- **License:** MIT
- **Maintainer:** Galaxy Project Contributors

## Installation Methods

Users can add this marketplace in several ways:

### GitHub (Recommended)
```shell
/plugin marketplace add arash77/galaxy-claude-marketplace
```

### Git URL
```shell
/plugin marketplace add https://github.com/arash77/galaxy-claude-marketplace.git
```

### Local Development
```shell
/plugin marketplace add ./galaxy-claude-marketplace
```

### Direct URL (Limited - relative paths won't work)
```shell
/plugin marketplace add https://raw.githubusercontent.com/arash77/galaxy-claude-marketplace/main/.claude-plugin/marketplace.json
```

## For Claude Code Instances

When working in this repository:

1. **This is not a software project** - it's a documentation/configuration repository
2. **No build/test/lint commands exist** for this repo itself
3. **Skills describe external tools** (Galaxy Project development workflows)
4. **Modifications are Markdown/JSON only** - no executable code
5. **"Testing" means validating and loading plugins** into Claude Code
6. **The only validation command is:** `claude plugin validate .`

### What This Repo Contains

- **Markdown files** with instructions for Claude Code
- **JSON manifests** describing plugins and marketplace structure
- **No Python, JavaScript, or other executable code**
- **No dependencies, package.json, requirements.txt, etc.**

### What This Repo Does NOT Contain

- Build systems (no webpack, rollup, etc.)
- Test frameworks (no pytest, jest, etc.)
- Linters or formatters (no eslint, black, etc.)
- CI/CD pipelines for this repo (though skills may describe Galaxy's CI/CD)
- Dependencies or package managers

If asked to "run tests", "build the project", or "install dependencies", clarify that this repository contains plugin definitions only. The skills within describe how to work with the Galaxy Project, not commands for this repository itself.

The only relevant command for this repository is:
```bash
claude plugin validate .
```

## Common Pitfalls and Troubleshooting

### Skills Not Appearing After Installation

**Symptom:** Installed a plugin but skills/commands don't appear

**Solutions:**
1. Clear the plugin cache: `rm -rf ~/.claude/plugins/cache`
2. Restart Claude Code
3. Reinstall the plugin: `/plugin install plugin-name@marketplace-name`
4. Check the `/plugin` Errors tab for loading errors

### Relative Path Plugins Fail in URL-Based Marketplaces

**Symptom:** Added marketplace via URL but plugins with `"source": "./plugins/..."` fail to install

**Cause:** URL-based marketplaces only download `marketplace.json`, not plugin files

**Solutions:**
- Use Git-based marketplace (host on GitHub/GitLab)
- Or change plugin entries to use GitHub/Git URL sources instead of relative paths

### Files Not Found After Installation

**Symptom:** Plugin installs but references to files fail

**Cause:** Plugins are copied to cache, so paths like `../shared-utils` won't work

**Solutions:**
- Use `${CLAUDE_PLUGIN_ROOT}` variable in hooks/MCP configs
- Don't reference files outside plugin directory
- See [Plugin Caching and File Resolution](#plugin-caching-and-file-resolution)

### Validation Errors

| Error | Solution |
|-------|----------|
| `Invalid JSON syntax` | Check for missing commas, extra commas, unquoted strings |
| `Duplicate plugin name` | Ensure each plugin has unique `name` in marketplace |
| `Path traversal not allowed` | Don't use `..` in source paths |
| `File not found: .claude-plugin/marketplace.json` | Create marketplace manifest at root |

## Useful Commands

```bash
# Validate marketplace and all plugins
claude plugin validate .

# View marketplace structure
cat .claude-plugin/marketplace.json | jq .

# List all skills
find plugins -type f -name "SKILL.md"

# List all agents
find plugins -type f -path "*/agents/*.md"

# View plugin metadata
cat plugins/galaxy-dev/.claude-plugin/plugin.json | jq .

# Test plugin locally
claude --plugin-dir ./plugins/galaxy-dev

# Add marketplace to Claude Code (GitHub)
/plugin marketplace add arash77/galaxy-claude-marketplace

# Add marketplace to Claude Code (local)
/plugin marketplace add ./

# Install plugin from marketplace
/plugin install galaxy-dev@galaxy-claude-marketplace

# List all added marketplaces
/plugin marketplace list

# Update marketplace listings
/plugin marketplace update galaxy-claude-marketplace

# View installed plugins
/plugin

# Clear plugin cache
rm -rf ~/.claude/plugins/cache
```

## Future Plugin Ideas

Potential plugins to add to this marketplace:

- **galaxy-workflow**: Workflow development patterns
- **galaxy-tools**: Galaxy tool development guide
- **galaxy-deployment**: Deployment and configuration guides
- **galaxy-debugging**: Common debugging patterns and tools
- **galaxy-performance**: Performance profiling and optimization

Each plugin should follow the same structure: `.claude-plugin/plugin.json`, skills/, agents/, README.md.
