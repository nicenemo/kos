# 6. Disabling CPU Security Mitigations

* **Status:** Accepted
* **Date:** 2026-11-18

## Context

`mitigations=off` was already on the kernel command line of kos before these playbooks existed, as on the other
machines. The Broadwell CPU is affected by Spectre, Meltdown, L1TF and MDS; the mitigations cost noticeable
performance on a 2-core/4-thread CPU. Unlike the cluster nodes, kos is a mobile laptop that runs a web browser and
mail client all day.

## Decision

Keep `mitigations=off` (and `nowatchdog`) in `files/kos-tuning.conf` so the setting is reproducible.

## Consequences

* **Positive:** best use of a slow CPU.
* **Negative:** untrusted JavaScript (the browser) can theoretically use speculative execution side channels. This risk
  is accepted for a personal machine with full-disk encryption; remove the token from `files/kos-tuning.conf` and run
  `sdboot-manage gen` to switch the mitigations back on.
