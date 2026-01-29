# ERC-8004 TEE Agents

An ERC-8004 Reference Implementation with TEE Registry Extension 
for Secure & Verifiable Trustless Agents.

## Overview

ERC-8004 proposes a standard for discovering and trusting AI agents 
on-chain. This repository provides:

- ✅ Full reference implementation of ERC-8004 (Identity, Reputation, Validation)
- 🔒 TEE Registry Extension for hardware-backed trust
- 📦 TypeScript/Python SDKs
- 🧪 Comprehensive test suite
- 📚 Documentation & examples

## Why TEE Extension?

ERC-8004 mentions TEE as a trust model but doesn't specify implementation.
This extension fills that gap by:

1. Verifying agents run inside secure enclaves (Intel SGX, AMD SEV, AWS Nitro)
2. Providing on-chain attestation verification
3. Enabling cryptographic proof that agent code hasn't been tampered

## Quick Start
```bash
# Install
npm install @erc8004/sdk

# Register an agent
npx erc8004 register --name "MyAgent" --uri "ipfs://..."

# Give feedback
npx erc8004 feedback --agent 123 --value 85 --tag "quality"

# Verify TEE attestation
npx erc8004 tee verify --agent 123
```

## Deployments

| Network | Identity | Reputation | Validation | TEE |
|---------|----------|------------|------------|-----|
| Sepolia | 0x... | 0x... | 0x... | 0x... |
| Base Sepolia | 0x... | 0x... | 0x... | 0x... |

## Contributing

PRs welcome! See [CONTRIBUTING.md](./CONTRIBUTING.md)

## License

MIT