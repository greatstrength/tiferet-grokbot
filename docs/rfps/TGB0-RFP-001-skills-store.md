# TGB0-RFP-001 - Skills Store and Document Compiler

**Status:** Draft | **Domain:** `grokbot` | **Branch:** docs-core-domain | **Issue:** not published
**Related:** `docs/domain.md` sections 2.4-2.6, 2.10, 3.1-3.3
**Depends on:** none - first RFP in this repo
**Blocks:** TGB0-RFP-002 (routing reads tags from this store), TGB0-RFP-003 (ledger shares the event shape), TGB0-RFP-005 (CLI commands over this store)

## 1. Summary

Store each real skill document whole as ordered sections (domain.md 2.5), with
its role and project tags and its derived productions (2.6). A document
compiler parses `SKILL.md` into sections, normalizes dashes and smart quotes
(3.2), and rebuilds it byte for byte (3.3). A sync command re-imports from the
team's skill directory (3.1) and logs only real changes (2.10). Core ships a
YAML-backed store; an optional `[h5]` extra persists through tiferet-h5.

## 2. Motivation

A bot team must tie work back to real skill documents, not names in a
spreadsheet. Pointers to files are not enough: the toolkit should query
sections and productions, see when a skill changed, and rebuild the exact
document. Routing (TGB0-RFP-002) needs the tags this store holds.

## 3. Current state

- The pattern is proven in a private precursor package that stores skills as
  ordered inputs in HDF5 through tiferet-h5, with a compiler that normalizes
  dashes and smart quotes and round-trips byte for byte.
- tiferet-h5 1.0.0 provides `TableObject`, `NodeObject`, `H5Repository` and
  pins `tiferet>=2.1.0,<2.2`.
- Skill files in the wild use frontmatter (`name`, `description`), an H1, and
  H2 sections. Two shapes recur: process skills and code-component skills.
- Non-ASCII in source files is common (dashes, arrows, box drawing). The
  compiler must normalize the six dash and quote characters and keep the rest
  so round-trip holds.

## 4. Proposal

1. **Layout (YAML core).** One seed or store directory holds skill records:
   id, file_name, name, description, ordered sections (ordinal, key, heading,
   content), tags (kind role|project, tag), and derived productions.
2. **Layout (`[h5]` extra).** One HDF5 file:

   ```
   /skills/<skill_id>/         attrs: id, file_name, name, description
       inputs                  ordinal, part, key, heading, content
       tags                    kind (role|project), tag
       productions             input_ordinal, ordinal, weight, text
   /events                     timestamp, actor, action, target, old, new, reason
   /index/skills               id
   ```

3. **Mappers.** Domain objects (`Skill`, `SkillSection`, `Production`) are
   `DomainObject`s; aggregates are the mutable form. H5 mappers follow the
   tiferet-h5 `TableObject` / `NodeObject` pattern. Long section text chunks
   on UTF-8 character boundaries when using fixed `StringCol`s.
4. **Compiler.** Normalize dashes and smart quotes; split frontmatter; cut at
   `## ` outside fences; key each section; parse frontmatter as YAML with a
   line-by-line `key: value` fallback.
5. **Productions.** Rule-bearing sections only by default, with an all-items
   flag (2.6). `rule_heading_words` is configurable; a small generic default
   ships (`guardrail`, `procedure`, `convention`, `rule`). Re-derived on every
   import.
6. **Sync.** For each `<dir>/SKILL.md`: a new skill logs one `import`; an
   existing skill logs one `revise` per changed, added, or removed section
   (sha256 digests); an unchanged skill writes nothing. Stored tags survive
   sync. A stored skill whose file is gone is reported `missing`, not removed.
7. **Retag.** `RetagSkill` adds or removes role or project tags (remove wins),
   validates against the allowed set, logs old and new lists.
8. **Style.** Tiferet code style and the import law: error ids in `assets`,
   mappers import domain only, interfaces and repos import from mappers.
   ASCII source. New content the package writes is strict ASCII (`to_ascii`).
9. **Compression (h5).** Tables use `Filters(complevel=5, complib='blosc',
   shuffle=True)` by default.

## 5. Out of scope

- Writing skills back to the team's skill directory (3.1 forbids it). Skill
  export to a folder is a later RFP.
- Non-SKILL.md skill files (for example evals JSON).
- Weights and routing: TGB0-RFP-002.
- The CLI surface: TGB0-RFP-005.

## 6. Risks and open questions

- **Resolved:** source of truth is the skill directory (domain.md 3.1).
- **Resolved:** dashes normalize on import; new content is strict ASCII (3.2).
- **Resolved:** YAML core with `[h5]` optional (maintainer default).
- **Open:** ordinal-based diffing. Inserting a section mid-document shifts
  later ordinals. Matching by key and heading would be tighter (stable section
  keys; also needed by later composition).
- **Open:** concurrent writers on h5. One writer at a time until a lock lands.
- **Open:** tiferet 2.2. Blocked on the h5/kb pin `<2.2`.

## 7. Acceptance criteria

- [ ] Importing a folder of SKILL.md files stores one skill per file, and each
  rebuilds equal to its ASCII-normalized source, byte for byte.
- [ ] No stored skill text contains U+2014, U+2013, U+2018, U+2019, U+201C, or
  U+201D.
- [ ] Every event field the store writes is strict ASCII.
- [ ] A section longer than the chunk size, including multi-byte characters,
  round-trips unchanged.
- [ ] A `## ` line inside a code fence does not start a section.
- [ ] Frontmatter that is not strict YAML still yields `name` and `description`.
- [ ] A second sync of an unchanged folder logs zero events.
- [ ] Editing one section logs exactly one `revise` for that ordinal, with
  differing old and new digests.
- [ ] Retag with an unknown tag fails with a one-line domain error; a no-op
  retag logs nothing.
- [ ] YAML-only mode works with no tiferet-h5 installed; `[h5]` mode creates
  blosc-filtered tables.

- major@grokbot via tif-ai
