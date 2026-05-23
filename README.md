# Langclaw Contracts

Foundry project for Langclaw's Mantle hackathon contracts.

The hackathon-critical contracts are `LangclawRegistry` and `LangclawTradingJournal`. The registry records AI agent decisions, evidence hashes, and signal categories on Mantle; the trading journal records backtest and paper-trading outcomes so every strategy run can be independently checked later.

## Deployed Mantle Contracts

| Contract | Purpose | Mantle mainnet address |
| --- | --- | --- |
| `LangclawRegistry` | Agent decision proof and benchmarking trail | `0xe69755e4249c4978c39fbe847ca9674ce7af3505` |
| `LangclawUsageVault` | Optional MNT billing vault | `0x7e93Ef361e7b54297cF963977bA829E47E59e8E1` |
| `LangclawTradingJournal` | Strategy backtest and paper-trade proof trail | Deploy for Strategy Lab demos |

ERC-8004 identity:

- Identity registry: `0x8004A169FB4a3325136EB29fA0ceB6D2e539a432`
- Langclaw agent ID: `94`

Registry deployment transaction:
`0xf6f8af14295c86d2f358c32ba15d0669903b122c086dcb0b432d9df8aaec6b6c`

Vault deployment transaction:
`0xb60ed9019c5c8bb4c2b32c6a3e62e1edaf3b1530528d8151dfce08c1fd8b44e0`

Live decision proof examples:

| Decision | ERC-8004 agent | Signal type | Transaction |
| --- | --- | --- | --- |
| `0` | `94` | `smart-money` | `0x8a598de98fac01d53e696df67a9527de280c4d8cece72ccc4ced91164efa5187` |
| `1` | `94` | `smart-money` | `0x39caaca5fe3a6792c427740342116f309ac02ee0a846c7dbe54f12c86a39a177` |
| `2` | `94` | `liquidity-anomaly` | `0x9956a7574f6144ce831deac3275305939d65503366bc11bd922bc4783eeb5faf` |

## LangclawRegistry

Source: [`src/LangclawRegistry.sol`](src/LangclawRegistry.sol)

Tests: [`test/LangclawRegistry.t.sol`](test/LangclawRegistry.t.sol)

```solidity
function recordAgentDecision(
    uint256 agentId,
    string calldata runId,
    bytes32 decisionHash,
    string calldata evidenceUri,
    string calldata signalType
) external returns (uint256 decisionId);

function getDecision(uint256 decisionId) external view returns (AgentDecision memory);
```

Each record stores:

- ERC-8004 `agentId`
- Langclaw `runId`
- deterministic `decisionHash`
- evidence URI
- signal type, such as `smart-money` or `liquidity-anomaly`
- recorder wallet
- block timestamp

This is the contract to highlight for Mantle Turing Test Hackathon scoring.

## LangclawTradingJournal

Source: [`src/LangclawTradingJournal.sol`](src/LangclawTradingJournal.sol)

Tests: [`test/LangclawTradingJournal.t.sol`](test/LangclawTradingJournal.t.sol)

```solidity
function recordStrategyRun(
    uint256 agentId,
    string calldata runId,
    string calldata strategyId,
    string calldata market,
    bytes32 decisionHash,
    bytes32 resultHash,
    string calldata evidenceUri,
    string calldata action,
    int256 pnlBps,
    string calldata status
) external returns (uint256 recordId);

function getRecord(uint256 recordId) external view returns (StrategyRecord memory);
```

Each record stores:

- ERC-8004 `agentId`
- Langclaw `runId`
- `strategyId` such as `mantle-liquidity-momentum-v1`
- Mantle market or pair address
- deterministic decision and result hashes
- evidence URI
- action, PnL bps, and status (`backtested`, `paper-opened`, or `paper-closed`)
- recorder wallet
- block timestamp

This is the contract to highlight as a score booster for the primary AI Alpha & Data submission without live-funds risk.

## LangclawUsageVault

Source: [`src/LangclawUsageVault.sol`](src/LangclawUsageVault.sol)

Tests: [`test/LangclawUsageVault.t.sol`](test/LangclawUsageVault.t.sol)

`LangclawUsageVault` is an optional billing contract. It holds user MNT deposits, lets the backend authorize withdrawals, and lets users withdraw only authorized balances.

Mantle mainnet deployment:

- Address: `0x7e93Ef361e7b54297cF963977bA829E47E59e8E1`
- Owner: `0x2cA915EF6be8D2D48ccD3c5dAF715546AF873A4c`
- Withdrawal authority: `0x2cA915EF6be8D2D48ccD3c5dAF715546AF873A4c`

Do not mix the vault with the agent proof flow:

- `LangclawRegistry` = Mantle AI decision proof
- `LangclawUsageVault` = optional billing/top-up infrastructure

## Setup

```bash
git submodule update --init
forge build
forge test
```

Requires Foundry: https://book.getfoundry.sh/getting-started/installation

## Deploy Registry

```bash
cp .env.example .env

forge script script/DeployLangclawRegistry.s.sol:DeployLangclawRegistryScript \
  --rpc-url "$MANTLE_RPC_URL" \
  --private-key "$PRIVATE_KEY" \
  --broadcast
```

After deployment, copy the deployed address to `LANGCLAW_REGISTRY_ADDRESS` in `backend/.env`.

## Deploy Optional Usage Vault

```bash
forge script script/DeployLangclawUsageVault.s.sol:DeployLangclawUsageVaultScript \
  --rpc-url "$MANTLE_RPC_URL" \
  --private-key "$PRIVATE_KEY" \
  --broadcast
```

After deployment, copy the deployed address to `LANGCLAW_USAGE_VAULT_ADDRESS` in `backend/.env`.

## Deploy Trading Journal

```bash
forge script script/DeployLangclawTradingJournal.s.sol:DeployLangclawTradingJournalScript \
  --rpc-url "$MANTLE_RPC_URL" \
  --private-key "$PRIVATE_KEY" \
  --broadcast
```

After deployment, copy the deployed address to `LANGCLAW_TRADING_JOURNAL_ADDRESS` in `backend/.env` and set `MANTLE_TRADING_JOURNAL_ENABLED=true`.
