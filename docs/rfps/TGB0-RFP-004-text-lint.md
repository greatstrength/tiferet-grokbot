# TGB0-RFP-004 - Text Lint: ASCII and Signature Lines

**Status:** Draft | **Domain:** `grokbot` | **Branch:** docs-core-domain | **Issue:** not published
**Related:** `docs/domain.md` sections 2.14, 3.2, 3.7
**Depends on:** none - pure checks; the `grokbot lint` command lands with the CLI (TGB0-RFP-005)
**Blocks:** TGB0-RFP-005 (the CLI exposes `grokbot lint`), TGB0-RFP-006 (advisory policy checks reuse lint findings)

## 1. Summary

Ship lint rules that catch the cheapest recurring defects before anything is
posted or committed (domain.md 2.14): em and en dashes, smart quotes, and a
missing shared-account signature line. Lint is advisory in v0.1 (3.7). New
content the package writes is strict ASCII by default (3.2).

## 2. Motivation

Dash and quote drift and missing attribution show up constantly in bot-written
text. Fixing them after the fact is noisy. A one-command check, and a
normalizer used on import and on write, removes a whole class of review
findings.

## 3. Current state

- Normalization of the six characters (U+2014, U+2013, U+2018, U+2019,
  U+201C, U+201D) on skill import is proven.
- Strict ASCII output for package-generated text (`to_ascii` with
  transliteration) is proven.
- Shared bot accounts sign with a final line
  `- <agent>@<platform> via <account>` (binding supplies the platform and
  account). Co-Authored-By trailers are not used on shared accounts.

## 4. Proposal

1. **`normalize_ascii(text)`.** Map the six dash and quote characters; leave
   other non-ASCII unchanged. Used on skill import.
2. **`to_ascii(text)`.** Transliterate to strict ASCII for new content the
   package writes (events, role text, rendered tables, lint messages).
3. **`lint_text(text, rules, binding)`.** Returns a list of findings
   (rule id, path or span, message). Built-in rules: `ascii_dashes_quotes`,
   `signature_line`.
4. **Signature rule.** When the binding says the actor uses a shared bot
   account, require a final line matching the configured format. Do not
   require Co-Authored-By.
5. **CLI.** `grokbot lint PATH...` prints findings and exits 0 when clean, 1
   when findings exist (advisory; does not block a hook unless configured
   later).
6. **Config.** Teams can disable a rule or add paths to skip.

## 5. Out of scope

- Guardrail policy checks (merge, tag, experiment-without-RFP): TGB0-RFP-006.
- Document template and section-order validation: TGB0-RFP-007.
- Enforcing lint in a git hook (later decision; v0.1 is advisory).

## 6. Risks and open questions

- **Resolved:** advisory first (maintainer default).
- **Resolved:** signature format comes from binding, not hardcoded account
  names.
- **Open:** whether lint should rewrite files in place under a flag, or only
  report.
- **Open:** non-ASCII that teams want to keep in prose (for example arrows in
  skill files): normalize on import for the six characters only; `to_ascii`
  is for package output.

## 7. Acceptance criteria

- [ ] `normalize_ascii` converts the six dash and quote characters and leaves
  other non-ASCII unchanged.
- [ ] `to_ascii` output is strict ASCII for representative inputs (dashes,
  arrows, accents, ellipses).
- [ ] `lint_text` reports em/en dashes and smart quotes with rule id
  `ascii_dashes_quotes`.
- [ ] With a shared-account binding, a body missing the signature line reports
  `signature_line`; a correct final line is clean.
- [ ] `grokbot lint` exits 0 on clean input and 1 on findings.
- [ ] No finding text embeds a real team's account name; examples use binding
  placeholders.

- major@grokbot via tif-ai
