# Pakt API Reference

Base URL: `https://api.pakt.world` (or your Chainsite's base URL)
Auth header: `Authorization: <JWT_TOKEN>` on all endpoints

---

## Authentication

### POST /v1/auth/create-account
Create a new user account.
```json
{
  "firstName": "Alice",
  "lastName": "Bob",
  "email": "alice@example.com",
  "password": "1234@Abcd",
  "confirmPassword": "1234@Abcd"
}
```
Returns: `data.tempToken` (use for verification step)

### POST /v1/auth/account/verify
Verify account with email code.
```json
{ "token": "123456", "tempToken": "eyJ..." }
```
Returns: permanent JWT `data.token`

### POST /v1/auth/login
```json
{ "email": "alice@example.com", "password": "1234@Abcd" }
```
Returns: `data.token`, `data.isVerified`
Note: May return `tempToken` if 2FA is enabled — follow up with `/v1/auth/login/2fa`

### POST /v1/auth/login/2fa
```json
{ "tempToken": "eyJ...", "code": "123456" }
```
Returns: full user profile + JWT token

### POST /v1/auth/2fa/email/code
Send 2FA code to email.
```json
{ "email": "alice@example.com" }
```

### POST /v1/auth/password/reset
Initiate password reset.
```json
{ "email": "alice@example.com" }
```
Returns: `data.tempToken`

---

## Account

### GET /v1/account
Get current user's full profile. No body required.
Returns: full user object including `_id`, profile, wallet status, achievements, KYC status.

### PATCH /v1/account/update
Update profile fields (all optional):
```json
{
  "firstName": "Alice",
  "lastName": "Bob",
  "profile": {
    "contact": { "country": "US", "state": "IL", "city": "Chicago", "phone": "+13109483734" },
    "bio": { "title": "Software Engineer", "description": "Builds things" },
    "talent": {
      "about": "Extended bio here",
      "availability": "available",
      "tags": ["product", "engineering", "web3"]
    }
  },
  "socials": {
    "github": "https://github.com/username",
    "twitter": "https://twitter.com/username",
    "linkedin": "https://linkedin.com/username",
    "website": "https://mysite.com"
  }
}
```
Availability options: `"available"` | `"limited"` | `"unavailable"`

### GET /v1/account/:userId
Get another user's public profile by their `_id`.

### POST /v1/account/password/change
```json
{ "oldPassword": "old", "newPassword": "new@Abcd" }
```

---

## Escrow

### POST /v1/escrow/initiate
Create a new escrow.
```json
{
  "title": "Project Payment",
  "description": "Payment for deliverable X",
  "amount": 100,
  "coin": "usdc",
  "dueDate": "2026-12-31",
  "isPrivate": false,
  "isFundingRequest": false,
  "receiverId": "USER_MONGO_ID"
}
```
Returns:
- `coin`, `address` (deposit address), `escrowAmount`, `coinEscrowAmount`
- `amountToPay`, `expectedFee`, `feePercentage`, `usdFee`
- `chainId`, `contractAddress`, `paymentMethods`
- `collectionId` — use this as the `escrowId` in subsequent calls

### POST /v1/escrow/validate
Confirm funds received on-chain.
```json
{ "escrowId": "COLLECTION_ID" }
```
Returns: `escrowId`, `paymentAmount`, `paymentAddress`, `receiverId`, `status`

### POST /v1/escrow/release
Release funds to receiver.
```json
{ "escrowId": "COLLECTION_ID" }
```

---

## Wallet

### GET /v1/wallet
Get all wallet balances.
Returns: `totalBalance` (USD), `wallets[]` with `{ id, coin, amount, usdValue, address, icon }`

### GET /v1/wallet/coin/:coin
Get single coin balance. `:coin` = ticker, e.g. `usdc`, `avax`

### POST /v1/wallet/export-secret-key
Export wallet private key (requires 2FA code + password).
```json
{ "code": "123456", "password": "your_password" }
```
Returns: `secretKey`, `walletAddress`

### POST /v1/withdrawals/
Initiate withdrawal.
```json
{
  "amount": 100,
  "coin": "USDC",
  "address": "0xRECIPIENT_ADDRESS",
  "password": "your_password"
}
```
Returns: `withdrawalId`

### GET /v1/withdrawals/
List withdrawal history (paginated).

### GET /v1/transaction
List transactions (paginated).

### GET /v1/transaction/exchange
Get exchange rate data for supported coins.

---

## File Upload

### POST /v1/upload
Upload public file. `Content-Type: multipart/form-data`
Form field: `file` (binary)
Returns: `data.url` (CloudFront CDN), `data._id`, `data.name`, `data.type`, `data.size`

### POST /v1/upload/private
Upload private file. Same format as above.
File is accessible only to the authenticated user.

---

## Payment / Blockchain

### GET /v1/payment/coins
List all supported cryptocurrencies.
Returns array of: `{ _id, name, symbol, reference, icon, contractAddress, decimal, isToken, rpcChainId, active }`

### GET /v1/payment/rpc
Get blockchain RPC configuration.
Returns: `{ rpcName, rpcChainId, rpcUrls[], blockExplorerUrls[], rpcNativeCurrency }`
Note: Chain is Avalanche Testnet C-Chain (chainId: 43113) in current deployment.
