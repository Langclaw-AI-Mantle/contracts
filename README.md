# Langclaw Contracts

Foundry project for **LangclawUsageVault** and proof tests for **LangclawRegistry**. The hackathon-facing configuration is Mantle-first (chain ID `5000`, native token `MNT`).

**Organization:** [Langclaw-AI](https://github.com/Langclaw-AI) · **Backend:** [Langclaw-AI/backend](https://github.com/Langclaw-AI/backend) · **Frontend:** [Langclaw-AI/frontend](https://github.com/Langclaw-AI/frontend)  

**Agent decision proof contract** (`LangclawRegistry`): [`../backend/contracts/LangclawRegistry.sol`](../backend/contracts/LangclawRegistry.sol) — deploy via `npm run deploy:registry` in the backend repo.

## LangclawUsageVault

Holds user deposits. The backend credits an off-chain usage ledger after verifying `Deposit` events. Users withdraw only up to amounts the backend authorizes on-chain.

```solidity
function deposit(bytes32 depositReference) external payable;
receive() external payable;
function authorizeWithdrawal(address payer, uint256 amount, bytes32 withdrawalId) external;
function withdraw(uint256 amount) external;
function pause() / unpause() external onlyOwner;
```

Key events: `Deposit`, `Withdrawal`, `WithdrawalAuthorized`.

Source: [`src/LangclawUsageVault.sol`](src/LangclawUsageVault.sol)  
Tests: [`test/LangclawUsageVault.t.sol`](test/LangclawUsageVault.t.sol)

## Setup

```bash
forge install
forge build
forge test
```

Requires [Foundry](https://book.getfoundry.sh/getting-started/installation).

## Deploy

```bash
cp .env.example .env
# LANGCLAW_USAGE_VAULT_OWNER=0x...
# LANGCLAW_USAGE_VAULT_WITHDRAWAL_AUTHORITY=0x...  # backend ops wallet

source .env  # or export manually
forge script script/DeployLangclawUsageVault.s.sol:DeployLangclawUsageVaultScript \
  --rpc-url https://rpc.mantle.xyz \
  --broadcast
```

Copy the deployed address to **`LANGCLAW_USAGE_VAULT_ADDRESS`** in the [backend `.env`](https://github.com/Langclaw-AI/backend/blob/main/.env.example).

Broadcast artifact: `broadcast/DeployLangclawUsageVault.s.sol/5000/run-latest.json`

## Integration

| Backend endpoint | On-chain |
| ---------------- | -------- |
| `POST /api/usage/deposit/verify` | Reads `Deposit` event |
| `POST /api/usage/withdraw/request` | Returns vault address; user calls `withdraw` after `authorizeWithdrawal` |

`authorizeWithdrawal` must be called by `withdrawalAuthority` (set at deploy). Backend automation for this is still being wired — see the [backend README](https://github.com/Langclaw-AI/backend/blob/main/README.md).

## Do not mix usage billing with LangclawRegistry

- **Vault** = billing only  
- **Registry** = Mantle agent decision hash + evidence URI only  

Requirements spec: [SMART_CONTRACT_TEAM_NOTES.md](https://github.com/Langclaw-AI/backend/blob/main/docs/SMART_CONTRACT_TEAM_NOTES.md)
