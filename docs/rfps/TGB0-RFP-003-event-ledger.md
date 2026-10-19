# TGB0-RFP-003 - Event Ledger and Seed Replay

**Status:** Draft | **Domain:** `grokbot` | **Branch:** docs-core-domain | **Issue:** not published
**Related:** `docs/domain.md` sections 2.10, 3.6
**Depends on:** TGB0-RFP-001 (event row shape and skill actions), TGB0-RFP-002 (roster actions)
**Blocks:** audit export in the CLI (TGB0-RFP-005); later handoff and notes actions

## 1. Summary

Keep one append-only ledger of every state change (domain.md 2.10) with
timestamp, actor, action, target, old, new and reason. A seed file replays
history so a fresh store always ends in the same events and weights (3.6).
Re-seeding an up-to-date store is a no-op; appending one seed entry logs
exactly one event.

## 2. Motivation

Without a ledger, weights and skill imports are hearsay. A team needs to
answer "who changed this, when, and why?" and to rebuild the same store from
the seed on any machine. Chat memory and ad-hoc notes cannot do that.

## 3. Current state

- The seven actions for skills and roster are fixed: create, assign, reweight,
  retire, import, revise, retag.
- A private precursor uses a pure `matrix_event_fields(...)` helper and builds
  the domain event at the call site (clock read at the call site), with
  folded-final re-seed semantics.
- Content old/new values are `sha256:<12 hex>` digests; text history stays in
  git.

## 4. Proposal

1. **Event row.** Columns: timestamp (ISO 8601 with offset), actor, action,
   target, old, new, reason. Actor and reason required.
2. **Construction.** A pure function returns the field dict; the call site
   reads the clock and builds the domain object. No domain object constructed
   inside a module-level helper.
3. **Seed replay.** Base creates and assigns, then ordered `reweights`, then
   ordered `retags`. Hold entries (same weight) log nothing but keep their
   reason for documentation.
4. **Folded re-seed.** On an existing store, apply only the folded final state
   for keys the seed history mentions. Keys the seed never mentions survive.
5. **Read path.** Listing and export open the store read-only. `--last N`
   requires N >= 1.
6. **ASCII.** Every field the ledger writes is strict ASCII (3.2).

## 5. Out of scope

- New action kinds for handoff, notes, or mirror sync (later RFPs).
- Multi-writer locking (named open in TGB0-RFP-001).
- Shipping one team's seed; examples use a generic minimal team only.

## 6. Risks and open questions

- **Resolved:** seven actions for v0.1; new actions named per later RFP.
- **Open:** retention and export format (JSONL vs table dump).
- **Open:** whether reason text should be length-capped in the store.

## 7. Acceptance criteria

- [ ] A fresh seed of the minimal-team example is reproducible: two runs yield
  the same event count and final weights.
- [ ] A second seed on the same store logs zero events.
- [ ] Appending one reweights entry and re-seeding logs exactly one event.
- [ ] A hold entry (same weight) logs nothing and keeps its reason in the seed.
- [ ] `events --last 0` and negative values fail with a one-line error, exit 2.
- [ ] Read commands do not change store mtime on an h5 backend.
- [ ] All event fields are strict ASCII.

- major@grokbot via tif-ai
