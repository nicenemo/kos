# 8. Intel Broadwell Graphics Stack

* **Status:** Accepted
* **Date:** 2026-11-18

## Context

kos has an Intel HD Graphics 5500 (Broadwell, Gen8) and no discrete GPU. mikmakmuk installs `intel-media-driver` for
newer iGPUs and sets `MESA_LOADER_DRIVER_OVERRIDE=iris`. `intel-media-driver` only supports Gen9 and later; Broadwell
video acceleration needs the legacy i965 VA-API driver. The new Intel compute runtime does not support Gen8, so
OpenCL/Rusticl is not configured. Nvidia and ROCm playbooks from the other repositories do not apply.

## Decision

Install `mesa`, `vulkan-intel`, `libva-intel-driver`, `libva-utils`, `vulkan-tools`, `clinfo` and `intel-gpu-tools`.
Set `MESA_LOADER_DRIVER_OVERRIDE=iris` and `LIBVA_DRIVER_NAME=i965` through `/etc/environment.d/`, load `i915` at boot
and set `i915 enable_fbc=1 fastboot=1` in `/etc/modprobe.d/`.

## Consequences

* **Positive:** hardware video decoding in browsers and Moonlight works on the iGPU.
* **Negative:** Vulkan on Gen8 is incomplete, and `libva-intel-driver` is in maintenance mode. Revisit when the
  hardware is replaced.
