# Narrative Docs — Chapter Series Guide

This `docs/` directory is a **story-style explainer** of the Couchbase MCP
Server, separate from the API reference in the root `README.md` and
`DOCKER.md`. It exists for readers who want to understand *what MCP is, why
this server exists, how it actually works under the hood, and how to run and
operate it* — building from zero knowledge up to advanced operational detail.

It is not:

- A replacement for README.md/DOCKER.md (those stay the authoritative,
  scannable reference for flags, env vars, and tool tables).
- API documentation. Don't restate parameter lists verbatim — link to the
  README section instead when a chapter needs to reference exact flags.
- A place for marketing copy or sales language.

## Audience

Someone who is comfortable reading code and running a terminal, but has
**never used MCP before** and may not know Couchbase well either. Assume
curiosity, not prior context. Each chapter should be readable on its own,
but later chapters can assume earlier ones were read.

## Tone and style rules

- Plain language first, jargon second. If you introduce a term (e.g.
  "transport", "elicitation", "JWKS"), define it in one sentence the first
  time it appears in a chapter, even if a prior chapter already defined it.
- Friendly and direct — write like you're explaining this to a colleague at
  a whiteboard, not writing a spec. Short paragraphs, concrete examples.
- Prefer a small worked example over an abstract description whenever one is
  possible. Show real tool names, real env vars, real command output shapes
  from *this* codebase — never invented API surface.
- No filler enthusiasm ("Exciting!", "Powerful!"). Let the mechanics speak.
- Every claim about behavior, defaults, or flags must be verified against the
  current code before being written down. If a chapter describes something
  covered by README.md/DOCKER.md, link there rather than re-deriving details
  that could drift.
- Diagrams are welcome as plain ASCII/mermaid code blocks, not images.
- Each chapter is a single Markdown file: `docs/chapter-N-slug.md`.
- Keep chapters focused — one concept arc per chapter. If a chapter is
  sprawling, that's a sign it should split into two chapters (update the
  outline below if so).

## Chapter outline (planned)

Numbers are stable once a chapter is written; only the *unwritten* tail may
be resequenced.

1. **MCP basics** — what MCP is, the problem it solves (why "just give the
   LLM API docs" doesn't scale), a plain-language worked example of a tool
   call round-trip.
2. **Why a Couchbase MCP server** — what this server adds on top of raw MCP:
   cluster/bucket/scope/collection model, read-only-by-default safety,
   RBAC as the real security boundary, where this fits next to the
   Couchbase SDK/CLI/Capella UI.
3. **A tour of the tools** — walking through the tool categories (cluster
   health, schema discovery, KV, query, query performance) at a narrative
   level — what each *category* is for and when you'd reach for it, not a
   restated parameter table.
4. **Running the server** — STDIO vs Streamable HTTP vs SSE transports,
   picking one for your use case, connecting from Claude Desktop/Cursor/VS
   Code at a conceptual level (link to README for exact JSON configs).
5. **Operating it safely** — read-only mode, disabling tools, confirmation/
   elicitation, RBAC, and how these layers compose (this is the "why" behind
   README's safety features, not a restatement of them).
6. **Deploying for a team** — Docker image, Streamable HTTP for shared/remote
   deployments, OAuth 2.1 as a resource server, logging and diagnostics for
   support.
7. **Extending the server** — how a new tool gets added (tool categories in
   `src/cb_mcp/tools/__init__.py`, annotations, read-only gating), for
   readers who want to contribute.

Feel free to insert a chapter between existing ones if a gap becomes obvious
mid-series — append it with the next free number and note the resequencing
here if any *unwritten* chapter numbers shifted.

## How to add the next chapter (process for a future run)

1. Read this file in full, then skim every existing `docs/chapter-*.md` file
   (titles and intros are enough unless the new chapter builds directly on
   one's content).
2. Find the first item in the outline above with no corresponding file yet —
   that's the next chapter to write.
3. Verify any code-derived claims you plan to make (tool names, defaults,
   flags, behavior) against the current source — do not trust this file or
   older chapters as ground truth for code facts, since code moves faster
   than docs.
4. Write **only that one chapter** as `docs/chapter-N-slug.md`. Do not edit
   earlier chapters' content unless they contain a factual error about the
   current code — if you must fix one, keep the fix minimal and call it out
   explicitly as a correction in the PR body (this mirrors the accuracy-pass
   discipline used for README/DOCKER elsewhere in this repo).
5. Update `docs/README.md` (the index/table of contents) to link the new
   chapter.
6. If the outline above needs adjusting (split, reorder, new topic
   discovered while writing), update it in this file as part of the same
   change.
