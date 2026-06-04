# Node.js / Express Implementation

```js
import express from 'express';

const app = express();

app.get('/:handle/payments.txt', async (req, res) => {
  const { handle } = req.params;

  const user = await User.findByHandle(handle);
  if (!user) return res.status(404).send('Not found');

  const lines = [
    '# PAYMENTS.TXT MANIFEST (X402 COMPLIANT)',
    `# Generated for @${handle}`,
    '',
    '[meta]',
    'version = "1.0"',
    'status  = "active"',
    `x402_payment_endpoint = "https://api.yoursite.com/v1/x402/${handle}/pay"`,
    '',
    '[payment_terms]',
    'due_days         = 14',
    'preferred_fiat   = "EUR"',
    'preferred_crypto = "USDC"',
  ];

  if (user.evmAddress) {
    lines.push('', '[crypto_rails]');
    lines.push('supported_standards = ["EVM"]');
    lines.push(`evm_address = "${user.evmAddress}"`);
  }

  if (user.iban) {
    lines.push('', '[fiat_rails]');
    lines.push(`sepa_iban = "${user.iban.replace(/\s/g, '').toUpperCase()}"`);
  }

  lines.push('', '[hosted_checkout]');
  lines.push(`human_fallback_url = "https://yoursite.com/pay/${handle}"`);

  res
    .type('text/plain')
    .set('Cache-Control', 'public, max-age=3600')
    .send(lines.join('\n') + '\n');
});
```

Add the discovery tag to your HTML:

```html
<link rel="payments" type="text/plain" href="https://yoursite.com/payments.txt">
```
