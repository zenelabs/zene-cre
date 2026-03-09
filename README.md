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
User Device
  → Passkey (Face ID / Touch ID) → credential + salt derived from PRF
  → Redirects to immutable Arweave sign-in page

Arweave Sign-In Page (immutable, auditable)
  → Google sign-in via Google Identity Services (JWT stays in browser)
  → Sends auth code + salt to Chainlink CRE (NOT to any server)

Chainlink CRE / DON (Decentralized Oracle Network)
  → Multiple nodes in TEEs (Trusted Execution Environments)
  → Each node independently exchanges code for Google sub
    via Confidential HTTP (encrypted — even Chainlink can't see it)
  → Nodes reach consensus on result
  → Computes keccak256(salt + sub + chainId + appId)
  → Writes ONLY the hash to ZeneRegistry on Avalanche

Avalanche Fuji
  → ZeneRegistry: resolve(hash) → address ✓
```

**Why this matters:** No single party ever sees the Google sub ID. Not our server (we don't have one in the auth path), not any single Chainlink node. The token exchange is split across multiple TEEs with consensus. The Arweave page is immutable — anyone can verify it doesn't exfiltrate data.

**Daily sign-in** is passkey-only: Face ID → on-chain resolve → signed in. Sub-second, no server, no Google. The full Arweave + CRE flow only runs during **registration** and **account recovery**.

## Current Status

- ✅ CRE workflow coded and **simulation passing**
- ✅ Arweave sign-in page deployed and wired into live demo
- ✅ ZeneRegistry deployed on Fuji with working resolve
- ⏳ CRE deploy access pending (Chainlink team approval)
- 📝 Workflow uses mock Google sub in simulation; production will use real Confidential HTTP to Google's token endpoint

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

- Arweave sign-in page: https://node1.irys.xyz/kDRgb85nkNFZ1D-dh894NScXqQ0s-RZg4WUX8qX8Qg0
- ZeneRegistry: 0x8C44d507C71Fa29E7eD15Ac36678F247ad5516CF (Fuji)
- Demo: https://zene.network/demo
