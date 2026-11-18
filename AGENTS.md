# AGENTS.md

Guidelines for AI assistants in this repository. The authoritative description of commands, architecture and
conventions is [CLAUDE.md](./CLAUDE.md); decisions are in [`adr/`](./adr/), how-tos in [`wiki/`](./wiki/).

* Keep every `*-setup.yml` paired with a `*-teardown.yml` and register both in `setup.yml` / `teardown.yml`.
* Static files live in `files/`, Jinja templates in `templates/`, never inside `playbooks/`.
* Do not add workloads that do not fit the hardware (see ADR 0010) and do not edit boot entries by hand (ADR 0004).
* Run `ansible-lint`, `yamllint .` and `markdownlint-cli2 "**/*.md"` before committing.
* Never commit secrets; the `pre-commit` hook (`./githooks-setup.sh`) runs gitleaks, do not use `--no-verify`.
