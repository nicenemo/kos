# 2. Ansible for Declarative Automation

* **Status:** Accepted
* **Date:** 2026-11-18

## Context

The laptop setup used to be a mix of manual steps and shell history. A reinstall meant remembering what was done.

## Decision

Ansible is the only configuration management tool. Every `playbooks/*-setup.yml` has a matching
`*-teardown.yml`; `setup.yml` and `teardown.yml` only `import_playbook` them (teardown in reverse order). Native
modules are preferred over `command`/`shell`. The repository is checked with `ansible-lint` (production profile),
`yamllint` and `markdownlint-cli2`.

## Consequences

* **Positive:** a reinstall is `controller-setup`, `ansible-user-setup`, `setup`; teardowns make experiments reversible.
* **Negative:** teardowns deliberately keep system-critical packages (git, base-devel, mesa, stock kernel), so a
  teardown is not a perfect inverse. Every new playbook needs its teardown registered in both orchestrators.
