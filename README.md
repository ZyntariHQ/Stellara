# Stellara

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Stellar](https://img.shields.io/badge/Blockchain-Stellar-blue.svg)](https://stellar.org)
[![Next.js](https://img.shields.io/badge/Frontend-Next.js%2014-black.svg)](https://nextjs.org)
[![NestJS](https://img.shields.io/badge/Backend-NestJS-red.svg)](https://nestjs.com)

Stellara is a privacy-focused AI-powered invoicing platform built on the Stellar blockchain. Designed for freelancers and small businesses, it enables effortless invoice creation and seamless cryptocurrency payments while prioritising user privacy and security.

## Table of Contents

- [Overview](#overview)
- [Key Features](#key-features)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Deployment](#deployment)
  - [Backend Setup](#backend-setup)
  - [Frontend Launch](#frontend-launch)
- [Authentication](#authentication)
- [Development Commands](#development-commands)
- [Recommended Integrations](#recommended-integrations)
- [Technology Stack](#technology-stack)
- [Future Enhancements](#future-enhancements)
- [Contributing](#contributing)
- [License](#license)
- [Acknowledgments](#acknowledgments)

## Overview

Stellara combines AI-assisted invoice generation with secure, on-chain payment processing on Stellar. A lightweight Soroban smart contract acts as a PaymentRouter, routing funds to merchants and emitting events for reliable off-chain reconciliation. This ensures fast, low-cost transactions with a strong emphasis on privacy—no unnecessary data collection, passwordless authentication, and user-controlled identities.

Target users: Freelancers and small businesses seeking efficient, crypto-native invoicing without compromising privacy.

## Key Features

- **AI-Assisted Invoicing**: Generate professional invoices via an intuitive, responsive UI powered by AI suggestions.
- **Passwordless Authentication**: Wallet-based sign-in using Stellar accounts for secure, decentralized access.
- **Crypto Payments**: Native support for XLM and USDC via the PaymentRouter contract.
- **Real-Time Reconciliation**: Backend event listeners track payments instantly for accurate accounting.
- **Privacy-First Design**: Minimal data storage, compliant with data protection best practices.

## Project Structure

The monorepo is structured for streamlined development:

```
.
├── webapp/          # Next.js 14 frontend application
├── backend/         # NestJS API for auth, invoices, and payment handling
├── contracts/       # Rust smart contracts (Soroban PaymentRouter)
├── soroban/         # Soroban CLI configuration and scripts
├── docs/            # Additional documentation
├── .github/         # CI/CD workflows and issue templates
├── README.md        # This file
└── LICENSE          # MIT License
```

## Getting Started

### Prerequisites

- Node.js ≥ 18
- Package manager: npm or pnpm
- Stellar wallet: Freighter or Coinbase Wallet
- Testnet XLM: Request from [Stellar Laboratory Faucet](https://laboratory.stellar.org)

### Deployment

#### 1. Deploy PaymentRouter Contract

**Environment Setup** (`soroban/.env`):
```
SECRET_KEY="YOUR_DEPLOYER_SECRET_KEY"
NETWORK=PASSPHRASE_TESTNET  # PASSPHRASE_PROD for mainnet
RPC_URL=https://soroban-testnet.stellar.org
```

**Compile and Deploy**:
```bash
cd soroban
cargo install --git https://github.com/stellar/rs-soroban-env soroban-cli
soroban contract build
soroban contract deploy \
  --wasm target/soroban/wasm32-unknown-unknown/release/payment_router.wasm \
  --source YOUR_DEPLOYER_SECRET_KEY \
  --network testnet \
  --permission-set-name Admin
```
Record the contract ID.

**Demo Transaction**:
```bash
export CONTRACT_ID=CD...  # Deployed contract ID
export MERCHANT_ADDRESS=G...  # Merchant public key
soroban contract invoke \
  --id $CONTRACT_ID \
  --source YOUR_PAYER_SECRET_KEY \
  --network testnet \
  pay --merchant $MERCHANT_ADDRESS --amount 10.0
```
View on [Stellar Expert](https://stellar.expert/explorer/testnet).

**Event Emitted**:
```
PaymentReceived(invoice_id: String, payer: Address, token: AssetCode, merchant: Address, amount: u128)
```

#### 2. Backend Setup

**Environment** (`backend/.env`):
```
STELLAR_RPC_URL=https://soroban-testnet.stellar.org
STELLAR_NETWORK=testnet
STELLAR_CONTRACT_ID=CD...
STELLAR_USDC_ASSET=USDC:GB...  # Testnet issuer
DATABASE_URL=postgresql://...
JWT_SECRET=...
```

**Run**:
```bash
cd backend
npm install
npx prisma generate
npx prisma migrate dev
npm run start:dev
```

The `StellarWatcherService` handles:
- `PaymentReceived` events
- Asset transfers (USDC)
- Direct XLM payments

#### 3. Frontend Launch

```bash
cd webapp
npm install
npm run dev
```
Visit `http://localhost:3000`.

Payment flows:
- **XLM**: `router.pay(merchant, invoiceId, amount)`
- **USDC**: Trust asset, then `router.pay_asset(issuer, code, merchant, amount, invoiceId)`

## Authentication

- Signature-based wallet flows via `@stellar/freighter-api`.
- Self-sovereign identities tied to Stellar accounts.
- Zero-knowledge proofs for optional verification.

## Development Commands

| Component | Command | Description |
|-----------|---------|-------------|
| **Frontend** | `npm run dev` | Start dev server |
| | `npm run build` | Production build |
| | `npm run start` | Serve production app |
| **Backend** | `npm run start:dev` | Dev mode with hot reload |
| | `npm run test` | Run unit tests |
| | `npm run test:e2e` | End-to-end tests |
| **Contracts** | `soroban contract build` | Compile |
| | `soroban contract deploy ...` | Deploy |
| | `soroban contract invoke ...` | Test invocations |

## Recommended Integrations

- **Federated Addresses**: Resolve readable names with Stellar SDK; fallback to truncated keys.
- **Account Creation**: Use SDK for frictionless onboarding; integrate Freighter for connections.

Document integrations in submissions with screenshots and links to demo addresses.

## Technology Stack

- **Frontend**: Next.js 14, TypeScript, Tailwind CSS (neumorphic theme)
- **UI**: Radix UI, shadcn/ui
- **Backend**: NestJS, Prisma (PostgreSQL)
- **Blockchain**: Soroban CLI, Stellar SDK (Rust)
- **Other**: AI via OpenAI API (configurable)

## Future Enhancements

- Full CRUD for invoices and client portals
- Advanced AI analytics and suggestions
- Multi-asset/fiat support
- Reporting dashboards
- Automated PDF exports and notifications

## Contributing

We welcome contributions! Please adhere to our guidelines:

1. Fork the repository.
2. Create a feature branch: `git checkout -b feature/your-feature`.
3. Commit: `git commit -m "feat: describe changes"`.
4. Push: `git push origin feature/your-feature`.
5. Open a Pull Request with detailed description.

See [CONTRIBUTING.md](CONTRIBUTING.md) for more details. Report issues via GitHub.

## License

This project is licensed under the MIT License - see [LICENSE](LICENSE) for details.

## Acknowledgments

- [Stellar](https://stellar.org) – Fast, inclusive blockchain
- [Next.js](https://nextjs.org) – React framework
- [Tailwind CSS](https://tailwindcss.com) – Utility-first CSS
- [Radix UI](https://www.radix-ui.com) – Accessible components
- [Soroban](https://soroban.stellar.org) – Smart contracts on Stellar

---

*Built with ❤️ for the open-source community. Star us on GitHub!*
