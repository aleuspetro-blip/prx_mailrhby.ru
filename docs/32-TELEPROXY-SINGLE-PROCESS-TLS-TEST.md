# Teleproxy single-process TLS test

Date: 2026-07-21

## Confirmed state before this stage

The corrected live probe established all of the following for the full 120-second observation window:

- Telegram client TCP connections reached Teleproxy on public TCP 443.
- Teleproxy established multiple TCP 8888 connections to Telegram MiddleProxy nodes.
- The relay connections used local AWG address `10.76.106.92`.
- AWG transferred several megabytes during the test.
- The corrected NAT pair remained active: `10.76.106.92:185.237.186.95`.
- Teleproxy, AWG, nginx, public HTTPS fallback, Fake-TLS domain parsing, relay configuration, secret, and connection link remained healthy.
- Telegram still reported `Server unavailable`.

This excludes DNS, public TCP reachability, nginx, certificate validation, AWG routing, Telegram MiddleProxy reachability, relay bootstrap files, and the NAT pair as the remaining cause.

## Remaining isolated parameter

The TOML configuration still contains:

`workers = 1`

With a configured TLS domain, Teleproxy logs:

`It is recommended to not use workers with TLS-transport`

The Teleproxy source shows that a non-zero worker count forks a child worker process. The current deployment therefore runs a parent and one child process.

The TOML parser uses `workers = -1` when the key is absent. Since the systemd command does not specify `-M`, removing the key leaves the runtime worker count at zero and runs Teleproxy as a single process.

## Approved change

Remove only:

`workers = 1`

Then:

1. Preserve the current configuration, systemd unit, secret, and private connection link.
2. Run the existing MiddleProxy relay preflight.
3. Restart only `teleproxy.service`.
4. Confirm exactly one Teleproxy process remains.
5. Confirm the TLS worker warning and `creating 1 workers` message disappear.
6. Confirm Fake-TLS parsing, MiddleProxy configuration, TCP 443, and HTTPS fallback remain healthy.
7. Observe one Telegram client connection attempt for 120 seconds using corrected socket parsing.
8. Keep the workers setting removed if server-health checks pass.
9. Keep Teleproxy autostart disabled until stable client operation is confirmed.

## Safety constraints

- No VPS reboot.
- No AWG configuration or route changes.
- No nginx configuration changes or restart.
- NAT pair remains `10.76.106.92:185.237.186.95`.
- Existing secret and client link remain unchanged.
- Only `teleproxy.service` is restarted.
- Automatic rollback restores the original TOML configuration on a server-health failure.
