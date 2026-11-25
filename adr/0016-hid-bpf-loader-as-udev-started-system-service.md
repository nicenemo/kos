# 16. The HID-BPF Loader as a System Service Started by udev

* **Status:** Accepted
* **Date:** 2026-11-25

## Context

The Nachtgespenst pad needs a HID-BPF program on the host so that the Compose and Kana LEDs and the locking keys work
(`docs/plans/hid-ebpf-leds.md` in the nachtgespenst repository). Loading a HID-BPF `struct_ops` program needs
`CAP_BPF` and, depending on the kernel, `CAP_PERFMON` and `CAP_SYS_ADMIN`. A systemd user service cannot hold those
capabilities, the user manager has none to give. The board gets a new HID device name (and so a new `hid_id`, the
number the program attaches to) after every flash and replug, so the loader has to run again for each new device.

## Decision

* A system unit template `nachtgespenst-hid-bpf@.service` loads the program. It is a oneshot with `RemainAfterExit`:
  the loader attaches the program, pins its link under `/sys/fs/bpf/nachtgespenst` and exits; the program stays in the
  kernel until `ExecStop` detaches it. The instance name is the kernel name of the HID device.
* The unit runs as a dynamic user with `AmbientCapabilities` and `CapabilityBoundingSet` of exactly `CAP_BPF`,
  `CAP_PERFMON` and `CAP_SYS_ADMIN`, `NoNewPrivileges` and a few more restrictions. It does not run as root. Only the
  pin directory is created with full privileges (`ExecStartPre=+`).
* The udev rule `71-nachtgespenst-hid-bpf.rules` starts an instance on `add` of the board's HID device (USB and
  Bluetooth, `1209:0001`), as upstream `udev-hid-bpf` does, so the program follows the board.
* The loader is `/opt/nachtgespenst-hid-bpf/loader.jar`, owned by root and not writable by the user. The unit has
  `ConditionPathExists` on it: until the jar is installed nothing runs and no unit fails at a hotplug.
* `hid-bpf-setup.yml` installs the packages (`clang`, `llvm`, `libbpf`, `bpf`), the rule, the unit and the directory,
  and the jar when `-e hid_bpf_loader_jar=...` is given. `hid-bpf-teardown.yml` stops the instances (which detaches the
  programs) and removes all of it; the packages stay.

## Consequences

* **Positive:** no `sudo` per test, no long-lived root process, the capabilities are listed in one place, and the
  program follows the board after a flash.
* **Security:** whoever can replace `loader.jar` can load kernel code, so the file and its directory belong to root; the
  playbook, not the user, installs a new jar. The unit holds `CAP_SYS_ADMIN`, which is wide; if the kernel of the day
  accepts a narrower set the unit is the one place to change.
* **Unverified:** the unit and the rule are written from the documentation of systemd and HID-BPF and have not been run:
  no loader exists yet. The first run decides whether the capability set is enough and whether the attach by `hid_id`
  works for the Bluetooth (uhid) device.
