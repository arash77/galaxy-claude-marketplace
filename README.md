# Galaxy Claude Marketplace

Claude Code plugins for Galaxy Project development - all in one place!

## 🚀 Quick Install

**Add the marketplace:**
```bash
/plugin marketplace add arash77/galaxy-claude-marketplace
```

**Install the galaxy-dev plugin:**
```bash
/plugin install galaxy-dev@galaxy-claude-marketplace
```

That's it! All skills will be available: `/galaxy-db-migration`, `/galaxy-api-endpoint`, `/galaxy-testing` (plus automatic `galaxy-context`)

### Alternative: Use directly without marketplace

```bash
# Clone and use directly
git clone https://github.com/arash77/galaxy-claude-marketplace.git ~/galaxy-claude-marketplace
claude --plugin-dir ~/galaxy-claude-marketplace/plugins/galaxy-dev
```

## 📦 Available Plugins

### galaxy-dev

Galaxy development tools including:

**Skills:**
- `galaxy-context` (automatic) - Galaxy conventions, routing, and architecture reference (loads automatically)
- `/galaxy-db-migration [create|upgrade|downgrade|status|troubleshoot]` - Database migration workflows
- `/galaxy-api-endpoint [resource-name]` - Guide for creating new API endpoints
- `/galaxy-testing [run|write|unit|api|integration]` - Test running and writing guide

**Agent:**
- `galaxy-explorer` - Architecture-aware codebase exploration agent

**Location:** `plugins/galaxy-dev/`

## 📖 Usage Examples

### Database Migrations
```bash
/galaxy-db-migration create    # Create new migration
/galaxy-db-migration upgrade   # Upgrade database
/galaxy-db-migration status    # Check migration status
```

### API Endpoints
```bash
/galaxy-api-endpoint credentials    # Guide for creating credentials endpoint
/galaxy-api-endpoint               # Show general workflow
```

### Testing
```bash
/galaxy-testing run           # Show test running commands
/galaxy-testing api           # Guide for writing API tests
/galaxy-testing integration   # Guide for integration tests
```

## 🔄 Updating

Pull the latest changes:
```bash
cd ~/galaxy-claude-marketplace
git pull
```

Restart Claude to load updates.

## 🛠️ For Plugin Developers

### Adding a New Plugin

1. Create your plugin in `plugins/your-plugin-name/`
2. Add `.claude-plugin/plugin.json` manifest
3. Update `marketplace.json`:
   ```json
   {
     "name": "your-plugin-name",
     "path": "plugins/your-plugin-name",
     "version": "1.0.0",
     "description": "Your plugin description"
   }
   ```
4. Submit a pull request

### Plugin Structure

```
plugins/
└── your-plugin/
    ├── .claude-plugin/
    │   └── plugin.json
    ├── skills/
    │   └── skill-name/
    │       └── SKILL.md
    ├── agents/
    │   └── agent-name.md
    └── README.md
```

## 📁 Repository Structure

```
galaxy-claude-marketplace/
├── marketplace.json          # Plugin registry
├── README.md                # This file
└── plugins/                 # All plugins
    └── galaxy-dev/          # Galaxy development tools
        ├── .claude-plugin/
        ├── skills/
        ├── agents/
        └── README.md
```

## 🎯 Benefits

- ✅ **Single repository** - All plugins in one place
- ✅ **Simple installation** - Clone once, use forever
- ✅ **Easy updates** - `git pull` to get latest
- ✅ **No marketplace configuration** - Just point to plugin directory
- ✅ **Works offline** - Everything is local

## 📝 Usage

After adding the marketplace, you can:

**List available plugins:**
```bash
/plugin search galaxy
```

**Install plugins:**
```bash
/plugin install galaxy-dev@galaxy-claude-marketplace
```

**Update plugins:**
```bash
/plugin update galaxy-dev@galaxy-claude-marketplace
```

**Update marketplace:**
```bash
/plugin marketplace update galaxy-claude-marketplace
```

## 🤝 Contributing

1. Fork this repository
2. Create a feature branch
3. Add or update plugins in `plugins/` directory
4. Update `marketplace.json` if adding new plugin
5. Submit a pull request

## 📄 License

MIT License - see LICENSE file for details

## 🔗 Links

- Repository: https://github.com/arash77/galaxy-claude-marketplace
- Galaxy Project: https://galaxyproject.org/
- Galaxy GitHub: https://github.com/galaxyproject/galaxy

---

Built with ❤️ for the Galaxy developer community
