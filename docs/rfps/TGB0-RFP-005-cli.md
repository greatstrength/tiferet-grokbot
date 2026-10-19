# TGB0-RFP-005 - CLI and Configuration

**Status:** Draft | **Domain:** `grokbot` | **Branch:** docs-core-domain | **Issue:** not published
**Related:** `docs/domain.md` sections 2.13, 3.1, 3.7; TGB0-RFP-001..004
**Depends on:** TGB0-RFP-001, TGB0-RFP-002, TGB0-RFP-003, TGB0-RFP-004
**Blocks:** user-facing adoption of v0.1; later commands extend this surface

## 1. Summary

Ship one entry point, `grokbot`, that runs skills sync, roster route, ledger
listing, lint, seed and matrix render from a single config and binding
(domain.md 2.13). Domain and service errors print one line and exit 2.
Read-only commands open the store read-only. YAML core works with no extras;
`[h5]` selects the HDF5 backend.

## 2. Motivation

A library without a CLI leaves every team writing its own glue. One command
surface, stable exit codes and env-overridable paths make the toolkit usable
from scripts and from bots that shell out.

## 3. Current state

- Precursor CLI patterns: `--config` before subcommand, env overrides for
  config and skills dir, one-line errors, exit 2 on domain failure, read-only
  opens for stats/events/export/route/matrix, `--last` validation.
- Binding file pattern exists in Tiferet collab docs; this RFP adopts a
  machine-readable binding the CLI loads beside the seed.

## 4. Proposal

1. **Entry point.** Console script `grokbot` -> `tiferet_grokbot.cli:main`.
2. **Commands (v0.1).**

   | Command | Purpose | Store mode |
   |---|---|---|
   | `seed` | replay seed into the store | write |
   | `sync` | re-import skills from the skill directory | write |
   | `retag` | add/remove skill tags | write |
   | `weight` / `retire` | set or retire an assignment | write |
   | `route` | print winner, scores, partner | read |
   | `events` | list ledger rows (`--last N`, N >= 1) | read |
   | `export` | rebuild a skill to stdout or a path | read |
   | `matrix` | render the assignments view | read |
   | `lint` | run text lint on paths | none |
   | `stats` | counts of skills, agents, events | read |

3. **Config.** YAML seed path via `--config` or `GROKBOT_CONFIG`. Skills
   directory via config key or `GROKBOT_SKILLS_DIR`. Store path via `--h5` /
   config or in-memory / YAML store when `[h5]` is not installed.
4. **Binding.** Optional binding file (RFP prefix, account, signature format,
   role and project tag lists, policy flags including the candidate
   `experiment_requires_rfp` check). Missing binding uses safe defaults.
5. **Errors.** Missing config, domain errors and service errors: one line to
   stderr, exit 2. Lint findings: exit 1. Success: exit 0.
6. **ASCII.** Help text, errors and rendered output are strict ASCII.

## 5. Out of scope

- `check-action` / policy enforcement commands (TGB0-RFP-006).
- Template validate and handoff commands (later RFPs).
- Interactive TUI.

## 6. Risks and open questions

- **Resolved:** YAML core, h5 optional (maintainer default).
- **Resolved:** advisory lint exit 1 on findings, not a hard block.
- **Open:** exact env var prefix (`GROKBOT_` vs `TIFERET_GROKBOT_`).
- **Open:** whether `matrix --out` must refuse to overwrite without `--force`.

## 7. Acceptance criteria

- [ ] `pip install -e .` exposes `grokbot` and `grokbot --help` lists the v0.1
  commands.
- [ ] `grokbot --config minimal-team/seed.yml seed` is idempotent on a second
  run.
- [ ] `grokbot route ...` and `grokbot events` work against a read-only store
  file mode.
- [ ] Missing config prints one line and exits 2.
- [ ] `grokbot lint` on a file with an em dash exits 1 and prints a finding;
  on a clean file exits 0.
- [ ] Without `tiferet-h5` installed, YAML store commands succeed; h5-only
  flags fail with a clear install hint.
- [ ] Help, errors and matrix output are strict ASCII.

## 8. Suggested TRD slicing

Reconstruction debt, not a gate. Drawn from the Proposal items above; re-read
at freeze time.

1. Entry point, config, errors and ASCII output (Proposal 1, 3, 5, 6).
2. Command table wired to the TGB0-RFP-001..004 services (Proposal 2).
3. Binding file loader with safe defaults (Proposal 4).

- major@grokbot via tif-ai
