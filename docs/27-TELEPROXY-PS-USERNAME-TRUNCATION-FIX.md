# Teleproxy process username truncation fix

Date: 2026-07-20

## Observed result

Teleproxy passed the independent preflight, started successfully under systemd, and created two processes (master and worker). The validation script then stopped the service because `ps -eo user` displayed the account name `teleproxy` as the truncated form `telepro+`.

## Cause

The failure was in the validation script, not in Teleproxy. The script compared the formatted USER column from `ps` with the full string `teleproxy`. The formatted column may abbreviate long account names, so the comparison produced a false negative.

## Safe state after rollback

- `teleproxy.service`: inactive
- Teleproxy autostart: disabled
- public TCP 443: free
- AmneziaWG split tunnel: active
- default route: unchanged through eth0
- existing Teleproxy secret and configuration: preserved

## Correction

Validate process ownership using the numeric real UID from `/proc/<pid>/status`, compared with `id -u teleproxy`, instead of comparing a formatted `ps` username column.

The next test keeps Teleproxy autostart disabled and preserves the existing secret/configuration.
