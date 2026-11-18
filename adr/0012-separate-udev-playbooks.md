# 12. Separate udev Playbooks

* **Status:** Accepted
* **Date:** 2026-11-18

## Context

[ADR 0009](./0009-development-tooling-playbook.md) put the udev rules for the ESP32 boards and the `uucp` membership
in `dev-setup.yml`. The rules are device access policy, not development tooling: they are needed to flash, test and
use boards even when none of the development packages is installed, and they change independently (a new board, a new
USB ID). The nachtgespenst repository also refers to `udev-setup.yml` / `udev-teardown.yml` in this repository for its
hidraw rule (its ADR 0015 and `docs/wiki/lamparray-testing.md`).

## Decision

* `udev-setup.yml` and `udev-teardown.yml` own all udev rules and the `uucp` membership. `dev-setup.yml` installs
  packages only (`platformio-core-udev` stays there because it is part of that package).
* Rule files live in `files/`: `99-kos-esp32-serial.rules` (CP210x, CH9102, CH340 and Espressif USB serial adapters,
  group `uucp`) and `70-nachtgespenst-hidraw.rules` (`/dev/hidrawN` of the Nachtgespenst pad, USB and Bluetooth, ID
  `1209:0001`, `uaccess`).
* After installing, the playbook reloads the rules and triggers the `tty`, `hidraw` and `usb` subsystems, so connected
  boards pick up the rules without re-plugging.

## Consequences

* **Positive:** a rule change is one small playbook run; the teardown removes exactly the rules this repository added.
* **Negative:** one more pair of playbooks to keep in both orchestrators. The `uucp` membership still needs a new login
  session, and the teardown removes the user from `uucp`, which other tools may also rely on.
