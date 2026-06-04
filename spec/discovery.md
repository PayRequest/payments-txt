# Discovery — How agents find payments.txt

## 1. HTML link tag (recommended)

Add to your page `<head>`:

```html
<link rel="payments" type="text/plain" href="https://yoursite.com/payments.txt">
```

Agents parsing your page HTML will find this tag and know where your manifest is.

## 2. Well-known path

Serve at a predictable URL:

- `/{handle}/payments.txt` (per-user, e.g. `payrequest.me/geertjan/payments.txt`)
- `/payments.txt` (site-wide)

## 3. HTTP Link header

Optionally include in any HTTP response:

```
Link: <https://yoursite.com/payments.txt>; rel="payments"
```

## 4. X402 discovery JSON

The `x402_discovery_url` in `[meta]` points to a JSON endpoint for programmatic discovery:

```json
{
  "x402_version": 2,
  "username": "handle",
  "status": "active",
  "manifest_url": "https://payrequest.me/handle/payments.txt",
  "payment_endpoint": "https://payrequest.app/api/v1/x402/handle/pay",
  "accepts": [
    {
      "scheme": "exact",
      "network": "eip155:8453",
      "asset": "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
      "amount": "1000",
      "payTo": "0xMerchantAddress",
      "maxTimeoutSeconds": 30
    }
  ]
}
```

## Priority order

Agents SHOULD use this priority order:

1. HTML `<link rel="payments">` tag
2. `/{handle}/payments.txt` (if accessing a user profile URL)
3. `/payments.txt` site root
4. HTTP `Link` header
