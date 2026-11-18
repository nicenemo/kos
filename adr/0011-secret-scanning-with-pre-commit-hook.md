# 11. Secret Scanning with a Pre-Commit Hook

* **Status:** Accepted
* **Date:** 2026-11-18

## Context

The repository manages SSH access (the ansible key, sudoers, sshd) and may later hold passwords or tokens. It is on
GitHub, and a pushed secret has to be treated as leaked even after the commit is removed. The inventory and playbooks
contain no secrets today; the private key lives in `~/.ssh` and is not part of the repository.

## Decision

* Scan every commit with [gitleaks](https://github.com/gitleaks/gitleaks) in a tracked `pre-commit` hook
  (`.githooks/pre-commit`, enabled with `./githooks-setup.sh` which sets `core.hooksPath`). The hook also runs
  `yamllint`, and it refuses to commit when gitleaks is missing.
* A second, independent scanner, [trufflehog](https://github.com/trufflesecurity/trufflehog), runs in `.githooks/pre-push`
  over the commits that are about to be pushed and fails the push on any finding (exit 183). It is not in the official
  repositories (installed here as the `trufflehog-bin` package), so the hook refuses to push when it is missing.
  Verification sends only a found candidate key to its own provider.
* Configuration is `.gitleaks.toml`: the default ruleset, a rule for Ansible vault passwords and an allowlist for
  `*.pub` files. `gitleaks` is installed by `base-setup.yml`.
* Secrets belong in an encrypted Ansible vault or outside the repository; `*.vault*`, `.vault_pass`, `credentials/`
  and `certs/` are in `.gitignore`.
* A hook is a local convenience and can be skipped with `--no-verify`; `gitleaks git` and
  `trufflehog git file://$PWD --results=verified,unknown,unverified` can be run on the full history at any time.

## Consequences

* **Positive:** a secret is caught before it enters history; no external service is needed.
* **Negative:** hooks are not cloned automatically, so `./githooks-setup.sh` must be run once per checkout. A GitHub
  Actions scan is not configured; add one if the repository becomes public or gets other contributors.
