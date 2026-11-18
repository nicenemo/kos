# 7. TLP and thermald for Power Management

* **Status:** Accepted
* **Date:** 2026-11-18

## Context

mikmakmuk disables every power daemon and pins the performance governor; that fits always-on desktops, not a laptop.
On kos `power-profiles-daemon` (PPD) is masked and TLP and `thermald` already run. The CPU runs `intel_cpufreq` in
passive mode with `schedutil`, and the kernel exposes no `platform_profile`, so PPD would have little to control.
TLP and PPD conflict; TLP and thermald do not.

## Decision

* `laptop-setup.yml` installs and enables TLP and `thermald`, keeps PPD masked, and masks the `systemd-rfkill` units
  that TLP replaces.
* TLP battery charge thresholds for the internal battery (BAT0) are set from `battery_start_threshold` and
  `battery_stop_threshold` (75 and 80 percent) in `/etc/tlp.d/99-kos.conf`.
* The CPU governor is not forced to `performance`.
* `fstrim.timer` is enabled for the SSD.

## Consequences

* **Positive:** ThinkPad-aware battery care through `thinkpad_acpi`, no daemon conflicts.
* **Negative:** the thresholds are a choice; set both to the full range to charge fully. The external ultrabase battery
  (BAT1) is not configured.
