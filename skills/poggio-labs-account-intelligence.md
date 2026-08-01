---
name: Research a company account with Poggio
description: >-
  Use the Poggio MCP server to gather and read AI account intelligence on a target
  company — create the account if needed, search its intelligence documents, fetch
  full document text, and ask the Poggio assistant a research question.
api: mcp/poggio-labs-mcp.yml
transport: https://mcp.poggio.io/mcp
operations: [create_account, search, fetch, ask_poggio]
generated: '2026-07-20'
method: generated
---

# Research a company account with Poggio

Poggio's hosted MCP server (`https://mcp.poggio.io/mcp`) exposes account
intelligence to AI agents. Connect with OAuth (Dynamic Client Registration for
desktop clients such as Claude Desktop / Cursor / Windsurf) or a Bearer token in
the `Authorization` header. The authorization server is `https://api.poggio.io/`.

## Steps

1. **Ensure the account exists.** Call `create_account` with `account_domain`
   (the target company's domain, e.g. `acme.com`). This establishes the account
   in your workspace and kicks off immediate intelligence gathering. It is safe
   to call if the account may already exist.
2. **Find relevant intelligence.** Call `search` with `query` set to the
   company domain to get back a list of account intelligence documents, each
   with an `id`.
3. **Read the detail.** Call `fetch` with the `id` of a document from step 2 to
   retrieve its full text.
4. **Ask a focused question.** Call `ask_poggio` with a `question` to have the
   Poggio assistant research across web search, Salesforce, Gong transcripts,
   and company intelligence and return a synthesized answer.

## Rules

- Authenticate every call with an OAuth access token or Bearer token; the
  organization is inferred from the token (see
  `authentication/poggio-labs-authentication.yml`).
- Treat account intelligence as confidential customer data.
- Prefer `search` + `fetch` for grounded document retrieval; use `ask_poggio`
  for open-ended synthesis.
