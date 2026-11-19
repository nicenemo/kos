# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Ansible automation for **kos**, a ThinkPad T450s (i7-5600U Broadwell, Intel HD 5500 only, 12 GB) running CachyOS with
KDE and systemd-boot. Scope: host OS baseline, desktop applications, Docker and development tooling. Not in scope: no
Nvidia, gaming, Elasticsearch/Kibana or local AI (ADR 0010). Rationale for deliberate decisions is in `adr/`; read the
relevant ADR before changing one (e.g. `mitigations=off`, TLP instead of power-profiles-daemon, i965 VA-API).

## Running playbooks

All playbooks are executable and run from the repo root (`ansible.cfg`: `inventory.yml`, `become = True`, key
`~/.ssh/id_ed25519_ansible`). kos is reached over SSH on 127.0.0.1 as `myansibleuser` (ADR 0005).

```bash
./controller-setup.yml          # create the ansible keypair (no -K)
./ansible-user-setup.yml -K     # runs locally as kruse: sshd on localhost, myansibleuser, NOPASSWD sudo
./setup.yml                     # everything (no -K)
./teardown.yml                  # revert
ansible-playbook playbooks/laptop-setup.yml   # one part
ansible-playbook setup.yml --check --diff     # dry run
```

## Linting

```bash
ansible-lint                    # production profile (.ansible-lint)
yamllint .                      # .yamllint, 120 columns
markdownlint-cli2 "**/*.md"     # .markdownlint-cli2.yaml (ignores playbooks/, asterisk lists, 120 columns)
```

Secrets: the tracked `pre-commit` hook (`.githooks/`, enable with `./githooks-setup.sh`) runs `gitleaks git --staged`,
`yamllint` and `ansible-lint`; `.githooks/pre-push` runs `trufflehog` (ADR 0011).
Never commit private keys, tokens or vault passwords; never bypass with `--no-verify`.

There are no tests; the teardown, setup cycle is the idempotency test. All three linters must pass before a commit.

## Architecture

* **Orchestrators**: `setup.yml` and `teardown.yml` are `import_playbook` lists. Every `*-setup.yml` needs a matching
  `*-teardown.yml`; register both, teardown in reverse order. Teardowns keep system-critical packages.
* **Inventory** (`inventory.yml`): group `systems`, host `kos` (SSH to 127.0.0.1), `target_user: kruse`. Package lists
  and tunables are variables here (`common_packages`, `kernel_packages`, `desktop_packages`,
  `boot_default_entry`, battery thresholds).
* **Bootloader**: systemd-boot via `sdboot-manage` (ADR 0004), not Limine or `/etc/kernel/cmdline`. Kernel options go
  in `/etc/sdboot-manage.conf.d/99-kos.conf`; the default kernel is the `default` line in `/boot/loader/loader.conf`;
  regenerate entries with `sdboot-manage gen`. `/boot` is root-only.
* **Development and udev**: `dev-setup.yml` installs packages only (PlatformIO, esptool, JBang, JDK). udev rules and the
  `uucp` membership are in `udev-setup.yml` / `udev-teardown.yml` (ADR 0012); rule files are in `files/`.
* **Graphics**: Broadwell needs `libva-intel-driver` (i965), not `intel-media-driver` (ADR 0008).
* **Power**: TLP + thermald; power-profiles-daemon stays masked (ADR 0007).
* **Path resolution**: playbooks under `playbooks/` define `project_root: "{{ playbook_dir | dirname }}"` to reach
  `files/`; do not use relative `src` paths (ansible-lint `no-relative-paths`).
* **Conventions**: Arch/CachyOS only (`community.general.pacman`), FQCN module names, tasks grouped under numbered
  `# ----` comments where a playbook is long, `changed_when` on command tasks, `become` set whenever `become_user` is
  used. **Quotes**: single quotes for strings without templating (`mode: '0644'`, `name: 'Install packages'`), double
  quotes only when the value contains Jinja (`{{ }}`, `{% %}`) or a single quote; the same in samos, multiberry and
  mikmakmuk.
* **Copying from other repositories**: samos is partly derived from multiberry and still contains wrong references to
  it (host names, users, domains). When copying a playbook, remove host-specific names, `secondary_user` and
  `firestorm_users`, and grep for `multiberry`, `samos` and `hanskruse` before committing.
* **Project page**: `index.html` (the only tracked HTML file, `*.html` is gitignored otherwise) is deployed to GitHub
  Pages by `.github/workflows/static.yml`. Keep it short and plain; keep it in step with the README playbook list.
  `style.css`, `menu.js`, `hero-bg.svg` and `favicon.svg` are identical in kos, mikmakmuk, multiberry and samos (menu
  and wave banner as on the nachtgespenst site, without its ghost); change them in all four. The `hanskruse.eu` button
  is always the last button of the banner.
* **Docs**: new decisions get the next ADR number in `adr/` and a line in the README list; how-to content goes in
  `wiki/`.

## Keeping the documentation up to date

This is the one place for these rules ([AGENTS.md](./AGENTS.md) only points here). With every change that is committed
to `main`, in the same commit or the next one on the same day:

* **`CHANGELOG.md`**: add an entry under today's date (`### Added`, `### Changed`, `### Fixed`), newest day first, one
  line per change, with a link to the ADR when there is one. Changes that stay on an unmerged branch are listed once
  under `### Branches` for that day, not as normal entries. Use British English; no entry for pure typo fixes.
* **`README.md`**: a new or changed playbook updates the playbook table, a new ADR gets a line in the ADR list.
* **`index.html`**: keep the feature list in step with the README (short and plain).
* **ADR**: a deliberate choice (a tunable, a service, a parameter that costs battery or changes the boot) gets a new
  ADR or an update of the existing one, with the measurement or reason.
* Run the three linters (above) before committing.
