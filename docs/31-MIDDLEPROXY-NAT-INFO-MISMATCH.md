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

The Telegram client still reported that the proxy was unavailable before the NAT correction.

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

## False rollback identified

The corrected pair was first applied and then rolled back because an earlier validation script required at least two established MiddleProxy sockets immediately after startup.

That requirement was invalid. Teleproxy may create relay sockets only when client traffic requires them. The source implementation of background outbound creation returns without creating connections, and a later post-rollback diagnostic confirmed that client activity produces both inbound TCP 443 sessions and established MiddleProxy TCP 8888 sessions.

## Live observation parser bug

The first 120-second live observation after the final NAT correction used commands in this form:

`ss -H -tanp state established`

When an explicit state filter is used, `ss` omits the state column from its output. The script still parsed the output as if the state column were present. Therefore these reported values were invalid:

- `Maximum inbound ESTABLISHED sockets`
- `Maximum relay ESTABLISHED sockets`
- `Maximum relay SYN-SENT sockets`
- the derived end-to-end path result

The AWG byte deltas from that run remain real, but they do not independently prove which Teleproxy sockets carried the traffic.

## Corrected validation plan

1. Keep the active NAT pair `10.76.106.92:185.237.186.95` unchanged.
2. Do not restart Teleproxy.
3. Do not change AWG, nginx, workers, or secrets.
4. Observe one Telegram client attempt for 120 seconds.
5. Sample `ss -H -tanp` without an explicit state filter so the state remains column 1.
6. Count inbound TCP 443 and MiddleProxy TCP 8888 sockets from the correct columns.
7. Sample four times per second to capture short-lived relay sockets.
8. Record AWG transfer deltas and new Teleproxy journal lines.
9. Redact the client IP and never print the secret.
10. Keep Teleproxy autostart disabled until the client reports a stable connection.

## Workers warning

The current TOML still contains:

`workers = 1`

Teleproxy logs:

`It is recommended to not use workers with TLS-transport`

This setting is not being changed together with the NAT correction. It will be evaluated only after the corrected live observation, so that one variable is changed at a time.

## Safety constraints

- No VPS reboot.
- No changes to AWG `AllowedIPs`.
- No temporary diagnostic routes.
- Default route remains on `eth0`.
- SSH remains outside the VPN.
- nginx and AWG are not restarted.
- Existing secret and connection link remain unchanged.
- Teleproxy autostart remains disabled pending stable client confirmation.
