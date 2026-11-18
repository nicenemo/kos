# 14. Linters in the Pre-Commit Hook

* **Status:** Accepted
* **Date:** 2026-11-19

## Context

[ADR 0011](./0011-secret-scanning-with-pre-commit-hook.md) put `gitleaks` and `yamllint` in the `pre-commit` hook.
`ansible-lint` and `markdownlint-cli2` must also pass before a commit (see `CLAUDE.md`), but nothing enforced them, and
the same lint configuration was missing in samos, multiberry and mikmakmuk.

## Decision

* The `pre-commit` hook runs `gitleaks`, `yamllint` and `ansible-lint -q .`. It refuses to run when `gitleaks` or
  `ansible-lint` is missing. `ansible-lint` gets an explicit path and `</dev/null`; without them it reads `stdin` as a
  playbook when it is not run from a terminal and hangs.
* `ansible-lint`, `yamllint`, `markdownlint-cli2` and `gitleaks` are installed by `common_packages` in `inventory.yml`
  and removed again by `base-teardown.yml`. `trufflehog` (pre-push) is not a pacman package and stays a manual install.
* `.ansible-lint`, `.yamllint`, `.markdownlint-cli2.yaml`, the hooks and `githooks-setup.sh` are the same in kos, samos,
  multiberry and mikmakmuk. `build/`, `certs/` and `credentials/` are excluded from linting.

## Consequences

* **Positive:** a commit that does not lint cannot be made, and all four playbook repositories follow one standard.
* **Negative:** a commit takes about a minute because `ansible-lint` runs on the whole repository. `markdownlint-cli2`
  is not in the hook; run it by hand.
