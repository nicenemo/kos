# 15. BlueZ LE Connection Parameters

* **Status:** Accepted
* **Date:** 2026-11-19

## Context

The Nachtgespenst macro pad (Core2 and CoreS3) talks to kos over Bluetooth LE HID and asks for a connection interval of
7.5 to 15 ms. Measurements with both laptop controllers (the Intel Bluetooth 5 controller and a Bluetooth 4.0 CSR dongle)
show about 30 delivered reports per second on both, so the limit is the host, not the radio: BlueZ sends one queued
write per connection event and its default interval is about 30 to 50 ms.

## Decision

* `playbooks/bluez-setup.yml` sets `MinConnectionInterval` 6, `MaxConnectionInterval` 9 (7.5 and 11.25 ms),
  `ConnectionLatency` 0 and `ConnectionSupervisionTimeout` 400 in the `[LE]` section of `/etc/bluetooth/main.conf`
  (the template lines are commented out, the playbook replaces them) and restarts `bluetooth`.
* `playbooks/bluez-teardown.yml` comments them out again. Bonds survive the restart; devices must reconnect.
* The parameters apply to every LE device, not only the pad: the cost is slightly more radio activity for mice and
  keyboards, which is what they usually ask for anyway.

## Consequences

* **Positive:** a chance of several times more reports per second to the pad; one reversible change.
* **Negative:** whether BlueZ applies the values to HID over GATT connections is measured, not assumed; if the rate
  stays at 30 per second, run the teardown. Bluetooth audio devices are BR/EDR and are not affected.
