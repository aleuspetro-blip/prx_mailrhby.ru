# Teleproxy verbose RPC probe V2

Date: 2026-07-21

## Confirmed state before the probe

- Teleproxy is active in MiddleProxy relay mode.
- Teleproxy runs as one process; the `workers` TOML setting remains removed.
- Corrected NAT pair remains `10.76.106.92:185.237.186.95`.
- Teleproxy autostart remains disabled.
- AWG and nginx remain active and enabled.
- The existing server secret and Telegram connection link remain unchanged.

## First verbose-probe failure

The first `--verbosity=2` diagnostic script did not reach the live Telegram client test.

It failed while checking for the text `MiddleProxy relay preflight: SUCCESS` in only the last 300 journal lines. Verbosity level 2 generated hundreds of low-level socket and RPC lines almost immediately, so the early systemd preflight line had already been pushed outside that fixed tail.

This was a diagnostic-script false negative, not a Teleproxy runtime failure.

The cleanup completed successfully:

- the original systemd service was restored;
- temporary verbosity was removed;
- Teleproxy returned active;
- TCP 443 returned to Teleproxy;
- autostart remained disabled.

The startup fragment did confirm that TCP connections to MiddleProxy nodes were created and RPC nonce packets were accepted, but it did not capture the requested live client interval.

## V2 correction

The V2 probe:

1. requires the Telegram proxy entry to be removed before capture, stopping old automatic retries;
2. runs the independent relay preflight directly;
3. records a journald cursor before the temporary restart;
4. adds only `--verbosity=2` to the systemd command;
5. restarts only `teleproxy.service`;
6. verifies Fake-TLS startup and the main loop by searching all entries after the cursor rather than a fixed journal tail;
7. observes one Telegram client attempt for 90 seconds;
8. captures the complete verbose interval to a root-only file;
9. counts MTProto decoding, full RPC readiness, relay answers, ACKs, dropped queries, mapping failures and handshake errors;
10. automatically classifies the failing stage;
11. restores the original systemd service byte-for-byte and restarts Teleproxy with normal logging.

## Source-code basis

Teleproxy verbosity level 2 logs the key application stages needed for this diagnosis:

- decoded encrypted and unencrypted MTProto client packets;
- inability to select a ready relay target;
- `RPC_PROXY_ANS` responses;
- responses forwarded back to the Telegram client;
- simple acknowledgements;
- missing client mappings;
- full `Connected to RPC Middle-End` readiness.

## Safety constraints

- No VPS reboot.
- No AWG or nginx restart.
- No secret or connection-link regeneration.
- No persistent verbosity setting.
- Teleproxy autostart remains disabled.
- Raw verbose output is stored with mode 0600 and is not intended for publication.
