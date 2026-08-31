# Narrative Docs — Series Guide

This `docs/` directory is a friendly, story-style explainer of the Couchbase MCP
Server — **not** API reference. API/config reference already lives in
[README.md](../README.md) and [DOCKER.md](../DOCKER.md); this series exists to
build the *mental model* that makes that reference make sense. It's written for
a reader who is new to MCP, or new to this server, and wants to go from "what
is this thing" to "I can run and operate this in production" one chapter at a
time.

## Structure

- One file per chapter, named `NN-slug.md` (e.g. `01-what-is-mcp.md`),
  numbered in reading order.
- [`index.md`](index.md) is the table of contents — one line per chapter, kept
  in sync whenever a chapter is added.
- Chapters build on each other. Chapter *N* may assume everything taught in
  chapters `1..N-1`, but should not assume anything from later chapters.

## Planned chapter list

This is the full outline. Only write the *next unwritten* chapter in any
given run — see "How to pick the next chapter" below.

1. **What is MCP?** — the problem MCP solves, plain-language, with a tiny
   worked example. (No Couchbase specifics yet.)
2. **Why a Couchbase MCP Server?** — what changes when the tool-caller is
   pointed at a real database; the shape of the problem (agents need schema
   discovery, safe writes, query performance visibility) and how this server
   answers it.
3. **A tour of the tools** — walk the tool catalog by category (cluster
   health, schema discovery, KV, query, indexing, query performance), grouped
   the same way `src/cb_mcp/tools/__init__.py` groups them, with a couple of
   worked natural-language-prompt examples per category.
4. **How it's built** — the architecture: FastMCP, the `ClusterProvider`
   contract (`src/cb_mcp/core/contracts.py`) that lets the same tool code run
   standalone or inside Capella's managed runtime, the lifespan-scoped
   `AppContext`, and how a tool call flows from MCP request to Couchbase SDK
   call and back.
5. **The safety model** — read-only mode, per-tool disabling, elicitation/
   confirmation-required tools, and why RBAC on the Couchbase user is the
   real security boundary (tool gating is a convenience layer on top of it).
6. **Running it** — the three transports (stdio, Streamable HTTP, SSE),
   PyPI vs. source vs. Docker, and picking the right one for a given client.
7. **Operating it in production** — logging and log rotation, OAuth 2.1 as a
   resource server, diagnostics/health tools, and troubleshooting playbooks.
8. **Extending the server** — for readers who want to add a new tool: the
   registration checklist, tool annotations, and where read-only/write
   classification and the disabled/confirmation-required tool lists plug in.

Chapter boundaries above are a starting plan, not a contract — if, when
you sit down to write chapter *N*, the natural split has shifted (e.g.
material turns out to belong a chapter earlier or later), adjust the outline
and say so in the PR description. Don't silently drift from it without a note.

## Tone and style

- **Plain language first.** Explain the concept before naming it. Define
  jargon the first time it's used.
- **Second person, conversational, not breathless.** Talk to "you", the
  reader. No marketing superlatives ("blazing fast", "seamless").
- **Concrete over abstract.** Every concept gets a small worked example —
  an actual prompt, an actual tool call, actual JSON — not just prose.
- **Honest about tradeoffs and limits.** If something is a convenience layer
  and not a security boundary (e.g. tool disabling), say so plainly, the way
  README.md does.
- **Couchbase-specific chapters ground claims in the code.** When a chapter
  describes behavior (tool lists, config defaults, safety semantics), check
  it against the current source in `src/cb_mcp/` rather than against a
  previous chapter's memory of it — the code is the source of truth.
- **Short sections, descriptive headers.** Prefer several short sections a
  reader can skim over one long unbroken one.
- **No API-reference duplication.** Don't restate the full tool/parameter
  tables from README.md — link to them. This series explains *why* and
  *how things fit together*; README/DOCKER explain *exact parameters*.

## How to pick the next chapter (process for future runs)

1. Read this file's "Planned chapter list" and `index.md`.
2. Find the highest-numbered chapter file that already exists in `docs/`.
3. Write **only** the next chapter in the outline — do not write two chapters
   in one run, and do not rewrite existing chapters.
4. Exception: if an existing chapter contains a claim that's now factually
   wrong about the code (e.g. a tool was renamed or removed), fix *only*
   that claim, minimally, and call it out explicitly as a correction in the
   PR description — don't reword or restyle the surrounding prose.
5. Update `index.md` to list the new chapter.
6. If all planned chapters exist, propose a new chapter 9+ outline extension
   here (in this file) before writing it, so the outline stays authoritative.
