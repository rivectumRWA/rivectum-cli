# RivectumRWA CLI

Command-line interface for the **RivectumRWA vault protocol** — autonomous allocation for tokenized real-world assets on Base Sepolia.

## Install

**Requires [Bun](https://bun.sh) >= 1.1.**

```bash
# From GitHub (recommended)
npm install -g github:rivectumRWA/rivectum-cli

# Or from npm registry
npm install -g rivectum-cli

# Or clone + link locally
git clone https://github.com/rivectumRWA/rivectum-cli
cd rivectum-cli
bun install
npm link
```

After install, the `rivectum` command is available globally:

```bash
rivectum --help
```

## Quick Start

```bash
# Setup (first time only)
cp .env.example .env   # edit .env with your contract addresses and keys

# Show all commands
rivectum --help

# Agent namespace (operator / read-only)
rivectum agent status              # Vault snapshot from chain
rivectum agent decisions           # Recent rebalance decisions
rivectum agent decisions --limit 5 # Last 5 decisions
rivectum agent tick                # Dry-run rebalance (no tx)
rivectum agent tick --broadcast --yes  # Execute rebalance NOW
rivectum agent analyze             # AI vault health report
rivectum agent explain --id 1      # AI explains decision #1
rivectum agent suggest             # AI operator recommendations

# User namespace (deposit / withdraw)
rivectum user preview --deposit 100  # Preview shares for deposit
rivectum user balance                # Show USDC + vault balance
rivectum user approve --max --yes    # Approve USDC spending
rivectum user deposit --amount 100 --yes
rivectum user withdraw --assets 50 --yes
rivectum user withdraw --shares 1000 --yes
```

Namespaces
----------

| Namespace | Role | Commands |
|-----------|------|----------|
| `agent` | Operator (read-only or rebalance) | `status`, `decisions`, `tick`, `analyze`, `explain`, `suggest` |
| `user` | Vault user (deposit/withdraw) | `preview`, `balance`, `approve`, `deposit`, `withdraw` |

## All Commands

### `agent status` — Vault Snapshot

Reads vault state from chain: TVL, total supply, paused flag, agent identity, whitelisted underlyings.

```bash
rivectum agent status
rivectum agent status --json
```

**Required env**: `RPC_URL`, `VAULT_ADDRESS`

### `agent decisions` — Decision History

Lists recent rebalance decisions from the agent database.

```bash
rivectum agent decisions
rivectum agent decisions --limit 10
rivectum agent decisions --status confirmed
rivectum agent decisions --json
```

**Required env**: `DB_PATH`

### `agent tick` — Rebalance

Computes allocation based on APY and submits an intent to the vault. Dry-run by default.

```bash
rivectum agent tick                    # Dry-run (no tx)
rivectum agent tick --broadcast        # Show preview, ask confirm
rivectum agent tick --broadcast --yes  # Execute NOW
rivectum agent tick --json             # Dry-run, JSON output
```

**Required env**: `RPC_URL`, `VAULT_ADDRESS`, `AGENT_PRIVATE_KEY`, `UNDERLYING_1`, `UNDERLYING_2`

> **Key safety**: `agent tick` defaults to dry-run. Requires explicit `--broadcast --yes` to send a transaction.

### `agent analyze` — AI Vault Health Report

Uses an LLM to produce a structured health report: TVL summary, risk assessment, and operator recommendations.

```bash
rivectum agent analyze
rivectum agent analyze --no-db         # Skip decision history
rivectum agent analyze --json
```

**Required env**: `RPC_URL`, `VAULT_ADDRESS` + LLM key
**Optional**: `DB_PATH` (for richer context)

### `agent explain` — AI Decision Explanation

Explains a single rebalance decision in plain English — what happened, why, and on-chain result.

```bash
rivectum agent explain --id 1
rivectum agent explain --id 3 --json
```

**Required env**: `RPC_URL`, `VAULT_ADDRESS`, `DB_PATH` + LLM key

### `agent suggest` — AI Operator Recommendations

Recommends operator actions: when to rebalance, portfolio adjustments, risk flags, gas/timing notes.

```bash
rivectum agent suggest
rivectum agent suggest --no-db
rivectum agent suggest --json
```

**Required env**: `RPC_URL`, `VAULT_ADDRESS` + LLM key
**Optional**: `DB_PATH`

### `user preview` — Simulate Deposit/Withdraw

Estimates shares for a deposit or assets for a redeem.

```bash
rivectum user preview --deposit 100     # How many shares for 100 USDC?
rivectum user preview --redeem 50       # How much USDC for 50 shares?
rivectum user preview --deposit 100 --json
```

**Required env**: `RPC_URL`, `VAULT_ADDRESS`

### `user balance` — Show Balances

Shows USDC balance and vault shares. Optionally query any address.

```bash
rivectum user balance
rivectum user balance --address 0x...
rivectum user balance --json
```

**Required env**: `RPC_URL`, `VAULT_ADDRESS`, `USDC_ADDRESS`

### `user approve` — Approve USDC

Approves the vault to spend your USDC. `--max` for unlimited.

```bash
rivectum user approve --amount 500 --yes
rivectum user approve --max --yes
rivectum user approve --amount 500 --yes --json
```

**Required env**: `RPC_URL`, `VAULT_ADDRESS`, `USDC_ADDRESS`, `USER_PRIVATE_KEY`

### `user deposit` — Deposit USDC

Deposits USDC into the vault. Use `--auto-approve` to approve first.

```bash
rivectum user deposit --amount 100 --yes
rivectum user deposit --amount 500 --auto-approve --yes
rivectum user deposit --amount 100 --yes --json
```

**Required env**: `RPC_URL`, `VAULT_ADDRESS`, `USDC_ADDRESS`, `USER_PRIVATE_KEY`

### `user withdraw` — Withdraw

Redeems vault shares for USDC. Specify `--assets` or `--shares`.

```bash
rivectum user withdraw --assets 100 --yes
rivectum user withdraw --shares 1000 --yes
rivectum user withdraw --assets 100 --yes --json
```

**Required env**: `RPC_URL`, `VAULT_ADDRESS`, `USDC_ADDRESS`, `USER_PRIVATE_KEY`

---

## Environment Variables

| Variable | Required For | Description |
|----------|-------------|-------------|
| `RPC_URL` | all | Base Sepolia RPC endpoint |
| `VAULT_ADDRESS` | all | RivectumRWA vault contract |
| `USDC_ADDRESS` | user commands | USDC token on Base Sepolia |
| `AGENT_PRIVATE_KEY` | `agent tick` | Agent signer (operator) |
| `USER_PRIVATE_KEY` | user write commands | Your wallet private key |
| `UNDERLYING_1` | `agent tick` | First whitelisted underlying (mTBILL) |
| `UNDERLYING_2` | `agent tick` | Second whitelisted underlying (mCREDIT) |
| `DB_PATH` | `agent decisions`, `explain` | Path to agent.db (e.g., `../agent/agent.db`) |

### LLM Provider (for AI commands)

Set **one** of these:

| Variable | Provider | Default Model |
|----------|----------|---------------|
| `OPENAI_API_KEY` | OpenAI | `gpt-4o-mini` |
| `ANTHROPIC_API_KEY` | Anthropic | `claude-3-5-haiku-latest` |
| `OLLAMA_HOST` | Ollama (local) | `llama3.2` |

Optional model overrides: `OPENAI_MODEL`, `ANTHROPIC_MODEL`, `OLLAMA_MODEL`.

No key? Commands print a helpful message — CLI still works normally.

---

## Global Flags

| Flag | Description |
|------|-------------|
| `--json` | Machine-readable JSON output on stdout (human messages → stderr) |
| `--yes` | Skip confirmation prompts |
| `--help`, `-h` | Show help for any command |
| `--api-key <key>` | API key for thin-client mode (CLI → API → chain) |
| `--api-url <url>` | API base URL (default `https://app.rivectum.xyz`) |

### API Mode (Thin Client)

When `--api-key` is set, the CLI becomes a thin client — it reads vault data through the web API instead of talking to chain directly. Writes are signed locally and broadcast via the API.

```bash
rivectum agent status --api-key rivectum_xxxx
rivectum agent decisions --api-key rivectum_xxxx --limit 5
```

Get your API key at https://app.rivectum.xyz/settings.

---

## Key Safety

- **Read-only commands** (`status`, `decisions`, `analyze`, `explain`, `suggest`, `preview`, `balance --address`) never load private keys.
- Agent commands only load `AGENT_PRIVATE_KEY`, never `USER_PRIVATE_KEY`.
- User commands only load `USER_PRIVATE_KEY`, never `AGENT_PRIVATE_KEY`.
- `agent tick` defaults to dry-run. Requires explicit `--broadcast --yes` to send a tx.

## Output

- **Default**: Colored, human-readable tables (Unicode box-drawing when TTY).
- **`--json`**: Single JSON object on stdout; human messages go to stderr.

```json
{ "ok": true, "command": "agent.status", "data": { "vault": "0x...", "paused": false } }
```

## Error Codes

| Exit | Meaning |
|------|---------|
| 0 | Success |
| 1 | Generic error |
| 2 | User aborted / confirmation declined |
| 3 | Missing or invalid env variable |
| 4 | RPC error / LLM error |
| 5 | Transaction reverted |

---

## AI Commands (LLM-powered)

Three read-only commands use an LLM to provide intelligent analysis. **Zero additional npm dependencies** — uses raw `fetch()` to call the API directly.

| Command | What It Does |
|---------|-------------|
| `agent analyze` | Vault health: TVL, utilization rate, risk score, recommendations |
| `agent explain --id <n>` | Plain-English breakdown of one rebalance decision |
| `agent suggest` | Operator guidance: when to rebalance, what to adjust, risk flags |

Provider auto-detection order:
1. `OPENAI_API_KEY` → OpenAI GPT-4o-mini
2. `ANTHROPIC_API_KEY` → Anthropic Claude 3.5 Haiku
3. `OLLAMA_HOST` → Ollama (local, free)

---

## Development

```bash
bun install        # Install dependencies
bun test           # Run all tests (21 tests)
bun run typecheck  # TypeScript check
bun run cli --help # Test CLI locally
```

## Network

- **Chain**: Base Sepolia (testnet, chain ID 84532)
- **RPC**: `https://sepolia.base.org`
- **Contracts**: Deployed from `rivectumRWA/rivectumRWA` monorepo

---

## License

MIT
