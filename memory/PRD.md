# FozPay — Crypto Payment Gateway (PRD)

## Original problem statement
Clone of an open crypto-payment product (repo: github.com/maksymnykytiuk92-cpu/wwe) with:
real deposit / withdraw / swap, purple glassmorphism redesign, rebrand to **FozPay**,
remove public registration (admin creates users & changes their passwords), fix direct
(non-invoice) deposit crediting + history, real swap & withdrawal, and a **hot wallet**
that all deposits are swept to and from which withdrawals/swaps/commissions are handled;
admin can withdraw any currency from the hot wallet. API must work.

## Architecture
- FastAPI + MongoDB (motor) backend; React (CRACO) + Tailwind + shadcn frontend.
- HD wallet (BIP-44) from `WALLET_MNEMONIC`; per-user deposit addresses (index ≥1),
  hot wallet at index 0 (EVM `TREASURY_EVM`, TRON index 0).
- Real chain access: Alchemy (EVM), TronGrid (TRON), mempool.space (BTC), 1inch (EVM swaps for auto-swap).
- Live prices from Binance public API.

## User personas
- **Admin** (single superadmin): creates/deletes users, resets passwords, sets platform fees,
  toggles networks, manages hot wallet (view on-chain balances, withdraw any currency).
- **User/Merchant**: deposit, withdraw, internal swap, invoices, merchant API keys.

## Core requirements (static)
- No public registration; JWT email/password login (+ optional Google + 2FA).
- Deposits (invoice AND direct) credit balance + write history, then sweep to hot wallet.
- Internal swap by live rate (CoinGecko/Binance), fee stays in platform pool on hot wallet.
- Real EVM withdrawals paid out from hot wallet by a background worker.
- Merchant private API (X-Auth-Token + sha256 signature).

## Implemented (2026-06)
- Rebrand OKIPAYS/MaksPAY → **FozPay** across backend + frontend.
- Purple glassmorphism theme (index.css global re-skin + glass Layout/Login).
- Removed `/auth/register` (returns 403); admin user management endpoints:
  `GET/POST /api/admin/users`, `PUT /api/admin/users/password`, `DELETE /api/admin/users/{id}`.
- Settings → **Users** and **Hot Wallet** admin tabs.
- `direct_deposit_worker`: detects on-chain deposits to per-user addresses (no invoice),
  credits balance + records tx, then sweeps to hot wallet (fixes the reported bug).
- `withdrawal_worker`: real EVM payout from hot wallet for Pending withdrawals.
- `sweep_to_hot_wallet`: moves confirmed deposits to hot wallet.
- Hot wallet admin: `GET /api/admin/hot-wallet` (live balances), `POST /api/admin/hot-wallet/withdraw`.

## Backlog (P1/P2)
- P1: Real TRON/BTC withdrawals & sweeps (currently EVM real; non-EVM operator-assisted).
- P2: Per-address deposit txid tracking; email notifications; AML review queue UI.

## Test credentials
- Admin: admin@fozpay.io / FozPay#Admin2026 (see /app/memory/test_credentials.md).
