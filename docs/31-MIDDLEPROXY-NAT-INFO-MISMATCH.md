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

The Telegram client still reports that the proxy server is unavailable.

## Mismatch found

The running systemd service originally used:

`--nat-info 185.237.186.95:185.237.186.95`

However, the actual outbound relay sockets used local address:

`10.76.106.92`

and were routed through `awg-egress`.

Teleproxy defines `--nat-info` as:

`<local-addr>:<global-addr>`

for the RPC protocol handshake.

The implementation stores each local/global pair and translates an address only when the local side exactly equals the address being translated. Therefore the original rule did not match the actual relay socket source address and was not applied.

## Corrected interpretation

The earlier plan to discover and advertise the VPN exit IPv4 was incorrect and was abandoned before any service change.

The local side must match the real Teleproxy relay socket source:

`10.76.106.92`

The global side remains the public, client-reachable VPS address:

`185.237.186.95`

The corrected pair is therefore:

`--nat-info 10.76.106.92:185.237.186.95`

## Corrected NAT pair applied

On 2026-07-21 the corrected pair was applied successfully and remains active.

Confirmed after restart:

- Teleproxy active.
- Teleproxy autostart still disabled.
- AWG active and unchanged.
- nginx active.
- Fake-TLS domain check successful.
- `proxy-multi.conf` active.
- Public HTTPS fallback returned HTTP 200.
- TCP 443 remained owned by Teleproxy.
- TOML configuration hash unchanged.
- Private connection-link hash unchanged.
- Default route remained on `eth0`.

## Corrected live observation

A read-only 120-second probe sampled `ss -H -tanp` four times per second while the Telegram client attempted to connect.

Confirmed results:

- Maximum inbound established TCP 443 sockets: 60.
- Inbound established connections were observed for the full 120 seconds.
- Maximum established MiddleProxy TCP 8888 sockets: 16.
- Established relay connections were observed for the full 120 seconds.
- Maximum MiddleProxy sockets in `SYN-SENT`: 114.
- The first inbound client connection was observed within 0.25 seconds.
- The first established relay connection was observed within 0.25 seconds.
- AWG received traffic increased by 2,538,556 bytes.
- AWG sent traffic increased by 5,985,519 bytes.
- Multiple Telegram MiddleProxy addresses were simultaneously established through local source `10.76.106.92`.
- Teleproxy, nginx, AWG, TCP 443, default route, TOML config, systemd service, and private connection file remained unchanged throughout the probe.

The complete TCP path is therefore proven:

`Telegram client -> Teleproxy TCP 443 -> AWG -> Telegram MiddleProxy TCP 8888`

The Telegram client still reports `Server unavailable`. The remaining failure is above the TCP layer, inside Teleproxy/Fake-TLS/RPC session handling rather than DNS, routing, firewall, certificate, AWG, or MiddleProxy reachability.

## Workers warning and next isolated hypothesis

The current TOML contains:

`workers = 1`

At every startup Teleproxy logs:

`It is recommended to not use workers with TLS-transport`

The official Fake-TLS documentation shows a single-process example, while the general tuning documentation says one worker is the generic default. The explicit runtime warning is specific to TLS transport.

The next isolated test should:

1. Back up the current TOML configuration.
2. Remove only `workers = 1`.
3. Preserve the corrected NAT pair and every other setting.
4. Restart only `teleproxy.service`.
5. Confirm that the worker warning disappears and only one Teleproxy process handles TCP 443.
6. Repeat one Telegram client test.
7. Roll back automatically if server health checks fail.

This remains a hypothesis until the client test succeeds.

## Safety constraints

- No VPS reboot.
- No changes to AWG `AllowedIPs`.
- No temporary diagnostic routes.
- Default route remains on `eth0`.
- SSH remains outside the VPN.
- nginx and AWG are not restarted.
- Existing secret and Telegram connection link remain unchanged.
- Teleproxy autostart remains disabled until the Telegram client reports a stable connection.
