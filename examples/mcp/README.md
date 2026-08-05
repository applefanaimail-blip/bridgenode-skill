# BridgeNode MCP example

Call BridgeNode through its MCP server with x402 payment.

## Server

- URL: `https://bridgenode.cc/mcp` (streamable-http)
- Tool: `chat_completions` (model | mode, messages, max_tokens)
- Payment: x402 handshake per tool call (via `_meta["x402/payment"]`)

Prices are annotated in `tools/list` (`x-x402`) as an indication; the actual
amount is in the 402 response — always check it before signing.

## Option A — one-command wrapper (any MCP client)

```bash
npx -y @bridgenode/mcp@latest
```

Use this as the MCP server command in your client (Claude Code, Cursor,
Windsurf, etc.). The wrapper connects to `https://bridgenode.cc/mcp` and
handles x402 payment automatically with the agent's wallet.

Claude Code example:

```bash
claude mcp add bridgenode -- npx -y @bridgenode/mcp@latest
```

## Option B — direct streamable-http

Configure your MCP client with:

```
URL: https://bridgenode.cc/mcp
Transport: streamable-http
```

## What the agent sees

1. `tools/list` → `chat_completions` with `x-x402` price annotation.
2. `tools/call` → server returns payment required (result with
   `structuredContent` PaymentRequired object).
3. Client signs the USDC transaction, retries the call with
   `_meta["x402/payment"]`.
4. Server settles, runs inference, returns the completion with
   `_meta["x402/payment-response"]` (settlement receipt).

## Notes

- Solana mainnet, real USDC. Minimum charge: $0.002 per request.
- Gas sponsored by BridgeNode — the agent only needs USDC, no SOL.
- Free tools (e.g., model list) skip payment.
