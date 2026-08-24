# model-price-repo

Filtered model pricing data for CRS and sub2api projects.

> **This fork is maintained manually.** The scheduled GitHub Actions workflow was
> removed intentionally so upstream synchronization cannot overwrite local price
> overrides. Read [Manual pricing overrides](docs/manual-pricing-overrides.md)
> before changing prices or running the bundled synchronization scripts.

## How it works

Sub2API reads the two published files from the `main` branch:

1. `model_prices_and_context_window.json` contains the model prices.
2. `model_prices_and_context_window.sha256` contains the exact SHA-256 of the
   JSON file and lets consumers detect a change.

There is no active scheduled synchronization in this fork. Price changes are
made directly in the JSON file, the hash is regenerated, and both files are
committed together.

## Configuration

All settings live in [`config.json`](config.json):

| Field | Description |
|---|---|
| `upstream_url` | URL to the upstream litellm pricing JSON |
| `output_file` | Output filename (default: `model_prices_and_context_window.json`) |
| `hash_file` | SHA-256 hash filename for change detection |
| `sync_mode` | `"additive"` (only add new) or `"full"` (replace each run) |
| `update_existing` | Whether to update pricing data for models already in the output |
| `prefix_filters` | List of prefixes — a model key must start with one to be included |
| `exclude_patterns` | Substring patterns to exclude (applied before prefix matching) |
| `aliases` | Map alias model keys to existing source models (deep copy pricing) |
| `custom_models` | Manually defined pricing objects, always injected |

### Adding new model prefixes

Edit the `prefix_filters` array in `config.json`:

```json
{
  "prefix_filters": [
    "claude-",
    "gpt-",
    "your-new-prefix/"
  ]
}
```

### Adding aliases

Aliases create copies of an existing model's pricing under a new key:

```json
{
  "aliases": {
    "claude-opus-4-6-thinking": {
      "source": "claude-opus-4-6",
      "description": "Thinking variant, same pricing"
    }
  }
}
```

If the source model doesn't exist in the filtered data, the alias is skipped with a warning.

## Running locally

The original synchronization tooling remains in the repository for reference.
Do not run it for routine maintenance of this fork: current `config.json` rules
do not represent every manual override and can replace those overrides.

```bash
python3 scripts/sync_prices.py --config config.json --repo-root .
```

No pip dependencies — uses Python standard library only.

## CRS integration

Point CRS to the raw output file from this repo:

```
MODEL_PRICES_URL=https://raw.githubusercontent.com/<owner>/model-price-repo/main/model_prices_and_context_window.json
```

The output JSON structure is identical to what litellm produces (model key -> pricing object), so CRS `pricingService.js` works without changes.

## License

[MIT](LICENSE)
