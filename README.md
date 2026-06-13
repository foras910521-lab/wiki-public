# wiki-public

A curated, **public-friendly** subset of the local Hermes Agent Wiki.

This is the parts of the Wiki that are useful to the broader Hermes community
without leaking personal decisions, holdings, conversations, or NAS-local
configuration. The full Wiki (4899 markdown files) lives locally and on the
author's NAS — it includes stock research, WeChat group knowledge assets,
private operational details, and other content not appropriate for a public
repo.

## Scope of this repo

What's **here** (governance and methodology that generalizes):

- `system/hermes-ops/` — governance docs for Hermes config sync, key
  management follow-up, and local toolchain (Homebrew on macOS).
- `concepts/cross-domain/` — cross-domain concept notes (currently the
  wikilinks reference).
- `entities/hermes-ops/` — concept cards for Hermes-specific terms
  (Roundtable Runtime, Hermes Gateway, Hindsight, LLM Wiki).
- `learning/` — synthesis notes from external sources (kevimaistack Hermes
  series, external tools evaluation).
- `areas/personal-knowledge-radar/` — methodology for personal knowledge
  capture (cardbox templates and rationale). **Sub-directories**
  (`cards/`, `daily-actions/`, `kanban-poc-2026-05-17/`, `tools/`) are
  deliberately excluded — they contain personal data.
- `SCHEMA.md` — the LLM Wiki / Obsidian Vault schema definition.

What's **deliberately not here**:

- `concepts/hermes-ops/stock-roundtable-*` — stock-related operational
  protocols (private decision frameworks).
- `concepts/stock/`, `entities/stock/`, `cases/stock/`, `queries/stock/`,
  `projects/stock/`, `areas/stock-data-tools/` — all stock research.
- `concepts/self-media/`, `experiments/self-media/`, `projects/self-media/`,
  `queries/self-media/`, `system/self-media/`, `tools/self-media/`,
  `style-system/self-media/`, `_imports/self-media/`,
  `content-drafts/self-media/` — self-media notes (unreviewed for sharing).
- `wechat-knowledge-assets/` — WeChat group knowledge corpus (private).
- `_meta/`, `_archive/`, `_imports/`, `.obsidian/`, `.hermes/` —
  meta-internal artifacts and Obsidian app state.
- Sub-directories under `areas/personal-knowledge-radar/` — personal
  captured knowledge, not methodology.

## Sanitization

All environment-specific identifiers — home paths, LAN IPs, hostnames,
account IDs, SSH aliases, NAS internal data roots — have been mechanically
replaced with explicit `<...>` placeholders before publishing. No
machine-identifying string from the author's environment appears in any
committed file in this repository.

If you find one, file an issue — that's a sanitization gap, not a feature.

The Hermes convention `~/.hermes` (without an explicit absolute prefix) is
**kept as-is** because it is the public path documented by the Hermes Agent
project and is not personal.

Wikilinks to local-only pages (`[[some-private-page]]`) may still appear in
some files; those will resolve as broken links in a fresh clone. This is
intentional — it signals "this concept is local-only".

## Reading order (suggested)

1. `SCHEMA.md` — what the Wiki **is** and how it is structured.
2. `system/hermes-ops/hermes-config-sync-governance.md` — what belongs in
   `config.yaml` vs `.env` (a core boundary rule).
3. `concepts/cross-domain/wikilinks.md` — the cross-domain navigation rule.
4. `entities/hermes-ops/*.md` — concept cards for the Hermes stack.
5. `learning/kevimaistack-hermes-series-synthesis-2026-06-07.md` — distilled
   external reading on the Hermes series.
6. `system/hermes-ops/hermes-key-system-followup-checklist.md` —
   what to do after touching a key.
7. `areas/personal-knowledge-radar/personal-knowledge-cardbox-v1.md` and
   `cardbox-templates-v1.md` — personal knowledge capture templates.

## Provenance

Maintained by foras910521-lab. Wiki content was produced during the
2026-06-06 migration of an LLM Wiki from three legacy vaults to a single
Obsidian-compatible structure. All artifacts were self-collected; no
external content is mirrored without explicit attribution in the source
files.

## License

MIT for original content. Where a page is a synthesis of an external
source, the source link is preserved in the file.