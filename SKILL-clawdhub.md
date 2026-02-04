---
name: autonomous-agent
description: CreditNexus x402 agent. Use when the user wants stock predictions, backtests, or to open a bank account. Payment-protected MCP tools (run_prediction, run_backtest, open_bank_account) with x402 flow (Aptos + Base). Agent handles 402 → pay → retry autonomously.
metadata: {"clawdbot":{"emoji":"📈","homepage":"https://github.com/FinTechTonic/autonomous-agent","requires":{"bins":["node","npm"]}}}
---

# CreditNexus x402 Agent Skill

Autonomous agent that calls x402-protected MCP tools: stock prediction, backtest, open bank account. Handles payment flow (402 → pay → retry) with Aptos (prediction/backtest) and Base (banking).

## Installation

Clone and install from the repository root:

```bash
git clone https://github.com/FinTechTonic/autonomous-agent.git && cd autonomous-agent
npm install
```

Set `MCP_SERVER_URL` to your x402 MCP server (e.g. `https://borrower.replit.app`). Copy `.env.example` to `.env` and configure LLM and wallet paths.

## Run the agent

From the **repository root** (where `package.json` and `src/` live):

```bash
node src/run-agent.js "Run a 30-day prediction for AAPL"
# Or interactive
node src/run-agent.js
```

**Source:** [FinTechTonic/autonomous-agent](https://github.com/FinTechTonic/autonomous-agent)
