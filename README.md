# Kudosity SMS — GitHub Action

Send an SMS from a GitHub Actions workflow using the [Kudosity](https://kudosity.com) messaging platform.

Drop this into any workflow as a step — `on: push`, `on: schedule`, on deploy success, on a failed build — and it will deliver an SMS via Kudosity's V2 API.

## Runner requirements

Requires a Linux or macOS runner with `jq` and `bash` available — the default `ubuntu-latest` and `macos-latest` runners qualify out of the box. Windows runners are not supported.

## Usage

```yaml
- uses: kudosity/kudosity-sms-action@v1
  with:
    api-key: ${{ secrets.KUDOSITY_API_KEY }}
    to: '0491570156'
    from: ${{ secrets.KUDOSITY_SENDER }}
    message: "Deploy ${{ github.sha }} shipped"
```

## Inputs

| Name | Required | Description |
|------|----------|-------------|
| `api-key` | ✅ | Kudosity API key from **Developers → API Settings**. Store as a repository secret. |
| `to` | ✅ | Recipient phone number — local (e.g. `0491570156`) or E.164 international (e.g. `61491570156`). Must be in the same country as the `from` sender. |
| `from` | ✅ | Sender number assigned to your Kudosity account, or an alphanumeric sender ID (max 11 chars, letters/numbers/`_`/`-`/space). |
| `message` | ✅ | SMS body text. Use `[opt-out-link]` for an opt-out URL if required. |
| `message-ref` | optional | Reference (max 500 chars) returned in delivery webhooks. Useful for correlating sends with later events. |
| `track-links` | optional | `'true'` to shorten and track links in the body. Defaults to `'false'`. |

## Outputs

| Name | Description |
|------|-------------|
| `message-id` | Kudosity message ID (UUID). Use for downstream queries to `/v2/sms/{id}`. |
| `status` | Message status returned by Kudosity (e.g. `queued`, `delivered`). |

## Setup

1. **Sign up** at [kudosity.com/signup](https://kudosity.com/signup) (free).
2. **Get an API key** — Developers → API Settings.
3. **Add it as a repo secret** — your GitHub repo → Settings → Secrets and variables → Actions → New repository secret. Name: `KUDOSITY_API_KEY`. Value: your API key.
4. **(Optional) Store the sender ID** — add another secret named `KUDOSITY_SENDER` for the number/sender ID you send from, and reference it as `${{ secrets.KUDOSITY_SENDER }}` in the workflow `from:` input. Keeps sender IDs out of the workflow YAML and lets you rotate them centrally.
5. **(Optional) Store the on-call number** — same pattern: add `ONCALL_PHONE` (or whatever name suits you) and reference it as `${{ secrets.ONCALL_PHONE }}` in the `to:` input.

## Examples

### Notify on deploy

```yaml
name: Deploy
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      - name: Deploy
        run: ./scripts/deploy.sh
      - name: Notify by SMS
        uses: kudosity/kudosity-sms-action@v1
        with:
          api-key: ${{ secrets.KUDOSITY_API_KEY }}
          to: ${{ secrets.ONCALL_PHONE }}
          from: ${{ secrets.KUDOSITY_SENDER }}
          message: "Deploy ${{ github.sha }} of ${{ github.repository }} shipped to prod."
```

See [`examples/notify-on-deploy.yml`](examples/notify-on-deploy.yml) for a copy-paste-ready version.

### Alert on failure

```yaml
- name: Alert on failure
  if: failure()
  uses: kudosity/kudosity-sms-action@v1
  with:
    api-key: ${{ secrets.KUDOSITY_API_KEY }}
    to: ${{ secrets.ONCALL_PHONE }}
    from: ${{ secrets.KUDOSITY_SENDER }}
    message: "ALERT: ${{ github.workflow }} failed in ${{ github.repository }} (run ${{ github.run_id }})"
    message-ref: "alert-${{ github.run_id }}"
```

See [`examples/alert-on-failure.yml`](examples/alert-on-failure.yml) for the full scheduled-job version.

### Using the outputs

```yaml
- name: Send SMS
  id: sms
  uses: kudosity/kudosity-sms-action@v1
  with:
    api-key: ${{ secrets.KUDOSITY_API_KEY }}
    to: ${{ secrets.ONCALL_PHONE }}
    from: ${{ secrets.KUDOSITY_SENDER }}
    message: 'Hello from CI'

- name: Log the message id
  run: echo "Kudosity message id was ${{ steps.sms.outputs.message-id }}"
```

## API choice

This Action targets the **Kudosity V2 SMS API** (`https://api.transmitmessage.com/v2/sms`) using `x-api-key` authentication. V2 is the modern path — JSON body, simpler auth, no separate secret required.

V1-only operations (list-based sends, scheduling, multi-recipient comma-separated sends) are intentionally not exposed here. The audience for this Action is CI/CD pipelines sending a single notification SMS; multi-recipient and list workflows are better handled through the Kudosity dashboard or the V1 API directly.

If you need richer messaging logic from an IDE, see the related plugins below.

## Related Kudosity integrations

- **Claude Code**: [`kudosity-claude-sms-plugin`](https://github.com/kudosity/kudosity-claude-sms-plugin)
- **Gemini CLI**: [`gemini-kudosity-sms-extension`](https://github.com/kudosity/gemini-kudosity-sms-extension)
- **GitHub Copilot (VS Code)**: [`kudosity-copilot-extension`](https://github.com/kudosity/kudosity-copilot-extension)

## Security

- Always store the API key as a **repository (or organization / environment) secret**. Never inline it in workflow YAML.
- The Action body never echoes the API key, and explicitly registers it with `::add-mask::` so the runner's log scrubber will redact it from any output (including response bodies logged on non-2xx HTTP errors).
- The Action runs entirely on the runner — no third-party servers are involved beyond the Kudosity API.

## Troubleshooting

### `HTTP 401`

Your API key is wrong, expired, or scoped to a different environment. Re-copy from Developers → API Settings in your Kudosity account.

### `HTTP 400 — sender/recipient invalid`

Sender must be a number assigned to your account, or an approved alphanumeric sender (max 11 chars, no consecutive spaces/hyphens). Recipient must be in the same country as the sender unless you have a global routing arrangement — see Kudosity's [Global SMS delivery list](https://support.transmitsms.com/s/article/page-products-connectivity-global-delivery-list).

### Self-test workflow is skipped

The bundled `.github/workflows/test.yml` requires three repo secrets to be set: `KUDOSITY_SANDBOX_API_KEY`, `KUDOSITY_SANDBOX_RECIPIENT`, `KUDOSITY_SANDBOX_SENDER`. Without them the workflow logs a warning and skips the live send.

### No automatic retry

The Action does not retry on transient failures (5xx, network errors) — a single failed call fails the step. If your use case needs retries, wrap the step with [`nick-fields/retry@v3`](https://github.com/nick-fields/retry) or similar. The connect timeout is 10s and the overall request timeout is 30s.

## Versioning

This Action follows [Semantic Versioning](https://semver.org/). Pin to `@v1` to get patch-level updates automatically, or pin to a specific SHA / tag (`@v1.0.0`) if you need exact reproducibility.

## License

MIT — see [LICENSE](LICENSE).
