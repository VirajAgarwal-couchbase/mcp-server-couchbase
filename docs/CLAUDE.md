# docs/ — narrative documentation series

This directory is a story-style, from-scratch explainer of the Couchbase MCP
Server. It is deliberately **not** API reference — the README and DOCKER.md
already cover flags, env vars, and tool tables precisely. This series exists
for the reader who wants to understand *what this thing is and why it's built
this way*, in prose, before or instead of skimming a table.

Chapters are added incrementally, one per maintenance run, following the
outline below. Do not write more than one new chapter per run.

## Planned chapter outline

1. **MCP basics** — what the Model Context Protocol is, the problem it
   solves, a small worked example. (Written.)
2. **Why a Couchbase MCP server** — what an AI agent can do against a
   database once it has one, and why hand-rolled tool code per-agent doesn't
   scale.
3. **Anatomy of this server** — the moving parts: `mcp_server.py` entrypoint,
   `tool_registration.py`, the `tools/` package, `ClusterProvider` in
   `core/contracts.py`, and how a tool call flows from client to Couchbase
   cluster and back.
4. **Read-only mode and safety controls** — `CB_MCP_READ_ONLY_MODE`,
   per-tool disabling, confirmation/elicitation, and why these exist as
   guardrails on top of (not instead of) Couchbase RBAC.
5. **Transports: STDIO, Streamable HTTP, and SSE** — when to use each, what
   changes operationally (one local process vs. a shared network service).
6. **OAuth 2.1 for remote deployments** — resource-server model, JWT/JWKS
   validation, scopes (`couchbase-mcp:read`/`couchbase-mcp:write`), and how
   this maps to the read-only/write tool split from chapter 4.
7. **Running it in Docker** — the image, relevant environment variables,
   and how this differs from running the PyPI package directly.
8. **Operating in production** — logging/log rotation, telemetry, cluster
   diagnostics tools, and troubleshooting a misbehaving deployment.
9. **Extending the server** — adding a new tool end to end (this overlaps
   with CONTRIBUTING.md's checklist, but tells it as a walkthrough with a
   concrete example tool rather than a checklist).

This list may be adjusted by a future chapter author if the code has evolved
in a way that changes what's worth explaining — note any such change
explicitly in that run's PR body.

## Writing style and tone

- Write for a curious engineer who has never used MCP, not for someone
  already fluent in the protocol. Define terms before using them.
- Prose first, building from basics to advanced. Bullet lists and tables are
  fine for enumerations, but the connective reasoning ("why this exists",
  "what problem this solves") should be sentences, not fragments.
- Use one small, concrete worked example per chapter where it helps — a
  literal request/response, a literal tool call, a literal config snippet.
  Prefer showing over asserting.
- Keep it friendly and direct. Second person ("you") is fine. Avoid marketing
  language ("blazing fast", "seamless") — this is an explainer, not a pitch.
- Ground every claim in the actual code or actual protocol behavior. If you
  are not sure something is still true, check the source before writing it
  down (see "Keeping chapters accurate" below).
- Each chapter should be readable on its own but may reference earlier
  chapters by name (e.g., "as chapter 1 covered, ...") rather than repeating
  their content.
- Target length: enough to properly explain the topic, not padded to hit a
  word count. A few hundred to ~1500 words is typical.

## File naming and structure

- Chapters live at `docs/chapter-N-slug.md` (e.g. `docs/chapter-1-what-is-mcp.md`).
- `docs/README.md` is the table of contents: a short intro to the series plus
  a linked list of chapters, updated every time a chapter is added.
- This file (`docs/CLAUDE.md`) is the only place that holds the full planned
  outline and process — don't duplicate the outline elsewhere.

## Process for adding the next chapter

1. Read this file in full, plus every existing chapter (don't skip — later
   chapters assume earlier ones and you need to avoid contradicting them).
2. Determine the next unwritten chapter in the outline above.
3. Verify against the current code that the topic's outline description is
   still accurate; adjust the chapter's scope slightly if the code has moved
   on, and note the adjustment in your PR body.
4. Write only that one chapter. Do not rewrite earlier chapters.
5. Update `docs/README.md`'s table of contents to link the new chapter.
6. Open a PR titled `docs: add chapter N — <title>`.

## Keeping chapters accurate

If, while writing a new chapter, you discover an **earlier** chapter now
makes a factually wrong claim about the code (not just stylistically dated),
fix only that specific claim — a minimal, surgical correction, the same
standard as the docs-accuracy track for README/DOCKER — and call it out
explicitly in the PR body as a correction, separate from the new chapter.
Do not use this as license to reorganize or reword prior chapters.
