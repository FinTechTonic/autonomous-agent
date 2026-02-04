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
---
name: autonomous-agent
description: CreditNexus x402 agent. Use when the user wants to create and use wallets (Aptos/EVM), fund wallets, check balances, run stock predictions, backtests, open bank accounts, or transfer/swap tokens. Payment-protected MCP tools with x402 flow (Aptos + Base). Agent handles 402 → pay → retry autonomously.
metadata: {"clawdbot":{"emoji":"📈","homepage":"https://github.com/FinTechTonic/autonomous-agent","requires":{"bins":["node","npm"]}}}
---

# Autonomous Agent (x402) – Skill

Autonomous agent that **creates, funds, and uses** Aptos and EVM wallets, then calls x402-paid MCP tools (stock prediction, backtest, open bank account). Handles payment flow (402 → pay → retry) with Aptos and Base. Use this skill when the user wants wallet setup, funding, balances, predictions, backtests, banking, or token operations.

---

## Tasks you can do

### Wallets – create and manage

- **Create Aptos wallet** – `create_aptos_wallet` (optionally `network: "testnet"` or `"mainnet"`). Agent can have multiple Aptos wallets.
- **Create EVM wallet** – `create_evm_wallet` (optionally `network: "testnet"` or `"mainnet"`). Agent can have multiple EVM wallets.
- **List wallet addresses** – `get_wallet_addresses` returns all Aptos and EVM addresses (with network tags) for whitelisting and funding.

### Fund wallets

- **Fund Aptos wallet** – `credit_aptos_wallet`: on devnet uses programmatic faucet; on testnet returns instructions and Aptos faucet URL. Needed for `run_prediction` and `run_backtest` (~6¢ USDC).
- **Fund EVM wallet** – `fund_evm_wallet`: returns address and instructions (Base Sepolia faucet, etc.). Needed for `open_bank_account` (~$3.65 on Base).

User must **whitelist** every agent address at the onboarding flow (e.g. `http://localhost:4024/flow.html` or your MCP server’s flow) before paid tools succeed.

### Check balances

- **Aptos balance** – `balance_aptos` (USDC for the agent wallet).
- **EVM balance** – `balance_evm` (native token on a chain: base, baseSepolia, ethereum, etc.).

### Paid MCP tools (x402)

- **Stock prediction** – `run_prediction` (symbol, horizon in days). Cost ~6¢ (Aptos).
- **Backtest** – `run_backtest` (trading strategy). Cost ~6¢ (Aptos).
- **Open bank account** – `open_bank_account` (CornerStone bank link). Cost ~$3.65 (EVM/Base).

The agent handles 402 Payment Required automatically: verify → settle → retry with payment signature.

### CLI (from repo root)

| Task | Command |
|------|--------|
| Generate Aptos wallet | `node src/setup-aptos.js` |
| Generate EVM wallet | `node src/setup.js` |
| Show addresses for whitelist | `node src/show-agent-addresses.js` |
| Credit Aptos (devnet) | `npm run credit:aptos` (set `APTOS_FAUCET_NETWORK=devnet`) |
| EVM balance | `node src/balance.js <chain>` |
| Transfer ETH/tokens | `node src/transfer.js <chain> <to> <amount> [tokenAddress]` |
| Swap tokens (Odos) | `node src/swap.js <chain> <fromToken> <toToken> <amount>` |
| Run agent | `node src/run-agent.js "Run a 30-day prediction for AAPL"` or `node src/run-agent.js` (interactive) |

---

## When to use this skill

Use when the user wants to:

- Create or use **Aptos** or **EVM** wallets (testnet or mainnet).
- **Fund** agent wallets (faucet instructions or programmatic credit).
- **Check** Aptos or EVM balances.
- Run **stock predictions** or **backtests** (paid via Aptos).
- **Open a bank account** (paid via Base).
- **Transfer** or **swap** tokens from the agent wallet (via CLI or context).

---

## Setup

1. **Install:** From repo root: `npm install`. Copy `.env.example` to `.env`.
2. **Configure:** Set `MCP_SERVER_URL`, `X402_FACILITATOR_URL`, `HUGGINGFACE_API_KEY` (or `HF_TOKEN`), `LLM_MODEL`, and wallet paths (`APTOS_WALLET_PATH`, `EVM_WALLET_PATH` or `EVM_PRIVATE_KEY`).
3. **Wallets:** Create via agent tools (`create_aptos_wallet`, `create_evm_wallet`) or CLI (`node src/setup-aptos.js`, `node src/setup.js`). Fund and whitelist all addresses at the MCP server’s flow (e.g. `/flow.html`).

---

## Run the agent

From the **repository root** (where `package.json` and `src/` live):

```bash
node src/run-agent.js "Create an Aptos wallet, then run a 30-day prediction for AAPL"
# Or interactive
node src/run-agent.js
```

**Source:** [FinTechTonic/autonomous-agent](https://github.com/FinTechTonic/autonomous-agent)