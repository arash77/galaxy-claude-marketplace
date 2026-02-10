# Galaxy Dev - Claude Code Plugin

A Claude Code plugin providing development tools for the Galaxy Project, including database migrations, API endpoint creation, testing, and codebase exploration.

## Features

### Skills

#### `galaxy-context` (Automatic) - Galaxy Development Context & Routing
**Not user-invocable** - Loads automatically when working in a Galaxy codebase.

Provides Galaxy-specific conventions and intelligent routing:
- Enforces critical conventions (always use `run_tests.sh`, avoid reading large files)
- Routes user intent to appropriate skills (`/db-migration`, `/api-endpoint`, `/testing`)
- Provides Galaxy architecture reference (Manager pattern, FastAPI structure)
- Acts as a coordination layer between user requests and specialized skills

This skill works behind the scenes to ensure Claude follows Galaxy best practices and uses the right tools for each task.

#### `/db-migration` - Database Migration Management
Comprehensive guidance for Galaxy's Alembic-based database migrations:
- Create new migration revisions
- Upgrade/downgrade database
- Check migration status
- Troubleshoot migration errors

**Usage:**
```bash
/db-migration              # Show task menu
/db-migration create       # Guide through creating new migration
/db-migration upgrade      # Show upgrade commands
/db-migration status       # Check database version
/db-migration troubleshoot # Diagnose errors
```

#### `/api-endpoint` - API Endpoint Creation
Step-by-step guide for creating new Galaxy API endpoints following project patterns:
- Define Pydantic schemas
- Create manager methods
- Build FastAPI routers
- Write comprehensive tests

**Usage:**
```bash
/api-endpoint              # Show creation workflow
/api-endpoint credentials  # Guide for specific resource
```

#### `/testing` - Test Running and Writing
Complete guide to Galaxy's test infrastructure:
- Run unit, API, and integration tests
- Write new tests following patterns
- Use test fixtures and populators
- Debug test failures

**Usage:**
```bash
/testing           # Show test types menu
/testing run       # Test running reference
/testing write     # Guide for writing tests
/testing unit      # Unit test patterns
/testing api       # API test patterns
/testing integration # Integration test patterns
```

### Agent

#### `galaxy-explorer` - Codebase Explorer
Architecture-aware agent for exploring the Galaxy codebase:
- Find code patterns and implementations
- Explain architecture and design patterns
- Locate functionality across the codebase
- Provide file paths and line numbers
- Read-only exploration (no edits)

The explorer understands Galaxy's structure and knows to avoid reading massive auto-generated files.

## Installation

### From Custom Marketplace (Recommended)

Add the Galaxy Development Marketplace to your Claude configuration:

```bash
# Add marketplace to Claude config
claude config set pluginMarketplaces.0 https://raw.githubusercontent.com/arash77/galaxy-claude-marketplace/main/marketplace.json

# Install galaxy-dev plugin
claude plugin install galaxy-dev
```

Or manually edit `~/.config/claude/config.json`:

```json
{
  "pluginMarketplaces": [
    "https://raw.githubusercontent.com/arash77/galaxy-claude-marketplace/main/marketplace.json"
  ]
}
```

Then install the plugin:

```bash
claude plugin install galaxy-dev
```

### From Source (Development)

For development or contributing, clone the monorepo and use locally:

```bash
# Clone the marketplace monorepo
git clone https://github.com/arash77/galaxy-claude-marketplace.git ~/galaxy-claude-marketplace

# Use the plugin
claude --plugin-dir ~/galaxy-claude-marketplace/plugins/galaxy-dev
```

## Usage Examples

### Creating a Database Migration

```bash
# Start migration workflow
/db-migration create

# Claude will guide you through:
# 1. Confirming model updates
# 2. Creating revision file
# 3. Implementing upgrade/downgrade
# 4. Running and verifying migration
```

### Adding a New API Endpoint

```bash
# Guide for creating credentials endpoint
/api-endpoint credentials

# Claude will walk through:
# 1. Finding similar endpoints as reference
# 2. Defining Pydantic schemas
# 3. Creating manager methods
# 4. Building FastAPI router
# 5. Registering router
# 6. Writing comprehensive tests
# 7. Running and verifying
```

### Running Tests

```bash
# Get test running reference
/testing run

# Shows commands like:
# ./run_tests.sh -api lib/galaxy_test/api/test_workflows.py
# ./run_tests.sh -integration test/integration/test_vault.py
# ./run_tests.sh --coverage -unit test/unit/managers/
```

### Writing New Tests

```bash
# Guide for writing API tests
/testing api

# Shows:
# - Test structure and patterns
# - ApiTestCase usage
# - HTTP method helpers
# - Assertion methods
# - Populator usage
# - Example test code
```

### Exploring the Codebase

Delegate to the galaxy-explorer agent for questions about the codebase:

```
User: "Where is workflow execution handled?"

Claude will use the galaxy-explorer agent to:
1. Search for relevant managers in lib/galaxy/managers/
2. Find the WorkflowsManager class
3. Locate execution methods
4. Provide file paths with line numbers
5. Explain the architecture
6. Suggest related files to check
```

## Architecture Overview

Galaxy is a scientific workflow platform with:

**Backend (Python):**
- FastAPI for REST APIs
- SQLAlchemy 2.0 for ORM
- Manager pattern for business logic
- Pydantic schemas for validation

**Frontend (Vue.js + TypeScript):**
- Vue 2.7 (migrating to Vue 3)
- Pinia for state management
- TypeScript throughout

**Testing:**
- pytest with custom runner
- Unit, API, integration, and E2E tests
- Comprehensive test infrastructure

## Key Directories

```
lib/galaxy/
├── managers/           # Business logic (Manager pattern)
├── model/             # SQLAlchemy models
├── schema/            # Pydantic schemas
├── webapps/galaxy/api/ # FastAPI routers
└── workflow/          # Workflow engine

client/
├── src/components/    # Vue components
├── src/stores/        # Pinia stores
└── src/composables/   # Composition functions

test/
├── unit/             # Fast unit tests
├── integration/      # Full system tests
└── integration_selenium/ # Browser E2E tests

lib/galaxy_test/api/  # API endpoint tests
```

## Skill Details

### Galaxy Context Skill (Automatic)

The `galaxy-context` skill is **not user-invocable** - it loads automatically when Claude works in a Galaxy codebase. It provides three key functions:

#### 1. Galaxy Development Conventions

**Critical Rules Enforced:**
- **Always use `./run_tests.sh`** - Never run `pytest` directly. Galaxy's test suite requires special configuration and fixtures that only `run_tests.sh` provides.
- **Avoid reading large files** - Files like `client/src/api/schema/schema.ts` (46,529 lines) and `lib/galaxy/model/__init__.py` (12,677 lines) will exhaust token budgets. Use Grep with patterns or the galaxy-explorer agent instead.
- **Manager pattern** - Business logic belongs in manager classes (`lib/galaxy/managers/`) following the flow: API Router → Manager → Model.

#### 2. Intelligent Skill Routing

Routes user intent to specialized skills:
- **Database operations** → `/db-migration` (schema changes, migrations, Alembic)
- **API development** → `/api-endpoint` (FastAPI routers, Pydantic schemas)
- **Testing operations** → `/testing` (running tests, writing tests, test patterns)
- **Architecture questions** → `galaxy-explorer` agent (codebase exploration, pattern identification)

**Example:** When a user mentions "add a column to the workflow table", the context skill recognizes this as a database operation and routes to `/db-migration create`.

#### 3. Architecture Reference

Provides quick reference for Galaxy structure:
- **Backend:** FastAPI routers, Pydantic schemas, SQLAlchemy models, Manager pattern
- **Frontend:** Vue 2.7 components, Pinia stores, TypeScript
- **Testing:** Unit tests, API tests, integration tests via `run_tests.sh`
- **Key directories:** `lib/galaxy/managers/`, `lib/galaxy/webapps/galaxy/api/`, `client/src/`, `test/`

**Why it's automatic:** The context skill ensures every interaction follows Galaxy conventions and uses the right specialized skills, preventing common mistakes like running pytest directly or reading massive auto-generated files.

### Database Migration Skill

**Covers:**
- Alembic basics (gxy and tsi branches)
- Creating new revisions
- Migration utilities (create_table, add_column, etc.)
- Upgrade/downgrade operations
- Status checking
- Common troubleshooting scenarios

**Reference files:**
- Migration utilities: `lib/galaxy/model/migrations/util.py`
- Example migrations: `lib/galaxy/model/migrations/alembic/versions_gxy/`
- Models: `lib/galaxy/model/__init__.py`

### API Endpoint Skill

**Covers:**
- Finding similar endpoints as patterns
- Pydantic schema definition
- Manager pattern implementation
- FastAPI router creation
- Router registration
- Comprehensive test writing
- Manual testing and verification

**Includes:**
- Complete code examples
- Field type reference
- Manager best practices
- Router patterns
- Test patterns
- Common gotchas

### Testing Skill

**Covers:**
- Test runner usage (`./run_tests.sh`)
- Test type flags (-unit, -api, -integration)
- Unit test patterns (BaseTestCase)
- API test patterns (ApiTestCase)
- Integration test patterns (IntegrationTestCase)
- Populators (DatasetPopulator, WorkflowPopulator)
- Test decorators and fixtures

**Includes:**
- Complete test examples
- Base class reference
- Assertion methods
- Helper functions
- Common patterns
- Troubleshooting guide

### Galaxy Explorer Agent

**Capabilities:**
- Read-only codebase exploration
- Architecture-aware searches
- Pattern identification
- File and line number references
- Component relationship mapping
- Smart avoidance of large generated files

**Understands:**
- Manager pattern
- FastAPI router structure
- Pydantic schema organization
- SQLAlchemy model layout
- Test infrastructure
- Directory organization

## Contributing

This plugin is developed for the Galaxy Project community. Contributions welcome!

1. Fork the repository at https://github.com/arash77/galaxy-claude-marketplace
2. Create a feature branch
3. Make your changes in the `plugins/galaxy-dev/` directory
4. Test locally: `claude --plugin-dir ./plugins/galaxy-dev`
5. Submit a pull request

## License

MIT License - see LICENSE file for details

## Support

- GitHub Issues: https://github.com/arash77/galaxy-claude-marketplace/issues
- Galaxy Project: https://galaxyproject.org/
- Galaxy Gitter: https://gitter.im/galaxyproject/Lobby

## Version History

### 0.1.0 (Initial Release)
- Galaxy context skill (automatic) - Conventions, routing, and architecture reference
- Database migration skill
- API endpoint creation skill
- Testing skill with comprehensive patterns
- Galaxy codebase explorer agent
- Complete reference documentation

## Related Resources

- [Galaxy Project](https://galaxyproject.org/)
- [Galaxy GitHub](https://github.com/galaxyproject/galaxy)
- [Galaxy Development Docs](https://docs.galaxyproject.org/en/master/dev/)
- [Claude Code Documentation](https://docs.anthropic.com/claude-code)

---

Built with ❤️ for the Galaxy developer community
