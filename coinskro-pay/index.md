---
layout: default
title: Coinskro Pay
nav_order: 2
has_children: true
description: "Coinskro Pay - Accept cryptocurrency payments on your website"
permalink: /coinskro-pay/
---

# Coinskro Pay Integration Guide
{: .fs-9 }

Accept cryptocurrency payments on your website in minutes.
{: .fs-6 .fw-300 }

[Quick Start](#quick-start){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }
[View Examples](/coinskro-pay/examples){: .btn .fs-5 .mb-4 .mb-md-0 }

---

## What is Coinskro Pay?

Coinskro Pay is a cryptocurrency payment gateway that lets you accept crypto payments from your customers. Whether you run an e-commerce store, subscription service, or any online business, Coinskro Pay makes it easy to receive payments in USDT, BTC, ETH, and more.

### Why Choose Coinskro Pay?

✅ **Easy Integration** - Add payments in under 10 minutes  
✅ **Low Fees** - Competitive transaction fees  
✅ **Real-time Webhooks** - Get notified instantly when payments complete  
✅ **No Chargebacks** - Cryptocurrency payments are final  
✅ **Global Payments** - Accept payments from anywhere in the world  

---

## Quick Start

### Step 1: Get Your API Keys

1. Sign up at [coinskro.com](https://coinskro.com)
2. Create a new integration for your project
3. Copy your **Secret Key** from the dashboard

{: .warning }
> Keep your Secret Key secure! Never expose it in frontend code.

### Step 2: Create a Payment

When a customer wants to pay, create a payment from your server:

```bash
curl -X PUT https://api.coinskro.com/payment/create \
  -H "Authorization: Bearer YOUR_SECRET_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 50.00,
    "currency": "USDT",
    "payment_type": "deposit",
    "customer_email": "customer@example.com",
    "success_url": "https://yoursite.com/thank-you",
    "failure_url": "https://yoursite.com/payment-failed"
  }'
```

### Step 3: Redirect Customer

The API returns a `PaymentURL`. Redirect your customer to this URL to complete payment:

```json
{
  "message": "Payment created",
  "data": {
    "PaymentData": {
      "payment_reference": "PAY_abc123xyz",
      "amount": 50.00,
      "currency": "USDT"
    },
    "PaymentURL": "https://pay.coinskro.com?reference=PAY_abc123xyz&app=..."
  }
}
```

### Step 4: Handle the Result

After payment, the customer is redirected to your `success_url` or `failure_url`. For reliable order fulfillment, always use [webhooks](/coinskro-pay/webhooks).

---

## Integration Flow

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Your Site  │────▶│  Your API   │────▶│ Coinskro Pay│
│  (Frontend) │     │  (Backend)  │     │    API      │
└─────────────┘     └─────────────┘     └─────────────┘
       │                   │                   │
       │  1. Checkout      │                   │
       │─────────────────▶│                   │
       │                   │  2. Create Payment│
       │                   │─────────────────▶│
       │                   │                   │
       │                   │  3. Payment URL   │
       │                   │◀─────────────────│
       │  4. Redirect      │                   │
       │◀─────────────────│                   │
       │                   │                   │
       │         5. Customer pays on Coinskro  │
       │─────────────────────────────────────▶│
       │                   │                   │
       │                   │  6. Webhook       │
       │                   │◀─────────────────│
       │                   │                   │
       │  7. Order Complete│                   │
       │◀─────────────────│                   │
       │                   │                   │
```

---

## Next Steps

| Guide | Description |
|:------|:------------|
| [Create Payments](/coinskro-pay/payments) | Learn all payment options |
| [Handle Webhooks](/coinskro-pay/webhooks) | Get real-time payment notifications |
| [Code Examples](/coinskro-pay/examples) | Copy-paste examples in your language |
| [Testing](/coinskro-pay/testing) | Test your integration before going live |
| [Error Handling](/coinskro-pay/errors) | Handle errors gracefully |

---

## Need Help?

- 📧 Email: support@coinskro.com
- 💬 Telegram: [Join our community](https://t.me/coinskro)
- 🐦 X: [Follow us](https://x.com/coinskro)
- 📚 FAQ: [Frequently Asked Questions](/coinskro-pay/faq)
