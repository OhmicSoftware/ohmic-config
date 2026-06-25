# Ohmic Config

Remote configuration for [Ohmic](https://github.com/OhmicSoftware), an AI-assisted music production tool for Ableton Live.

The Ohmic app fetches these configuration files on startup (after EULA acceptance) to get the latest settings. This lets us update models, endpoints, and links without releasing a new version of the app.

## Files

| File | Purpose |
|------|---------|
| `models.json` | LLM provider/model registry, default models, max output tokens |
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

1. Edit `models.json`
2. Add the model ID to the appropriate provider's `models` array
3. Submit a pull request with the model name and a link to its documentation

### Guidelines

- Use the exact model ID as accepted by the provider's API
- Order models from most capable to most affordable
- Only add models that support text generation (no image-only, audio-only, or embedding models)
- Test that the model ID works before submitting

## License

This repository is public domain. The configuration data is factual information (API model identifiers, public URLs) and is not copyrightable.
