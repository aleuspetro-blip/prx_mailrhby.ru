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

and were routed through `awg-egress`, where a VPN exit performs the external NAT.

Teleproxy defines `--nat-info` as:

`<local-addr>:<global-addr>`

for the RPC protocol handshake. The project's current pair therefore describes the VPS public interface instead of the actual AWG relay path.

## Planned correction

1. Preserve the current relay configuration and service.
2. Determine the real external IPv4 visible through `awg-egress` using temporary host routes only.
3. Confirm the local relay source address from the routing table.
4. Replace `--nat-info` with the actual AWG local and external IPv4 pair.
5. Restart only `teleproxy.service`.
6. Confirm relay connections and Fake-TLS remain healthy.
7. Preserve the existing secret and connection link.
8. Keep Teleproxy autostart disabled until the Telegram client test succeeds.

## Safety constraints

- No VPS reboot.
- Default route remains on `eth0`.
- SSH remains outside the VPN.
- Only temporary `/32` diagnostic routes are added through `awg-egress`, then removed.
- nginx and AWG are not restarted.
- Automatic rollback restores the prior systemd service on failure.
