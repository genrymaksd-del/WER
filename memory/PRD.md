# FozPay — Crypto Payment Gateway (PRD)

## Origin
Cloned from GitHub repo `feromaksua-gif/fff` and deployed into `/app`.
Stack: FastAPI + React + MongoDB. HD (BIP-44) wallet, EVM + TRON + BTC/LTC/SOL.

## Core Requirements (static)
- Crypto payment gateway: merchant invoices, cabinet balances, deposits, withdrawals, swap.
- Single BIP-39 mnemonic → per-user deposit addresses; funds swept to a platform hot wallet.
- Public registration disabled; admin creates/manages users.
- **Hot-wallet key encrypted at rest** (DB leak alone must not reveal it).

## Personas
- Admin/operator: manages users, hot wallet, withdrawals, platform fees.
- Merchant/user: receives crypto, exchanges, withdraws.

## Implemented (2026-06 session)
- **Deploy**: repo copied to `/app`, deps installed (bip-utils, web3, eth-account, 1inch, etc.), `.env` created with user keys (ALCHEMY_KEY, TRONGRID_KEY, ONEINCH_KEY), generated JWT_SECRET + Fernet MNEMONIC_ENC_KEY. Admin: admin@fozpay.io / FozPay#Admin2026.
- **Wallet encryption at rest**: mnemonic stored ONLY as Fernet token (`system.wallet.mnemonic_enc`); key `MNEMONIC_ENC_KEY` lives only in `backend/.env`, never in DB. Verified no plaintext mnemonic in DB.
- **Hot wallet address**: now derived from HD seed at EVM index 0 (deposit addresses start at index 1). Enables auto gas-funding + auto-sweep. `/api/admin/hot-wallet` returns populated evm_address (previously blank because TREASURY_EVM was unset).
- **Scan all EVM networks**: `direct_deposit_worker` now uses `scan_evm_address` to detect every supported asset (native + USDT/USDC) across Ethereum/BSC/Polygon/Arbitrum on each deposit address, then auto-sweeps to the hot wallet (gas auto-funded from hot wallet for token sweeps).
- **Real on-chain exchange (no simulation)**: `/api/wallet/exchange` now executes a REAL 1inch swap on the hot wallet for same-chain EVM pairs and credits the actual on-chain received amount (minus platform swap %). Unsupported pairs (BTC↔ETH, cross-chain, non-EVM) are rejected with a clear message instead of a simulated ledger conversion.

## Testing
- 12/12 new backend tests pass (`backend/tests/test_realswap_and_encryption.py`).
- 18/19 regression (`test_fozpay.py`); the 1 failure is a pre-existing stale hardcoded merchant token in the test, unrelated.

## MOCKED / untested with real funds
- Real on-chain sweep / withdraw / hot-wallet payout / exchange require the hot wallet to actually hold crypto + native gas. Logic is real (verified it queries live balances) but cannot be confirmed end-to-end until the hot wallet is funded. TRON/BTC payouts remain operator-handled (real automation is EVM only).

## Backlog / Next
- P1: Real automatic TRON (USDT-TRC20) sweeps/withdrawals.
- P1: Pending-exchange worker to remove the ledger-vs-onchain desync window in synchronous swap.
- P2: Deposit alerts (email/Telegram) to admin on sweep.
- P2: Admin analytics (volume, fees, hot-wallet trends); user freeze.
- P2: Add more EVM networks (Base/Optimism/Avalanche) to scanning + catalog.
