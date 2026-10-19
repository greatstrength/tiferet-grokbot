# TGB0-RFP-002 - Roster, Weights and Routing

**Status:** Draft | **Domain:** `grokbot` | **Branch:** docs-core-domain | **Issue:** not published
**Related:** `docs/domain.md` sections 2.1-2.3, 2.7-2.9, 3.4-3.6
**Depends on:** TGB0-RFP-001 - skills and tags live in that store
**Blocks:** TGB0-RFP-003 (seed replay drives weights), golden routing tests (v0.2)

## 1. Summary

Weight each agent against roles, projects and single skills (domain.md 2.7),
pair agents with a lead through affinities (2.8), mark domain subject matter
experts (2.9), and route a message to the recipient with the highest base plus
project plus affinity score (3.5). Weights change only through controlled
inputs with an actor and a reason (2.10). The seed keeps the whole history so
a fresh store always ends in the same state (3.6).

## 2. Motivation

When a lead relays a message, it should go to the right teammate by skill, not
by habit. The relay rule must be explainable (a score anyone can recompute)
and auditable (every weight has a who and a why). SME projects need feedback
to go to the expert and implementation to go to the builders, without baking
one team's names into the engine.

## 3. Current state

- The score `base + project + affinity`, seed-order tie-break, affinity
  partner, documentation-ignores-named-skill rule, and SME feedback /
  implementer split are proven in a private precursor.
- Assignment kinds in use: `role`, `project`, `skill`, `affinity`, `sme`.
- Default role classes: feedback = review, triage, scouting; implementer =
  leadership, implementation, testing; documentation ignores SMEs and named
  skill weights.
- Team-specific agents, weights and reasons stay in each team's private seed.
  This RFP ships only the engine and a generic minimal-team example.

## 4. Proposal

1. **Agent store.** Agents with id, name, role text, and an assignments table
   (kind, target, weight 0-5). Index lists ids in seed order.
2. **Events.** `CreateAgent`, `SetWeight` (assign or reweight; kind checked;
   same weight is a no-op), `RetireWeight`. All require `actor` and `reason`.
3. **Default skill weight** (3.4): highest role weight among the skill's role
   tags. Deviations need a one-line reason. A role reweight does not cascade.
4. **Relay** (3.5): `RouteMessage(recipients, role, project, skill_id, lead)`
   returns winner, partner, smes found, sme_rule applied (`feedback`,
   `implementer`, or empty), and every score. Read-only, logs nothing.
5. **SME rule** (2.9): with an SME among the recipients, feedback roles go to
   SMEs only; implementer roles go to non-SMEs only; other roles (including
   documentation by default) ignore the marker. Role class lists are config.
6. **Documentation rule:** roles in `skill_weight_ignored_roles` (default:
   documentation) score role + project only when a skill is named.
7. **Tie-break:** equal scores go to the earlier recipient; default recipient
   list is seed order.
8. **Seed:** base weights plus ordered `reweights` and `retags` lists. Fresh
   seed replays all; re-seed of an up-to-date store logs nothing; a newly
   appended entry logs exactly one.
9. **Matrix view:** render a human-readable assignments table from the store
   and seed notes. No team names in the generic example.

## 5. Out of scope

- Sending or posting the routed message.
- Production-level weights in routing.
- Guardrail enforcement of who may reweight (advisory policy is TGB0-RFP-006).
- Golden routing CI tables (v0.2 companion).

## 6. Risks and open questions

- **Resolved:** SME is the primary feedback provider, not the main implementer
  (domain.md 2.9).
- **Resolved:** documentation stays outside the SME filter by default (2.2).
- **Open:** whether role weights should later be derived from skill weights.
- **Open:** affinity scope beyond a single role (for example testing on a
  heavy task).
- **Watch:** close calls on ties; the matrix must label every tie.

## 7. Acceptance criteria

- [ ] Seeding the minimal-team example loads the declared agents, role and
  project weights, and skill weights, and a second seed logs zero events.
- [ ] Weight outside 0-5 fails with a one-line domain error; empty actor or
  reason fails required-parameter.
- [ ] Assign then reweight logs `assign` then `reweight`; retire logs the old
  weight; same weight again logs nothing.
- [ ] Leadership on a project with a clear lead goes to that lead; a named
  skill overrides the role weight except for ignored roles.
- [ ] On an SME project, review goes to the SME; implementation goes to a
  non-SME; with the SME absent, scores decide.
- [ ] With a lead set, affinity boosts apply and the winner's partner is named.
- [ ] Documentation ignores a named skill's weight.
- [ ] Rendered matrix and all event text are strict ASCII.
- [ ] No example or test embeds a real team's agent names or weights.

- major@grokbot via tif-ai
