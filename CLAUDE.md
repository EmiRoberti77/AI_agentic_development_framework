# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

A **documentation-only** repository: two companion technical guides on building production
agentic AI systems, authored by Emiliano Roberti. There is no application code, no build
system, no package manifest, and no test suite. The deliverable *is* the Markdown — written
to render directly on GitHub (prose, code blocks, tables, and inline SVG diagrams).

There are no build, lint, or test commands. Validation is by reading the rendered Markdown
and checking that links and diagram paths resolve.

## Structure

- `01-architecting-agentic-ai/` — the **architecture-level** guide (14 sections, 11 diagrams):
  *what* to build and *why* — data structures, retrieval, RAG, frameworks, model
  customization, the safety boundary.
- `02-langchain-langgraph-implementation/` — the **implementation-level** guide (12 sections,
  5 diagrams): *how* to build it on the LangChain 1.0 / LangGraph 1.0 runtime.
- `assets/diagrams/agentic/*.svg` and `assets/diagrams/langgraph/*.svg` — diagrams referenced
  by sections in guide 01 and 02 respectively.
- Each guide has a `README.md` (overview + ordered table of contents); the root `README.md`
  ties both together.

Section files are named `NN-kebab-case-title.md` and are meant to be read in numeric order.
The two guides are designed as a pair: guide 01 decides *what/why*, guide 02 explains *how*.

## Authoring conventions (follow these exactly when editing or adding sections)

- **Navigation bars.** Every section file carries an identical nav bar at the **top and
  bottom**, linking previous · index · next. First and last sections omit the missing
  neighbor. Format:
  `[← Previous Title](NN-prev.md) · [Guide index](README.md) · [Next Title →](NN-next.md)`
  After the top bar, a `---` rule precedes the body; a `---` rule precedes the bottom bar.
- **Adding/removing/reordering a section** is a multi-file edit: update the new file's nav
  bar, the nav bars of *both* adjacent files, the guide `README.md` table of contents, and
  the section count in the guide README intro (and root README if it cites counts).
- **Diagrams.** Reference SVGs with a relative path from the section file, e.g.
  `../assets/diagrams/agentic/figNN.svg`. The pattern is an image with a short alt caption,
  immediately followed by a full italic caption line starting `***Figure N.**` (guide 01) or
  `***Fig. N** —` (guide 02). Keep the numbering consistent with the rest of that guide.
- **Callouts** use blockquote lines with bold tags: `> **KEY — ...**`, `> **WARNING — ...**`,
  prose-leading `>` quotes for section epigraphs. Match the surrounding style rather than
  introducing new callout types.
- **Cross-references** within a guide cite sections by `§N` (e.g. "the trust boundary (§11)").
- **Voice.** Mechanism first, then the engineering decision (when to use it, what it costs,
  how it fails). The guides intentionally describe patterns meant to outlive specific tool
  versions; version pins (e.g. `langchain 1.3.x`) appear in the implementation guide's intro.

## Footer / authorship

Author attribution (Emiliano Roberti) and the `© 2026` line in the root README are
intentional — preserve them.
