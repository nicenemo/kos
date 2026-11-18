# CLAUDE.md

Ansible setup for the ThinkPad T450s `kos` (CachyOS, Broadwell, Intel iGPU only, no Nvidia, no gaming, no
Elasticsearch/Kibana). Run from the repo root: `./setup.yml`, `./teardown.yml`, or one `playbooks/*-setup.yml`.
Every `*-setup.yml` has a matching `*-teardown.yml`; `setup.yml`/`teardown.yml` are `import_playbook` lists.
The host group is `systems` (host `kos`, local connection); variables live in `inventory.yml`.
Nachtgespenst tooling (PlatformIO, JBang, ESP32 udev rules; no clone) is in `playbooks/nachtgespenst-setup.yml`.
Bootloader is systemd-boot via `sdboot-manage` (not Limine, not `/etc/kernel/cmdline`).
Lint: `ansible-lint`, `yamllint .`.
Broadwell needs the i965 VA-API driver (`libva-intel-driver`), not intel-media-driver.
