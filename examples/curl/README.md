# BridgeNode curl example

Manual x402 flow with curl. This shows the protocol steps; for automatic
payment handling use [x402curl](https://github.com/second-state/x402-skill)
(drop-in curl replacement) or the SDK examples.

> ⚠️ Mainnet warning: Solana mainnet, real USDC. Minimum charge: $0.002.
> Check the 402 `amount` before signing.

## Step 1 — Request, get 402

```bash
curl -sS -D - -o /dev/null https://bridgenode.cc/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model":"deepseek-v4-flash","messages":[{"role":"user","content":"hello"}],"max_tokens":100}'
```

Expected: `HTTP/1.1 402 Payment Required` with a `PAYMENT-REQUIRED` header
(base64 JSON: scheme, network, amount, asset, payTo, memo, feePayer).

## Step 2 — Sign the partial transaction

Decode the `PAYMENT-REQUIRED` header and sign a partial transaction:
USDC `TransferChecked` (amount from the 402) + Memo instruction, signed with
the agent's Solana keypair. The fee payer is NOT signed by the agent —
BridgeNode sponsors gas.

Use any x402-capable signer: `x402curl`, the official SDKs (`x402[svm]`,
`@x402/svm`), or the Solana `pay` CLI. This produces a base64 JSON payload
(`x402Version`, `resource`, `accepted`, `payload.transaction`).

## Step 3 — Retry with PAYMENT-SIGNATURE

```bash
PAYLOAD="<base64 JSON from step 2>"
curl -sS https://bridgenode.cc/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "PAYMENT-SIGNATURE: $PAYLOAD" \
  -d '{"model":"deepseek-v4-flash","messages":[{"role":"user","content":"hello"}],"max_tokens":100}'
```

Expected: `200 OK` with the completion and a `PAYMENT-RESPONSE` header
(settlement receipt).

## Automatic alternative — x402curl

```bash
# install: https://github.com/second-state/x402-skill
# config: X402_PRIVATE_KEY=... (agent's Solana keypair)
x402curl -sS https://bridgenode.cc/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model":"deepseek-v4-flash","messages":[{"role":"user","content":"hello"}],"max_tokens":100}'
```

x402curl detects the 402, signs the payment, and retries automatically.

## Notes

- Network: `solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp` (mainnet)
- Asset: USDC `EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v`
- The agent needs a USDC ATA, but no SOL (gas sponsored).
- Free: `GET https://bridgenode.cc/v1/models` (models + prices).
