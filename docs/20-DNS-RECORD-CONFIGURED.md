# DNS record configured

Date: 2026-07-21

## Configured record

The DNS control panel shows the following record:

- Type: `A`
- Host: `prx.mailrhby.ru`
- Value: `185.237.186.95`
- TTL: `600`

## Status

The record has been entered in the authoritative DNS zone.

Runtime verification from the VPS and external resolvers is still required before certificate issuance.

## Next step

Verify propagation, install nginx and certbot, obtain a certificate for `prx.mailrhby.ru`, and configure nginx as a local TLS 1.3 backend on `127.0.0.1:8443` while keeping public TCP `443` free for Teleproxy.
