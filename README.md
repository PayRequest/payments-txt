# payments.txt

> The open standard for machine-readable payment profiles.  
> Like `robots.txt` and `llms.txt` — but for getting paid by AI agents.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Standard: payments.txt v1.0](https://img.shields.io/badge/standard-payments.txt%20v1.0-2eb0c9)](spec/payments-txt-v1.md)
[![X402 compatible](https://img.shields.io/badge/X402-compatible-a8ff78)](spec/x402-integration.md)

---

## What is payments.txt?

`payments.txt` is a plain-text file served at `/{handle}/payments.txt` that tells AI agents, billing systems, and autonomous software how to pay you — without any human in the loop.

It describes:
- Your crypto wallets (EVM, Solana)
- Your bank details (SEPA, ACH)
- Your payment terms (due days, late fees, preferred currency)
- Your X402 endpoint for autonomous HTTP-level payments
- A human fallback URL for anything that needs a checkout page

Inspired by `robots.txt` (1994) and `llms.txt` (2024). `payments.txt` is the payment layer for the agentic web.

---

## The problem it solves

AI agents increasingly handle scheduling, procurement, invoicing, and payments on behalf of humans. Today, every payment requires either:

1. A human to click through a checkout page, or
2. A custom API integration per vendor

Neither scales. `payments.txt` gives agents a **standard discovery mechanism** — one file, consistent format, no integration work.

```
Agent reads:  https://vendor.com/payments.txt
Agent knows:  EVM wallet, preferred USDC, payment terms, X402 endpoint
Agent pays:   Autonomously, no human needed
```

---

## File format

`payments.txt` uses an INI-style format. All sections are optional — include only what applies to you.

```ini
# PAYMENTS.TXT
# Human-readable payment profile for autonomous agents

[meta]
version                  = "1.0"
status                   = "active"
x402_discovery_url       = "https://api.example.com/v1/x402/handle"
x402_payment_endpoint    = "https://api.example.com/v1/x402/handle/pay"

[payment_terms]
due_days                 = 14
late_fee_percentage      = 2.0
preferred_fiat           = "EUR"
preferred_crypto         = "USDC"

[crypto_rails]
supported_standards      = ["EVM", "Solana"]
primary_stablecoin       = "USDC"
evm_address              = "0xYourEVMAddress"
solana_address           = "YourSolanaAddress"

[fiat_rails]
sepa_iban                = "NL15BUNQ2106989466"
sepa_bic                 = "BUNQNL2AXXX"
ach_routing              = "021000021"
ach_account              = "000123456789"

[hosted_checkout]
human_fallback_url       = "https://payrequest.me/handle"
```

Full specification: [spec/payments-txt-v1.md](spec/payments-txt-v1.md)

---

## X402 integration

`payments.txt` pairs with the [X402 protocol](spec/x402-integration.md) for autonomous HTTP-level payments. X402 uses the `402 Payment Required` HTTP status code to negotiate and settle USDC payments without a human checkout flow.

```
GET  /{handle}/payments.txt            → discover wallets + X402 endpoint
POST /api/v1/x402/{handle}/pay         → 402 with payment requirements
POST /api/v1/x402/{handle}/pay         → X-Payment header → 200 + receipt
```

See [spec/x402-integration.md](spec/x402-integration.md) for the full protocol.

---

## Quick start

### Auto-generated (PayRequest)

Every [PayRequest](https://payrequest.me) page auto-publishes a `payments.txt` manifest. Fill in your wallet in settings — done.

Live example: [payrequest.me/geertjan/payments.txt](https://payrequest.me/geertjan/payments.txt)

### Static file

Drop a `payments.txt` in your site root served as `text/plain; charset=UTF-8`.

### HTML discovery tag

```html
<link rel="payments" type="text/plain" href="https://yoursite.com/payments.txt">
```

### Laravel

```php
Route::get('/{handle}/payments.txt', [PaymentManifestController::class, 'show']);
```

Full guide: [implementations/laravel.md](implementations/laravel.md)

### Node.js

```js
app.get('/:handle/payments.txt', (req, res) => {
  res.type('text/plain').send(`[meta]\nversion = "1.0"\nstatus = "active"\n\n[crypto_rails]\nevm_address = "${process.env.EVM_WALLET}"\n`);
});
```

Full guide: [implementations/nodejs.md](implementations/nodejs.md)

---

## Specification

| Document | Description |
|---|---|
| [spec/payments-txt-v1.md](spec/payments-txt-v1.md) | Full file format spec — all fields, types, parsing rules |
| [spec/x402-integration.md](spec/x402-integration.md) | X402 HTTP 402 autonomous payment flow |
| [spec/discovery.md](spec/discovery.md) | How agents discover payments.txt |

---

## Examples

| File | Description |
|---|---|
| [examples/minimal.txt](examples/minimal.txt) | Crypto-only, minimal setup |
| [examples/freelancer.txt](examples/freelancer.txt) | Freelancer with SEPA + USDC |
| [examples/business.txt](examples/business.txt) | Full business profile (USDC + SEPA + ACH) |
| [examples/crypto-native.txt](examples/crypto-native.txt) | All chains, no fiat |

---

## Implementations

| Language | Status | Link |
|---|---|---|
| PHP | ✅ Reference | [implementations/php.md](implementations/php.md) |
| Node.js / Express | ✅ Example | [implementations/nodejs.md](implementations/nodejs.md) |
| Python / FastAPI | 📋 Planned | — |
| WordPress plugin | 📋 Planned | — |

---

## How payments.txt compares

| | `robots.txt` | `llms.txt` | `payments.txt` |
|---|---|---|---|
| **Purpose** | Crawl control | LLM context | Payment discovery |
| **Audience** | Web crawlers | LLM agents | AI agents / billing systems |
| **Format** | Rules-based | Markdown | INI-style |
| **Year proposed** | 1994 | 2024 | 2025 |
| **Agent action** | Passive (skip) | Passive (read) | **Active (pay)** |
| **Requires server logic** | No | No | Optional (X402) |

---

## Contributing

This is an open standard. Contributions welcome:

- **New implementations** — Python, Go, Ruby, WordPress, etc.
- **Spec improvements** — edge cases, new rails (Lightning, Farcaster frames)
- **Tooling** — validators, parsers, agent SDKs

Open an issue or PR. No CLA required.

---

## License

MIT — free to use, implement, and extend.

Built by [PayRequest](https://payrequest.me) — the open payment layer for the agentic web.
