# 10. Scope and Excluded Workloads

* **Status:** Accepted
* **Date:** 2026-11-18

## Context

The playbooks started from samos, multiberry and mikmakmuk. These contain playbooks for hardware and workloads that
kos does not have or should not run: Elasticsearch/Kibana, Traefik, Caddy, a local CA, Ollama/Lemonade models, ROCm,
Nvidia drivers, Steam, Firestorm and benchmark pipelines. A dual-core laptop with 12 GB RAM and no GPU is not a
sensible host for them.

## Decision

kos covers the host OS baseline (packages, kernel and bootloader, iGPU, power), desktop applications (Flatpak,
Chrome, Vivaldi, Proton Mail, Moonlight), Docker and development tooling. Playbooks from the other repositories are
copied only when they apply, and are then rewritten for kos (single user, no secondary user, no copied host names).

## Consequences

* **Positive:** small, reviewable repository where every playbook applies to the hardware.
* **Negative:** improvements in the other repositories are not picked up automatically; compare by hand when needed.
