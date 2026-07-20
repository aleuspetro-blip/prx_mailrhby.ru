# Teleproxy client probe and TLS cipher root cause

Date: 2026-07-20

## Confirmed client-side symptom

Telegram client spends a long time in `Connecting`, then reports that the proxy server is unavailable.

## Server-side probe result

During a controlled 120-second client connection attempt:

- Teleproxy remained active.
- TCP 443 remained owned only by Teleproxy.
- The client reached the server.
- Maximum TCP sockets on port 443: 58.
- Maximum established TCP sockets: 54.
- AWG traffic increased significantly during the attempt.
- DNS, public port, HTTPS fallback, and Telegram DC routes were all working.

Therefore the failure is not caused by DNS, firewall, TCP reachability, certificate validity, or Telegram egress routing.

## Root cause found in Teleproxy v4.15.0 source

The Teleproxy TLS parser checks the chosen TLS 1.3 cipher suite at a fixed offset and requires bytes `13 01`, which corresponds to `TLS_AES_128_GCM_SHA256`.

The current nginx backend negotiates `TLS_AES_256_GCM_SHA384` (`13 02`). Teleproxy then logs:

- `Failed to parse upstream TLS response: TLS <= 1.2: expected x25519 as a chosen cipher`
- `Failed to update response data about prx.mailrhby.ru, so default response settings will be used`

The error text is misleading: the failing check is the cipher suite value, not X25519.

## Planned correction

Configure the local nginx TLS backend to allow only:

`TLS_AES_128_GCM_SHA256`

Then:

1. Validate nginx configuration.
2. Reload nginx without reboot.
3. Confirm the local backend negotiates TLS 1.3 with `TLS_AES_128_GCM_SHA256`.
4. Run Teleproxy self-check with the custom domain.
5. Restart only `teleproxy.service`.
6. Confirm the startup journal no longer contains the TLS parser failure.
7. Re-test the Telegram client.

## Safety constraints

- No VPS reboot.
- AWG service remains active.
- SSH and default route remain unchanged.
- Teleproxy autostart remains disabled until the Telegram client test succeeds.
- Existing Teleproxy secret and connection links are preserved.
- nginx configuration is backed up and automatically restored on failure.
