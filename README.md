# kos: ThinkPad T450s setup

Ansible playbooks that configure **kos**, a Lenovo ThinkPad T450s (Broadwell CPU, Intel HD Graphics 5500, no
discrete GPU, 12 GB RAM) running CachyOS with KDE Plasma. Layout and conventions follow
[samos](https://github.com/nicenemo/samos), [multiberry](https://github.com/nicenemo/multiberry) and
[mikmakmuk](https://github.com/nicenemo/mikmakmuk). No Elasticsearch, Kibana or gaming playbooks.

## Playbooks

| Playbook | What it does |
| --- | --- |
| `base-setup.yml` | keyrings, full upgrade, common packages (incl. neovim, jq), user groups, `nvim` as editor |
| `kernel-setup.yml` | `linux-cachyos` + headers, `intel-ucode`, systemd-boot via `sdboot-manage`, default entry, sysctl |
| `intel-gpu-setup.yml` | Mesa (iris), `vulkan-intel`, i965 VA-API (Broadwell is not supported by intel-media-driver), i915 |
| `laptop-setup.yml` | `thermald`, weekly `fstrim.timer`, v4l/usb tools |
| `flatpak-setup.yml`, `chrome-setup.yml` | Flatpak + Flathub, Google Chrome (as in samos) |
| `desktop-setup.yml` | Vivaldi, Proton Mail, Moonlight |
| `docker-setup.yml` | Docker + compose, `/opt/compose` |
| `nachtgespenst-setup.yml` | build tooling for [nachtgespenst](https://github.com/nicenemo/nachtgespenst), no clone |

`nachtgespenst-setup.yml` installs `platformio-core` (+ its udev rules), esptool, CMake/Ninja, the JDK, system-wide
JBang (`/opt/jbang`), the udev rules for the M5Stack Core2/CoreS3 serial ports (`files/99-kos-esp32-udev.rules`) and
adds your user to `uucp`. Log out and in again afterwards so the group applies.

The bootloader is systemd-boot managed by `systemd-boot-manager` (`/etc/sdboot-manage.conf`). Extra kernel options go
in `kernel_options` (inventory) and end up in `/etc/sdboot-manage.conf.d/99-kos.conf`; `boot_default_entry` is set in
`/boot/loader/loader.conf`.

## Usage

The machine configures itself (`connection: local`), so there is no ansible management user; `ansible.cfg` asks for
your sudo password on every run.

```bash
./setup.yml                                  # everything
./teardown.yml                               # revert (keeps system-critical packages and your checkout)
ansible-playbook playbooks/nachtgespenst-setup.yml   # a single part
ansible-playbook setup.yml --check --diff    # dry run
```

Requires `ansible` (`ansible-galaxy collection install -r requirements.yml`), `community.general`, `community.docker` and `ansible.posix` (already present on kos).

## Notes

* The CPU governor stays on power-profiles-daemon (unlike the always-on mikmakmuk nodes, this is a laptop).
* Lint: `ansible-lint`, `yamllint .`, `markdownlint-cli2 "**/*.md"`
