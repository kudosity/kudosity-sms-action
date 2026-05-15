# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-05-15

### Added

- `kudosity-sms` composite GitHub Action — sends a single SMS via the Kudosity V2 API (`https://api.transmitmessage.com/v2/sms`) using `x-api-key` authentication.
- Inputs: `api-key` (required), `to` (required), `from` (required), `message` (required), `message-ref` (optional), `track-links` (optional).
- Outputs: `message-id` — the Kudosity message ID returned by the API, for downstream verification or logging.
- Self-test workflow at `.github/workflows/test.yml` runs on push using `KUDOSITY_SANDBOX_API_KEY` + `KUDOSITY_SANDBOX_RECIPIENT` repo secrets.
- Example workflows: `examples/notify-on-deploy.yml`, `examples/alert-on-failure.yml`.
- `LICENSE` (MIT), `CHANGELOG.md`, `.gitignore`.
