# HTTPS backend ready

Date: 2026-07-20

## Result

The HTTPS camouflage backend for `prx.mailrhby.ru` is ready.

Confirmed:

- AmneziaWG split tunnel is active and enabled at boot.
- DNS A record resolves to `185.237.186.95` through the system resolver, Cloudflare, Google, and Quad9.
- nginx `1.24.0-2ubuntu7.14` is installed and enabled.
- Public HTTP on TCP 80 returns HTTP 200.
- Let's Encrypt certificate was successfully issued for `prx.mailrhby.ru`.
- Certificate validity: 2026-07-20 through 2026-10-18.
- Certificate renewal timer is enabled and active.
- Local TLS backend listens only on `127.0.0.1:8443`.
- TLS 1.3 negotiation succeeded with `TLS_AES_256_GCM_SHA384`.
- Certificate verification returned code 0.
- Public TCP 443 remains free for Teleproxy.

## Final port layout

- TCP 80 public: nginx / ACME and ordinary web page
- TCP 443 public: free, reserved for Teleproxy
- TCP 8443 loopback only: nginx TLS 1.3 backend

## Sensitive files

Do not commit or publish:

- `/etc/letsencrypt/live/prx.mailrhby.ru/privkey.pem`
- Teleproxy secret files or connection links

## Next step

Install a pinned Teleproxy release, configure public TCP 443 with Fake-TLS SNI `prx.mailrhby.ru`, and forward invalid or probe traffic to `127.0.0.1:8443`.
