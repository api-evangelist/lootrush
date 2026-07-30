---
name: Read a LootRush account over MCP
description: Use the LootRush MCP server to read your own balance, history, and card data from an AI assistant.
api: openapi/lootrush-openapi-original.json
operations: [callMcpTool]
---

# Read a LootRush account over MCP

LootRush publishes a hosted, read-only MCP server that scopes every call to the API
key's user. Use it to answer account questions without leaving the chat.

## Connect
- Endpoint: `https://mcp.lootrush.com/mcp` (Streamable HTTP, JSON-RPC 2.0, `POST` only, stateless).
- Auth: your LootRush API key as `Authorization: Bearer <token>` (or `?token=` for clients that
  cannot set headers — header is preferred).
- Claude Code: `claude mcp add --transport http lootrush https://mcp.lootrush.com/mcp --header "Authorization: Bearer YOUR_API_KEY"`.

## Tools (all via `callMcpTool`, `method: tools/call`)
- `getAccountBalance` — current balances.
- `getAccountHistory` — account activity.
- `getUserCards` — list cards; `includeCardSecrets` reveals PAN/CVV **encrypted** and needs a
  `sessionId` built from `getCardRevealPublicKey` plus the `mcp_card_reveal` scope on the key.
- `getUserCardTransactions` — card transactions.
- `getCardRevealPublicKey` — public key for the encrypted card-reveal flow.

## Rules
- Read-only and self-scoped: no tool accepts an identity argument.
- Batch (array) JSON-RPC bodies are rejected — one message per POST.
- Rate limit: 60 requests / 10 seconds per user (HTTP 429 on exceed).
- `403` with JSON-RPC code `-32004` = the request IP is not in the key's allowlist, or the key
  lacks the required scope (`mcp`, or `mcp_card_reveal` for a reveal).
