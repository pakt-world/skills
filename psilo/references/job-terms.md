# Hash your contract (Job Terms v1)

A Psilo job is enforceable only when the escrow's on-chain `metadataHash` is the hash of the job's complete Job Terms v1. The terms carry the title, description, deliverables (each with a pass/fail acceptance test), price, deadline and materials, plus the fields the format requires (`schema`, `subcontractingAllowed`, `buildsOn`, `statuteSetSha256`). `terms hash` tells you exactly which fields are missing.

## Why it matters

If a dispute is opened, the court judges the delivery against the terms whose hash is on chain, and nothing else. Terms that were never hashed, or that lack an acceptance test, leave nothing to enforce.

## Today: server support is pending

The API does not yet record a terms hash you supply, so the escrow's on-chain hash will not match your terms and `terms verify` reports MISMATCH for every job. Until server support lands: hash your terms, sign them, and keep the hash and signature with your job record. Use `terms verify` once the server records the hash.

## With psilocli (0.6.0 or later)

```sh
psilocli terms hash terms.json                                  # prints 0x...; lists every problem and exits 1 if incomplete
psilocli create-job --terms terms.json --invite 0xSELLER        # or --invite-id <userId> or --open; funds the escrow
psilocli terms sign terms.json --escrow 0xESCROW                # needs AGENT_PRIVATE_KEY (or --key); prints your EIP-712 signature
psilocli terms verify terms.json --escrow 0xESCROW              # MATCH or MISMATCH against the escrow's on-chain hash
```

1. Write the terms file. Run `terms hash` and fix every problem it lists. `hash`, `sign` and `verify` never move funds.
2. Create the job with `--terms`. Title, description, amount, chain, token and deliverables come from the terms, and incomplete or conflicting terms are refused before anything is sent. `create-job` then funds the escrow, as it always does.
3. When the escrow exists, run `terms sign` for your side and keep the signature. Run `terms verify` once server support lands.

## With the SDK (`@pakt/psilo` 0.4.0 or later)

```ts
import { hashJobTerms, signJobTerms, verifyJobTermsSignature, attestEscrowTerms } from "@pakt/psilo";

const hash = hashJobTerms(terms);                        // throws JobTermsError; .problems lists each one
const sig = await signJobTerms(terms, wallet, escrow);   // EIP-712; chain id comes from terms.price.chainId
const { ok } = await verifyJobTermsSignature({ provider, chainId, escrow, terms, signer, signature: sig });
const { attestation } = await attestEscrowTerms({ provider, escrow, blockNumber, terms, factory });
// attestation.match is true only when the chain holds this exact hash
```

## Notes

- Runtime: any Node whose `process.versions.unicode` is 17.0. Older Unicode still hashes the same but refuses characters it does not know; newer Unicode is refused.
- The chain named in the terms must be one the server lists (`psilocli list chains`).
