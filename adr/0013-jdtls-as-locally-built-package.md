# 13. jdtls as a Locally Built Package

* **Status:** Accepted
* **Date:** 2026-11-19

## Context

kos is used for Java development (see [ADR 0009](./0009-development-tooling-playbook.md)), and editors need the
Eclipse JDT language server (`jdtls`). It is not in the Arch or CachyOS repositories. Unpacking a tarball into `/opt`
(as is done for JBang) leaves a program that pacman does not know about: no version, no file ownership, no clean
removal. samos already builds `jdtls` as a pacman package with a set of playbooks.

## Decision

* kos uses the same playbooks as samos: `jdtls-download`, `jdtls-package`, `jdtls-distribute`, `jdtls-install` and
  `jdtls-remove-package`, called in this order by `jdtls-setup.yml`. `jdtls-teardown.yml` removes the package.
* The version is pinned in `inventory.yml` (`jdtls_version`). The download playbook reads the tarball name from the
  `latest.txt` of that milestone on download.eclipse.org and verifies the published SHA-256 checksum.
* The package is built from `templates/jdtls-pkg/PKGBUILD.j2` as `target_user` (not as the SSH user, which cannot enter
  the checkout) and installed from `/tmp`, which `jdtls-remove-package.yml` cleans up afterwards.
* A package that is installed with another version is replaced; the install is skipped only when the pinned version is
  already present.

## Consequences

* **Positive:** `pacman -Q jdtls` shows the version, the teardown is a plain package removal, and kos and samos share one
  procedure.
* **Negative:** five playbooks instead of one, and a build (`makepkg`) on the machine. A new `jdtls` version means
  changing `jdtls_version` in `inventory.yml` and running `jdtls-setup.yml` again.
