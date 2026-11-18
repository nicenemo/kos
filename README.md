# kos: ThinkPad T450s setup

Ansible automation for **kos**, a Lenovo ThinkPad T450s running CachyOS with KDE Plasma and systemd-boot. The layout
and conventions come from [mikmakmuk](https://github.com/nicenemo/mikmakmuk) (closest hardware generation, ADR and wiki
structure), with desktop and application playbooks adapted from the author's other machine repositories. Playbooks that
do not apply to this hardware (Elasticsearch, Kibana, gaming, Nvidia, local AI) are left out on purpose, see
[ADR 0010](./adr/0010-scope-and-excluded-workloads.md).

## Hardware

| Part | Details |
| :--- | :--- |
| Model | Lenovo ThinkPad T450s (20BX) |
| CPU | Intel Core i7-5600U (Broadwell, 2 cores / 4 threads) |
| GPU | Intel HD Graphics 5500, no discrete GPU |
| RAM / disk | 12 GB, 512 GB SATA SSD (LUKS + btrfs) |
| Network | Intel I218-LM ethernet, Intel Wireless 7265 |
| OS / boot | CachyOS, systemd-boot managed by `sdboot-manage` |

## Playbooks

| Playbook | What it does |
| --- | --- |
| `base-setup.yml` | keyrings, full upgrade, common packages (neovim, jq, tmux, btop, ...), groups, `nvim` as editor |
| `kernel-setup.yml` | `linux-cachyos`, headers, `intel-ucode`, sdboot-manage options, default entry, sysctl tuning |
| `intel-gpu-setup.yml` | Mesa (iris), `vulkan-intel`, i965 VA-API, i915 options |
| `laptop-setup.yml` | TLP (battery thresholds) + `thermald`, power-profiles-daemon masked, weekly `fstrim.timer` |
| `flatpak-setup.yml`, `chrome-setup.yml` | Flatpak + Flathub, Google Chrome |
| `desktop-setup.yml` | Vivaldi, Proton Mail, Moonlight |
| `docker-setup.yml` | Docker + compose, `/opt/compose` |
| `udev-setup.yml` | udev rules and `uucp` for ESP32 serial adapters and the Nachtgespenst hidraw device |
| `dev-setup.yml` | development tooling: PlatformIO, esptool, JBang, JDK, CMake/Ninja |

Every `*-setup.yml` has a `*-teardown.yml`. `setup.yml` and `teardown.yml` import them in order and reverse order.

## Usage

kos manages itself over SSH on `127.0.0.1` as `myansibleuser` (passwordless sudo, dedicated key
`~/.ssh/id_ed25519_ansible`), see [ADR 0005](./adr/0005-ssh-on-localhost-with-ansible-user.md).

```bash
./controller-setup.yml          # create the ansible SSH keypair (no -K)
./ansible-user-setup.yml -K     # sshd on localhost + myansibleuser with NOPASSWD sudo (needs -K)
./setup.yml                     # everything (no -K)
./teardown.yml                  # revert (keeps system-critical packages)
./ansible-user-teardown.yml -K  # remove myansibleuser and the localhost sshd config
./controller-teardown.yml       # delete the keypair
ansible-playbook playbooks/dev-setup.yml        # a single part
ansible-playbook setup.yml --check --diff       # dry run
```

Requires `ansible` plus the collections in `requirements.yml` (`ansible-galaxy collection install -r requirements.yml`).
Step by step: [wiki/first-run.md](./wiki/first-run.md).

## Configuration

All settings are in `inventory.yml`: `common_packages`, `kernel_packages`, `boot_default_entry`,
`desktop_packages`, `battery_start_threshold` / `battery_stop_threshold`, `jbang_install_dir`. Static files are in
`files/` (kernel parameters in `kos-tuning.conf`, i915 options, iGPU environment, sysctl, udev rules for ESP32
boards and the Nachtgespenst hidraw device).

## Linting

```bash
ansible-lint                     # production profile, config in .ansible-lint
yamllint .                       # 120 columns
markdownlint-cli2 "**/*.md"
```

## Secret scanning

Run `./githooks-setup.sh` once per checkout. The `pre-commit` hook in `.githooks/` runs `gitleaks` on the staged
changes and `yamllint`; the `pre-push` hook runs `trufflehog` on the commits being pushed
([ADR 0011](./adr/0011-secret-scanning-with-pre-commit-hook.md)). Scan the whole history with
`gitleaks git --redact` and `trufflehog git file://$PWD`.

## Project page and licence

`index.html` is published with GitHub Pages by `.github/workflows/static.yml` (only that file is deployed). The
repository is public and licensed under the [MIT License](./LICENSE), (c) 2026 Hans Kruse.

## Documentation

* [Wiki](./wiki/): [first run](./wiki/first-run.md), [troubleshooting](./wiki/troubleshooting.md),
  [development tooling](./wiki/development-tooling.md)
* Architecture decisions in [`adr/`](./adr/):
  * [0001 Record architecture decisions](./adr/0001-record-architecture-decisions.md)
  * [0002 Ansible for declarative automation](./adr/0002-ansible-for-declarative-automation.md)
  * [0003 CachyOS as host operating system](./adr/0003-cachyos-as-host-operating-system.md)
  * [0004 systemd-boot with sdboot-manage](./adr/0004-systemd-boot-with-sdboot-manage.md)
  * [0005 SSH on localhost with a dedicated ansible user](./adr/0005-ssh-on-localhost-with-ansible-user.md)
  * [0006 Disabling CPU security mitigations](./adr/0006-disabling-cpu-security-mitigations.md)
  * [0007 TLP and thermald for power management](./adr/0007-tlp-and-thermald-for-power-management.md)
  * [0008 Intel Broadwell graphics stack](./adr/0008-intel-broadwell-graphics-stack.md)
  * [0009 Development tooling playbook](./adr/0009-development-tooling-playbook.md)
  * [0010 Scope and excluded workloads](./adr/0010-scope-and-excluded-workloads.md)
  * [0011 Secret scanning with a pre-commit hook](./adr/0011-secret-scanning-with-pre-commit-hook.md)
  * [0012 Separate udev playbooks](./adr/0012-separate-udev-playbooks.md)
