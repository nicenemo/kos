# Development tooling

`playbooks/dev-setup.yml` installs a generic ESP32/Java development toolbox. It does not clone projects.

| Tool | Source | Use |
| --- | --- | --- |
| `pio` | `platformio-core` | build and upload firmware (PlatformIO) |
| PlatformIO udev rules | `platformio-core-udev` | access to common boards |
| `esptool` | `esptool` | flash and inspect ESP32 chips |
| `jbang` | `/opt/jbang`, linked in `/usr/local/bin` | run single-file Java tools; downloads the JDK a script asks for |
| JDK, CMake, Ninja | pacman | build prerequisites |

Projects may still keep PlatformIO inside the project. For nachtgespenst this means
`python3 -m venv .pio/venv && .pio/venv/bin/pip install platformio` in its own checkout; see that repository's
`docs/wiki/tooling.md`. The decision is recorded in [ADR 0009](../adr/0009-development-tooling-playbook.md).

The udev rules (serial adapters in group `uucp`, and `/dev/hidrawN` of the Nachtgespenst pad) are installed by
`udev-setup.yml`, see [ADR 0012](../adr/0012-separate-udev-playbooks.md). To add a board, find its USB IDs with
`lsusb`, add a line to `files/99-kos-esp32-serial.rules` and run `ansible-playbook playbooks/udev-setup.yml`.
