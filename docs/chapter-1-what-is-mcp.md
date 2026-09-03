# Chapter 1: What is MCP, and why does it exist?

## The problem: every AI assistant needs its own glue code

Say you want to ask an AI assistant, in plain English, "how many active users do we have in the `users` collection?" The assistant itself — the language model — can't answer that. It has no idea your Couchbase cluster exists, let alone how to connect to it, authenticate, or run a query against it. Language models are good at understanding language and reasoning about text; they are not, by themselves, connected to anything.

So someone has to build a bridge: code that takes the model's intent ("run this query") and turns it into an actual database call, then hands the result back in a form the model can read. Before the Model Context Protocol existed, that bridge was usually bespoke. A team building a Claude-based support bot would write one integration to Couchbase. A different team using a different AI product would write another, incompatible one, to the same database. If either team switched AI vendors, the integration had to be rewritten from scratch, because there was no shared contract for "how a database exposes itself to an AI assistant."

This is the same problem that motivated USB, or ODBC for databases before it: N tools and M applications need N×M custom integrations, unless everyone agrees on one interface. Model Context Protocol (MCP) is that agreed interface, but for AI assistants and the external systems they need to act on.

## What MCP actually standardizes

MCP is an open protocol, introduced by Anthropic and now used across the AI assistant ecosystem, that defines a standard way for two things to talk to each other:

- An **MCP client** — the AI assistant, or the application embedding it (Claude Desktop, Cursor, VS Code with Copilot, a custom agent you've built, etc.)
- An **MCP server** — a program that exposes a specific capability (a database, a filesystem, a ticketing system, an internal API) through a small, well-defined set of operations

The protocol mostly standardizes three kinds of things a server can offer:

- **Tools** — actions the assistant can invoke, each with a name, a description, and a schema for its parameters. "Run this SQL++ query" or "fetch this document by ID" are tools.
- **Resources** — data the assistant can read, addressed by URI, similar in spirit to files.
- **Prompts** — reusable prompt templates the server can suggest to the client.

The Couchbase MCP Server you're reading about in this series is, almost entirely, a collection of **tools**. When you connect it to Claude Desktop or another MCP client, the client asks the server "what can you do?", the server answers with a list of tool definitions (names, descriptions, parameter schemas), and from then on the assistant can decide, based on what the user asks for, to call one of those tools — the same way it would decide to write a paragraph or run a calculation, except this time the "calculation" is a real network call into your Couchbase cluster.

Crucially, the protocol is the same regardless of which AI assistant is on the other end. A Couchbase MCP Server built once works with Claude, Cursor, Windsurf, VS Code Copilot, or any other client that speaks MCP — because the server doesn't integrate with the *assistant*, it integrates with the *protocol*.

## A worked example: one tool call, end to end

Suppose you've connected the Couchbase MCP Server to an MCP client and you type: "How many buckets do I have in this cluster?"

Roughly, here's what happens:

1. **Discovery (happens once, at connection time).** The client asked the server which tools exist. The server's answer included an entry like this (simplified from the real tool, `get_buckets_in_cluster`):

   ```json
   {
     "name": "get_buckets_in_cluster",
     "description": "Get a list of all the buckets in the cluster",
     "inputSchema": { "type": "object", "properties": {} }
   }
   ```

2. **Reasoning.** The model reads your message and the list of available tools, and decides `get_buckets_in_cluster` is the one that answers your question. It doesn't know how the tool is implemented — it only knows its name, description, and what parameters it takes.

3. **Tool call.** The client sends a request to the server, structurally similar to:

   ```json
   { "method": "tools/call", "params": { "name": "get_buckets_in_cluster", "arguments": {} } }
   ```

4. **Execution.** The server receives this, and its `get_buckets_in_cluster` function (defined in `src/cb_mcp/tools/server.py` in this codebase) uses an already-established connection to your Couchbase cluster to list the actual buckets — this is regular Python code calling the Couchbase SDK, nothing magical.

5. **Response.** The server returns a result, something like:

   ```json
   { "content": [ { "type": "text", "text": "[\"travel-sample\", \"orders\"]" } ] }
   ```

6. **Answer.** The model turns that structured result back into a natural-language reply: "You have two buckets: `travel-sample` and `orders`."

Notice what didn't have to happen: you didn't write a query, open a database client, or even know the tool's name. And notice what the server didn't have to do: it didn't need to know anything about Claude versus Cursor versus any other client — it just answered a standard MCP request the same way for all of them.

## Why this matters for a database in particular

A database is a good example of *why* MCP's tool model — small, named, schema'd actions, rather than a raw query console — is the right shape for AI access. Handing a language model unrestricted database access (e.g., "here's a connection string, write your own SQL++") is powerful but risky: the model could run something destructive, something slow enough to affect production, or something outside the scope it should have. By instead exposing specific tools — `get_document_by_id`, `run_sql_plus_plus_query`, `create_index`, and so on — the server gets to decide exactly what's possible, add guardrails around the dangerous ones, and describe each capability clearly enough that the model uses it correctly.

That guardrail design — read-only mode, per-tool disabling, confirmation prompts — is specific to how this server chooses to expose Couchbase, and it's substantial enough to deserve its own chapter later in this series. For now, the takeaway is simpler: MCP gives this server a standard vocabulary for exposing Couchbase to any AI assistant, and the rest of this series is about how this particular server uses that vocabulary well.

---

Next: **Chapter 2 — Why a Couchbase MCP server**, on what becomes possible once an AI agent has structured, safe access to a real database, and why that's worth building as a dedicated server rather than one-off integration code.
