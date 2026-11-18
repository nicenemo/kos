# 9. Development Tooling as a Generic Playbook

* **Status:** Accepted
* **Date:** 2026-11-18

## Context

kos is used to build and flash ESP32 firmware (for example
[nachtgespenst](https://github.com/nicenemo/nachtgespenst), an M5Stack Core2/CoreS3 macro pad). That project builds
with PlatformIO and ships its helper tools as JBang scripts that download their own JDK. The machine setup should
provide the tools, not a particular project.

## Decision

* `dev-setup.yml` / `dev-teardown.yml` install `platformio-core` and `platformio-core-udev` from the official
  repositories, `esptool`, `jdk-openjdk`, CMake, Ninja, system-wide JBang in `/opt/jbang`, a udev rule file for the
  M5Stack serial adapters (`files/99-kos-esp32-udev.rules`) and put the user in `uucp`.
* The playbook does not clone any project and does not create project-local virtual environments; projects bootstrap
  themselves as their own documentation describes.

## Consequences

* **Positive:** the same playbook serves any PlatformIO or JBang project; the machine setup does not depend on
  another repository.
* **Negative:** JBang is downloaded from GitHub (`latest`) and is not pinned or updated by pacman. A new `uucp`
  membership only applies after logging in again.
