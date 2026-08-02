# Hermes Agent — Mistral AI Provider

A model provider plugin for [Hermes Agent](https://github.com/NousResearch/hermes-agent) that adds native Mistral AI API support.

Stars are appreciated! ⭐

## Installation

Copy into your Hermes plugins directory:

```bash
cp -r plugins ~/.hermes/
```

Or symlink for automatic updates:

```bash
ln -s $(pwd)/plugins/model-providers/mistral ~/.hermes/plugins/model-providers/mistral
```

## Setup

Set your Mistral API key in `~/.hermes/.env`:

```
MISTRAL_API_KEY=your-key-here
```

For an enterprise endpoint or a compatible proxy, optionally override the
default `https://api.mistral.ai/v1` endpoint:

```
MISTRAL_BASE_URL=https://mistral.example.com/v1
```

Then run `hermes model` and select **Mistral AI** from the provider list.

## Supported Models

| Model | Tool Calling | Vision | Reasoning |
|---|---|---|---|
| `mistral-large-latest` | ✅ | ✅ | ❌ (current) |
| `mistral-medium-latest` | ✅ | ✅ | ✅ |
| `mistral-small-latest` | ✅ | ✅ | ✅ |
| `codestral-latest` | ✅ | ❌ | ❌ |
| `pixtral-12b-latest` | ✅ | ✅ | ❌ |

## Features

- Standard OpenAI-compatible chat completions
- Tool/function calling (tested on Small, Medium, Large, Codestral, Ministral)
- Vision (base64 images)
- `reasoning_effort` pass-through via **version-threshold gating** — future date-stamped models (e.g. `mistral-small-2703`) automatically supported, no code changes needed
- 48 unit tests covering model gating, wire shape, metadata, and edge cases
- Auto-registers with Hermes provider discovery — just set `MISTRAL_API_KEY`

## Reasoning Gate Design

Rather than maintaining a static allowlist, the plugin uses a version-threshold heuristic:

- Extracts the 4-digit date code from model names (e.g. `2603` from `mistral-small-2603`)
- Compares against per-family thresholds: small >= 2603, medium >= 2604, large >= 2600
- Future models are automatically covered — no plugin updates needed

Known non-reasoning families (codestral, ministral, pixtral) are explicitly excluded. Magistral models (native reasoning) are also excluded since they reject the `reasoning_effort` parameter.

## Tests

```bash
cd /path/to/hermes-agent
python3 -m pytest tests/plugins/model_providers/test_mistral_profile.py -v
```

## PR

This plugin was contributed via [PR #65450](https://github.com/NousResearch/hermes-agent/pull/65450) to the upstream Hermes Agent repository.

## License

Same as Hermes Agent.
