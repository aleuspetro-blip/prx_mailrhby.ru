# Telegram bootstrap and cleanup result

Date: 2026-07-20

## Result

The temporary AmneziaWG egress tunnel worked successfully.

Confirmed:

- VPN handshake established.
- `https://core.telegram.org/getProxySecret` downloaded successfully through the tunnel.
- `https://core.telegram.org/getProxyConfig` downloaded successfully through the tunnel.
- `proxy-secret` saved with mode `0600`, size 128 bytes.
- `proxy-config` saved with mode `0600`, size 821 bytes.
- 19 Telegram IPv4 endpoints were extracted from the current proxy configuration.
- Temporary Telegram `/32` routes were removed.
- Temporary interface `awg-egress` was removed.
- Default route remained via `185.237.186.1` on `eth0`.
- SSH client route remained direct via `eth0`.
- No AmneziaWG interface remained active after cleanup.

## Current safe state

- Public VPS IPv4: `185.237.186.95`
- Main interface: `eth0`
- VPN interface: inactive
- Permanent split tunnel: not configured yet
- MTProto proxy: not installed yet
- Domain DNS: not changed yet

## Sensitive files

Never commit or publish these files:

- `/root/hidemy-awg.conf`
- `/root/telegram-bootstrap/proxy-secret`
- `/root/telegram-bootstrap/proxy-config`

Only checksums and non-secret operational facts may be documented.

## Next step

Create a persistent AmneziaWG split tunnel that routes only the Telegram endpoint networks through Hide.mn while preserving direct SSH and inbound proxy traffic on `185.237.186.95`.
