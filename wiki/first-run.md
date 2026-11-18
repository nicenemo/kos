# First run on a fresh install

Requirements: CachyOS with KDE, systemd-boot, a user `kruse` in `wheel`, `ansible`, `ansible-lint` and `yamllint`.

```bash
./githooks-setup.sh                # enable the hooks; trufflehog (trufflehog-bin, AUR) must be installed
ansible-galaxy collection install -r requirements.yml
./controller-setup.yml           # creates ~/.ssh/id_ed25519_ansible (no -K)
./ansible-user-setup.yml -K      # sshd on localhost, myansibleuser, NOPASSWD sudo
./setup.yml                      # everything else, no password prompt
```

Afterwards:

* Log out and in again (new `uucp` group for the serial ports).
* Check the default boot entry with `sudo bootctl list` and reboot into `linux-cachyos`.
* Check the graphics stack with `vainfo` and `vulkaninfo --summary`.
* Check power management with `sudo tlp-stat -s` and `sudo tlp-stat -b` (charge thresholds).

Run one part only with `ansible-playbook playbooks/<name>-setup.yml`, and dry-run with `--check --diff`.
