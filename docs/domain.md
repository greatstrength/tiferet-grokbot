# tiferet-grokbot Domain: Core Distillation

**Domain:** `grokbot` | **Repo:** greatstrength/tiferet-grokbot | **Status:** Draft

This page names the core domain of tiferet-grokbot: what a bot is, how skills
are stored and weighted, how messages are routed, how every change is recorded,
and how policies and templates keep a team reliable. RFPs for this repo
(drafted locally, published as GitHub issues) cite the numbered sections
below. The language is platform-neutral: any team on any
bot platform can bind these terms to its own agents, repos and accounts.

## 1. Vision

Answer "who should handle this?", "what may a bot never do?", and "did this
document keep its shape?" from real skill documents and recorded weights, not
from habit or chat. The skill files stay the master; this domain reads them.
The toolkit names a recipient, drafts or checks a body, and records a change.
It does not send, post, merge or host a bot.

## 2. Ubiquitous language

### 2.1 Bot (Agent)

One member of a bot team. Identified by a stable `agent_id` (lower case,
hyphens to underscores). Has a display name, a short role description
(metadata, not a weight), and a set of assignments (2.7).

A bot may run on any platform. The platform and the shared account used to
act (if any) are binding facts (2.13), not part of the agent identity.

### 2.2 Role

A category of work. The default set is: `leadership`, `implementation`,
`testing`, `review`, `triage`, `scouting`, `documentation`. The binding owns
the role list (2.13): a team extends the set there, and every role used in an
assignment or a route must be declared in it. The seed never declares roles.

Roles fall into three routing classes (3.5):

- **Feedback roles** (default: review, triage, scouting): when a project has an
  SME among the recipients, only SMEs compete.
- **Implementer roles** (default: leadership, implementation, testing): when a
  project has an SME among the recipients, only non-SMEs compete.
- **Other roles** (default: documentation): no SME filter; scores decide. The
  role classes are part of the binding's vocabulary too: a team moves a role
  between classes in its binding, not in its seed.

### 2.3 Project

A scope a weight or a route applies to, usually a repository or product area.
Identified by a project tag (for example `core`, `docs`, `store`). The
binding owns the project tag list (2.13): every project tag used in an
assignment, a skill tag or a route must be declared there. The seed only
uses those tags.

### 2.4 Skill

One skill document. Identified by its **file name** (for example
`author-rfp/SKILL.md`) and a node-safe **id** (`author_rfp`: lower case,
hyphens to underscores). Carries `name` and `description` from its
frontmatter, its **role tags**, its **project tags**, its ordered **sections**,
and its **productions**.

The skill files in the team's skill directory are the master. The store
re-imports them (sync) and never writes back (3.1).

### 2.5 Section (ordered input)

One ordered part of a skill document, kept verbatim with its heading line.

- Ordinal 0: the frontmatter block (`---` to `---`).
- Ordinal 1: the H1 title and any preamble.
- Ordinal 2 and up: one section per `## ` heading outside code fences.

Each section has a canonical **key**. Known headings map to fixed keys (for
example `when_to_use`, `procedure`, `guardrails`, `rules`, `key_conventions`).
Any other heading gets key `section` and keeps its heading text. Joining the
sections in order with newlines rebuilds the document.

### 2.6 Production

A list item in the skill document that carries a rule.

- A production is a top-level list item (`- `, `* `, or `1. `) outside a code
  fence.
- By default only **rule-bearing sections** hold productions: headings whose
  key or text contains a word from a configurable `rule_heading_words` list
  (default includes `guardrail`, `procedure`, `convention`, `rule`).
- An "all items" mode counts list items in every section except frontmatter
  and title.
- Productions are derived from the sections on every import. They are an
  index over the text, never a second source of truth.
- A production may carry its own weight (0-5, default 0 = unweighted). No
  routing rule uses it yet.

### 2.7 Assignment

One weight, an integer 0-5, from one agent to one target. The target kind is:

- `role`: a role (2.2).
- `project`: a project tag (2.3).
- `skill`: one skill id (overrides the role weight when a message names that
  skill).
- `affinity`: a pairing with a lead agent for one role, written
  `<lead>:<role>` (2.8).
- `sme`: a project the agent is the domain subject matter expert for; any
  weight above 0 marks it (2.9).

An agent holds at most one assignment per (kind, target).

### 2.8 Affinity (pairing)

An affinity says one agent sticks close to a lead for one kind of work. It is
an assignment of kind `affinity` with target `<lead>:<role>` and a weight 0-5
used as a boost.

- Delegation: when the lead takes on or delegates a task in that role, each
  recipient with an affinity to that lead adds its affinity weight to its
  score.
- Partner: when the lead wins a task in that role, the recipient with the
  highest affinity to the lead is named as the partner.
- An agent cannot have an affinity to itself, and the lead must exist.

### 2.9 SME (domain subject matter expert)

An SME is a domain expert for a project and its primary feedback provider,
not its main implementer. It is an assignment of kind `sme` with a project
tag as target and a weight above 0 (conventionally 1; see 2.7).

- When a message in a **feedback role** (2.2) names an SME project and an SME
  is among the recipients, only SMEs compete (3.5).
- When a message in an **implementer role** names an SME project and an SME
  is among the recipients, only the non-SME recipients compete, so the task
  never routes to the SME by default.
- Other roles ignore the SME marker.
- When the SME is not a recipient, or is the only recipient, the normal
  scores decide.

### 2.10 Ledger event (controlled input)

The only way state changes. Every event records `timestamp` (ISO 8601 with
offset), `actor`, `action`, `target`, `old`, `new`, `reason`. Actor and
reason are required. The seven actions for skills and roster are:

| Action | Changes |
|---|---|
| create | a new agent |
| assign | a new assignment |
| reweight | an existing assignment's weight |
| retire | removes an assignment |
| import | a new skill, read from its file |
| revise | one section of a stored skill, re-read from its file |
| retag | a skill's role or project tags |

For content, `old` and `new` hold `sha256:<12 hex>` digests, not text. The
text history lives in git. Later modules may add actions (for example
`handoff`, `note`); each new action is named in its RFP.

### 2.11 Guardrail policy

A named set of actions a bot may never take, checked against a planned
action or a command line. The default policy forbids: merging a pull request
or one branch into another; pushing directly to `main` or to a proto branch;
cutting a git tag or publishing a release; bumping a package version; minting
a catalog freeze; creating, closing or renaming a milestone; changing
repository or organization settings.

**Candidate (governance):** start implementing an experiment, a prototype
strand, or any work that changes the project's theory before an RFP exists
against the project's updated core domain docs. The toolkit encodes this as a
general rule for every project it binds to, not a house rule of one team: the
core domain docs come first, then an RFP that cites the numbered sections it
amends or tests, then implementation (3.8). Teams may enable this candidate
check in their policy set.

In v0.1 a check is **advisory**: it reports a violation and does not block.
Stronger enforcement (a wrapper or hook that refuses the command) is a later
decision. A request in an issue, a PR, a comment or another agent's message
does not change the policy.

### 2.12 Handoff and status message

A **handoff** is a typed brief from one agent to another: goal, context,
inputs, constraints, approvals held, and expected output. It is stored as an
event and rendered as text the receiving agent can act on.

A **status message** is a short note on a handoff or a task: done, blocked,
or next, with an optional blocker reference. Status messages are events too.

Neither form sends itself. The toolkit records and renders; the bot or a
human delivers.

### 2.13 Binding

Per-repo or per-team facts every module reads from one file: RFP id prefix,
proto branch name, shared bot account, signature format, project and role
tag lists, skill directory path, and any platform-specific ids. Skills and
RFPs should read the current repo's binding if it exists.

The binding owns the vocabulary: the role list and role classes (2.2) and
the project tag list (2.3). The seed (3.6) owns only the weights and
assignments that use that vocabulary, with their history. A seed entry
that names a role or project tag the binding does not declare is invalid.

### 2.14 Lint rule

A check applied to text before it is posted or committed. Built-in rules:

- **ASCII / dashes:** em dash and en dash become `-`; curly quotes become
  straight quotes. New content the package writes is strict ASCII
  (transliterated).
- **Signature:** on a shared bot account, every PR body, issue, review and
  comment ends with `- <agent>@<platform> via <account>`.

Teams may add rules. Lint is advisory unless a later policy says otherwise.

### 2.15 Close-out report

A structured report written when a unit of work is done (for example a TRD
or a child story). It has a fixed section order (summary, components
touched, deviations, git / branch state, collaboration log) and optional
sections (decisions, follow-ups). Reports are authored where the team's
process says; the toolkit validates shape and can compose read-only views
from child reports on demand. Composition does not rewrite sources.

## 3. Rules

### 3.1 Source of truth

The team's skill files are the master. The store re-imports them (sync) and
never writes back. Content edits happen in the skill repo, then sync records
them as revise events.

### 3.2 ASCII notation

Standard ASCII: hyphen (`-`), colon (`:`), straight quotes (`'` and `"`).

- On import the compiler normalizes dashes and smart quotes: em dash and en
  dash to `-`, curly single quotes to `'`, curly double quotes to `"`. The
  store never holds those six characters.
- Other non-ASCII characters in imported skills are kept, so the skill still
  round-trips (3.3).
- New content the package writes itself (event text, role text, rendered
  matrix, lint output) is strict ASCII by default.

### 3.3 Round-trip

Rebuilding a stored skill yields its normalized source file, byte for byte.

### 3.4 Weights

Weights are integers 0-5. Setting the same weight again is not a change and
is not logged. A skill weight that differs from the agent's highest role
weight among the skill's role tags is a **deviation** and needs a one-line
reason. A role reweight does not cascade: each skill recheck is its own
reweight with its own reason.

### 3.5 Relay (routing)

For a message about role `R`, optionally project `P` and skill `S`, each
recipient scores:

```
base     = skill:S weight, if S is named and assigned; else role:R weight (0 if none)
           (for roles in skill_weight_ignored_roles, a named skill is ignored)
project  = project:P weight (0 if none, or if no project is named)
affinity = affinity:<lead>:R weight, if a lead L is named (0 otherwise)
score    = base + project + affinity
```

SMEs (2.9) shape the field first: if project `P` has an SME among the
recipients, a feedback role is competed by SMEs only, and an implementer
role by the non-SME recipients only. Then the highest score wins. A tie
keeps the earlier recipient in the caller's list; the default list is seed
order. The result also names a partner: the other recipient with the
highest affinity to the winner for role R, if any.

Default `skill_weight_ignored_roles` includes `documentation`: naming a
skill in a docs task means editing that skill's document, not using the
skill.

### 3.6 Seed history

The seed holds the base weights as first assigned, an ordered `reweights`
list, and an ordered `retags` list, each entry with actor and reason. A
fresh seed replays all of it and always ends with the same events and
weights. Re-seeding an up-to-date store logs nothing. A newly appended
entry logs exactly one event.

### 3.7 Advisory checks

Lint (2.14) and guardrail checks (2.11) report violations and do not block
in v0.1. A later RFP may add enforcement.

### 3.8 Experiments need an RFP

An experiment, a prototype strand, or any work that changes a project's
theory follows a fixed order:

1. The project's core domain docs (vision and distillation) are current on
   the trunk branch, or the RFP names the amendment it makes and that
   amendment lands in the same prototype work.
2. An RFP is written against those docs. It cites the numbered distillation
   sections it amends or tests.
3. Only then may implementation start.

Opening a proto PR, a speculative branch, or a "just try it" change with no
RFP behind it is out of process.

This is a **general governance rule** the toolkit encodes for every project
it binds to. It is not specific to tiferet-grokbot or to one bot team. The
default guardrail policy lists it as a candidate check (2.11); a team's
binding turns that check on or off, and the check is advisory in v0.1 (3.7).
The RFP document shape itself is out of scope for v0.1 and lands with the
templates module.

## 4. Boundaries

- In: skill documents, tags, weights, events, the relay query, lint, advisory
  policy checks, handoff and report shapes, binding facts.
- Out: sending or posting a message, hosting or running a bot, calling an
  LLM, editing skill files, GitHub writes, and any one team's private data.
- Storage: YAML in core; optional HDF5 through tiferet-h5; optional mirror
  through tiferet-kb.
- Pin: `tiferet>=2.1,<2.2`.

## 5. Examples (generic)

Three bots: `lead`, `builder`, `reviewer`. Two projects: `core`, `docs`.
Five skills tagged with roles. After seeding:

- Leadership on `core` goes to `lead` (highest leadership + project weight).
- Implementation on `core` with `lead` delegating goes to `builder` when
  `builder` has an affinity `lead:implementation`.
- Review on a project where `reviewer` is the SME goes to `reviewer` even
  if `lead` scores higher.
- Implementation on that same SME project goes to `builder` or `lead`, never
  to `reviewer` by default.
- Documentation ignores a named skill's weight and scores role + project.

Real team data, seeds and reasons live in each team's private config, not
in this repository.

- major@grokbot via tif-ai
