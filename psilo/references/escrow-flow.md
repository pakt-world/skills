# Pakt Escrow Flow

Escrow on Pakt is a 4-step lifecycle: **Initiate → Fund → Validate → Release**

## Step 1: Initiate

The payer creates the escrow contract.

```bash
curl -s -X POST "$PAKT_BASE_URL/v1/escrow/initiate" \
  -H "Authorization: $PAKT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Website Redesign",
    "description": "Payment for phase 1 delivery",
    "amount": 500,
    "coin": "usdc",
    "dueDate": "2026-06-30",
    "isPrivate": false,
    "isFundingRequest": false,
    "receiverId": "66fbc91b9e1d33779aec8e80"
  }'
```

**Response (save these):**
```json
{
  "data": {
    "collectionId": "67fd70faedb5a6dcc87170cb",  ← this is your escrowId
    "address": "0xABC...",                          ← send crypto HERE
    "coinEscrowAmount": "500.00",
    "amountToPay": "510.00",                        ← includes fee
    "feePercentage": 2,
    "contractAddress": "0xDEF...",
    "chainId": "43113"
  }
}
```

## Step 2: Fund (On-Chain)

The payer sends `amountToPay` in `coin` to `address` on chain `chainId`.

This happens **outside the API** — it's a blockchain transaction. Options:
- Use the Pakt web UI
- Use a wallet (MetaMask, etc.) connected to Avalanche Testnet C-Chain (43113)
- Use an agent wallet with the exported `secretKey`

RPC URL for Avalanche Testnet: `https://api.avax-test.network/ext/bc/C/rpc`

## Step 3: Validate

After on-chain funding is confirmed, call validate to record it:

```bash
curl -s -X POST "$PAKT_BASE_URL/v1/escrow/validate" \
  -H "Authorization: $PAKT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"escrowId": "67fd70faedb5a6dcc87170cb"}'
```

Response includes `status` — should be `"funded"` or similar confirmed state.

## Step 4: Release

The payer releases funds to the receiver after work is verified:

```bash
curl -s -X POST "$PAKT_BASE_URL/v1/escrow/release" \
  -H "Authorization: $PAKT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"escrowId": "67fd70faedb5a6dcc87170cb"}'
```

Funds transfer to the receiver's Pakt wallet.

---

## Agent Escrow Pattern (POS Integration)

For autonomous agent-to-agent payments via the Pack Operating System:

1. **Get receiver's Pakt userId** — look up in `tq_agent_registry` or via `GET /v1/account/:userId`
2. **Initiate escrow** from payer agent's account
3. **Fund on-chain** using agent wallet's private key (stored in `.env` as `PAKT_WALLET_KEY`)
4. **Validate** once tx confirms
5. **Release** when task completion is verified (by human or automated check)

This makes `tq_tasks` economically binding — funds held in escrow until the assigned work is done.

---

## Key Fields Summary

| Field | Where | Purpose |
|-------|-------|---------|
| `collectionId` | initiate response | Use as `escrowId` in validate/release |
| `address` | initiate response | Blockchain address to send funds to |
| `amountToPay` | initiate response | Total including Pakt fee |
| `receiverId` | initiate request | Pakt `_id` of the person receiving funds |
| `coin` | initiate request | Lowercase: `"usdc"`, `"avax"` |
