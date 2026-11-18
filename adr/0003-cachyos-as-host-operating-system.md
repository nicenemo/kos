# 3. CachyOS as the Host Operating System

* **Status:** Accepted
* **Date:** 2026-11-18

## Context

kos is a ThinkPad T450s (Intel Core i7-5600U, HD Graphics 5500, 12 GB RAM, 512 GB SATA SSD, no discrete GPU). The same
owner runs CachyOS on mikmakmuk, samos and multiberry, and the playbooks for those machines are the starting point.

## Decision

Run CachyOS (Arch-based) with KDE Plasma. The i7-5600U supports AVX2, FMA and BMI2, so the `x86-64-v3` CachyOS
repositories and the optimized `linux-cachyos` kernel are usable. The root filesystem is btrfs (`@` subvolume,
`compress=zstd:1`, `noatime`) on LUKS.

## Consequences

* **Positive:** the playbooks share `pacman`, keyring handling and package names with the other machines.
* **Negative:** a rolling distribution on a laptop needs regular upgrades; `base-setup.yml` performs a full
  `pacman -Syu` on every run.
