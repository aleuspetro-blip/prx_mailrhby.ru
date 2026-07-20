# Hide.mn AmneziaWG connectivity test

Date: 2026-07-20

## Result

The temporary split-routing test succeeded.

- AmneziaWG interface `awg-egress` was created successfully.
- The default route remained on `eth0` via `185.237.186.1`.
- The current SSH client route remained on `eth0`.
- The Hide.mn peer completed a successful handshake.
- `core.telegram.org` returned HTTP 200 through the VPN.
- TCP 443 connected successfully to all tested Telegram addresses:
  - `149.154.167.99`
  - `149.154.175.50`
  - `149.154.175.100`
  - `91.105.192.100`

## Important operational note

The test command was pasted into an interactive shell. Because `trap cleanup EXIT` was registered in that shell, the `EXIT` handler does not run at the end of the pasted block; it runs only when the shell exits. The absence of the `CLEANUP` section means the temporary interface and routes may still be active.

Before continuing, explicitly remove the temporary routes/interface and clear the shell trap, or intentionally replace them with the persistent production configuration.

## Security

No private key, peer public key, obfuscation payload, or full provider configuration is stored in this repository.
