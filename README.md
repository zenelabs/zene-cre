# Zene Identity Registration — Chainlink CRE Workflow

Privacy-preserving decentralized identity registration using Chainlink CRE.

## What it does

1. Receives Google OAuth code + passkey-derived salt via HTTP trigger
2. Exchanges code for Google sub using **Confidential HTTP** (sub never leaves CRE)
3. Computes `keccak256(salt || sub || chainId || appId)` inside CRE
4. Writes only the salted hash to ZeneRegistry on Avalanche Fuji

**No server ever sees the user's Google identity.**

## Privacy Track

This workflow uses CRE's Confidential HTTP to ensure:
- Google client_secret stored as DON secret
- Google sub ID processed inside confidential compute
- Only a salted hash (brute-force resistant) goes on-chain
- Passkey-derived salt (via WebAuthn PRF) makes hash unguessable

## Architecture

```
User Device (Arweave page)
  → Passkey + PRF → salt
  → Google PKCE → auth code
  → POST to CRE: { code, salt, address, verifier }

Chainlink CRE (Confidential Compute)
  → Exchange code → Google sub (PRIVATE)
  → keccak256(salt + sub + chainId + appId)
  → Write hash to ZeneRegistry

Avalanche Fuji
  → ZeneRegistry: resolve(hash) → address ✓
```

## Setup

```bash
cre login
cd zene-identity-workflow
bun install
cre workflow simulate zene-identity-workflow \
  --http-payload '{"code":"...", "salt":"...", "address":"0x...", "verifier":"..."}' \
  -T staging-settings
```

## Deploy

```bash
cre workflow deploy zene-identity-workflow -T staging-settings
```

## Links

- Arweave sign-in page: https://gateway.irys.xyz/CW83UzFa5DPHCp52B5E5tSWLvhLYYdDbz8gc3XNQv5PV
- ZeneRegistry: 0x8C44d507C71Fa29E7eD15Ac36678F247ad5516CF (Fuji)
- Demo: https://zene-auth.claws.page/demo/register
