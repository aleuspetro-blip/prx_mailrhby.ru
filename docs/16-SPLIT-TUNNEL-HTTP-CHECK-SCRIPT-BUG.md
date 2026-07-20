# Split tunnel HTTP check script bug

Date: 2026-07-20

## What happened

The permanent split-tunnel configuration was created and started successfully.

Confirmed before rollback:

- `awg-egress` became active with address `10.76.106.92/32`.
- The default route remained on `eth0`.
- The Hide.mn endpoint remained routed through `eth0`.
- The SSH client route remained routed through `eth0`.
- All 19 Telegram addresses from the downloaded proxy configuration were routed through `awg-egress`.
- `core.telegram.org` was routed through `awg-egress`.
- HTTPS request to `core.telegram.org` returned `HTTP=200`.

The script nevertheless treated the successful response as a failure because of a fragile multiline string test.

## Safety result

The automatic rollback executed correctly:

- `awg-egress` was removed.
- No VPN interface remained active.
- The default route remained direct via `185.237.186.1` on `eth0`.
- SSH connectivity remained direct.

## Current state

- Permanent configuration file exists at `/etc/amnezia/amneziawg/awg-egress.conf`.
- Split tunnel service is inactive after rollback.
- Autostart is not enabled.

## Next step

Start the already-installed configuration again and validate the HTTP status using a single-value curl result instead of parsing multiline output.
