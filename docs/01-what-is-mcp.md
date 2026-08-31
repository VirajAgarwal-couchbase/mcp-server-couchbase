# Chapter 1: What is MCP?

## The problem: an AI assistant that can't actually do anything

Talk to a chatbot and it can tell you *about* things — it was trained on a
huge amount of text, so it can explain what a database index is, or draft
you a SQL query. But it can't check whether *your* database actually has
that index. It can't run the query against *your* data. Everything it knows
is frozen at training time, and everything it says is a guess dressed up as
an answer, because it has no way to go look.

That's fine for a lot of questions. It falls apart the moment you want the
assistant to *do* something with live, specific, external state: "is the
`orders` bucket healthy right now", "insert this document", "which of my
queries are slow this week". The model needs a way to reach outside itself
— to call a real function, get a real result back, and reason about that
result. That capability is usually called **tool use** or **function
calling**, and most modern AI assistants support some form of it.

The trouble is *how* they reach outside themselves. Before MCP, if you
wanted Claude Desktop, Cursor, and five other AI tools to all be able to
query your database, you'd write a different integration for each one —
they each expected tool definitions and results shaped their own way. Add a
second data source and multiply the integrations again. Every assistant
times every tool: an N-clients × M-tools problem.

## The fix: one protocol, both sides implement it once

The **Model Context Protocol (MCP)** is an open standard, published by
Anthropic, that defines a single way for an AI application to discover and
call external tools — regardless of which AI model or which tool is on the
other end. It turns the N×M integration problem into N+M: a tool author
implements the protocol once (as an **MCP server**), and an assistant author
implements the protocol once (as an **MCP client**), and after that, every
MCP server works with every MCP client without either side knowing anything
specific about the other.

Three pieces make this work:

- **MCP server** — a small program that exposes a specific capability
  (querying a database, reading files, calling an API) as a set of
  well-described **tools**. It knows nothing about which AI model is
  calling it.
- **MCP client** — lives inside the AI application (Claude Desktop, Cursor,
  VS Code, etc.). It connects to one or more MCP servers, asks each one
  "what tools do you have?", and hands that list to the model.
- **The protocol itself** — a JSON-RPC-based message format for listing
  tools, calling a tool with arguments, and returning a result, sent over a
  **transport** (see below for the options).

The server describes each tool with a name, a plain-language description,
and a schema for its parameters — the same shape an API endpoint's docs
would have. The model reads those descriptions, decides a tool call is
useful for answering the user, and the client executes it on the model's
behalf.

## A worked example

Say an MCP server exposes one tool:

```json
{
  "name": "get_weather",
  "description": "Get the current weather for a city.",
  "inputSchema": {
    "type": "object",
    "properties": { "city": { "type": "string" } },
    "required": ["city"]
  }
}
```

You ask your AI assistant: *"Should I bring an umbrella in Boston today?"*

1. The client already fetched this tool list from the server at
   connection time and gave it to the model.
2. The model reads the description, recognizes it's relevant, and asks the
   client to call `get_weather` with `{"city": "Boston"}` — it doesn't run
   any code itself; it just requests the call.
3. The client sends that request to the MCP server over the transport.
4. The server runs the *actual* logic — calls a real weather API, gets back
   `{"condition": "rain", "precip_chance": 80}` — and returns that as the
   tool's result.
5. The client hands the result back to the model, which now has a real,
   current fact to reason from and answers: *"Yes — 80% chance of rain in
   Boston today."*

Nothing about steps 3–4 involved the model. The server did real work against
a real, live system; the model's job was only to decide *when* to ask for
that work and *how* to use the answer. That division of labor — the model
decides, the server executes — is the whole idea. It's also why tool
descriptions matter so much: the model's only guide to what a tool does and
when to use it is the text the server author wrote.

## Transports: how the messages actually travel

MCP defines a couple of transports for carrying those JSON-RPC messages
between client and server:

- **STDIO** — the client launches the server as a local subprocess and talks
  to it over its standard input/output streams. This is what most desktop
  AI apps use for locally-installed servers: no network involved, one
  client per server process.
- **Streamable HTTP** — the server runs independently (e.g. as a long-lived
  process or in a container) and clients connect to it over HTTP. This lets
  multiple clients share one running server, and supports standard HTTP
  auth (MCP also defines an OAuth 2.1 flow for this transport).
- **SSE (Server-Sent Events)** — an older HTTP-based transport, now
  deprecated in favor of Streamable HTTP, but still seen in the wild.

Which transport a given server supports (and which one you pick) affects
*how* you connect, not *what* the tools do — the tool-calling model above is
identical regardless of transport.

## Where this leaves us

MCP's job is narrow and deliberately so: describe tools, carry tool calls
and results, standardize how any client and any server negotiate that. It
doesn't say anything about what a specific server's tools should *be* — that
part is up to the server author, shaped by whatever system they're exposing.

That's where the next chapter picks up: it looks at what an MCP server looks
like when the system behind it is a real, live Couchbase cluster — schemas
to discover, writes that need guardrails, queries that need to be safe to
run on someone's production data.
