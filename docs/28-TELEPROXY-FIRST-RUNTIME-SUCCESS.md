# Teleproxy first runtime validation

Date: 2026-07-20

## Result

Teleproxy v4.15.0 successfully started with the production Fake-TLS configuration.

Confirmed state:

- systemd service `teleproxy.service` is active;
- autostart remains disabled pending client-side Telegram validation;
- main and worker processes run as system user `teleproxy` (UID 999, GID 988);
- Teleproxy listens only on public TCP 443;
- HTTPS fallback through Teleproxy returns HTTP 200 from the local nginx TLS 1.3 backend;
- certificate validation succeeds for `prx.mailrhby.ru`;
- all five built-in Telegram production DC IPv4 addresses route through `awg-egress`;
- default route remains on `eth0`;
- AmneziaWG split tunnel remains active and enabled;
- nginx remains active.

## Validation details

The independent preflight passed:

- Teleproxy configuration: OK;
- DC1 through DC5 connectivity: OK;
- clock drift: 0.0 seconds;
- 7 passed, 0 failed, 0 warnings.

Runtime validation passed:

- process ownership verification: SUCCESS;
- listener layout: only TCP 443;
- HTTPS fallback: SUCCESS;
- TLS version: TLS 1.3;
- certificate verify return code: 0.

## Known startup warnings

Teleproxy logs a TLS response parsing warning for the nginx backend and falls back to default response settings. This warning is related to Teleproxy's internal TLS response parser; real TLS 1.3 and the public HTTPS fallback were independently verified successfully with OpenSSL and curl.

The logs also show informational `getifaddrs(): Address family not supported by protocol` messages under the current hardened systemd address-family restrictions. The service nevertheless starts and operates correctly.

## Security notes

The private MTProto server secret and client connection links remain stored only in:

`/root/teleproxy-connection.txt`

They are not committed to GitHub.

## Next stage

Validate the private connection link in a Telegram client. Only after successful client validation should `teleproxy.service` autostart be enabled.
