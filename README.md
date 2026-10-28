# Ohmic Config

Remote configuration for [Ohmic](https://github.com/OhmicSoftware), an AI-assisted music production tool for Ableton Live.

Current Ohmic builds read one signed file, `config-release.json`, after startup. A user reviews and approves each update in Ohmic; it applies the next time Ohmic opens. This lets us update models, endpoints, and links without releasing a new version of the app.

## Files

| File | Purpose |
|------|---------|
| `config-release.json` | Signed release of `models.json`, `links.json` and `system_prompt_links.json`; the only file current builds download |
| `config_signing_public_key.pem` | Publisher public key, for reference only; Ohmic trusts the copy built into the app |
| `models.json` | Model registry, API adapters, defaults, reasoning options, and output limits |
| `alpha.json` | Alpha build kill switch and minimum version gate |
| `endpoints.json` | API key validation endpoints and pricing fallback URLs |
| `links.json` | User-facing browser links |
| `system_prompt_links.json` | System prompt resource links |
| `mcp_access.json` | MCP Access client setup templates and compatibility pins |

## How it works

- Each Ohmic release selects one version branch. Current builds use `v2`.
- `models.json`, `links.json` and `system_prompt_links.json` are signer inputs. Current builds never download them separately; they read only `config-release.json` and accept it only when its Ed25519 signature matches the public key built into Ohmic.
- Release numbers only increase. Ohmic rejects older releases and a reused number with different content.
- Offline or rejected checks leave the user's configuration unchanged. Ohmic ships a signed copy as its starting configuration.
- `alpha.json` stays on `main` for already distributed alpha builds.
- No data leaves the user's machine until the EULA is accepted.

## Publishing

1. Edit the source JSON files on `v2`.
2. From an Ohmic checkout, run `& .\tools\sign_config_release.ps1` and enter the passphrase at the prompt. It signs the next release number and verifies it. Every edit needs a new signed release; Ohmic ignores unsigned changes.
3. Commit the edited files and `config-release.json`, and push `v2` without force.

The private key never enters this repository.

## Contributing

Know about a new model that should be added? Open a PR!

1. Edit `models.json`. A maintainer signs and publishes accepted changes.
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
- Anthropic: `messages` or `messages_bound_thinking`
- Gemini: `generate_content`
- OpenRouter: provider-level `chat_completions`; its model list remains dynamically discovered

Configuration selects adapters already shipped in Ohmic. It cannot define executable behavior, request templates, imports, or arbitrary protocol implementations. A model using a new wire protocol requires an Ohmic release.

`messages_bound_thinking` opts documented compatible models such as
`claude-fable-5-1` into adaptive thinking with the fixed
`thinking-binding-controls-2026-08-01` beta header and
`block_binding.prefix_mismatch_behavior: drop_block`. Anthropic handles invalid
thinking when context changes; Ohmic preserves conversation history and tool-result
links. Ordinary `messages` and provider defaults stay unchanged.
Future compatible models can select this shipped mode without model-name rules.

The `v1` branch remains for legacy Ohmic builds. Older unsigned v2 builds are not supported by this signed release. Only `alpha.json` remains on `main`. Do not add v2 fields to `v1`. A future branch such as `v3` must carry the complete configuration bundle before an Ohmic release selects it.

### Guidelines

- Use the exact model ID accepted by the provider's API and cite official provider documentation
- Order models from most capable to most affordable
- Only add models that support text generation (no image-only, audio-only, or embedding models)
- Do not add deprecated or shut-down models
- Test the exact model ID and selected adapter before submitting

## License

This repository is public domain. The configuration data is factual information (API model identifiers, public URLs) and is not copyrightable.
