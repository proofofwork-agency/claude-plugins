---
description: Switch Claude Code to a different model provider (Anthropic, OpenRouter, Z.AI) by updating settings with API keys and endpoints. Use when the user wants to change providers, switch to a different API, or configure a new model endpoint.
---

# Switch Provider

Provider argument: **$ARGUMENTS**

Follow these steps EXACTLY.

---

## PHASE 1: DETERMINE PROVIDER

### Step 1.1: Check if provider argument is provided

If `$ARGUMENTS` is empty or blank, use AskUserQuestion to ask:

Question: "Which provider would you like to switch to?"
Options:
- anthropic: Official Anthropic API
- openrouter: OpenRouter.ai proxy
- zai: Z.AI API

Wait for user response before continuing.

### Step 1.2: Load provider config

Read `${CLAUDE_PLUGIN_ROOT}/providers.json`.

If the file doesn't exist:
```
providers.json not found
```
Then STOP.

### Step 1.3: Find and extract provider values

Find the provider object for the chosen provider name.

If not found, list available providers and STOP.

Note these values:
- base_url: (value or null)
- api_key_env: (the env var name)
- uses_native_token: (true if can use ~/.claude token, false/missing otherwise)
- requires_empty_anthropic_key: (true or false/missing)
- haiku: default_models.haiku (value or null)
- sonnet: default_models.sonnet (value or null)
- opus: default_models.opus (value or null)

---

## PHASE 2: READ API KEY

### Step 2.1: Ask about custom API key (for providers with uses_native_token)

**Only if `uses_native_token` is true (e.g., Anthropic):**

Use AskUserQuestion to ask:

Question: "Do you want to use a custom API key instead of the default ~/.claude token?"
Options:
- no: Use default ~/.claude token (Recommended)
- yes: Use a custom API key from .env

Store the answer as `use_custom_key` (true if "yes", false if "no").

**If user selects "no":**
- Set `api_key_value` to `null`
- Skip to Step 2.3

**If user selects "yes":**
- Continue to Step 2.2 to read the API key from .env

### Step 2.2: Read or prompt for API key

Read `.env` in the project root.

Find the line: `<api_key_env>=<value>`

Extract the value (remove surrounding quotes if present). Treat empty string as "not found".

**For providers with `uses_native_token: true` AND `use_custom_key` is false:**
- Skip this step entirely (already handled in Step 2.1)

**For providers with `uses_native_token: true` AND `use_custom_key` is true:**
- If .env doesn't exist or key is empty/not found:
  - Use AskUserQuestion to prompt: "Please enter your Anthropic API key (or leave blank to use default ~/.claude token instead):"
  - This should be a free-text input (user selects "Other" and types the key)
  - If user provides a key: use it as `api_key_value`
  - If user leaves blank or skips: set `use_custom_key` to false, `api_key_value` to null (fall back to defaults)

**For all other providers:**
- If .env doesn't exist or key not found:
```
API key not found. Add <api_key_env>=your-key to .env file
```
Then STOP.

### Step 2.3: Ask about model overrides (for providers with uses_native_token)

**Only if `uses_native_token` is true (e.g., Anthropic):**

Use AskUserQuestion to ask:

Question: "Do you want to override the default Anthropic models?"
Options:
- no: Use default Claude models (Recommended)
- yes: Specify custom model IDs

If user selects "yes", ask for each model tier they want to override (haiku, sonnet, opus).
Store these in `custom_haiku`, `custom_sonnet`, `custom_opus` (or null if not provided).

### Step 2.4: Output extracted values

```
PROVIDER CONFIGURATION:
- Provider: <name>
- API Key: <first 8 chars>... (or "Using ~/.claude token" if null for Anthropic)
- Custom Key: <yes if use_custom_key is true, else no (only shown for Anthropic)>
- Base URL: <value or "none">
- Empty API Key Mode: <yes if requires_empty_anthropic_key, else no>
- Haiku: <model or "default">
- Sonnet: <model or "default">
- Opus: <model or "default">
```

---

## PHASE 3: UPDATE SETTINGS

### Step 3.1: Read current settings

Read `.claude/settings.local.json`

If file doesn't exist, start with: `{}`

### Step 3.2: Build settings JSON

Preserve all existing keys (like `permissions`).

**CASE A: Provider with `uses_native_token: true` and `use_custom_key` is false (using ~/.claude token):**
- If user did NOT request model overrides:
  - **REMOVE the entire `env` section** from settings.local.json
  - This allows Claude Code to use its native token from ~/.claude folder
  - Only keep other sections like `permissions`
- If user requested model overrides:
  - Add only the model override env vars (no auth token needed):
    - `ANTHROPIC_DEFAULT_HAIKU_MODEL`: <custom_haiku> (OMIT if not specified)
    - `ANTHROPIC_DEFAULT_SONNET_MODEL`: <custom_sonnet> (OMIT if not specified)
    - `ANTHROPIC_DEFAULT_OPUS_MODEL`: <custom_opus> (OMIT if not specified)
  - **REMOVE** `ANTHROPIC_AUTH_TOKEN`, `ANTHROPIC_API_KEY`, and `ANTHROPIC_BASE_URL` if they exist

**CASE B: Provider with `uses_native_token: true` and `use_custom_key` is true (custom API key):**
- Only add `ANTHROPIC_AUTH_TOKEN`: <the API key from .env>
- If user requested model overrides:
  - `ANTHROPIC_DEFAULT_HAIKU_MODEL`: <custom_haiku> (OMIT if not specified)
  - `ANTHROPIC_DEFAULT_SONNET_MODEL`: <custom_sonnet> (OMIT if not specified)
  - `ANTHROPIC_DEFAULT_OPUS_MODEL`: <custom_opus> (OMIT if not specified)
- **REMOVE** `ANTHROPIC_API_KEY` if it exists (clean up from previous OpenRouter config)
- **REMOVE** `ANTHROPIC_BASE_URL` if it exists (not needed for official Anthropic API)

**CASE C: OpenRouter (requires_empty_anthropic_key is true):**
- `ANTHROPIC_API_KEY`: "" (empty string - REQUIRED to prevent Anthropic-specific headers)
- `ANTHROPIC_AUTH_TOKEN`: <the API key>
- `ANTHROPIC_BASE_URL`: <base_url>
- `ANTHROPIC_DEFAULT_HAIKU_MODEL`: <haiku> (OMIT if null/empty)
- `ANTHROPIC_DEFAULT_SONNET_MODEL`: <sonnet> (OMIT if null/empty)
- `ANTHROPIC_DEFAULT_OPUS_MODEL`: <opus> (OMIT if null/empty)

**CASE D: Other providers (Z.AI, etc):**
- `ANTHROPIC_AUTH_TOKEN`: <the API key>
- `ANTHROPIC_BASE_URL`: <base_url> (OMIT if null)
- `ANTHROPIC_DEFAULT_HAIKU_MODEL`: <haiku> (OMIT if null/empty)
- `ANTHROPIC_DEFAULT_SONNET_MODEL`: <sonnet> (OMIT if null/empty)
- `ANTHROPIC_DEFAULT_OPUS_MODEL`: <opus> (OMIT if null/empty)
- **REMOVE** `ANTHROPIC_API_KEY` if it exists (clean up from previous OpenRouter config)

### Step 3.3: Write settings file

Write the complete JSON to `.claude/settings.local.json`

---

## PHASE 4: VALIDATE AND REPORT

### Step 4.1: Read back and verify

Verify file is valid JSON and values match.

### Step 4.2: Report success

```
Provider switched to: <name>

Updated: <absolute_path>/.claude/settings.local.json

RESTART Claude Code for changes to take effect.
```
