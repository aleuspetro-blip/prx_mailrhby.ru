# Teleproxy binary installation result

Date: 2026-07-20

## Result

Teleproxy `v4.15.0` was downloaded from the official GitHub release and installed successfully.

Confirmed:

- Release tag: `v4.15.0`
- Asset: `teleproxy-linux-amd64`
- Asset size: `14660032` bytes
- Expected SHA-256: `3c9812fca0808e34033fd8e92a3b6a009d1f6b496dac7bb007f1b35be7d1efe2`
- Downloaded SHA-256 matched the official GitHub Release digest.
- Installed path: `/usr/local/bin/teleproxy`
- Installed mode: `0755`
- Binary type: static Linux x86-64 ELF
- Version banner: `teleproxy-4.15.0`
- Required options confirmed: `-H`, `-S`, `-D`, `--direct`, and automatic ClientHello fragmentation support.

## Safe state after installation

- Teleproxy process: not running
- Teleproxy systemd service: not created
- Client secret: not generated
- Public TCP 443: free
- nginx HTTPS backend: active on `127.0.0.1:8443`
- AmneziaWG split tunnel: active and enabled

## Next step

Create a dedicated `teleproxy` system user, generate a private EE-mode secret, write a protected TOML configuration, run `teleproxy check`, create a hardened systemd unit, and start Teleproxy on TCP 443 without enabling autostart yet.

Secrets, connection links, and private configuration values must never be committed to GitHub.
