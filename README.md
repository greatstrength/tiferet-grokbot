# tiferet-softbot

A Tiferet-based toolkit that makes teams of softbots lean and reliable. Skills as ordered-section documents, weighted routing, an event-sourced ledger, advisory guardrails and document linting: data instead of prompts.

A softbot is a software robot: an agent that acts within software environments, such as repositories, issue trackers, chat and documents, rather than in the physical world. Any AI assistant bot that works through tools and accounts is a softbot in this sense, whatever platform it runs on.

Distribution: `tiferet-softbot`. Import: `tiferet_softbot`. CLI: `softbot`.

## What it is for

A softbot team fails in predictable ways. Work goes to the wrong bot, a bot repeats or contradicts another, state lives in chat and gets lost, and a bot does something only a human should do. Reports and PR bodies drift in shape.

tiferet-softbot addresses that with:

- **Skills as documents.** A skill is a SKILL.md file split into ordered sections. Its rules (productions) are indexed. The files stay the master; the store re-imports them and never writes back.
- **A roster with weighted routing.** Agents carry integer weights (0-5) on roles, projects, skills, affinities and SME markers. "Who should handle this?" becomes one explainable score.
- **An event ledger.** Every change records timestamp, actor, action, target, old, new and reason. A seed replays history so a fresh store ends in the same state.
- **Policies and templates as data.** Guardrails, signature rules and document templates live in config and are checked by linters.

It runs as a library plus a CLI. Any softbot team, on any bot platform and any GitHub org, installs it and points it at its own skill files, roster and policies.

## Contents

- `docs/domain.md` - the core domain (Bot, Role, Project, Skill, Assignment, Affinity, Balance, Routing, Ledger, Policy, Handoff, Report, Lint, Binding).
- Code package: not yet; see the RFPs.

## RFPs

Prototype work follows the Tiferet RFP model. An RFP is drafted locally in a gitignored `.rfp/` folder, then published as a GitHub issue on this repo; the issue body is the public copy. Ids follow the Tiferet `<PREFIX><major>-RFP-<nnn>` shape. This repo's binding prefix is `TSB0` (`TSB` for tiferet-softbot plus major version `0`, marking the 0.x line until a 1.0 freeze), so ids are `TSB0-RFP-<nnn>` (three digits, for example `TSB0-RFP-001`) and issue titles are `TSB0-RFP-<nnn> - <Plain Title>`. The prefix is a binding value (`docs/domain.md` 2.13), so other teams and repos bind their own. Every RFP cites `docs/domain.md` section numbers and passes the experiment gate (`docs/domain.md` 3.8). The v0.1 RFPs (TSB0-RFP-001 to TSB0-RFP-005) cover the skills store, roster and routing, ledger, lint and CLI; TSB0-RFP-011 covers bot affinity and balance, and TSB0-RFP-012 covers delegation builds.

## Dependencies (planned)

- Core: `tiferet>=2.1,<2.2`, `pyyaml`. YAML-only mode needs nothing else.
- Extra `[h5]`: `tiferet-h5>=1.0.0,<2` for a persistent skills store and ledger.
- Extra `[kb]`: `tiferet-kb` for the GitHub mirror and composed reports (later).
- Extra `[test]`: `pytest>=7`.

## House rules

- ASCII: hyphen, colon, straight quotes. No em or en dashes, no smart quotes. New content the package writes is strict ASCII.
- Skill files are mastered in the team's skill directory. This toolkit reads them and never rewrites them.
- Agents on a shared bot account sign `- <agent>@<platform> via <account>`. The platform label and the account are binding values chosen by each team, not part of the core.
- Agents never merge, tag, push to main, cut a release, mint a freeze, or touch milestones or settings. The default guardrail policy says so; it is advisory in v0.1.
- Experiments need an RFP against the project's updated core domain docs before any implementation starts. The toolkit encodes that as a general governance rule (and a candidate guardrail) for every project it binds to, not a house rule of one team.

## Maintainers

Maintained by Great Strength Systems. Contributions come through pull requests under the RFP gate (`docs/domain.md` 3.8).

## License

BSD 3-Clause. See `LICENSE`.
