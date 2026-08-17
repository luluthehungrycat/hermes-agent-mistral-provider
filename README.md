# Hermes Agent — Mistral AI Provider

A model-provider plugin for [Hermes Agent](https://github.com/NousResearch/hermes-agent) that adds native Mistral AI API support.

## Installation

This repository is now shaped as a standalone Hermes plugin: `plugin.yaml` and
`__init__.py` are at the repository root.

### Direct copy — works with current Hermes

Copy the plugin payload into the user model-provider directory:

```bash
mkdir -p "${HERMES_HOME:-$HOME/.hermes}/plugins/model-providers/mistral-provider"
cp __init__.py plugin.yaml \
  "${HERMES_HOME:-$HOME/.hermes}/plugins/model-providers/mistral-provider/"
```

If installing from the repository root, the source files are intentionally
root-level. The destination directory name may be different, but keeping it
aligned with `name: mistral-provider` is recommended.

### Symlink — recommended for a development clone

A symlink keeps the installed provider synchronized with the clone:

Run this from the repository root. The guard refuses to replace a real
installation directory, so migrating from a direct copy requires an explicit
manual removal rather than silently creating a nested symlink:

```bash
plugin_dir="${HERMES_HOME:-$HOME/.hermes}/plugins/model-providers/mistral-provider"
mkdir -p "$(dirname "$plugin_dir")"
if [ -e "$plugin_dir" ] && [ ! -L "$plugin_dir" ]; then
  printf 'Refusing to replace existing directory: %s\n' "$plugin_dir" >&2
  exit 1
fi
if [ -L "$plugin_dir" ]; then
  rm "$plugin_dir"
fi
ln -s "$(pwd)" "$plugin_dir"
```

The symlink target must be the repository root because the plugin payload is
now at the root. Remove only the symlink when uninstalling:

```bash
rm "${HERMES_HOME:-$HOME/.hermes}/plugins/model-providers/mistral-provider"
```

### `hermes plugins install`

The manifest correctly declares:

```yaml
kind: model-provider
```

However, current released/upstream Hermes still has an installer/discovery
mismatch: `hermes plugins install` installs Git repositories under
`$HERMES_HOME/plugins/<name>/`, while the model-provider registry discovers
providers under `$HERMES_HOME/plugins/model-providers/<name>/`. Therefore this
command can report a successful installation but does **not** currently make
this provider discoverable:

```bash
hermes plugins install luluthehungrycat/hermes-agent-mistral-provider
```

This repository is intentionally flattened, so no explicit subdirectory
argument is required or valid. The former nested path
`plugins/model-providers/mistral` has been removed; do not use it with this
repository. For repositories that still contain a nested plugin payload,
Hermes accepts an explicit suffix, but it does not change the generic install
destination and subdirectory installs have a separate update limitation.

Use the direct-copy or symlink method above until the upstream installer fix is
available. The relevant upstream tracking is:

- [Issue #76372](https://github.com/NousResearch/hermes-agent/issues/76372) —
  model-provider Git installs land in the wrong directory.
- [PR #76387](https://github.com/NousResearch/hermes-agent/pull/76387) — routes
  `kind: model-provider` installs into `model-providers/`.
- [PR #76554](https://github.com/NousResearch/hermes-agent/pull/76554) — an
  overlapping fix for the same issue.
- [PR #64277](https://github.com/NousResearch/hermes-agent/pull/64277) — broader
  standalone model-provider distribution support.
- [Issue #65314](https://github.com/NousResearch/hermes-agent/issues/65314) —
  subdirectory installs discard `.git`, so `hermes plugins update` cannot use
  the resulting directory as a Git checkout.

Once the routing fix is merged and included in the Hermes version in use, the
root-level repository can be installed with the normal command:

```bash
hermes plugins install luluthehungrycat/hermes-agent-mistral-provider
```

No repository-specific installer should be necessary.

## Setup

Set your Mistral API key in the Hermes profile environment:

```text
MISTRAL_API_KEY=your-key-here
```

For an enterprise endpoint or a compatible proxy, optionally set:

```text
MISTRAL_BASE_URL=https://mistral.example.com/v1
```

Then start a fresh Hermes process and run `hermes model` to select **Mistral
AI**.

## Supported Models

- `mistral-large-latest` — tool calling and vision
- `mistral-medium-latest` — tool calling, vision, and reasoning
- `mistral-small-latest` — tool calling, vision, and reasoning
- `codestral-latest` — tool calling
- `pixtral-12b-latest` — tool calling and vision

## Features

- Standard OpenAI-compatible chat completions
- Tool/function calling
- Vision through base64 images
- `reasoning_effort` pass-through with version-threshold gating
- Three independently selectable endpoint profiles: Global, EU, and US
- Global endpoint override through `MISTRAL_BASE_URL`
- Fixed EU and US regional endpoints that do not accept the generic override
- Automatic Hermes provider registration after the provider is placed in the
  model-provider discovery directory

## Regional inference profiles

The plugin adds three entries to Hermes' provider list:

- **Mistral AI (Global endpoint)** — default behavior and broadest coverage
- **Mistral AI (EU endpoint — +10% price)** — EU regional inference
- **Mistral AI (US endpoint — +10% price)** — US regional inference

All profiles use `MISTRAL_API_KEY`. The EU and US profiles use fixed regional
base URLs, so selecting one cannot silently fall back to Global. Regional
endpoints may have narrower feature or model coverage.

## Reasoning gate design

The plugin uses a version-threshold heuristic instead of a static allowlist:

- extracts the four-digit date code from model names;
- compares it with per-family thresholds;
- supports future date-stamped models without a plugin update.

Known non-reasoning families such as Codestral, Ministral, and Pixtral are
explicitly excluded. Magistral models are also excluded because they reject
the `reasoning_effort` parameter.

## Tests

The focused tests require a Hermes checkout or environment that provides
Hermes' runtime modules such as `providers` and `model_tools`:

```bash
PYTHONPATH=/path/to/hermes-agent \
  python3 -m pytest tests/plugins/model_providers/test_mistral_profile.py -v
```

For this checkout, the Hermes source is available at
`/home/hermes/.hermes/hermes-agent`, so the verified local command is:

```bash
PYTHONPATH=/home/hermes/.hermes/hermes-agent \
  uv run pytest --import-mode=importlib \
  tests/plugins/model_providers/test_mistral_profile.py -q
```

Standalone syntax and whitespace checks do not require the Hermes runtime:

```bash
python3 -m compileall -q .
git diff --check
```

## License

Same as Hermes Agent.
