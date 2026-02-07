# Autonomous Agent – x402 MCP + LangChain.js

**Published as [cornerstone-autonomous-agent](https://www.npmjs.com/package/cornerstone-autonomous-agent)** on npm; **source: [FinTechTonic/autonomous-agent](https://github.com/FinTechTonic/autonomous-agent)**.

Autonomous AI agent for **x402-paid MCP tools**: predict tickers, backtest strategies, link bank accounts, and query agent/borrower scores. Pays with **Aptos** or **EVM** via the x402 facilitator—no OpenAI key; uses **Hugging Face** for the LLM. Built for Cursor, OpenClaw/Moltbot, and headless runs.

## Why?

Most agent demos need a full backend and separate API keys. This agent talks to an **x402 MCP server**, pays with its own Aptos and EVM wallets (verify → settle), and uses an OpenAI-compatible LLM (e.g. Hugging Face). You get **run_prediction**, **run_backtest**, **link_bank_account**, and **score tools** (get_agent_reputation_score, get_borrower_score, by-email variants) with minimal setup—whitelist your agent, fund wallets, run. The server offers both Aptos and EVM payment options; the agent pays with whichever you prefer or have funded.

## Install

### From npm (recommended)

```bash
npm install cornerstone-autonomous-agent
```

Copy the package’s `.env.example` into your project and set `MCP_SERVER_URL`, `X402_FACILITATOR_URL`, `HUGGINGFACE_API_KEY`, `LLM_MODEL`, and wallet paths. See [Config](#config).

### From source

```bash
git clone https://github.com/FinTechTonic/autonomous-agent.git && cd autonomous-agent
npm install
```

**OpenClaw / Moltbot:** See [adapters/openclaw/SKILL.md](adapters/openclaw/SKILL.md) or use your platform’s skill install (e.g. ClawHub). Then point `MCP_SERVER_URL` at your MCP server.

## Quick Start

**Important:** Another npm package is named [autonomous](https://www.npmjs.com/package/autonomous). To run **this** package without installing, always use the full name: **`npx cornerstone-autonomous-agent`**. After you `npm install cornerstone-autonomous-agent`, the short binary `autonomous` in that project runs this CLI.

```bash
# Run without installing (use full package name so the other "autonomous" package isn’t used)
npx cornerstone-autonomous-agent setup:aptos
npx cornerstone-autonomous-agent setup
npx cornerstone-autonomous-agent addresses
npx cornerstone-autonomous-agent attest:aptos
npx cornerstone-autonomous-agent "Run a 30-day prediction for AAPL"

# Or install first, then you can use the short command in that project
npm install cornerstone-autonomous-agent
npx autonomous setup:aptos
npx autonomous attest:aptos

# Or install globally for the short command everywhere
npm i -g cornerstone-autonomous-agent
autonomous setup:aptos
autonomous "Run a 30-day prediction for AAPL"

# From source:
node src/setup-aptos.js
node src/run-agent.js "Run a 30-day prediction for AAPL"
```

**CLI:** This package registers the **`autonomous`** binary (short name) and **`cornerstone-autonomous-agent`** (matches package name). Use **`npx cornerstone-autonomous-agent <command>`** when you haven’t installed the package; use **`npx autonomous <command>`** only inside a project that has `cornerstone-autonomous-agent` installed, or **`autonomous <command>`** after a global install.

## Config

Copy `.env.example` to `.env` and set:

| Variable | Description |
|----------|-------------|
| `MCP_SERVER_URL` | x402 MCP server base URL (e.g. http://localhost:4023 or https://3001-borrower.replit.app) |
| `X402_FACILITATOR_URL` | Facilitator base URL (Aptos + EVM verify/settle). Use public (e.g. https://x402-navy.vercel.app/facilitator) for full demo. |
| `X402_EVM_FACILITATOR_URL` | Optional. EVM facilitator; defaults to X402_FACILITATOR_URL. |
| `PREFERRED_PAYMENT_ORDER` | Optional. Comma-separated `network|asset` (e.g. `aptos:2|usdc,eip155:84532|usdc`) to choose payment option when server returns multiple. |
| `LLM_BASE_URL` | OpenAI-compatible base URL (default https://router.huggingface.co/v1) |
| `HUGGINGFACE_API_KEY` or `HF_TOKEN` | Hugging Face API key |
| `LLM_MODEL` | Model ID (e.g. meta-llama/Llama-3.2-3B-Instruct) |
| `APTOS_WALLET_PATH` | Aptos wallet JSON path. Multi-wallet: ~/.aptos-agent-wallets.json. |
| `EVM_WALLET_PATH` | EVM wallet path. Multi-wallet: ~/.evm-wallets.json. Or set EVM_PRIVATE_KEY. |
| `BASE_SEPOLIA_RPC` | Optional; Base Sepolia RPC. Agent supports all EVM chains in lib/chains.js (Base, Ethereum, Polygon, etc.). |

## Commands

Use **`npx cornerstone-autonomous-agent <command>`** (or, in a project that has this package installed, **`npx autonomous <command>`**). From source: `node src/<script>.js` or `npm run <script>`.

| Command | Description |
|---------|-------------|
| `autonomous setup` | Generate EVM wallet (single; for multi use agent tool create_evm_wallet) |
| `autonomous setup:aptos` | Generate Aptos wallet (single; for multi use create_aptos_wallet) |
| `autonomous setup:evm:multichain` | Generate EVM multi-chain wallet |
| `autonomous addresses` | Print all Aptos and EVM addresses for whitelisting at flow.html |
| `autonomous attest:aptos` | Sign attestation for Aptos wallet (set `APTOS_PRIVATE_KEY` or `APTOS_TESTNET_*`); submit to POST /attest/aptos |
| `autonomous attest:evm` | Sign attestation for EVM wallet; submit to POST /attest/evm |
| `autonomous balance [chain]` | EVM balance |
| `autonomous transfer` | Transfer (see script for args) |
| `autonomous contract` | Contract interaction helper |
| `autonomous swap` | Swap helper |
| `autonomous` or `autonomous agent` or `autonomous start [message]` | Run agent (default: demo balance + prediction) |
| `npm run credit:aptos` | Credit Aptos agent (devnet: programmatic; testnet: instructions) |
| `npx cornerstone-agent [message]` | Run agent (legacy bin name) |

**Crediting Aptos:** Testnet has no programmatic faucet—use [Aptos testnet faucet](https://aptos.dev/network/faucet). Devnet: `APTOS_FAUCET_NETWORK=devnet npm run credit:aptos`. See [Canteen – Aptos x402](https://canteenapp-aptos-x402.notion.site/).

## MCP Tools

| Tool | Description | Payment |
|------|-------------|---------|
| `run_prediction` | Stock prediction (symbol, horizon) | x402: Aptos or EVM |
| `run_backtest` | Backtest trading strategy | x402: Aptos or EVM |
| `link_bank_account` | CornerStone bank link (Plaid) | x402: Aptos or EVM |
| `get_agent_reputation_score` | Agent reputation score (allowlisted wallet) | x402 or lender credits |
| `get_borrower_score` | Borrower score (100 or 100+X when bank linked) | x402 or lender credits |
| `get_agent_reputation_score_by_email` | Reputation score by email (resolves to agent) | x402 or lender credits |
| `get_borrower_score_by_email` | Borrower score by email | x402 or lender credits |

All paid tools accept **both Aptos and EVM**; the server returns 402 with multiple options. Use `PREFERRED_PAYMENT_ORDER` to prefer one chain/asset.

## Supported Networks

| Network | Use |
|---------|-----|
| Aptos testnet (aptos:2) / mainnet (aptos:1) | run_prediction, run_backtest, score tools |
| Base Sepolia (eip155:84532), Base (eip155:8453), Ethereum, Polygon, Arbitrum, Optimism | link_bank_account, score tools (when server offers them) |

## x402 Flow

```
Agent calls MCP tool
  → Server returns 402 + payment requirements (single or array of options: USDC, APT, native ETH)
  → Agent picks one option (by preferredPaymentOrder or first), builds payload (Aptos or EVM)
  → Agent calls facilitator /verify then /settle
  → Agent retries request with payment_payload
  → Server returns result + payment_receipt
```

**Multi-asset:** The server may return multiple `paymentRequirements` (e.g. USDC on Aptos, USDC on EVM, APT on Aptos, native ETH on EVM). The client uses `preferredPaymentOrder` (e.g. `["aptos:2|usdc", "eip155:84532|native"]`) to choose, or the first option. Native ETH: client submits the ETH transfer, then sends the `txHash` in the payload; facilitator verifies on-chain.

## Architecture

```
autonomous/
├── src/
│   ├── agent/           # ReAct agent, LLM, tool wiring
│   ├── lib/
│   │   ├── mcp/          # MCP client + 402 handle/retry
│   │   ├── aptos/        # Aptos wallet, balance, signPayment
│   │   ├── evm/          # EVM wallet, signPayment (Base)
│   │   └── x402/         # Payment types, verify/settle flow
│   ├── cli.js            # CLI router (autonomous / cornerstone-autonomous-agent)
│   ├── run-agent.js      # Agent entrypoint
│   ├── setup.js          # EVM wallet generation
│   ├── setup-aptos.js    # Aptos wallet generation
│   ├── attest-aptos-wallet.js / attest-evm-wallet.js
│   └── show-agent-addresses.js
├── adapters/             # OpenClaw, OpenAI, Anthropic, local
├── .env.example
└── package.json
```

**Core pieces:** `lib/mcp` — MCP client and 402 retry; `lib/aptos` / `lib/evm` — wallets and payment signing; `lib/x402` — verify/settle; `agent/` — LangChain.js ReAct agent and tools.

## Tech Stack

- **Runtime:** Node.js 18+
- **Agent:** LangChain.js (ReAct), OpenAI-compatible LLM (e.g. Hugging Face)
- **MCP:** [Model Context Protocol](https://modelcontextprotocol.io) + x402 payment flow
- **Chains:** Aptos (viem-style + @aptos-labs/ts-sdk), EVM (viem) for Base Sepolia/Base
- **Payments:** x402 facilitator (verify/settle), local wallet storage

## Security

- **Wallets:** Stored locally (e.g. `~/.aptos-agent-wallets.json`, `~/.evm-wallets.json`); private keys not logged or sent except as signed payloads to the facilitator.
- **Payments:** Only verify/settle go to the facilitator; no custody of funds by the MCP server.
- **Whitelist:** Agent addresses must be allowlisted at the onboarding flow before paid tools succeed.

## Capability + adapters

- **Capability:** Core (`src/`) — MCP client, x402 flow, local tools. No OpenAI/Claw/Anthropic logic in code.
- **Adapters:** `adapters/` — how each platform uses the capability:
  - [adapters/openclaw/SKILL.md](adapters/openclaw/SKILL.md) — OpenClaw / Moltbot
  - [adapters/openai/openapi.yaml](adapters/openai/openapi.yaml) — Custom GPTs / Assistants
  - [adapters/anthropic/tools.json](adapters/anthropic/tools.json) — Claude tools
  - [adapters/local/README.md](adapters/local/README.md) — LM Studio, AutoGen, CrewAI

## Deployment order

1. **x402 facilitator** — Use public (e.g. https://x402-navy.vercel.app/facilitator) for full demo; or run local and set X402_EVM_FACILITATOR_URL to public for link_bank_account.
2. **MCP server** — x402-enabled (run locally or use Replit MCP URL).
3. **Agent** — `node src/run-agent.js` or PM2 (`pm2 start ecosystem.config.cjs --only agent-autonomous` from repo root).

## References

- [Canteen App – Aptos x402](https://canteenapp-aptos-x402.notion.site/) — wallet hydration and crediting
- [LangChain.js MCP](https://js.langchain.com/docs/integrations/toolkits/mcp_toolbox)
- [Hugging Face Inference – OpenAI-compatible](https://huggingface.co/docs/api-inference/en/index)
- [evm-wallet-skill](https://github.com/surfer77/evm-wallet-skill) — self-sovereign EVM wallet pattern for agents

## License

GPL-2.0-only. Use of this software is also subject to the [Responsible AI License (RAIL)](https://www.licenses.ai/). See [LICENSE.md](../LICENSE.md) (GPL-2) and [RAIL](https://www.licenses.ai/).
