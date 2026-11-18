# Troubleshooting

## The machine boots the wrong kernel

The default entry is the `default` line in `/boot/loader/loader.conf`. Choose another kernel in the boot menu (press
space during the timeout), then list the entries and fix the line:

```bash
sudo bootctl list
sudo sdboot-manage gen
```

Set `boot_default_entry` in `inventory.yml` to the file name shown by `bootctl list` and run
`ansible-playbook playbooks/kernel-setup.yml`.

## Ansible cannot connect to 127.0.0.1

```bash
systemctl status sshd
ssh -i ~/.ssh/id_ed25519_ansible myansibleuser@127.0.0.1
```

If sshd is not running or the user is missing, run `./ansible-user-setup.yml -K` again. Repeated wrong sudo passwords
trigger `pam_faillock`: `sudo faillock --user kruse --reset`.

## No access to /dev/ttyACM0 or /dev/ttyUSB0

Check `id`: the user must be in `uucp` (log in again after `dev-setup.yml`). Check that the rule matches the board with
`udevadm info -a /dev/ttyACM0 | grep -E 'idVendor|idProduct'` and add the IDs to `files/99-kos-esp32-udev.rules`.

## Battery does not stop charging at 80 percent

`sudo tlp-stat -b` shows whether the thresholds are applied. The external ultrabase battery (BAT1) is not configured.
Change `battery_start_threshold` and `battery_stop_threshold` in `inventory.yml`, then run `laptop-setup.yml`.

## No hardware video decoding

`LIBVA_DRIVER_NAME=i965 vainfo` must list profiles. The setting comes from `/etc/environment.d/99-kos-igpu-dri.conf`
and only applies to new login sessions; Flatpak applications may need `flatpak override --env=LIBVA_DRIVER_NAME=i965`.
