# Teleproxy self-check backend/SNI fix

Date: 2026-07-20

## Observed result

The first Teleproxy setup attempt safely stopped before starting the proxy.

Successful checks:

- pinned Teleproxy v4.15.0 binary hash matched;
- AmneziaWG split tunnel active;
- nginx TLS backend returned HTTP 200;
- public TCP 443 remained free;
- all five production Telegram DCs were reachable through `awg-egress`;
- Teleproxy configuration parsed successfully;
- system user, private configuration and private connection file were created.

Failure:

- `teleproxy check --config /etc/teleproxy/config.toml` tested the configured backend `127.0.0.1:8443` using `127.0.0.1` as TLS SNI;
- nginx expects SNI `prx.mailrhby.ru`;
- the built-in TLS 1.3 validation therefore failed;
- rollback left TCP 443 free and did not affect the VPN.

## Root cause in Teleproxy v4.15.0

When the TOML domain entry has a separate backend, the v4.15.0 `check` command selects the backend string for its TLS probe. Its probe then uses that selected host as both connection target and ClientHello SNI.

With:

```toml
domain = [{ name = "prx.mailrhby.ru", backend = "127.0.0.1:8443" }]
```

`teleproxy check --config` probes `127.0.0.1:8443` with SNI `127.0.0.1`, which is not the intended production SNI.

## Safe fix

1. Add an idempotent local hosts entry:

```text
127.0.0.1 prx.mailrhby.ru
```

2. Keep the runtime TOML domain/backend mapping unchanged.
3. Run the self-check with an explicit CLI domain override:

```bash
teleproxy check --config /etc/teleproxy/config.toml -D prx.mailrhby.ru:8443
```

The CLI domain takes precedence for the check, resolves locally to `127.0.0.1`, connects to port 8443, and sends the correct SNI `prx.mailrhby.ru`.

4. Use the same explicit `-D` override in systemd `ExecStartPre`.

## Security state

- Server secret remains only in `/etc/teleproxy/config.toml` and `/root/teleproxy-connection.txt`.
- Secret values are never committed to GitHub.
- Teleproxy autostart remains disabled until the first complete live test succeeds.
