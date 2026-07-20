# MTProto proxy implementation selection

Date: 2026-07-20

## Selected implementation

Recommended implementation: **Teleproxy** (`teleproxy/teleproxy`).

Target release reviewed: **v4.15.0** (published 2026-06-03).

No proxy software has been installed yet.

## Why Teleproxy

The project requires maximum practical resistance to current Russian DPI filtering while keeping the Telegram proxy simple for one private user.

Teleproxy currently provides:

- MTProto Fake-TLS mode.
- Automatic ClientHello fragmentation by advertising a 256-byte MSS on the proxy listener.
- JA4 fingerprint visibility in statistics and metrics.
- Active-probing fallback to a real HTTPS backend.
- Dynamic TLS record sizing.
- Anti-replay protection.
- Static Linux binaries and a systemd installation workflow.
- Support for Telegram middle-end mode, avoiding direct-mode functional limitations.

## Comparison notes

### Official Telegram MTProxy

The official implementation remains the protocol reference but has very limited recent release activity and lacks the newer anti-DPI mechanisms required by this project.

### Telemt

Telemt is actively maintained and feature-rich. However, its README dated 2026-06-05 explicitly states that the project was still analyzing a new JA4/JA4+ blocking wave. It remains a viable fallback, but Teleproxy currently documents and ships an automatic MSS-based ClientHello fragmentation mechanism designed for that blocking method.

### Teleproxy direct mode

Direct mode is not selected for the initial deployment because its own documentation notes feature limitations, including possible media delivery issues for non-Premium accounts and missing sponsored-channel delivery.

The initial deployment should use Telegram middle-end mode with the official `proxy-secret` and `proxy-config` files already downloaded through the AmneziaWG split tunnel.

## Planned ingress camouflage

For maximum active-probing resistance:

1. Point `prx.mailrhby.ru` to the VPS public IPv4.
2. Obtain a valid TLS certificate for `prx.mailrhby.ru`.
3. Run a small real HTTPS backend on loopback, for example `127.0.0.1:8443`.
4. Run Teleproxy on public TCP 443 in Fake-TLS mode.
5. Forward invalid or probing connections to the real HTTPS backend.
6. Keep Telegram upstream networks routed through `awg-egress`.

## Current prerequisites

Confirmed complete:

- AmneziaWG installed.
- Telegram split tunnel active.
- Split tunnel autostart enabled.
- Telegram HTTPS and representative TCP endpoints reachable through the tunnel.
- SSH/default route remain direct through `eth0`.
- TCP 443 remains available for the proxy.

Pending:

- User approval for Teleproxy installation.
- DNS A record for `prx.mailrhby.ru`.
- TLS certificate and local HTTPS camouflage backend.
- Teleproxy binary, configuration, service, secret, and client link.

## Secret handling

Never commit the proxy secret, full client link, VPN configuration, or Telegram bootstrap files to GitHub.
