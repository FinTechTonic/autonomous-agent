# Autonomous Agent – x402 MCP + LangChain.js

Autonomous AI agent for **x402-paid MCP tools**: predict tickers, backtest strategies, open bank accounts. Pays with **Aptos** or **EVM** via the x402 facilitator—no OpenAI key; uses **Hugging Face** for the LLM. Built for Cursor, OpenClaw/Moltbot, and headless runs.

## Why?

Most agent demos need a full backend and separate API keys. This agent talks to an **x402 MCP server**, pays with its own Aptos and EVM wallets (verify → settle), and uses an OpenAI-compatible LLM (e.g. Hugging Face). You get **run_prediction**, **run_backtest**, and **open_bank_account** with minimal setup—whitelist your agent, fund wallets, run.

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

**OpenClaw / MoltBook / Moltbot:** Load the skill from `skills/autonomous-agent/` or [adapters/openclaw/SKILL.md](adapters/openclaw/SKILL.md). See [MoltBook / OpenClaw](#moltbook--openclaw) below. Then point `MCP_SERVER_URL` at your MCP server.

## Quick Start

```bash
# Generate Aptos wallet (for prediction/backtest)
node src/setup-aptos.js

# Generate EVM wallet (for open_bank_account)
node src/setup.js

# Whitelist your agent at the onboarding flow (e.g. https://borrower.replit.app/flow.html)
node src/show-agent-addresses.js

# Run the agent (demo: balance + AAPL prediction)
npx cornerstone-agent "Run a 30-day prediction for AAPL"
# or from source:
node src/run-agent.js "Run a 30-day prediction for AAPL"
```

## Config

Copy `.env.example` to `.env` and set:

| Variable | Description |
|----------|-------------|
| `MCP_SERVER_URL` | x402 MCP server base URL (e.g. https://borrower.replit.app or http://localhost:4023) |
| `X402_FACILITATOR_URL` | Facilitator base URL for Aptos (verify/settle). Use public (e.g. https://x402-navy.vercel.app/facilitator) for open_bank_account. |
| `X402_EVM_FACILITATOR_URL` | Optional. EVM facilitator; defaults to X402_FACILITATOR_URL. Set to public when using local Aptos facilitator. |
| `LLM_BASE_URL` | OpenAI-compatible base URL (default https://router.huggingface.co/v1) |
| `HUGGINGFACE_API_KEY` or `HF_TOKEN` | Hugging Face API key |
| `LLM_MODEL` | Model ID (e.g. meta-llama/Llama-3.2-3B-Instruct) |
| `APTOS_WALLET_PATH` | Aptos wallet JSON path. Multi-wallet: ~/.aptos-agent-wallets.json. |
| `EVM_WALLET_PATH` | EVM wallet path. Multi-wallet: ~/.evm-wallets.json. Or set EVM_PRIVATE_KEY. |
| `BASE_SEPOLIA_RPC` | Optional; Base Sepolia RPC for open_bank_account |

## Commands

| Command | Description |
|---------|-------------|
| `node src/setup.js` | Generate EVM wallet (single; for multi use agent tool create_evm_wallet) |
| `node src/setup-aptos.js` | Generate Aptos wallet (single; for multi use create_aptos_wallet) |
| `node src/show-agent-addresses.js` | Print all Aptos and EVM addresses for whitelisting at flow.html |
| `npm run credit:aptos` | Credit Aptos agent (devnet: programmatic; testnet: instructions) |
| `node src/balance.js <chain>` | EVM balance |
| `node src/run-agent.js [message]` | Run agent (or `npx cornerstone-agent [message]` when installed from npm) |
| `npm run agent` | Same as run-agent.js |

**Crediting Aptos:** Testnet has no programmatic faucet—use [Aptos testnet faucet](https://aptos.dev/network/faucet). Devnet: `APTOS_FAUCET_NETWORK=devnet npm run credit:aptos`. See [Canteen – Aptos x402](https://canteenapp-aptos-x402.notion.site/).

## MCP Tools

| Tool | Description | Cost |
|------|-------------|------|
| `run_prediction` | Stock prediction (symbol, horizon, strategy) | ~6¢ (Aptos) |
| `run_backtest` | Backtest trading strategy | ~6¢ (Aptos) |
| `open_bank_account` | CornerStone bank link / open bank account | ~$3.65 (Ethereum/Base) |

## Supported Networks

| Network | Use | Cost |
|---------|-----|------|
| Aptos testnet | run_prediction, run_backtest | ~6¢ USDC |
| Base Sepolia | open_bank_account (testnet) | ~$3.65 |
| Base (mainnet) | open_bank_account (production) | ~$3.65 |

## x402 Flow

```
Agent calls MCP tool
  → Server returns 402 + payment requirements
  → Agent builds payment (Aptos or EVM), calls facilitator /verify then /settle
  → Agent retries request with PAYMENT-SIGNATURE header
  → Server returns result + request_payload, response_payload, payment_receipt
```

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
│   ├── run-agent.js      # Entrypoint
│   ├── setup.js          # EVM wallet generation
│   ├── setup-aptos.js    # Aptos wallet generation
│   └── show-agent-addresses.js
├── skills/               # MoltBook/OpenClaw (AgentSkills layout)
│   └── autonomous-agent/
│       └── SKILL.md
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
- **Adapters:** How each platform uses the capability:
  - **MoltBook / OpenClaw / Moltbot:** `skills/autonomous-agent/SKILL.md` (AgentSkills-compatible; [see below](#moltbook--openclaw)) or [adapters/openclaw/SKILL.md](adapters/openclaw/SKILL.md)
  - **OpenAI:** [adapters/openai/openapi.yaml](adapters/openai/openapi.yaml) — Custom GPTs / Assistants
  - **Claude / Anthropic:** [adapters/anthropic/tools.json](adapters/anthropic/tools.json) — Claude tools
  - **Local / OSS:** [adapters/local/README.md](adapters/local/README.md) — LM Studio, AutoGen, CrewAI

## MoltBook / OpenClaw

This repo is optimized for **easy skill loading** in MoltBook and OpenClaw (Claude, Anthropic, OpenAI, and other providers work via the same agent; the skill tells the assistant how to run it).

- **Skill layout:** The skill lives in `skills/autonomous-agent/SKILL.md` (AgentSkills-compatible, single-line frontmatter, `metadata.openclaw` gating). OpenClaw/MoltBook loads skills from `skills/` subfolders.
- **Load options:**
  1. **extraDirs:** Add this repo path to `~/.openclaw/openclaw.json` under `skills.load.extraDirs`. OpenClaw will scan `skills/` and load `autonomous-agent`.
  2. **Workspace:** Clone the repo and use it as your OpenClaw workspace; workspace skills are loaded from `<workspace>/skills`.
  3. **Managed skills:** Copy `skills/autonomous-agent` to `~/.openclaw/skills/` for all agents on the machine.
  4. **ClawHub:** When published, install with `clawhub install autonomous-agent` (installs into `./skills` by default).
- **Config:** In `skills.entries["autonomous-agent"]` you can set `enabled`, `env`, or `apiKey` (maps to `primaryEnv`). Ensure `MCP_SERVER_URL`, x402 facilitator URLs, and LLM/env are set for the agent run.
- **Run:** From the **repository root** (parent of `skills/`), run `node src/run-agent.js "your message"`.

Example `~/.openclaw/openclaw.json` to load this repo’s skills:

```json
{
  "skills": {
    "load": {
      "extraDirs": ["/path/to/autonomous-agent"]
    },
    "entries": {
      "autonomous-agent": {
        "enabled": true,
        "env": { "MCP_SERVER_URL": "https://borrower.replit.app" }
      }
    }
  }
}
```

### Publishing this skill on MoltBook / OpenClaw (ClawHub)

[ClawHub](https://clawhub.ai) is the public skill registry for MoltBook and OpenClaw. Publishing lets others install your skill with `clawhub install autonomous-agent`.

1. **Install the ClawHub CLI**
   ```bash
   npm i -g clawhub
   ```
   or `pnpm add -g clawhub`.

2. **Log in** (GitHub account required; must be at least one week old to publish)
   ```bash
   clawhub login
   ```
   This opens a browser to authenticate. For CI or headless: `clawhub login --token <token>`.

3. **Publish the skill** from the repo root:
   ```bash
   clawhub publish ./skills/autonomous-agent --slug autonomous-agent --name "Autonomous Agent (x402)" --version 1.0.0 --changelog "Initial release" --tags latest
   ```
   - `--slug` must match the folder name (`autonomous-agent`) so installs work as `clawhub install autonomous-agent`.
   - Omit `--version` / `--changelog` to be prompted. For later releases, bump version (e.g. `--version 1.0.1`) and add a `--changelog` line.

4. **Update an existing skill** (after you change `SKILL.md` or metadata): run `publish` again with a new version:
   ```bash
   clawhub publish ./skills/autonomous-agent --slug autonomous-agent --name "Autonomous Agent (x402)" --version 1.0.1 --changelog "Description of changes" --tags latest
   ```
   Alternatively, `clawhub sync --root . --all --bump patch --changelog "Description"` scans the repo and publishes changed skills with a bumped version.

After publishing, the skill appears on [clawhub.ai](https://clawhub.ai) and users can run `clawhub install autonomous-agent` (into `./skills` by default).

## Deployment order

1. **x402 facilitator** — Use public (e.g. https://x402-navy.vercel.app/facilitator) for full demo; or run local and set X402_EVM_FACILITATOR_URL to public for open_bank_account.
2. **MCP server** — x402-enabled (e.g. https://borrower.replit.app or run locally).
3. **Agent** — `node src/run-agent.js` or PM2 (`pm2 start ecosystem.config.cjs --only agent-autonomous` from repo root).

## References

- **Source:** [FinTechTonic/autonomous-agent](https://github.com/FinTechTonic/autonomous-agent)
- [Canteen App – Aptos x402](https://canteenapp-aptos-x402.notion.site/) — wallet hydration and crediting
- [LangChain.js MCP](https://js.langchain.com/docs/integrations/toolkits/mcp_toolbox)
- [Hugging Face Inference – OpenAI-compatible](https://huggingface.co/docs/api-inference/en/index)
## License

GPL-2.0-only. Use of this software is also subject to the [Responsible AI License (RAIL)](https://www.licenses.ai/). See [LICENSE.md](LICENSE.md) (GPL-2) and [RAIL](https://www.licenses.ai/).
