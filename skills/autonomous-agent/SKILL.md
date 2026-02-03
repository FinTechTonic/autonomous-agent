---
name: autonomous-agent
description: CreditNexus x402 agent. Use when the user wants stock predictions, backtests, or to open a bank account. Payment-protected MCP tools (run_prediction, run_backtest, open_bank_account) with x402 flow (Aptos + Base). Agent handles 402 → pay → retry autonomously.
metadata: {"openclaw":{"emoji":"📈","homepage":"https://github.com/FinTechTonic/autonomous-agent","requires":{"bins":["node","npm"]},"primaryEnv":"MCP_SERVER_URL","skillKey":"autonomous-agent"},"clawdbot":{"emoji":"📈","homepage":"https://github.com/FinTechTonic/autonomous-agent","requires":{"bins":["node","npm"]}}}
---

# CreditNexus x402 Agent Skill

Autonomous agent that calls x402-protected MCP tools: stock prediction, backtest, open bank account. Handles payment flow (402 → pay → retry) with Aptos (prediction/backtest) and Base (banking).

## Installation

When this skill is loaded from the **autonomous-agent** repo, the repo root is the parent of the skill folder `{baseDir}`. Clone and install from the repo root:

```bash
# From repository root (parent of {baseDir} when using this repo)
git clone https://github.com/FinTechTonic/autonomous-agent.git && cd autonomous-agent
npm install
```

Set `MCP_SERVER_URL` to `https://borrower.replit.app`. Copy `.env.example` to `.env` and set:

- `MCP_SERVER_URL` – MCP server base URL
- `X402_FACILITATOR_URL` – x402 facilitator (verify/settle)
- `LLM_BASE_URL`, `HUGGINGFACE_API_KEY` or `HF_TOKEN`, `LLM_MODEL` – for inference
- `APTOS_WALLET_PATH`, `EVM_WALLET_PATH` (or `EVM_PRIVATE_KEY`) – for payments

## Run the agent

From the **repository root** (where `package.json` and `src/` live):

```bash
node src/run-agent.js "Run a 30-day prediction for AAPL"
# Or interactive
node src/run-agent.js
```

**x402 flow:** Agent calls tool → server returns 402 with payment requirements → agent pays via facilitator (verify → settle) → retries with PAYMENT-SIGNATURE → receives result.

## Tools

| Tool | Description | Cost |
|------|-------------|------|
| `run_prediction` | Stock prediction (symbol, horizon) | ~6¢ (Aptos) |
| `run_backtest` | Backtest trading strategy | ~6¢ (Aptos) |
| `open_bank_account` | CornerStone bank link / open bank account | ~$3.65 (Base) |
| `get_agent_reputation_score` / `get_borrower_score` | CornerStone scores | ~1¢ |
| `get_agent_reputation_score_by_email` / `get_borrower_score_by_email` | CornerStone scores by email (extra fee) | base + extra |

Whitelist your agent at https://borrower.replit.app/flow.html so the server allows your wallet.
