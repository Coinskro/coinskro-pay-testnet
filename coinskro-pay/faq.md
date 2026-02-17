---
layout: default
title: FAQ
parent: Coinskro Pay
nav_order: 6
permalink: /coinskro-pay/faq
---

# Frequently Asked Questions
{: .no_toc }

Common questions about integrating Coinskro Pay.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## General

### What is Coinskro Pay?

Coinskro Pay is a cryptocurrency payment gateway that allows you to accept crypto payments (USDT, BTC, ETH, etc.) on your website or application.

### What currencies are supported?

Currently supported currencies:
- **USDT** (Tether USD)
- **USDC** (USD Coin)
- **BTC** (Bitcoin)
- **ETH** (Ethereum)

Contact us to request additional currencies.

### What are the fees?

Transaction fees vary by currency and volume. Check your dashboard for current rates or contact sales for volume pricing.

### How long do payments take?

Most payments complete within 1-5 minutes, depending on blockchain confirmation times:
- USDT/USDC: ~1-2 minutes
- ETH: ~2-5 minutes
- BTC: ~10-30 minutes

---

## Integration

### Do I need a backend server?

**Yes.** For security, payment creation must happen from your server using your Secret Key. Never expose your Secret Key in frontend code.

### Can I use Coinskro Pay with my existing e-commerce platform?

Yes! We have guides for popular platforms:
- WooCommerce (WordPress)
- Shopify
- Magento

Contact us for platform-specific integration support.

### How do I test my integration?

Use **Test Mode** with `sk_test_` keys. Test payments don't involve real cryptocurrency. See our [Testing Guide](/coinskro-pay/testing).

### What happens if my webhook endpoint is down?

We retry failed webhooks up to 5 times over 24 hours. You can also manually retry webhooks from the dashboard.

---

## Payments

### How do I know when a payment is complete?

Use [webhooks](/coinskro-pay/webhooks) - we'll POST to your URL when payment completes. Don't rely only on redirect URLs.

### Can I refund a payment?

Cryptocurrency payments are final and cannot be reversed. For refunds, you'll need to send a separate payment to the customer.

### What if a customer pays the wrong amount?

Our system validates payment amounts. If there's a discrepancy, the customer can raise an appeal through the payment interface.

### How do I handle disputes?

Disputes (appeals) are managed through your dashboard. Both you and the customer can submit evidence, and our team helps resolve disputes fairly.

### Can I accept partial payments?

Not currently. Customers must pay the full amount in a single transaction.

---

## Security

### How secure is Coinskro Pay?

We use industry-standard security practices:
- All API calls over HTTPS
- API keys are never logged
- Webhook signatures for verification
- Regular security audits

### How should I store my API keys?

- Store in environment variables, never in code
- Use different keys for test and production
- Rotate keys periodically
- Never commit keys to version control

### Can I restrict API key usage?

Yes, in your dashboard you can:
- Whitelist IP addresses
- Set spending limits
- Enable/disable specific features

---

## Webhooks

### Why am I not receiving webhooks?

Common causes:
1. Webhook not activated in dashboard
2. URL not publicly accessible
3. Server returning non-200 status
4. Firewall blocking requests

### How do I verify webhooks are from Coinskro?

Check the `X-Coinskro-Signature` header. See [Webhook Verification](/coinskro-pay/webhooks#verifying-webhook-signatures).

### What if I process the same webhook twice?

Your code should be **idempotent** - check if you've already processed a payment before fulfilling the order.

---

## Troubleshooting

### "Integration is not yet approved" error

Your integration needs approval before accepting live payments. You can:
1. Wait for approval (usually 24-48 hours)
2. Use test mode while waiting

### Payment stuck in "pending" status

This can happen if:
1. Customer hasn't completed the payment
2. Blockchain confirmation is delayed
3. Payment amount mismatch

Check the payment details in your dashboard.

### Webhook returns 404 error

Verify your webhook URL:
1. Is the path correct? (e.g., `/webhooks/coinskro` not `/webhook/coinskro`)
2. Is your server running?
3. Is the route publicly accessible?

### "Unauthorized" error on API calls

Check your Authorization header:
- Format: `Authorization: Bearer sk_live_xxxxx`
- No extra spaces or characters
- Using the correct key (test vs live)

---

## Account & Billing

### How do I get my earnings?

Funds accumulate in your integration wallet. You can withdraw to your personal wallet from the dashboard.

### When can I withdraw?

Withdrawals are available once your balance exceeds the minimum withdrawal amount (varies by currency).

### How do I view my transaction history?

Log in to your [dashboard](https://coinskro.com) to view all payments, withdrawals, and account activity.

---

## Still Have Questions?

- 📧 **Email:** support@coinskro.com
- 💬 **Discord:** [Join our community](https://discord.gg/coinskro)
- 📞 **Enterprise Support:** Contact your account manager
