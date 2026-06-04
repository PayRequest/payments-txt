# payments.txt v1.0 Specification

## Overview

`payments.txt` is a plain-text INI-style file that describes how to pay a person or business. It is served over HTTP and intended to be consumed by AI agents, billing systems, and any software that needs to initiate a payment without human interaction.

## Serving the file

- **Content-Type**: `text/plain; charset=UTF-8`
- **Recommended path**: `/{handle}/payments.txt` or `/payments.txt`
- **Cache**: Recommend `Cache-Control: public, max-age=3600`
- **Discovery tag**: Add to HTML `<head>`:
  ```html
  <link rel="payments" type="text/plain" href="https://example.com/payments.txt">
  ```

## File structure

The file consists of comments (lines starting with `#`) and sections (`[section_name]`). All sections are optional. Unknown fields within a section MUST be ignored by parsers.

```
# Comment
[section]
key = value
key = "string value"
key = 42
key = ["array", "of", "strings"]
```

---

## Sections

### `[meta]`

Required section. Describes the file itself and links to machine-payment endpoints.

| Field | Type | Required | Description |
|---|---|---|---|
| `version` | string | yes | Spec version. Currently `"1.0"`. |
| `status` | string | yes | `"active"` or `"inactive"`. Agents MUST NOT attempt payment if `"inactive"`. |
| `x402_discovery_url` | string (URL) | no | URL returning the X402 discovery JSON (see [x402-integration.md](x402-integration.md)). |
| `x402_payment_endpoint` | string (URL) | no | URL accepting `POST` with `X-Payment` header (X402 protocol). |

### `[payment_terms]`

Describes the payment expectations for invoices and services.

| Field | Type | Default | Description |
|---|---|---|---|
| `due_days` | integer | 14 | Days until payment is due after invoice date. |
| `late_fee_percentage` | decimal | 2.0 | Monthly late fee as a percentage of the outstanding amount. |
| `preferred_fiat` | string | `"EUR"` | ISO 4217 currency code for fiat invoices. |
| `preferred_crypto` | string | `"USDC"` | Preferred crypto token for on-chain payments. |

### `[crypto_rails]`

On-chain payment addresses.

| Field | Type | Description |
|---|---|---|
| `supported_standards` | array | Networks supported. Values: `"EVM"`, `"Solana"`, `"Bitcoin"`. |
| `primary_stablecoin` | string | Preferred stablecoin. Typically `"USDC"`. |
| `evm_address` | string | ERC-55 checksummed Ethereum/EVM address. |
| `solana_address` | string | Base58-encoded Solana public key. |
| `btc_address` | string | Bitcoin address (any format). |

**Validation rules:**
- `evm_address` MUST match `/^0x[0-9a-fA-F]{40}$/`
- `solana_address` MUST be 32–44 base58 characters
- Agents SHOULD verify addresses are checksum-valid before sending funds

### `[fiat_rails]`

Traditional bank payment details.

| Field | Type | Description |
|---|---|---|
| `sepa_iban` | string | IBAN (spaces stripped, uppercase). |
| `sepa_bic` | string | BIC/SWIFT code. |
| `ach_routing` | string | US ACH routing number (9 digits). |
| `ach_account` | string | US bank account number. |

### `[hosted_checkout]`

Human-facing fallback when autonomous payment is not possible.

| Field | Type | Description |
|---|---|---|
| `human_fallback_url` | string (URL) | URL of a checkout or payment page. |

---

## Parsing rules

1. Lines starting with `#` are comments and MUST be ignored.
2. Empty lines MUST be ignored.
3. Section headers are `[name]` on their own line.
4. Key-value pairs are `key = value` (spaces around `=` are optional).
5. String values MAY be quoted with `"`. Quotes are stripped.
6. Arrays are `["item1", "item2"]`.
7. Unknown sections and unknown fields within known sections MUST be silently ignored.
8. If a required field in `[meta]` is missing, the file is considered invalid.

---

## Security considerations

- Parsers MUST NOT execute any content from the file.
- Agents MUST validate wallet addresses before sending funds.
- Agents SHOULD verify the file is served over HTTPS.
- Agents SHOULD respect `status = "inactive"` and not attempt payment.
- The file is public — do not include private keys or secrets.

---

## Versioning

The `version` field in `[meta]` indicates the spec version. Current version: `1.0`.

Future versions will be backwards compatible. Parsers that encounter `version = "2.0"` SHOULD still attempt to parse known fields.

---

## MIME type

`payments.txt` files should be served as `text/plain`. There is no registered MIME type for this format — `text/plain` is intentional for maximum compatibility.
