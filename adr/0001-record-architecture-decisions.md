# 1. Record Architecture Decisions

* **Status:** Accepted
* **Date:** 2026-11-18

## Context

kos is a laptop that is rebuilt from scratch from time to time. The reasons behind choices (why `mitigations=off`, why
the i965 VA-API driver, why TLP) are lost quickly and easy to undo by accident.

## Decision

Use Architecture Decision Records as framed by Michael Nygard, stored as Markdown in `adr/`, numbered sequentially.
The format follows the [mikmakmuk](https://github.com/nicenemo/mikmakmuk) repository: status, date, context, decision,
consequences.

## Consequences

* **Positive:** the README and `CLAUDE.md` stay short; rationale lives in one place.
* **Negative:** an ADR has to be written or superseded whenever a deliberate decision changes.
