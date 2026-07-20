# Teleproxy pinned installation plan

Date: 2026-07-20

## Selected release

- Repository: `teleproxy/teleproxy`
- Version: `v4.15.0`
- Asset: `teleproxy-linux-amd64`
- Installation path: `/usr/local/bin/teleproxy`

## Verification policy

The binary must not be installed until all checks pass:

1. Query the official GitHub Release API for tag `v4.15.0`.
2. Require an exact asset name match: `teleproxy-linux-amd64`.
3. Require the release asset to provide a `sha256:` digest.
4. Download from the asset's official `browser_download_url`.
5. Verify the downloaded binary SHA-256 against the GitHub release digest.
6. Confirm the file is an x86-64 Linux executable.
7. Inspect `--help` before creating a secret or service.
8. Confirm public TCP 443 remains free.

## Planned runtime architecture

- Public listener: `185.237.186.95:443`
- Fake-TLS SNI: `prx.mailrhby.ru`
- Camouflage backend: `127.0.0.1:8443`
- Public HTTP / ACME: nginx on TCP 80
- Telegram egress: AmneziaWG interface `awg-egress`
- Direct SSH and ordinary server traffic: `eth0`

## Secret policy

The server secret, generated EE client secret, and Telegram connection link must never be committed to GitHub.

## Current state

Teleproxy has not yet been installed or started. Public TCP 443 remains free.
