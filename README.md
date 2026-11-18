# kos: ThinkPad T450s setup

Ansible playbooks that configure **kos**, a Lenovo ThinkPad T450s (Broadwell CPU, Intel HD Graphics 5500, no
discrete GPU, 12 GB RAM) running CachyOS with KDE Plasma. Layout and conventions follow
[samos](https://github.com/nicenemo/samos), [multiberry](https://github.com/nicenemo/multiberry) and
[mikmakmuk](https://github.com/nicenemo/mikmakmuk). No Elasticsearch, Kibana or gaming playbooks.

## Playbooks

| Playbook | What it does |
|---|---|
| `base-setup.yml` | keyrings, full upgrade, common packages, user groups, `nvim` as editor |
| `kernel-setup.yml` | `linux-cachyos` + headers, `intel-ucode` (stock `linux` stays as fallback) |
| `intel-gpu-setup.yml` | Mesa, `vulkan-intel`, i965 VA-API (Broadwell is not supported by intel-media-driver), i915 options |
| `laptop-setup.yml` | `thermald`, weekly `fstrim.timer`, v4l/usb tools |
| `flatpak-setup.yml`, `chrome-setup.yml` | Flatpak + Flathub, Google Chrome |
| `docker-setup.yml` | Docker + compose, `/opt/compose` |
| `nachtgespenst-setup.yml` | build tooling for [nachtgespenst](https://github.com/nicenemo/nachtgespenst) |

`nachtgespenst-setup.yml` installs Python, esptool, CMake/Ninja, the JDK, system-wide JBang (`/opt/jbang`),
udev rules and `uucp` membership for the M5Stack Core2/CoreS3 serial ports, clones the repo to
`~/Projects/Nachtgespenst` and creates its project-local PlatformIO venv (`.pio/venv`). Switch the clone and venv off
with `nachtgespenst_clone: false` / `nachtgespenst_bootstrap_pio: false` in `inventory.yml`. Log out and in again
afterwards so the new `uucp` group applies.

## Usage

The machine configures itself (`connection: local`), so there is no ansible management user; `ansible.cfg` asks for
your sudo password on every run.

```bash
./setup.yml                                  # everything
./teardown.yml                               # revert (keeps system-critical packages and your checkout)
ansible-playbook playbooks/nachtgespenst-setup.yml   # a single part
ansible-playbook setup.yml --check --diff    # dry run
```

Requires `ansible`, `community.general`, `community.docker` and `ansible.posix` (already present on kos).

## Notes

* Kernel command line is not touched: `mitigations=off` is already set, and the boot loader entries are managed
  outside this repo (see the multiberry/samos `kernel-setup.yml` if you want to manage `/etc/kernel/cmdline`).
* Lint: `ansible-lint`, `yamllint .`
