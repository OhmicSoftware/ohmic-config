# Ohmic Config

Remote configuration for [Ohmic](https://github.com/OhmicSoftware), an AI-assisted music production tool for Ableton Live.

The Ohmic app fetches these configuration files on startup (after EULA acceptance) to get the latest settings. This lets us update models, endpoints, and links without releasing a new version of the app.

## Files

| File | Purpose |
|------|---------|
| `models.json` | Model registry, API adapters, defaults, reasoning options, and output limits |
| `alpha.json` | Alpha build kill switch and minimum version gate |
| `endpoints.json` | API key validation endpoints and pricing fallback URLs |
| `links.json` | User-facing browser links |
| `system_prompt_links.json` | System prompt resource links |
| `mcp_access.json` | MCP Access client setup templates and compatibility pins |

## How it works

- Ohmic fetches the cacheable config files in parallel on launch (after EULA acceptance)
- If any fetch fails (offline, GitHub down, etc.), the app falls back to its built-in defaults
- No data leaves the user's machine until the EULA is accepted

## Contributing

Know about a new model that should be added? Open a PR!

1. Edit `models.json`.
2. Add the exact model ID as a key under the appropriate provider's `models` object.
3. Select one of Ohmic's shipped adapters with `api`, set a positive `max_output_tokens`, and add validated `reasoning` metadata only when the provider documents it.
4. Submit a pull request with links to the provider's official model and API documentation.

### Model registry v2

Static model records are keyed by exact provider model ID:

```json
"gpt-5.6-luna": {
  "api": "responses",
  "max_output_tokens": 128000,
  "reasoning": {
    "default": "none",
    "options": ["none", "low", "medium", "high", "xhigh", "max"]
  }
}
```

Allowed `api` values are provider-specific:

- OpenAI: `responses` or `chat_completions`
- Anthropic: `messages`
- Gemini: `generate_content`
- OpenRouter: provider-level `chat_completions`; its model list remains dynamically discovered

Configuration selects adapters already shipped in Ohmic. It cannot define executable behavior, request templates, imports, or arbitrary protocol implementations. A model using a new wire protocol requires an Ohmic release.

The `v1` branch remains for legacy Ohmic builds. New builds using keyed records and adapter metadata fetch `models.json` from `v2`; do not add v2 fields to `v1`.

### Guidelines

- Use the exact model ID accepted by the provider's API and cite official provider documentation
- Order models from most capable to most affordable
- Only add models that support text generation (no image-only, audio-only, or embedding models)
- Do not add deprecated or shut-down models
- Test the exact model ID and selected adapter before submitting

## License

This repository is public domain. The configuration data is factual information (API model identifiers, public URLs) and is not copyrightable.
