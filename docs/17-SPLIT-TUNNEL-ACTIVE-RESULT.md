# Persistent AmneziaWG split tunnel — successful runtime test

Date: 2026-07-20

## Result

The permanent split-tunnel configuration was installed at:

`/etc/amnezia/amneziawg/awg-egress.conf`

Confirmed:

- Service `awg-quick@awg-egress.service` started successfully.
- Interface `awg-egress` is active with address `10.76.106.92/32`.
- Default route remains direct via `eth0`.
- SSH client route remains direct via `eth0`.
- VPN endpoint route remains direct via `eth0`.
- Telegram networks are routed through `awg-egress`:
  - `91.105.192.0/23`
  - `91.108.4.0/22`
  - `91.108.56.0/22`
  - `149.154.160.0/20`
- `core.telegram.org` returned HTTP 200 through the tunnel.
- AmneziaWG handshake was established.
- Representative Telegram endpoints on TCP 443 were reachable through the tunnel.
- Autostart is still disabled at this stage.

## Security and routing property

Only the selected Telegram networks use the Hide.mn VPN tunnel. General Internet traffic, SSH, package updates, GitHub, and future inbound MTProto proxy traffic continue to use the VPS public interface directly.

## Sensitive data

Do not commit or publish:

- `/root/hidemy-awg.conf`
- `/etc/amnezia/amneziawg/awg-egress.conf`
- VPN private/public keys
- obfuscation payload values

## Next step

Enable systemd autostart for `awg-quick@awg-egress.service`, restart the service without rebooting the VPS, and repeat route/handshake/connectivity verification.
