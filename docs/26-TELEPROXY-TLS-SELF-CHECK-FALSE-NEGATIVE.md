# Teleproxy TLS self-check false negative

Date: 2026-07-20

## Observed result

The corrected Teleproxy self-check still reported:

- all five production Telegram DC connections: OK
- clock synchronization: OK
- SNI/local mapping: OK
- TLS backend validation: FAIL

At the same time, independent checks against `127.0.0.1:8443` with SNI `prx.mailrhby.ru` confirmed:

- HTTP 200
- TLS 1.3 negotiated
- cipher `TLS_AES_256_GCM_SHA384`
- certificate verification return code 0
- valid Let's Encrypt certificate for `prx.mailrhby.ru`

## Root cause

Teleproxy v4.15.0 implements its own synthetic TLS ClientHello and parser in the `check` command. The generated probe and parser are stricter than a normal OpenSSL TLS 1.3 handshake and reject the valid nginx response with the internal message:

`TLS <= 1.2: expected x25519 as a chosen cipher`

This is a self-check false negative, not evidence that nginx or the certificate is invalid.

## Safe workaround

Do not use the TLS-domain portion of `teleproxy check` as the systemd start gate on this server.

Instead, use an independent preflight that:

1. verifies the main configuration and secret format;
2. checks all Telegram production DC routes through `awg-egress`;
3. runs `teleproxy check` on a temporary direct-mode configuration without a domain, so DC and clock checks still run;
4. verifies the nginx backend with OpenSSL TLS 1.3 and curl using SNI `prx.mailrhby.ru`;
5. starts the real Teleproxy configuration with its Fake-TLS domain/backend mapping;
6. verifies public HTTPS fallback through Teleproxy after startup.

## Safety state after failed check

- Teleproxy inactive
- TCP 443 free
- AmneziaWG active
- default route unchanged
- existing secret preserved
