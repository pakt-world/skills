---
name: pakt
description: Interact with the Pakt platform API to build and manage Chainsites (Web3 apps with blockchain baked in). Use when Brendan or Pakt Pack members ask to create accounts, manage escrow, handle wallets, upload files, or build/configure Pakt-powered applications. Triggers on: "build a pakt app", "create escrow", "check wallet", "upload to chainsite", "pakt api", "psilo", or any reference to Pakt's SDK/API.
---

# Pakt Skill

Pakt is a platform for building "Chainsites" — Web2 apps with blockchain infrastructure (escrow, wallets, crypto payments) built in. The API lives at `https://api.pakt.world` (production) or the chainsite's own base URL.

## Credentials

Check `~/.clawdbot/.env` for:
- `PAKT_API_KEY` — JWT token (obtained via login, long-lived)
- `PAKT_BASE_URL` — defaults to `https://api.pakt.world`
- `PAKT_USER_ID` — your user ID (needed for receiverId in escrow)

All requests require: `Authorization: <JWT_TOKEN>` header.

## Quick Reference

| Action | Method | Endpoint |
|--------|--------|----------|
| Login | POST | `/v1/auth/login` |
| Get account | GET | `/v1/account` |
| Update profile | PATCH | `/v1/account/update` |
| Create escrow | POST | `/v1/escrow/initiate` |
| Validate escrow | POST | `/v1/escrow/validate` |
| Release escrow | POST | `/v1/escrow/release` |
| Get wallet | GET | `/v1/wallet` |
| Get coin balance | GET | `/v1/wallet/coin/:coin` |
| Withdraw | POST | `/v1/withdrawals/` |
| Upload file (public) | POST | `/v1/upload` |
| Upload file (private) | POST | `/v1/upload/private` |
| Supported coins | GET | `/v1/payment/coins` |
| Blockchain RPC config | GET | `/v1/payment/rpc` |

## Common Workflows

### Authenticate
```bash
curl -s -X POST "$PAKT_BASE_URL/v1/auth/login" \
  -H "Content-Type: application/json" \
  -d '{"email":"EMAIL","password":"PASSWORD"}'
# Returns: data.token → save as PAKT_API_KEY
```

### Create Escrow
```bash
curl -s -X POST "$PAKT_BASE_URL/v1/escrow/initiate" \
  -H "Authorization: $PAKT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Project Payment",
    "description": "Payment for deliverable X",
    "amount": 100,
    "coin": "usdc",
    "dueDate": "2026-12-31",
    "isPrivate": false,
    "isFundingRequest": false,
    "receiverId": "RECEIVER_USER_ID"
  }'
# Returns: collectionId, contractAddress, amountToPay
```

### Release Escrow
```bash
curl -s -X POST "$PAKT_BASE_URL/v1/escrow/release" \
  -H "Authorization: $PAKT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"escrowId": "ESCROW_ID"}'
```

### Check Wallet
```bash
curl -s -H "Authorization: $PAKT_API_KEY" "$PAKT_BASE_URL/v1/wallet"
# Returns: totalBalance (USD), wallets[] with coin/amount/address
```

### Upload File
```bash
curl -s -X POST "$PAKT_BASE_URL/v1/upload" \
  -H "Authorization: $PAKT_API_KEY" \
  -F "file=@/path/to/file.png"
# Returns: data.url (CDN URL), data._id
```

## Supported Coins
Avalanche testnet (chain ID: 43113). Common coins: `usdc`, `avax`.
Run `GET /v1/payment/coins` for the full list with contract addresses.

## Notes
- Auth token is long-lived (no automatic refresh needed in most cases)
- Escrow flow: initiate → fund (on-chain) → validate → release
- File uploads go to CloudFront CDN; `url` in response is immediately accessible
- Chainsites need to be deployed & activated in the Pakt Command Center before API access works
- API key comes from the Control Panel tab in the Pakt Command Center (PCC)

## Full API Reference
See `references/api-reference.md` for detailed request/response schemas.
See `references/escrow-flow.md` for the complete escrow lifecycle with examples.
