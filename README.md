# Claude Plugins

Public plugin marketplace for Claude Code by [Proof of Work Agency](https://proofofwork.ventures/).

## Installation

Add this marketplace to Claude Code:

```
/plugin marketplace add proofofwork-agency/claude-plugins
```

## Available Plugins

### provider-switcher

Switch Claude Code between different model providers (Anthropic, OpenRouter, Z.AI) with a single command.

```
/plugin install provider-switcher@proofofwork.agency
```

**Commands:**
- `/switch-provider [provider]` - Switch to a different provider
- `/add-provider` - Add a custom provider to the catalog

[View documentation](./plugins/provider-switcher/README.md)

## Contributing

To add a new plugin:

1. Create a folder in `plugins/` with your plugin name
2. Add `.claude-plugin/plugin.json` with plugin metadata
3. Add your commands, hooks, or agents
4. Update `marketplace.json` with your plugin entry

## License

MIT
