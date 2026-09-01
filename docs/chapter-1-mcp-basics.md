# Chapter 1: What is MCP, and why does it exist?

## The problem

Say you want an AI assistant like Claude to help you with your Couchbase
database. You'd like to ask it things like "what buckets do I have?" or
"show me the schema of the `orders` collection" and get a real, live answer
— not a guess based on what Couchbase generally looks like, but the actual
state of *your* cluster.

The model itself can't do this. A large language model is, fundamentally, a
program that turns text into more text. It has no network socket, no
database driver, no way to authenticate to your cluster and run a query. If
you want it to act on live data, something else has to do the actual work —
connect to the database, run the query, get the result back — and then hand
the model something to read.

Before MCP, every AI assistant that wanted to do this had to invent its own
way of describing "here are the actions you can take" and "here's how to
call them." Every tool integration was a bespoke, one-off wire format between
one specific assistant and one specific backend. A team building a Couchbase
integration for Claude Desktop, and a separate team building one for Cursor,
would each write their own protocol, their own tool descriptions, their own
plumbing — even though the underlying idea ("let an LLM call `get_buckets`")
is identical.

## What MCP is

The **Model Context Protocol (MCP)** is an open standard that fixes this by
defining one shared way for an AI application (the **client** — Claude
Desktop, Cursor, VS Code, etc.) to talk to a **server** that exposes a set of
**tools**: named, described, callable actions with a defined input schema
and a defined output shape.

The client doesn't need to know anything about Couchbase. It just needs to
speak MCP. The server doesn't need to know anything about which AI assistant
is calling it. It just needs to speak MCP back. Write the server once,
and it works with any MCP-compatible client — that's the whole point.

Concretely, an MCP server does three things:

1. **Advertises its tools.** On startup, the client asks "what can you do?"
   and the server responds with a list of tools — each with a name, a
   plain-language description, and a schema for its parameters.
2. **Accepts tool calls.** The client (really, the LLM behind it, deciding
   on its own that a tool would help answer the user's question) sends a
   request naming a tool and its arguments.
3. **Returns results.** The server executes the underlying action and sends
   back a result the model can read and reason about, in plain text or
   structured data.

Everything in between — how the server actually does the work — is up to
the server. MCP only standardizes the conversation *about* the work, not the
work itself.

## A worked example, using this server

This repository, `mcp-server-couchbase`, is an MCP server for Couchbase. One
of its simplest tools is `test_cluster_connection` (defined in
[`src/cb_mcp/tools/server.py`](../src/cb_mcp/tools/server.py)), which checks
that the credentials you've configured can actually reach your cluster.

Here's the round trip, in plain language:

```
┌─────────────┐        "what tools do you have?"        ┌──────────────────┐
│  MCP client │ ───────────────────────────────────────▶│  This MCP server │
│ (e.g. Claude│                                          │ (mcp-server-     │
│  Desktop)   │◀─────────────────────────────────────── │  couchbase)      │
└─────────────┘   "test_cluster_connection, get_buckets_ └──────────────────┘
                    in_cluster, run_sql_plus_plus_query, ...
                    (22 read-only tools always available, plus
                    12 more write tools when read-only mode is
                    off — each with a description and parameter
                    schema)"

       User asks: "Is my Couchbase cluster reachable?"

┌─────────────┐   "call test_cluster_connection"         ┌──────────────────┐
│  MCP client │ ───────────────────────────────────────▶ │  This MCP server │
└─────────────┘                                          └──────────────────┘
                                                                    │
                                                    connects to the real
                                                    cluster using the
                                                    configured connection
                                                    string/credentials
                                                                    │
┌─────────────┐  { "status": "success",                  ┌──────────────────┐
│  MCP client │◀── "cluster_connected": true,             │  This MCP server │
└─────────────┘    "message": "Successfully connected     └──────────────────┘
                     to Couchbase cluster" }

       Claude reads the result and tells the user: "Yes, your cluster is
       reachable."
```

Nothing here is Couchbase-specific from the client's point of view. Claude
Desktop doesn't have Couchbase-specific code for this — it just saw a tool
called `test_cluster_connection` with a description explaining what it does,
decided (based on the user's question) that calling it would help, called
it, and read the JSON that came back. The same client, pointed at a
completely different MCP server (say, one for GitHub or Slack), would follow
the exact same protocol steps with completely different tools.

That's the trick: **the LLM chooses which tool to call and with what
arguments, based on the tool's name and description** — the same way it
decides what to say next in a conversation. The server's job is to make that
description accurate and the underlying action safe to run.

## Where this fits

For this specific server, "the work" means: connecting to a Couchbase
cluster (self-managed or [Capella](https://www.couchbase.com/products/capella/)),
and exposing operations like listing buckets, inspecting a collection's
schema, running a SQL++ query, or reading/writing a document — as MCP tools
an AI assistant can call on your behalf. [Chapter 2](chapter-2-why-couchbase-mcp.md)
picks up from here and looks at what this server specifically adds on top of
the bare MCP standard — starting with the Couchbase data model and the
read-only-by-default safety net.

For the full list of tools this server exposes today, and exactly how to
configure and run it, see the root [README.md](../README.md).
