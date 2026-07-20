# Teleproxy binary verification: help exit status

Date: 2026-07-20

## Verified release

The official `teleproxy-linux-amd64` asset for Teleproxy `v4.15.0` was downloaded from the GitHub release and checked successfully.

- Release: `v4.15.0`
- Asset size: `14660032` bytes
- SHA-256: `3c9812fca0808e34033fd8e92a3b6a009d1f6b496dac7bb007f1b35be7d1efe2`
- Binary format: statically linked x86-64 ELF
- Embedded commit: `1c989687ef3f163026c102bb5a2eed497e8611ba`

## Script issue

The initial verification script treated any non-zero result from `teleproxy --help` as an error. This build prints the complete help output but exits with status `2`. Therefore the script stopped before installing the verified binary.

No proxy process or systemd service was created, no secret was generated, and public TCP port 443 remained free.

## Corrective action

A corrected script accepts help exit statuses `0` and `2`, still verifies the expected version banner and required command-line options, and installs the pinned binary only after release metadata, file size, SHA-256 and ELF architecture validation.

## Operating workflow

Long server commands will be supplied as downloadable `.sh` files. The user uploads each file through FTP/SFTP to `/root`, then runs it explicitly. Each script writes a corresponding log file under `/root` when appropriate.
