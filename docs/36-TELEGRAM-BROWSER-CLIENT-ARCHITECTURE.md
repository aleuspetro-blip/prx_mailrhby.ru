# Telegram client through VPS browser interface

Date: 2026-07-23

## Confirmed network result

The VPS egress tunnel `awg-egress` works and can reach Telegram networks. Local requests from the VPS through the tunnel succeed.

Several native Telegram Desktop proxy tests were performed from the client network to the VPS using:

- SOCKS5 with username/password on TCP 1080;
- SOCKS5 with username/password on TCP 8443;
- SOCKS5 without authentication on TCP 9443;
- an independent PowerShell SOCKS5 protocol test.

In every remote Telegram Desktop test, TCP connections reached the VPS, but Dante received no complete SOCKS5 CONNECT request and closed sessions after negotiation timeouts. This behavior persisted after changing ports and removing authentication. The likely blocking point is the client ISP/network path, which interferes with recognizable proxy protocols.

## Source code reviewed

Primary upstream repositories:

- Official Telegram Desktop: `telegramdesktop/tdesktop`
- Official Telegram Web K: `TelegramOrg/Telegram-web-k`
- KasmVNC browser remote desktop: `kasmtech/KasmVNC`

Telegram Desktop is open source under GPLv3 with the OpenSSL exception. Its built-in proxy transport types are SOCKS5, HTTP and MTProto. It does not include an embedded WireGuard or AmneziaWG VPN client.

## Decision

Do not fork Telegram Desktop to embed AmneziaWG.

Embedding a VPN client would require a Windows TUN driver, elevated privileges, route management, VPN lifecycle management, secure key storage, code signing and continuous rebasing on Telegram Desktop releases. This is a separate long-term desktop software project and is not justified for the current VPS.

Use a server-side Telegram Desktop session instead:

```text
User browser
    -> HTTPS prx.mailrhby.ru
    -> nginx reverse proxy
    -> KasmVNC local web session
    -> Telegram Desktop for Linux on VPS
    -> host Telegram routes
    -> awg-egress
    -> Telegram datacenters
```

The user network sees only ordinary HTTPS traffic to `prx.mailrhby.ru`. Telegram protocols originate on the VPS and therefore are not exposed to the client ISP.

## Components to preserve

- `awg-egress` and its provider configuration;
- host routes for Telegram IPv4 networks through `awg-egress`;
- nginx for the HTTPS browser endpoint;
- the valid Let's Encrypt certificate and certbot renewal timer;
- SSH access and normal host networking.

## Components to remove after a verified backup

- Teleproxy service and configuration;
- authenticated Dante SOCKS5 service and configuration;
- temporary no-auth Dante test service and configuration;
- incoming `awg-client` gateway and its dedicated forwarding/NAT rules if no longer required;
- temporary proxy diagnostic scripts, logs and generated connection files;
- obsolete proxy listeners on TCP 1080, 8443 and 9443.

Removal must not affect `awg-egress`, nginx, the certificate, SSH or the default route.

## Browser-client deployment requirements

- dedicated unprivileged Linux user for Telegram;
- Telegram Desktop Linux client from the official upstream distribution;
- lightweight desktop session;
- KasmVNC bound only to loopback;
- nginx reverse proxy on HTTPS;
- independent browser authentication in addition to the Telegram account login;
- persistent Telegram profile stored with mode 700;
- automatic start after boot without rebooting during deployment;
- no public VNC port;
- resource limits appropriate for a 2 vCPU / 2 GiB RAM VPS;
- verification that Telegram traffic increases counters on `awg-egress`;
- backup and rollback scripts for every persistent change.

## Implementation stages

1. Read-only preflight and inventory.
2. Back up the proxy experiment configuration.
3. Remove obsolete proxy services while preserving `awg-egress`, nginx and certbot.
4. Install the dedicated Linux desktop and KasmVNC session.
5. Install Telegram Desktop and verify its outbound route through `awg-egress`.
6. Publish the browser interface through nginx and the existing certificate.
7. Complete Telegram authorization from the browser session.
8. Record final service state, security controls and rollback procedure.
