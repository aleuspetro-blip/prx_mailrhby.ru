# MiddleProxy NAT info mismatch

Date: 2026-07-20

## Confirmed relay state

The switch from direct mode to Telegram MiddleProxy relay mode completed successfully:

- Teleproxy loaded `proxy-multi.conf`.
- Fake-TLS domain validation succeeded.
- Public TCP 443 remained available through Teleproxy.
- Persistent outbound connections to Telegram MiddleProxy nodes on TCP 8888 were established through `awg-egress`.
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

## Planned correction

1. Preserve the current relay configuration and systemd service.
2. Confirm the relay route source address is still `10.76.106.92`.
3. Replace only the existing `--nat-info` pair.
4. Run the existing relay preflight.
5. Restart only `teleproxy.service`.
6. Confirm Fake-TLS, MiddleProxy connections, HTTPS fallback, and public TCP 443 remain healthy.
7. Preserve the existing secret and connection link.
8. Keep Teleproxy autostart disabled until the Telegram client test succeeds.

## Safety constraints

- No VPS reboot.
- No changes to AWG `AllowedIPs`.
- No temporary diagnostic routes.
- Default route remains on `eth0`.
- SSH remains outside the VPN.
- nginx and AWG are not restarted.
- Automatic rollback restores the prior systemd service on failure.
