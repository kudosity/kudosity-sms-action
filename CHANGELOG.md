# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.1] - 2026-05-22

Pre-Marketplace hardening pass — addresses external review. No functional changes for typical users.

### Security

- Explicitly mask the API key with `::add-mask::` so the runner's log scrubber redacts it even if the API ever echoes it back in an error body.
- Strip newlines from `message-id` / `status` outputs to prevent `$GITHUB_OUTPUT` injection from a malicious API response.
- Self-test workflow's Report step now passes outputs via `env:` instead of `${{ }}` interpolation in the shell `run:` block.

### Reliability

- Added `--connect-timeout 10` and `--max-time 30` to curl — prevents the step from hanging up to the job's 6h default timeout if the API stalls.
- Added an early `jq` availability check with a clear error message for self-hosted / custom runners.
- Added a `trap` to clean up the temp response file on exit.

### Changed

- `User-Agent` now reports the pinned action ref (`@v1`, `@v1.0.1`, or SHA) rather than a hardcoded version that drifts each release.

### Documentation

- README: documented runner requirements (Linux/macOS with `jq`).
- README: examples now reference `${{ secrets.KUDOSITY_SENDER }}` instead of a hardcoded sender number.
- README: documented "no automatic retry" with a pointer to a third-party retry wrapper.
- CHANGELOG: corrected v1.0.0 outputs list to include `status`.

## [1.0.0] - 2026-05-15

### Added

- `kudosity-sms` composite GitHub Action — sends a single SMS via the Kudosity V2 API (`https://api.transmitmessage.com/v2/sms`) using `x-api-key` authentication.
- Inputs: `api-key` (required), `to` (required), `from` (required), `message` (required), `message-ref` (optional), `track-links` (optional).
- Outputs: `message-id` (Kudosity message ID, UUID) and `status` (e.g. `queued`, `delivered`).
- Self-test workflow at `.github/workflows/test.yml` runs on push using `KUDOSITY_SANDBOX_API_KEY` + `KUDOSITY_SANDBOX_RECIPIENT` + `KUDOSITY_SANDBOX_SENDER` repo secrets.
- Example workflows: `examples/notify-on-deploy.yml`, `examples/alert-on-failure.yml`.
- `LICENSE` (MIT), `CHANGELOG.md`, `.gitignore`.
