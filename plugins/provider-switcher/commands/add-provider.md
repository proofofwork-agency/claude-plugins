---
description: Add a custom provider in the providers.json catalog
---

# Add Provider

Add a new provider configuration to providers.json and store the API key in .env.

**Note:** This command does NOT allow overwriting existing providers. To modify an existing provider, edit the providers.json file directly.

Follow these steps EXACTLY. Do not skip any step.

---

## PHASE 1: GATHER INFORMATION

### Step 1.1: Collect all provider details

Use AskUserQuestion to collect these fields. Provide real example values as selectable options that users could actually use, or they can type their own via "Other".

**Required fields:**
1. **provider** - Unique provider name (kebab-case)
   - Options: `my-custom-api`, `local-llm`
2. **base_url** - Provider's Anthropic-compatible endpoint URL
   - Options: `https://api.example.com/v1`, `http://localhost:8080/v1`
3. **api_key_env** - Environment variable name for the API key
   - Options: `MY_PROVIDER_API_KEY`, `CUSTOM_API_KEY`
4. **api_key** - The actual API key value (will be stored in .env)
   - Options: `sk-example-key-123`, `key-abc-456` (user will type their real key in Other)

**Model configuration:**
5. **haiku_model** - Model ID for haiku tier
   - Options: `none`, `gpt-4o-mini`
6. **sonnet_model** - Model ID for sonnet tier
   - Options: `none`, `gpt-4o`
7. **opus_model** - Model ID for opus tier
   - Options: `none`, `gpt-4-turbo`

**Special options:**
8. **requires_empty_anthropic_key** - Does this provider require ANTHROPIC_API_KEY="" to prevent header conflicts?
   - Options: `Yes` (proxy services), `No` (direct APIs)

### Step 1.2: Confirm values

Output the collected values for confirmation:
```
PROVIDER CONFIG:
- Name: <provider>
- Base URL: <base_url>
- API Key Env: <api_key_env>
- API Key: <first 8 chars>...
- Requires Empty API Key: <yes/no>
- Haiku: <haiku_model or "none">
- Sonnet: <sonnet_model or "none">
- Opus: <opus_model or "none">
```

Ask the user to confirm before proceeding.

---

## PHASE 2: UPDATE PROVIDERS FILE

### Step 2.1: Read current providers

Read `${CLAUDE_PLUGIN_ROOT}/providers.json`.

If file doesn't exist, start with:
```json
{}
```

### Step 2.2: Check for existing provider

**IMPORTANT: Do NOT allow overwriting existing providers.**

If a provider with the same name already exists in providers.json:
```
Provider "<provider>" already exists. Cannot overwrite existing providers.

Existing providers: <list of provider names>

To modify an existing provider, manually edit the providers.json file
```
Then STOP.

### Step 2.3: Build provider entry

Create the provider entry with this structure:
```json
{
  "<provider>": {
    "base_url": "<base_url>",
    "api_key_env": "<api_key_env>",
    "requires_empty_anthropic_key": <true if yes, omit if no>,
    "default_models": {
      "haiku": <haiku_model or null>,
      "sonnet": <sonnet_model or null>,
      "opus": <opus_model or null>
    }
  }
}
```

Notes:
- If `requires_empty_anthropic_key` is "no", OMIT the field entirely
- If a model is "none", use `null` in JSON
- Preserve all existing providers in the file

### Step 2.4: Write providers file

Write the complete JSON to `${CLAUDE_PLUGIN_ROOT}/providers.json`

---

## PHASE 3: UPDATE .ENV FILE

### Step 3.1: Read current .env

Read `.env` from the project root.

If file doesn't exist, start with empty content.

### Step 3.2: Add or update API key

Check if `<api_key_env>=` already exists in .env:
- If YES: Replace that line with `<api_key_env>=<api_key>`
- If NO: Append `<api_key_env>=<api_key>` to the end of .env

### Step 3.3: Write .env file

Write the updated content to `.env`

---

## PHASE 4: VALIDATE AND REPORT

### Step 4.1: Read back and verify

Read both files and verify:
- `${CLAUDE_PLUGIN_ROOT}/providers.json` is valid JSON with the new provider
- `.env` contains the API key line

If validation fails, report the error and STOP.

### Step 4.2: Report success

```
Provider added: <provider>

Files updated:
- providers.json (provider config)
- .env (<api_key_env>)

To activate this provider:
  /switch-provider <provider>
```
