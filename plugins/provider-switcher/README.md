# Provider Switcher

Switch Claude Code between different model providers (Anthropic, OpenRouter, Z.AI) with a single command.

## Installation

```
/plugin marketplace add proofofwork-agency/claude-plugins
/plugin install provider-switcher@proofofwork.agency
```

## Setup

Create a `.env` file in the root of your project with your API keys:

```env
# Anthropic (optional - Claude Code uses ~/.claude token by default)
ANTHROPIC_API_KEY=sk-ant-xxxxxxxxxxxxxxxxxxxxx

# OpenRouter
OPENROUTER_API_KEY=sk-or-xxxxxxxxxxxxxxxxxxxxx

# Z.AI
ZAI_API_KEY=xxxxxxxxxxxxxxxxxxxxx
```

Only add the keys for providers you plan to use.

## Commands

### `/switch-provider [provider]`

Switch to a different provider. If no provider is specified, you'll be prompted to choose.

```
/switch-provider openrouter
/switch-provider anthropic
/switch-provider zai
```

**What it does:**
- Reads the API key from your `.env` file
- Updates `.claude/settings.local.json` with the provider configuration
- Sets the appropriate base URL, auth token, and model mappings

**Restart Claude Code after switching for changes to take effect.**

### `/add-provider`

Add a custom provider to the catalog. Useful for self-hosted or alternative API endpoints.

```
/add-provider
```

You'll be prompted for:
- Provider name
- Base URL (must be Anthropic API compatible)
- API key environment variable name
- Model mappings for haiku/sonnet/opus tiers

## Supported Providers

| Provider | Base URL | Models |
|----------|----------|--------|
| anthropic | Native | claude-3.5-haiku, claude-3.5-sonnet, claude-3-opus |
| openrouter | openrouter.ai/api | mimo-v2-flash, grok-code-fast-1, deepseek-v3.2 |
| zai | api.z.ai/api/anthropic | glm-4.5-air, glm-4.7 |

## How It Works

1. Provider configurations are stored in `providers.json`
2. API keys are read from your project's `.env` file
3. Settings are written to `.claude/settings.local.json`
4. Claude Code reads these settings on startup

The plugin modifies these environment variables in your local settings:
- `ANTHROPIC_AUTH_TOKEN` - Your API key
- `ANTHROPIC_BASE_URL` - Provider endpoint
- `ANTHROPIC_API_KEY` - Set to empty string for proxy providers
- `ANTHROPIC_DEFAULT_HAIKU_MODEL` - Model for haiku tier
- `ANTHROPIC_DEFAULT_SONNET_MODEL` - Model for sonnet tier
- `ANTHROPIC_DEFAULT_OPUS_MODEL` - Model for opus tier

## License

MIT
