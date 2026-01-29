# ERC-8004-AgentTee

<p align="center">
  <img src="static/mascot.svg" width="128" height="128" alt="ERC-8004-AgentTee Mascot">
</p>

<p align="center">
  <strong>An ERC-8004 Reference Implementation with TEE Registry Extension for Secure & Verifiable Trustless Agents.</strong>
</p>

<p align="center">
  <a href="#quick-start">Quick Start</a> &bull;
  <a href="#architecture">Architecture</a> &bull;
  <a href="#api-endpoints">API</a> &bull;
  <a href="#contracts">Contracts</a> &bull;
  <a href="#configuration">Config</a> &bull;
  <a href="#contributing">Contributing</a>
</p>

---

## Overview

ERC-8004-AgentTee is a production-grade proof-of-concept for the [ERC-8004 (Trustless Agents)](https://eips.ethereum.org/EIPS/eip-8004) standard. It demonstrates how AI agents can obtain a **cryptographically verifiable on-chain identity** backed by **hardware-level TEE attestation**.

### Features

- **TEE-derived keys** — deterministic key generation inside Intel TDX enclaves via Phala dstack
- **On-chain identity** — ERC-721 NFT-based agent registration on Base Sepolia
- **Hardware attestation** — real Intel TDX attestation with on-chain TEE key registration
- **ERC-8004 compliant** — standard `/agent.json` (registration-v1) and CAIP-10 addresses
- **A2A protocol** — agent-to-agent task creation and execution
- **Config-driven endpoints** — enable/disable A2A, MCP, OASF, ENS, DID via `agent_config.json`
- **Web dashboard** — funding page with QR code and registration flow UI

---

## Quick Start

### 1. Clone and configure

```bash
git clone git@github.com:andreeARCdev/ERC-8004-AgentTee.git
cd ERC-8004-AgentTee
cp .env.example .env
```

Edit `.env`:
```bash
AGENT_DOMAIN=your-domain.com
AGENT_SALT=unique-salt
```

### 2. Run with Docker

```bash
docker compose up -d
```

Open http://localhost:8000 — the funding page will show a QR code for sending ETH.

### 3. Run locally (development)

```bash
pip install -e .
python deployment/local_agent_server.py
```

### 4. Register your agent

1. **Fund wallet** — send >= 0.001 ETH (Base Sepolia) to the displayed address
2. **Register** — click "Register" on the dashboard or call `POST /api/register`
3. **TEE verify** — click "Verify TEE" or call `POST /api/tee/register`
4. **Live** — your agent card is now served at `/agent.json`

---

## Architecture

```
┌──────────────────────────────────────────────────────┐
│               ERC-8004-AgentTee Server                │
│                                                      │
│  ┌─────────────┐  ┌──────────────┐  ┌─────────────┐ │
│  │ TEEAuth     │  │ RegistryClient│  │ EIP712Signer│ │
│  │ (dstack)    │  │ (Web3)       │  │             │ │
│  └──────┬──────┘  └──────┬───────┘  └─────────────┘ │
│         │                │                           │
│  ┌──────┴──────┐  ┌──────┴───────┐                   │
│  │ TEEVerifier │  │ AgentCard    │                   │
│  │ (on-chain)  │  │ Builder      │                   │
│  └─────────────┘  └──────────────┘                   │
└──────────────────────┬───────────────────────────────┘
                       │ Web3 / RPC
                       ▼
┌──────────────────────────────────────────────────────┐
│              Base Sepolia (Chain ID: 84532)           │
│                                                      │
│  IdentityRegistry     TEERegistry     DstackVerifier │
│  (ERC-721 NFTs)      (TEE Keys)     (Proof Verify)  │
└──────────────────────────────────────────────────────┘
```

### Registration Flow

```
Step 1 — Key Derivation
    TEE derives Ethereum address from domain + salt (deterministic)

Step 2 — Fund Wallet
    Send >= 0.001 ETH to the derived address on Base Sepolia

Step 3 — On-Chain Registration (POST /api/register)
    Mints ERC-721 NFT → receives Agent ID
    tokenURI → https://{domain}/agent.json

Step 4 — TEE Attestation (POST /api/tee/register)
    Generates Intel TDX quote → requests off-chain proof
    Submits proof → TEERegistry.addKey() on-chain

Step 5 — Agent Live
    GET /agent.json          → ERC-8004 registration card
    GET /.well-known/agent-card.json → A2A discovery
    POST /tasks              → accept A2A tasks
```

---

## Project Structure

```
erc-8004-ex-phala/
├── deployment/
│   └── local_agent_server.py  # FastAPI server (main entrypoint)
├── src/
│   ├── agent/
│   │   ├── base.py            # BaseAgent abstract class
│   │   ├── tee_auth.py        # TEE key derivation & attestation (dstack)
│   │   ├── tee_verifier.py    # On-chain TEE key registration
│   │   ├── registry.py        # ERC-8004 registry client (Web3)
│   │   ├── agent_card.py      # ERC-8004 agent card builders
│   │   └── eip712.py          # EIP-712 typed data signing
│   └── templates/
│       └── server_agent.py    # ServerAgent with sandbox integration
├── contracts/
│   ├── IdentityRegistry.sol   # ERC-721 agent registration
│   ├── TEERegistry.sol        # TEE key storage & verification
│   ├── DstackVerifier.sol     # dstack-specific proof verification
│   └── ITEERegistry.sol       # TEE Registry interface
├── static/
│   ├── funding.html           # Wallet funding page (QR code)
│   ├── dashboard.html         # Registration workflow dashboard
│   └── mascot.svg             # 8-bit TEE Guardian mascot
├── agent_config.json          # Agent metadata & endpoint config
├── docker-compose.yml         # Docker deployment
├── entrypoint.sh              # Docker entrypoint script
├── setup.py                   # Python package config
├── requirements.txt           # Python dependencies
└── .env.example               # Environment variable template
```

---

## API Endpoints

### ERC-8004 Identity

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/agent.json` | ERC-8004 registration-v1 format |
| `GET` | `/.well-known/agent-card.json` | A2A agent card |
| `GET` | `/api/card` | Agent card (alias) |

### Wallet & Registration

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/wallet` | Agent address, balance, QR code data |
| `GET` | `/api/status` | Registration and TEE verification status |
| `POST` | `/api/register` | Register agent on-chain (mint ERC-721) |
| `POST` | `/api/tee/register` | Submit TEE attestation to on-chain registry |
| `POST` | `/api/metadata/update` | Update on-chain metadata |

### TEE & Signing

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/attestation` | Get TEE attestation quote data |
| `POST` | `/api/sign` | Sign message with TEE-derived key (EIP-191 + raw) |

### A2A Protocol

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/tasks` | Create an asynchronous A2A task |
| `GET` | `/tasks/{id}` | Get task status and artifacts |

### Monitoring

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/` | Funding page (Web UI) |
| `GET` | `/dashboard` | Registration dashboard (Web UI) |
| `GET` | `/health` | Health check |

---

## Contracts

### Deployed Addresses (Base Sepolia)

| Contract | Address | Purpose |
|----------|---------|---------|
| IdentityRegistry | `0xd08eC0f6F00751993169103d2240A6E3aF6920f4` | ERC-721 agent registration |
| TEERegistry | `0x03eCA4d903Adc96440328C2E3a18B71EB0AFa60D` | TEE key storage |
| DstackVerifier | `0x481ce1a6EEC3016d1E61725B1527D73Df1c393a5` | dstack proof verification |

### IdentityRegistry

ERC-721 based. Each agent is an NFT. Supports:
- Multiple registration overloads (with/without tokenURI, with/without metadata)
- On-chain key-value metadata (`setMetadata` / `getMetadata`)
- Ownership-gated metadata updates

### TEERegistry

Stores TEE attestation keys per agent. Supports:
- Verifier whitelist (owner-managed)
- Multiple keys per agent (key rotation)
- Code measurement storage (SHA-256 of agent binary)

---

## Configuration

### Environment Variables (`.env`)

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `AGENT_DOMAIN` | yes | `localhost:8000` | Agent domain name |
| `AGENT_SALT` | yes | — | Unique salt for TEE key derivation |
| `RPC_URL` | no | `https://sepolia.base.org` | Blockchain RPC |
| `CHAIN_ID` | no | `84532` | Target chain ID |
| `IDENTITY_REGISTRY_ADDRESS` | no | `0xd08e...` | Identity contract |
| `TEE_REGISTRY_ADDRESS` | no | `0x03eC...` | TEE Registry contract |
| `TEE_VERIFIER_ADDRESS` | no | `0x481c...` | Verifier contract |
| `AGENT_HOST` | no | `0.0.0.0` | Server bind address |
| `AGENT_PORT` | no | `8000` | Server port |

### Agent Config (`agent_config.json`)

Controls agent metadata and endpoint discovery:

```json
{
  "name": "TEE Agent",
  "description": "Trustless AI agent with Intel TDX attestation",
  "supportedTrust": ["tee-attestation"],
  "endpoints": {
    "a2a": {"enabled": true, "version": "0.3.0"},
    "mcp": {"enabled": false, "endpoint": "", "version": "2025-06-18"},
    "oasf": {"enabled": false, "endpoint": "", "version": "0.7"},
    "ens": {"enabled": false, "endpoint": "", "version": "v1"},
    "did": {"enabled": false, "endpoint": "", "version": "v1"}
  },
  "evmChains": [
    {"name": "Base-Sepolia", "chainId": 84532},
    {"name": "Ethereum", "chainId": 1}
  ]
}
```

### Customization Examples

**Enable MCP endpoint:**
```json
"mcp": {
  "enabled": true,
  "endpoint": "https://mcp.myagent.com/",
  "version": "2025-06-18"
}
```

**Add more EVM chains:**
```json
{"name": "Polygon", "chainId": 137},
{"name": "Arbitrum", "chainId": 42161}
```

**Add trust models:**
```json
"supportedTrust": ["tee-attestation", "feedback", "inference-validation"]
```

---

## Tech Stack

| Component | Technology |
|-----------|-----------|
| TEE Runtime | Intel TDX via Phala dstack SDK |
| Blockchain | Base Sepolia (EVM, Chain ID 84532) |
| Backend | Python 3.10+ / FastAPI / Uvicorn |
| Web3 | web3.py 6.x / eth-account |
| Smart Contracts | Solidity ^0.8.20 / OpenZeppelin |
| Frontend | HTML / Tailwind CSS / Alpine.js |
| Container | Docker / Docker Compose |

---

## ERC-8004 Compliance

| Requirement | Status |
|------------|--------|
| `/agent.json` (registration-v1) | Implemented |
| CAIP-10 wallet addresses (`eip155:{chainId}:{address}`) | Implemented |
| A2A endpoints (`.well-known/agent-card.json`) | Implemented |
| Multi-endpoint support (A2A, MCP, OASF, ENS, DID) | Implemented |
| TEE attestation trust model | Implemented |
| On-chain Identity Registry | Implemented |
| Reputation feedback | Partial (contract deployed, UI pending) |
| Inference validation | Partial (contract interface ready) |

---

## Related Projects

- **[Sentinel-8004](https://github.com/sentinel-8004/sentinel-8004)** — Open-source SDK extracted from this implementation. Provides a pluggable TEE interface (`TEEProvider`), CLI tool, reputation scoring, and multi-TEE support. If you want to build your own ERC-8004 agent from scratch, start there.

---

## Contributing

Contributions are welcome. Here are some areas where help is needed:

- Production DCAP on-chain proof verification in `DstackVerifier.sol`
- Additional TEE backend support (SGX, AWS Nitro, AMD SEV)
- Reputation Registry UI in the dashboard
- TypeScript/Rust SDK ports
- Integration tests with a local Hardhat node
- Security audit and hardening

### How to Contribute

1. Fork this repository
2. Create a feature branch
3. Add tests for new functionality
4. Submit a Pull Request

---

## License

MIT

## Links

- **ERC-8004 Spec**: [eips.ethereum.org/EIPS/eip-8004](https://eips.ethereum.org/EIPS/eip-8004)
- **Reference**: [dstack-erc8004-poc](https://github.com/h4x3rotab/dstack-erc8004-poc)
- **Phala Network**: [phala.network](https://phala.network)
- **Base**: [base.org](https://base.org)
- **Sentinel-8004 SDK**: [github.com/sentinel-8004/sentinel-8004](https://github.com/sentinel-8004/sentinel-8004)
