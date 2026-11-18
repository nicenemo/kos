# 5. Self-Management over SSH on Localhost with a Dedicated Ansible User

* **Status:** Accepted
* **Date:** 2026-11-18

## Context

The other machines are managed from a control laptop over SSH as `myansibleuser` with passwordless sudo and a
dedicated key. kos is its own control node. Using `connection: local` would work, but then every run needs the sudo
password and the setup differs from the other repositories.

## Decision

* kos is reached over SSH on `127.0.0.1` as `myansibleuser`, using `~/.ssh/id_ed25519_ansible`.
* `controller-setup.yml` creates the key; `ansible-user-setup.yml -K` runs locally as `kruse` (the one play that needs
  a password) and creates the user, the NOPASSWD sudoers file and the authorized key.
* The same bootstrap installs `sshd`, restricted to `ListenAddress 127.0.0.1` with password login disabled.
* The key is stored unencrypted; the disk is protected by LUKS (same reasoning as mikmakmuk ADR 0015).

## Consequences

* **Positive:** identical workflow to the other repositories; `setup.yml` runs without a password prompt.
* **Negative:** a passwordless-sudo account and a listening (localhost-only) sshd exist on a laptop. Anyone who can read
  the key as `kruse` gets root. `ansible-user-teardown.yml` removes the user and the sshd drop-in again.
