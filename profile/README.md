# SkillChain

**Provenance infrastructure for artisan and cultural assets.**

---

> **For setup instructions, deployment guides, and implementation details, go directly to the individual repositories:**
>
> | Service | Repository |
> |---|---|
> | Frontend (React + Vite) | [SkillChainOrg/frontend](https://github.com/SkillChainOrg/frontend) |
> | Backend API (Flask) | [SkillChainOrg/backend](https://github.com/SkillChainOrg/backend) |
> | Vault Service | [SkillChainOrg/skillchain-vault](https://github.com/SkillChainOrg/skillchain-vault) |

---

## What Is SkillChain?

An artisan creates a handwoven textile. It is photographed, catalogued, and sold through a marketplace. Over the next decade, it changes hands three times, travels across two countries, and eventually enters a private collection. By the time it arrives there, no one can prove where it came from, who made it, or whether it is genuine.

SkillChain is built to solve this.

It is a provenance infrastructure platform — a system for recording, verifying, and transferring the ownership history of artisan artifacts using decentralized identifiers, cryptographic verification, and blockchain-settled ownership transitions. The goal is not to build another marketplace. It is to build the trust layer that any marketplace can use underneath.

**Provenance matters because:**

- Artisans lose attribution and revenue to counterfeit work
- Collectors have no reliable mechanism to verify authenticity across time
- Cultural heritage institutions cannot trace object history without manual archival work
- Marketplaces bear the liability of unauthenticated provenance claims
- Ownership continuity breaks the moment paper records are lost

SkillChain anchors provenance to cryptographic identity and on-chain records — not PDFs, not certificates of authenticity, not trust in a single platform.

---

## Live System

The MVP is deployed and functional on Algorand testnet.

| Component | Link |
|---|---|
| Frontend | [frontend-rho-eight-42.vercel.app](https://frontend-rho-eight-42.vercel.app/) |
| Backend API | [skillchain-dcyr.onrender.com](https://skillchain-dcyr.onrender.com) |
| Smart Contract (App 762870187) | [View on Explorer](https://testnet.explorer.perawallet.app/application/762870187/) |
| Successful x402 Acquisition | [Transaction Q6LAKG…](https://testnet.explorer.perawallet.app/tx/Q6LAKG4U3FQWIPQM6Q42HETXOT3VK3AQ6OWBVZODCUHOGIVXZESQ/) |

The x402 acquisition transaction demonstrates:
- Smart contract deployed and callable on Algorand testnet
- Grouped transaction (PaymentTxn + AppCallTxn) executing atomically
- Box references correctly resolving against registered artwork keys
- Pera Wallet signing working end-to-end
- Ownership settlement recorded immutably and explorer-verifiable

---

## Architecture

SkillChain is composed of four independently deployable services. Each has a defined responsibility boundary.

```
┌─────────────────────────────────────────────────────────────────┐
│                          End User                               │
│              (Artisan · Collector · Marketplace)                │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Frontend  (React + Vite)                     │
│                                                                 │
│  ┌──────────────────┐  ┌─────────────────┐  ┌───────────────┐  │
│  │  Marketplace UI  │  │  Provenance     │  │  Artisan      │  │
│  │  x402 Modal      │  │  Verification   │  │  DID Onboard  │  │
│  │  Grouped Txn UX  │  │  QR Scanner     │  │  Admin Panel  │  │
│  └──────────────────┘  └─────────────────┘  └───────────────┘  │
│                                                                 │
│  Pera Wallet SDK · react-i18next · Framer Motion               │
└────────────────────────────┬────────────────────────────────────┘
                             │  REST API
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Backend API  (Flask)                         │
│                                                                 │
│  ┌─────────────────┐  ┌──────────────┐  ┌────────────────────┐ │
│  │  DID Registry   │  │  Provenance  │  │  x402 Challenge    │ │
│  │  + Resolution   │  │  Orchestr.   │  │  + Settlement      │ │
│  └─────────────────┘  └──────────────┘  └────────────────────┘ │
│                                                                 │
│  PostgreSQL · IPFS (multi-gateway) · HMAC-SHA256 verification  │
└──────────┬──────────────────────────────────┬───────────────────┘
           │  Secret Fetch                    │  Algorand SDK
           ▼                                  ▼
┌──────────────────────┐       ┌───────────────────────────────────┐
│   Vault Service      │       │     Algorand Testnet              │
│                      │       │                                   │
│  HashiCorp Vault     │       │  ARC-4 Smart Contract             │
│  KV v2 (or local     │       │  Grouped Transaction Settlement   │
│  fallback)           │       │  On-chain Ownership State         │
│                      │       │  Box-reference Artwork Registry   │
└──────────────────────┘       └───────────────────────────────────┘
```

### Service Responsibilities

**Frontend** — All user interaction. Handles wallet connection, x402 challenge display, grouped transaction signing, provenance visualization, DID-gated artisan onboarding, QR-linked artifact verification, and multilingual rendering. Does not hold keys or sensitive state.

**Backend API** — Provenance orchestration and business logic. Issues x402 acquisition challenges, resolves DID documents, manages artifact registration, handles IPFS content addressing, and verifies post-settlement ownership transitions. Communicates with Vault for secrets; communicates with Algorand for state reads.

**Vault Service** — Isolated secret storage with a defined API surface. Wraps HashiCorp Vault KV v2 in production. Falls back to local encrypted storage for development. No other service accesses secrets directly — all secret access is routed through this service.

**Algorand Smart Contract** — Ownership settlement layer. Receives grouped transactions (PaymentTxn + AppCallTxn), validates box references against the registered artwork registry, and writes immutable ownership transitions on-chain. All acquisition events are publicly verifiable via Pera Wallet explorer.

---

## Core Flows

### Artisan Registration
An artisan submits their W3C DID to the frontend. The backend resolves the DID document and validates identity. On admin approval, the artisan gains access to artifact registration. Artifacts are registered with provenance metadata stored on IPFS, with the content address anchored to the on-chain record.

### x402 Acquisition
A collector selects an artifact and initiates acquisition. The backend issues an x402 payment challenge scoped to that artifact. The frontend presents the challenge in a modal with full transaction preview. On confirmation, Pera Wallet signs a grouped transaction: a PaymentTxn settling value and an AppCallTxn recording the ownership transition in the smart contract. The acquisition is verifiable on-chain within seconds.

### Provenance Verification
Anyone can verify an artifact by scanning its QR code or entering its ID. The backend returns the full provenance chain: original artisan, registration timestamp, IPFS metadata, acquisition history, and current on-chain owner. No account required.

---

## Ecosystem Components

### Frontend
React + Vite application with Pera Wallet integration, Framer Motion animations, and `react-i18next` for multilingual support (English and Hindi). Includes the x402 challenge modal, grouped transaction UX, DID-aware artisan dashboard, provenance visualization, QR-based verification, and an admin moderation panel.

### Backend API
Flask API handling provenance orchestration, DID registry operations, x402 challenge issuance and settlement verification, IPFS multi-gateway content fetching, and HMAC-SHA256 artifact integrity verification. Backed by PostgreSQL with advisory locking for concurrent registration safety.

### Vault Service
Thin abstraction over secret storage with a clean HTTP API. Supports HashiCorp Vault KV v2 in production and a local fallback mode for development. All sensitive material (signing keys, HMAC secrets) is fetched exclusively through this service boundary.

### Smart Contracts
ARC-4 Algorand contract handling grouped transaction acquisition, box-reference-based artwork registry lookups, and immutable ownership state writes. Deployed on Algorand testnet at [Application 762870187](https://testnet.explorer.perawallet.app/application/762870187/).

### Prototype Integrations *(architecture in place, not production)*
- **Razorpay** — payment UI and integration architecture exists; not wired to live API
- **DigiLocker** — identity verification flow architecture exists; currently mocked

---

## Roadmap

The following infrastructure is planned. None of it is currently implemented.

| Area | System |
|---|---|
| Counterfeit Detection | Perceptual hashing for duplicate artifact detection across the registry |
| Registration | Multi-angle artifact capture and structured visual registration |
| Content Authenticity | C2PA manifest generation at artifact registration time |
| Verification Network | Community verifier network with reputation-weighted provenance attestation |
| Performance | Redis-backed verification caching for high-volume provenance queries |
| Identity | Expanded DID registry integration (additional DID method support) |
| Ecosystem | Open provenance APIs for third-party marketplace integration |
| Institutional | Archive tooling for museums, heritage institutions, and cultural organizations |

The roadmap reflects an intent to move from a vertically integrated MVP toward an open provenance infrastructure layer — one that any marketplace, institution, or application can build on top of.

---

## Technology Stack

| Layer | Technology |
|---|---|
| Frontend | React, Vite, Framer Motion, react-i18next |
| Wallet | Pera Wallet SDK, Algorand JS SDK |
| Backend | Flask, PostgreSQL, IPFS |
| Identity | W3C DID Core 1.0, Ed25519 |
| Integrity | HMAC-SHA256, SHA-256 |
| Secret Storage | HashiCorp Vault KV v2 |
| Blockchain | Algorand, ARC-4 Smart Contracts |
| Deployment | Vercel (frontend), Render (backend) |

---

## Repository Index

| Repository | Description |
|---|---|
| [frontend](https://github.com/SkillChainOrg/frontend) | React + Vite marketplace, acquisition UX, provenance visualization |
| [backend](https://github.com/SkillChainOrg/backend) | Flask API, provenance orchestration, x402 challenge layer |
| [skillchain-vault](https://github.com/SkillChainOrg/skillchain-vault) | Vault-backed secret storage service |

---

*SkillChain — testnet MVP, 2025*
