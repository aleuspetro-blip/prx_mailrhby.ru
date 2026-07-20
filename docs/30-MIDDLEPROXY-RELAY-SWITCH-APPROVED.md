# MiddleProxy relay switch approved

Date: 2026-07-20

## Approval

The user explicitly approved switching Teleproxy from direct-to-DC mode to Telegram MiddleProxy relay mode.

## Reason

After the TLS cipher compatibility fix, the Telegram client connected successfully but later disconnected and could not reconnect. This confirms that:

- DNS works.
- TCP 443 works.
- Fake-TLS secret works.
- nginx camouflage works.
- The remaining instability is in the direct-to-DC operating mode.

Teleproxy documentation describes direct mode as bypassing Telegram MiddleProxy relays. The standard relay mode uses `proxy-secret` and `proxy-multi.conf` and is the selected project architecture.

## Planned change

1. Validate the existing Telegram bootstrap files and their known SHA-256 hashes.
2. Copy them into `/etc/teleproxy/relay/` with restricted permissions.
3. Verify every known relay IP is routed through `awg-egress`.
4. Back up the current Teleproxy configuration, preflight script, and systemd service.
5. Remove only `direct = true` from the Teleproxy TOML configuration.
6. Replace the preflight script with a relay-aware version.
7. Change `ExecStart` to include:
   - `--allow-skip-dh`
   - `--nat-info 185.237.186.95:185.237.186.95`
   - `--aes-pwd /etc/teleproxy/relay/proxy-secret`
   - `/etc/teleproxy/relay/proxy-multi.conf`
8. Restart only `teleproxy.service`.
9. Confirm TCP 443, nginx Fake-TLS fallback, AWG routing, and clean Teleproxy startup.
10. Keep Teleproxy autostart disabled until the Telegram client remains connected during a real client test.

## Safety guarantees

- Existing client secret and connection links are preserved.
- No VPS reboot.
- No nginx restart; only Teleproxy is restarted.
- AWG remains active and enabled.
- SSH and default route remain unchanged.
- Automatic rollback restores the previous direct-mode configuration on failure.
