# AmneziaWG split tunnel autostart verified

Date: 2026-07-20

## Result

The permanent AmneziaWG split tunnel was restarted successfully and autostart was enabled.

Confirmed:

- Service: `awg-quick@awg-egress.service`
- Active state: `active (exited)`
- Enabled state: `enabled`
- Interface: `awg-egress`
- Tunnel address: `10.76.106.92/32`
- Telegram networks routed through the tunnel:
  - `91.105.192.0/23`
  - `91.108.4.0/22`
  - `91.108.56.0/22`
  - `149.154.160.0/20`
- Default route remained direct through `eth0`.
- SSH client route remained direct through `eth0`.
- Configured VPN endpoint remained direct through `eth0`.
- Actual VPN peer endpoint remained direct through `eth0`.
- `core.telegram.org` returned HTTP 200 through the tunnel.
- Recent VPN handshake confirmed.
- Representative Telegram TCP endpoints on port 443 were reachable.
- No VPS reboot was performed.

## Current state

- Split tunnel: active
- Split tunnel autostart: enabled
- Full VPN tunnel: absent
- SSH route: direct
- Incoming TCP 443: still free for MTProto Proxy
- MTProto Proxy: not installed yet
- DNS for `prx.mailrhby.ru`: not changed yet

## Security note

Do not commit or publish:

- `/root/hidemy-awg.conf`
- `/etc/amnezia/amneziawg/awg-egress.conf`
- `/root/telegram-bootstrap/proxy-secret`
- `/root/telegram-bootstrap/proxy-config`

The next stage is selecting and installing the MTProto Proxy implementation on TCP 443, with outbound Telegram traffic using the established split tunnel.
