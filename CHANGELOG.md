# Changelog

All notable changes to the kos playbooks. The project has no tagged releases, so entries are grouped by day and follow
the commits. The format loosely follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## 2026-11-25

### Fixed

* `.githooks/pre-push`: trufflehog runs with `--no-update`. The self-update cannot replace a root-owned install
  (`cannot move binary`) and the hook reported that error as a finding, which blocked every push.

### Added

* `hid-bpf-setup.yml` and `hid-bpf-teardown.yml`: the root side of the HID-BPF loader of the Nachtgespenst pad (BPF
  packages, udev rule `71-nachtgespenst-hid-bpf.rules`, systemd unit `nachtgespenst-hid-bpf@.service` with
  `CAP_BPF`, `CAP_PERFMON` and `CAP_SYS_ADMIN` only, loader directory `/opt/nachtgespenst-hid-bpf`); registered in
  `setup.yml` and `teardown.yml` ([ADR 0016](./adr/0016-hid-bpf-loader-as-udev-started-system-service.md)).

## 2026-11-19

### Added

* `bluez-setup.yml` and `bluez-teardown.yml`: short LE connection intervals (`MinConnectionInterval` 6,
  `MaxConnectionInterval` 9, latency 0, supervision timeout 400) in `/etc/bluetooth/main.conf`, restarts `bluetooth`;
  both registered in `setup.yml` and `teardown.yml` ([ADR 0015](./adr/0015-bluez-le-connection-parameters.md)).
* jdtls as a locally built pacman package (`jdtls-*` playbooks, [ADR 0013](./adr/0013-jdtls-as-locally-built-package.md)).
* Hugo, Jekyll, actionlint and osv-scanner; `ansible-lint`, `yamllint` and `markdownlint-cli2` in the pre-commit hook
  ([ADR 0014](./adr/0014-linters-in-the-pre-commit-hook.md)).
* This changelog, and the rules for keeping it, the README, the site and the ADRs up to date (`CLAUDE.md`).

### Changed

* `laptop-setup.yml`: performance governor on AC; USB autosuspend stays on except for Bluetooth controllers and the
  Nachtgespenst pad (`USB_EXCLUDE_BTUSB`, `USB_DENYLIST` 1209:0001), see
  [ADR 0007](./adr/0007-tlp-and-thermald-for-power-management.md).
* Plain strings in playbooks use single quotes.

### Branches

* `lamp-test-tuning` (not merged): test-only TLP drop-in without USB autosuspend and with the performance governor on
  battery, for hardware tests; removed again by `laptop-teardown.yml`.

## 2026-11-18

### Added

* First playbooks for the ThinkPad T450s: base, kernel, Intel graphics, laptop, Flatpak, Chrome, desktop applications,
  Docker and development tooling, each with a teardown.
* `controller-setup.yml` and `ansible-user-setup.yml`: dedicated ansible key and user over SSH on localhost
  ([ADR 0005](./adr/0005-ssh-on-localhost-with-ansible-user.md)).
* TLP and `thermald` for power management, battery charge thresholds, power-profiles-daemon masked
  ([ADR 0007](./adr/0007-tlp-and-thermald-for-power-management.md)).
* `udev-setup.yml` and `udev-teardown.yml` with rules for ESP32 serial adapters and the Nachtgespenst hidraw device
  ([ADR 0012](./adr/0012-separate-udev-playbooks.md)).
* ADRs, wiki, `AGENTS.md`, MIT licence, project page with GitHub Pages workflow, gitleaks pre-commit hook and trufflehog
  pre-push hook ([ADR 0011](./adr/0011-secret-scanning-with-pre-commit-hook.md)).

### Changed

* Boot loader is systemd-boot with `sdboot-manage` ([ADR 0004](./adr/0004-systemd-boot-with-sdboot-manage.md)).
* Kernel parameters come from `files/kos-tuning.conf`; options that `sdboot-manage` already has are not repeated and a
  pending reboot is reported.
* The nachtgespenst playbooks became the generic `dev-setup.yml` and `dev-teardown.yml`; `teardown.yml` mirrors
  `setup.yml` in reverse.
