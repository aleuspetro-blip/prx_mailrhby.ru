# Split tunnel validation filename error

Date: 2026-07-20

## Result

The first permanent split-tunnel preparation attempt stopped safely before installing or starting the VPN service.

Observed error:

```text
awg-quick: The config file must be a valid interface name, followed by .conf
```

## Cause

The temporary validation file was created with `mktemp`, producing a basename that did not end in a valid interface name followed by `.conf`.

`awg-quick` validates the configuration path before parsing its contents. The configuration itself was not rejected.

## Safety state after rollback

- `awg-egress` interface absent
- default route unchanged via `eth0`
- VPN service inactive
- SSH route unchanged
- no permanent split-tunnel configuration installed by this attempt

## Fix

Use a temporary validation path with a valid name, for example:

```text
/etc/amnezia/amneziawg/awg-validate.conf
```

Then install the validated result as:

```text
/etc/amnezia/amneziawg/awg-egress.conf
```
