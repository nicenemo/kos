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
| `laptop-setup.yml` | TLP (battery thresholds, rfkill) + `thermald`, PPD masked, weekly `fstrim.timer`, v4l/usb tools |
| `flatpak-setup.yml`, `chrome-setup.yml` | Flatpak + Flathub, Google Chrome (as in samos) |
| `desktop-setup.yml` | Vivaldi, Proton Mail, Moonlight |
| `docker-setup.yml` | Docker + compose, `/opt/compose` |
| `dev-setup.yml` | development tooling: PlatformIO, esptool, JBang, JDK, CMake/Ninja, ESP32 serial udev rules |

`dev-setup.yml` installs `platformio-core` (+ its udev rules), esptool, CMake/Ninja, the JDK, system-wide
JBang (`/opt/jbang`), the udev rules for the M5Stack Core2/CoreS3 serial ports (`files/99-kos-esp32-udev.rules`) and
adds your user to `uucp`. Log out and in again afterwards so the group applies.

The bootloader is systemd-boot managed by `systemd-boot-manager` (`/etc/sdboot-manage.conf`). Extra kernel options go
in `kernel_options` (inventory) and end up in `/etc/sdboot-manage.conf.d/99-kos.conf`; `boot_default_entry` is set in
`/boot/loader/loader.conf`.

## Usage

kos manages itself over SSH on `127.0.0.1` as `myansibleuser` (passwordless sudo, dedicated key
`~/.ssh/id_ed25519_ansible`), like the mikmakmuk nodes. The bootstrap runs locally as `kruse` and also installs and
enables `sshd`, restricted to localhost with key-only login.

```bash
./controller-setup.yml          # create the ansible SSH keypair (no -K)
./ansible-user-setup.yml -K     # sshd on localhost + myansibleuser with NOPASSWD sudo (needs -K)
./setup.yml                     # everything (no -K)
./teardown.yml                  # revert (keeps system-critical packages)
./ansible-user-teardown.yml -K  # remove myansibleuser and the localhost sshd config
./controller-teardown.yml       # delete the keypair
ansible-playbook playbooks/dev-setup.yml   # a single part
ansible-playbook setup.yml --check --diff            # dry run
```

Requires `ansible` plus the collections in `requirements.yml` (`ansible-galaxy collection install -r requirements.yml`).

## Notes

* Power: TLP and thermald run together; power-profiles-daemon is masked (conflicts with TLP). Battery limit 75-80 %
  (`battery_start_threshold` / `battery_stop_threshold`); the governor stays `schedutil` (laptop, unlike mikmakmuk).
* Lint: `ansible-lint`, `yamllint .`, `markdownlint-cli2 "**/*.md"`
