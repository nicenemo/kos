# HID-BPF loader of the Nachtgespenst pad

The root side is installed by `playbooks/hid-bpf-setup.yml` ([ADR 0016](../adr/0016-hid-bpf-loader-as-udev-started-system-service.md)).
The loader itself is built in the nachtgespenst repository (`docs/plans/hid-ebpf-leds.md`).

## Install the loader

```bash
ansible-playbook playbooks/hid-bpf-setup.yml -e hid_bpf_loader_jar=/path/to/loader.jar
```

Without `hid_bpf_loader_jar` the playbook only prepares the system; the unit stays inert because
`/opt/nachtgespenst-hid-bpf/loader.jar` does not exist.

## Look at it

```bash
systemctl list-units 'nachtgespenst-hid-bpf@*'      # one instance per HID device of the board
journalctl -u 'nachtgespenst-hid-bpf@*' -b          # what the loader said
sudo bpftool prog list | grep -i hid                # the program in the kernel
ls /sys/fs/bpf/nachtgespenst                        # the pinned links
```

## Start an instance by hand

The kernel name of the board's HID device is the last part of the sysfs path, for example `0003:1209:0001.000A`:

```bash
ls /sys/bus/hid/devices | grep 1209:0001
sudo systemctl start nachtgespenst-hid-bpf@0003:1209:0001.000A.service
```

## Remove it

```bash
ansible-playbook playbooks/hid-bpf-teardown.yml
```
