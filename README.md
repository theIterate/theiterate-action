# theiterate — CI for prompts (GitHub Action)

Fail your build when a prompt change **quietly regresses**. On every pull request,
this Action runs a prompt's test set in [theiterate](https://theiterate.com),
compares the candidate's pass rate against the current **live** version, and exits
non-zero on a regression — so a bad prompt never ships.

## Setup

1. In theiterate, create a workspace **API key** (Settings → API keys).
2. Add it to your repo as a secret, e.g. `THEITERATE_API_KEY`.

## Usage

```yaml
# .github/workflows/prompts.yml
name: Prompt CI
on: pull_request

jobs:
  eval:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: theIterate/theiterate-action@v1
        with:
          api-key: ${{ secrets.THEITERATE_API_KEY }}
          prompt: support-classifier
          # Optional: evaluate a prompt file from THIS repo as the candidate
          # body-file: prompts/support-classifier.txt
          # Optional: also fail if pass rate is below an absolute floor
          # min-pass-rate: "0.9"
```

## What it does

- **No `body-file`** → re-evaluates the latest version stored in theiterate and
  gates it against the live baseline.
- **With `body-file`** → evaluates that file's contents as the candidate (saved as
  a new version), so the check tests the exact prompt in your PR.
- **Verdict** is `fail` when the candidate's pass rate drops below the live
  baseline (a regression) or below `min-pass-rate`. The step writes a summary to
  the job's **Summary** tab.

## Inputs

| Input | Required | Default | Notes |
|---|---|---|---|
| `api-key` | yes | — | Workspace key (`ti_live_…`), from a repo secret. |
| `prompt` | yes | — | The prompt name in theiterate. |
| `base-url` | no | `https://theiterate.com` | Override for self-hosted. |
| `body-file` | no | — | Path to a prompt file to evaluate as the candidate. |
| `min-pass-rate` | no | — | Absolute floor `0..1`. |

Requires `curl` and `jq` (present on GitHub's `ubuntu-latest`). The eval spends
LLM tokens against your plan budget, so it's rate-limited and admin-gated
(`ci_enabled`) on the theiterate side.

## Support

Questions, billing, and product bugs →
[GitHub Discussions](https://github.com/orgs/theIterate/discussions)
(our only contact channel). Fixes for this Action → open a PR here.
