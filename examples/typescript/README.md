# BridgeNode TypeScript example

Buyer-side example: AI agent pays for and calls BridgeNode with the official
x402 TS SDKs (`@x402/fetch` + `@x402/svm`).

## Setup

```bash
npm install
cp .env.example .env
# fill SVM_PRIVATE_KEY (agent's Solana keypair, must have USDC ATA)
```

## Run

```bash
npm run start
```

## What it does

1. Registers the Solana exact payment scheme.
2. `POST /v1/chat/completions` through `wrapFetchWithPayment` — 402 handled
   automatically: sign partial TX → retry with `PAYMENT-SIGNATURE` → 200.
3. Prints the payment response (settlement receipt).

## Notes

- Solana mainnet, real USDC. Minimum charge: $0.002 per request.
- Gas is sponsored by BridgeNode — the agent only needs USDC, no SOL.
- The agent must have an existing USDC ATA.
