# Changelog

All notable changes to the skills in this repository are documented here.

## 2026-11-22

### Added
- **`improve-logging`** ([`SKILL.md`](./improve-logging/SKILL.md), [`WIDE-EVENTS.md`](./improve-logging/WIDE-EVENTS.md)):
  - Stack-agnostic, user-invoked (`/improve-logging`) skill inspired by [loggingsucks.com](https://loggingsucks.com/).
  - Spawns a fresh sub-agent to audit missing or poor logging across execution boundaries.
  - Produces a prioritized Markdown report and waits for explicit approval before refactoring toward wide events, high-cardinality correlation, business context, tail sampling, and centralized redaction.
- Repository `README.md` with `npx skills@latest` installation instructions and `skills.sh` badge.
