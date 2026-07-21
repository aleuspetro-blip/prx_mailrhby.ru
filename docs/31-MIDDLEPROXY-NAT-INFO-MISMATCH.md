# MiddleProxy NAT info mismatch

Date: 2026-07-20 to 2026-07-21

## Confirmed relay state

The switch from direct mode to Telegram MiddleProxy relay mode completed successfully:

- Teleproxy loaded `proxy-multi.conf`.
- Fake-TLS domain validation succeeded.
- Public TCP 443 remained available through Teleproxy.
- Outbound connections to Telegram MiddleProxy nodes on TCP 8888 were created through `awg-egress` during client activity.
- Several relay sockets reached `ESTABLISHED`; many others remained in `SYN-SENT` while Teleproxy tried available relay nodes.
- The existing client secret and connection links were preserved.

The Telegram client still reported that the proxy was unavailable.

## Mismatch found

The running systemd service used:

`--nat-info 185.237.186.95:185.237.186.95`

However, the actual outbound relay sockets used local address:

`10.76.106.92`

and were routed through `awg-egress`.

Teleproxy defines `--nat-info` as:

`<local-addr>:<global-addr>`

for the RPC protocol handshake.

The implementation stores each local/global pair and translates an address only when the local side exactly equals the address being translated. Therefore the current rule does not match the actual relay socket source address and is not applied.

## Corrected interpretation

The earlier plan to discover and advertise the VPN exit IPv4 was incorrect and was abandoned before any service change.

The local side must match the real Teleproxy relay socket source:

`10.76.106.92`

The global side remains the public, client-reachable VPS address:

`185.237.186.95`

The corrected pair is therefore:

`--nat-info 10.76.106.92:185.237.186.95`

## False rollback identified

The corrected pair was temporarily applied and Teleproxy passed all startup checks, but the script rolled back because it required at least two established MiddleProxy sockets immediately after startup.

That requirement was invalid. Teleproxy may create relay sockets only when client traffic requires them. The source implementation of background outbound creation returns without creating connections, and the later post-rollback diagnostic confirmed that client activity produces both inbound TCP 443 sessions and established MiddleProxy TCP 8888 sessions.

The rollback restored:

- NAT pair `185.237.186.95:185.237.186.95`;
- active Teleproxy service;
- disabled Teleproxy autostart;
- public TCP 443 listener;
- unchanged TOML configuration and private connection link.

## Final validation plan

1. Preserve the current relay configuration and systemd service.
2. Confirm the relay route source address is still `10.76.106.92`.
3. Replace only the existing `--nat-info` pair with `10.76.106.92:185.237.186.95`.
4. Run the existing relay preflight.
5. Restart only `teleproxy.service`.
6. Validate service state, public TCP 443, Fake-TLS domain parsing, HTTPS fallback, and private-file hashes.
7. Do not treat absence of idle MiddleProxy sockets as an error.
8. Observe one active Telegram client attempt for 120 seconds and record only connection counts and AWG byte deltas.
9. Preserve the corrected NAT pair if server health checks pass, regardless of whether idle relay sockets remain open.
10. Keep Teleproxy autostart disabled until the Telegram client reports a stable connection.

## Safety constraints

- No VPS reboot.
- No changes to AWG `AllowedIPs`.
- No temporary diagnostic routes.
- Default route remains on `eth0`.
- SSH remains outside the VPN.
- nginx and AWG are not restarted.
- Automatic rollback restores the prior systemd service only for server-health failures.
