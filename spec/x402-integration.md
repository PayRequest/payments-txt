# X402 Integration

`payments.txt` is the **discovery** layer. X402 is the **payment** layer. Together they enable fully autonomous machine-to-machine payments over HTTP.

## What is X402?

X402 repurposes the HTTP `402 Payment Required` status code (defined in 1991, never widely used) as a machine-readable payment negotiation protocol.

- **Specified by**: Coinbase (open spec: [github.com/coinbase/x402](https://github.com/coinbase/x402))
- **Payment method**: USDC on Base (EVM) or Solana
- **Signing scheme**: ERC-3009 (`transferWithAuthorization`) for EVM

---

## The full flow

```
1. Agent reads /{handle}/payments.txt
   → finds x402_payment_endpoint

2. Agent POST {x402_payment_endpoint}   (no X-Payment header)
   ← HTTP 402 + JSON body with `accepts` array

3. Agent constructs payment off-chain:
   - Signs ERC-3009 transferWithAuthorization message
   - Encodes as X-Payment: base64(JSON)

4. Agent POST {x402_payment_endpoint}   (with X-Payment header)
   ← HTTP 200 + X-Payment-Response header + resource payload
   → USDC settles on-chain
```

---

## 402 Response body

When no `X-Payment` header is present, the server responds:

```
HTTP/1.1 402 Payment Required
Content-Type: application/json
WWW-Authenticate: X402
X-X402-Version: 2

{
  "x402Version": 2,
  "error": "payment_required",
  "accepts": [
    {
      "scheme": "exact",
      "network": "eip155:8453",
      "asset": "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
      "amount": "1000",
      "payTo": "0xMerchantEVMAddress",
      "maxTimeoutSeconds": 30
    }
  ],
  "resource": {
    "url": "https://payrequest.me/handle",
    "description": "Payment to @handle via PayRequest X402",
    "mimeType": "application/json"
  }
}
```

**Fields:**
- `network`: CAIP-2 format. `eip155:8453` = Base mainnet, `eip155:84532` = Base Sepolia
- `asset`: USDC contract address on the specified network
- `amount`: In atomic units (USDC has 6 decimals: `1000` = 0.001 USDC)
- `payTo`: Merchant's EVM wallet address

---

## X-Payment header

The agent constructs an ERC-3009 `transferWithAuthorization` authorization and encodes it as:

```
X-Payment: base64(JSON)
```

```json
{
  "x402Version": 2,
  "payload": {
    "signature": "0xEIP712Signature...",
    "authorization": {
      "from": "0xAgentWalletAddress",
      "to":   "0xMerchantWalletAddress",
      "value": "1000",
      "validAfter":  "1700000000",
      "validBefore": "1700086400",
      "nonce": "0x0000000000000000000000000000000000000000000000000000000000000001"
    }
  },
  "accepted": {
    "scheme": "exact",
    "network": "eip155:8453",
    "asset": "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
    "amount": "1000",
    "payTo": "0xMerchantWalletAddress",
    "maxTimeoutSeconds": 30
  }
}
```

The `signature` is an EIP-712 signature of a `TransferWithAuthorization` typed data message, signed by the agent's EVM wallet.

---

## 200 Response

On successful verification and settlement:

```
HTTP/1.1 200 OK
Content-Type: application/json
X-Payment-Response: base64(JSON)
X-X402-Version: 2

{
  "success": true,
  "transaction_id": 12345,
  "onchain_tx": "0xTxHash...",
  "payer": "0xAgentWalletAddress",
  "amount_usdc": 0.001
}
```

The `X-Payment-Response` header contains:
```json
{
  "success": true,
  "payer": "0xAgentWalletAddress",
  "transaction": "0xTxHash",
  "network": "eip155:8453",
  "amount": "1000"
}
```

---

## Invoice payments

To pay a specific invoice (not just a generic API fee):

```
POST /api/v1/x402/{handle}/invoice/{invoiceId}/pay
```

Same flow, but the `amount` in the 402 response equals the invoice amount, and on success the invoice is automatically marked as paid.

---

## Network reference

| Network | CAIP-2 | USDC Contract |
|---|---|---|
| Base mainnet | `eip155:8453` | `0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913` |
| Base Sepolia (testnet) | `eip155:84532` | `0x036CbD53842c5426634e7929541eC2318f3dCF7e` |
| Solana mainnet | `solana:5eykt4UsFv2P6xyS5Tnrsinsn47Y5ECON9UfuB3qwxM` | `EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v` |
| Solana devnet | `solana:EtWTRABZaYq6iMfeYKouRu166VU2xqa1wcaWoxPkrZBG` | `4zMMC9srt5Ri5X14GAgXhaHii3GnPAEERYPJgZJDncDU` |

---

## Settlement options

Resource servers can settle payments via:

1. **Coinbase CDP facilitator** — managed, no gas wallet needed. Get a free key at [cdp.coinbase.com](https://cdp.coinbase.com)
2. **Self-hosted via viem** — call `transferWithAuthorization` directly from your own Base wallet. Requires a small amount of ETH for gas (~$2 lasts thousands of transactions).

PayRequest uses option 2 by default, auto-selecting based on `.env` configuration.
