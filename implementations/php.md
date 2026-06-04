# PHP Implementation

Serve a `payments.txt` manifest from any PHP application. No framework required.

## Static file (simplest)

If your payment details never change, just drop a plain text file and serve it directly:

```
public/payments.txt
```

Configure your web server to serve it as `text/plain`:

```nginx
location ~ /payments\.txt$ {
    default_type text/plain;
}
```

```apache
<Files "payments.txt">
    ForceType text/plain
</Files>
```

---

## Dynamic PHP (single file)

Generate the manifest on the fly from your data. Create `payments.php` and route requests to it.

```php
<?php
// payments.php — serve at /{handle}/payments.txt via routing or .htaccess

declare(strict_types=1);

header('Content-Type: text/plain; charset=UTF-8');
header('Cache-Control: public, max-age=3600');
header('X-Robots-Tag: noindex');

// Load your user/merchant data however you store it
$handle        = $_GET['handle'] ?? 'you';
$evm_address   = '';   // e.g. from database
$solana_address = '';
$iban          = '';
$bic           = '';
$currency      = 'EUR';
$app_url       = 'https://api.example.com';
$site_url      = 'https://example.com';

// Only include a section if there is data for it
$lines = [];

$lines[] = '# PAYMENTS.TXT MANIFEST (X402 COMPLIANT)';
$lines[] = '# ' . $site_url . '/' . $handle . '/payments.txt';
$lines[] = '';
$lines[] = '[meta]';
$lines[] = 'version = "1.0"';
$lines[] = 'status  = "active"';

if ($app_url) {
    $lines[] = 'x402_discovery_url    = "' . $app_url . '/x402/' . $handle . '"';
    $lines[] = 'x402_payment_endpoint = "' . $app_url . '/x402/' . $handle . '/pay"';
}

$lines[] = '';
$lines[] = '[payment_terms]';
$lines[] = 'due_days           = 14';
$lines[] = 'late_fee_percentage = 2.0';
$lines[] = 'preferred_fiat     = "' . strtoupper($currency) . '"';
$lines[] = 'preferred_crypto   = "USDC"';

if ($evm_address || $solana_address) {
    $supported = array_filter([
        $evm_address    ? '"EVM"'    : null,
        $solana_address ? '"Solana"' : null,
    ]);

    $lines[] = '';
    $lines[] = '[crypto_rails]';
    $lines[] = 'supported_standards = [' . implode(', ', $supported) . ']';
    $lines[] = 'primary_stablecoin  = "USDC"';

    if ($evm_address) {
        $lines[] = 'evm_address    = "' . $evm_address . '"';
    }
    if ($solana_address) {
        $lines[] = 'solana_address = "' . $solana_address . '"';
    }
}

if ($iban || $bic) {
    $lines[] = '';
    $lines[] = '[fiat_rails]';

    if ($iban) {
        $lines[] = 'sepa_iban = "' . strtoupper(preg_replace('/\s+/', '', $iban)) . '"';
    }
    if ($bic) {
        $lines[] = 'sepa_bic  = "' . strtoupper(trim($bic)) . '"';
    }
}

$lines[] = '';
$lines[] = '[hosted_checkout]';
$lines[] = 'human_fallback_url = "' . $site_url . '/' . $handle . '"';

echo implode("\n", $lines) . "\n";
```

---

## Routing the URL

You want requests to `/{handle}/payments.txt` to hit your PHP file. Options:

**.htaccess (Apache)**

```apache
RewriteEngine On
RewriteRule ^([a-zA-Z0-9_-]+)/payments\.txt$ payments.php?handle=$1 [L,QSA]
```

**Nginx**

```nginx
location ~ ^/([a-zA-Z0-9_-]+)/payments\.txt$ {
    fastcgi_param QUERY_STRING handle=$1;
    include fastcgi_params;
    fastcgi_pass unix:/run/php/php-fpm.sock;
    fastcgi_param SCRIPT_FILENAME /var/www/html/payments.php;
}
```

**index.php front controller** (if you already have one)

```php
// in your router
if (preg_match('#^/([a-zA-Z0-9_-]+)/payments\.txt$#', $path, $m)) {
    $handle = $m[1];
    // include payments.php or call your payments manifest function
}
```

---

## HTML discovery tag

Add to every profile page `<head>` so agents can find the manifest automatically:

```html
<link rel="payments" type="text/plain"
      href="https://example.com/<?= htmlspecialchars($handle) ?>/payments.txt">
```

---

## Minimal static example

If you only need a single-owner file at `/payments.txt`:

```
# PAYMENTS.TXT
[meta]
version = "1.0"
status  = "active"

[crypto_rails]
evm_address = "0xYourAddressHere"

[hosted_checkout]
human_fallback_url = "https://example.com/pay"
```

Serve this file as-is. `Content-Type: text/plain` is all that is required.
