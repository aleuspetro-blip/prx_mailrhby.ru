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

This excluded DNS, public TCP reachability, nginx, certificate validation, AWG routing, Telegram MiddleProxy reachability, relay bootstrap files, and the NAT pair as the remaining cause.

## Tested parameter

The TOML configuration contained:

`workers = 1`

With a configured TLS domain, Teleproxy logged:

`It is recommended to not use workers with TLS-transport`

The Teleproxy source shows that a non-zero worker count forks a child worker process. The deployment therefore originally ran a parent and one child process.

The TOML parser uses `workers = -1` when the key is absent. Since the systemd command does not specify `-M`, removing the key leaves the runtime worker count at zero and runs Teleproxy as a single process.

## Completed change

The `workers = 1` line was removed successfully.

Confirmed after restart:

- Teleproxy active.
- Teleproxy autostart still disabled.
- Exactly one Teleproxy process.
- The worker warning disappeared.
- The `creating 1 workers` message disappeared.
- MiddleProxy relay preflight succeeded.
- Fake-TLS domain validation succeeded.
- `proxy-multi.conf` became active.
- Public HTTPS fallback returned HTTP 200.
- TCP 443 remained owned by Teleproxy.
- NAT pair remained `10.76.106.92:185.237.186.95`.
- AWG and nginx were not restarted.
- Systemd service hash was unchanged.
- Secret fingerprint and private connection-link hash were unchanged.

## Live test result

During the 120-second Telegram client attempt in single-process mode:

- Maximum inbound TCP 443 `ESTABLISHED` sockets: 62.
- Maximum MiddleProxy TCP 8888 `ESTABLISHED` sockets: 16.
- Maximum MiddleProxy TCP 8888 `SYN-SENT` sockets: 57.
- Inbound sessions were present for the full 120 seconds.
- Established relay sessions were present for the full 120 seconds.
- AWG received approximately 2.47 MB and sent approximately 4.96 MB.
- End-to-end TCP path `Telegram client -> Teleproxy -> MiddleProxy` was observed continuously.

Telegram still did not accept the proxy as available.

## Conclusion

The `workers = 1` setting was not the root cause. Single-process mode is retained because it removes Teleproxy's own TLS-transport warning and avoids an unnecessary child process, but the remaining failure is inside Teleproxy's Fake-TLS/RPC session handling rather than basic networking.

The next diagnostic stage is temporary verbosity level 2:

1. Preserve the current systemd service.
2. Add `--verbosity=2` temporarily.
3. Restart only `teleproxy.service`.
4. Capture one Telegram client attempt for 120 seconds.
5. Count RPC nonce processing, RPC readiness, proxy answers, forwarded answers, acknowledgements, close requests, dropped responses, and handshake errors.
6. Redact client IP and long hexadecimal values from the visible report.
7. Restore the original systemd service byte-for-byte.
8. Restart only `teleproxy.service` back to normal logging.
9. Keep single-process mode, NAT pair, secret, connection link, AWG, nginx, and routes unchanged.

## Safety constraints

- No VPS reboot.
- No AWG configuration or route changes.
- No nginx configuration changes or restart.
- NAT pair remains `10.76.106.92:185.237.186.95`.
- Existing secret and client link remain unchanged.
- Only `teleproxy.service` is restarted.
- Teleproxy autostart remains disabled until stable client operation is confirmed.
