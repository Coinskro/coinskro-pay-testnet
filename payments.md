---
layout: default
title: Creating Payments
nav_order: 2
permalink: /payments
---

# Creating Payments
{: .no_toc }

Learn how to create and manage payments with Coinskro Pay.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Basic Payment

Create a simple payment request:

```bash
curl -X PUT https://api.coinskro.com/payment/create \
  -H "Authorization: Bearer YOUR_SECRET_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 100.00,
    "currency": "PI",
    "payment_type": "deposit"
  }'
```

**Required Fields:**

| Field | Type | Description |
|:------|:-----|:------------|
| `amount` | number | Amount to charge |
| `currency` | string | Currency code (`PI`, `USDC`, `CSK`, `USDT`) |
| `payment_type` | string | Use `deposit` for customer payments |

---

## Payment with Customer Info

Track which customer made the payment:

```json
{
  "amount": 100.00,
  "currency": "PI",
  "payment_type": "deposit",
  "customer_email": "john@example.com",
  "customer_reference": "CUST_12345"
}
```

---

## Payment with Redirect URLs

Redirect customers after payment:

```json
{
  "amount": 100.00,
  "currency": "PI",
  "payment_type": "deposit",
  "success_url": "https://yoursite.com/order/success?order_id=123",
  "failure_url": "https://yoursite.com/order/failed?order_id=123"
}
```

{: .note }
> Include your order ID in the URL to identify the order when the customer returns.

---

## Payment with Items (E-commerce)

For e-commerce, include product details:

```json
{
  "amount": 150.00,
  "currency": "PI",
  "payment_type": "deposit",
  "customer_email": "customer@example.com",
  "item_list": {
    "items": [
      {
        "name": "Premium Plan",
        "price": 100.00,
        "quantity": 1
      },
      {
        "name": "Setup Fee",
        "price": 50.00,
        "quantity": 1
      }
    ]
  }
}
```

---

## Custom Payment Reference

Use your own order ID as the payment reference:

```json
{
  "amount": 100.00,
  "currency": "PI",
  "payment_type": "deposit",
  "payment_reference": "ORDER_2026_00123"
}
```

{: .warning }
> Payment references must be unique. If you don't provide one, we'll generate it automatically.

---

## Add Metadata

Store custom data with the payment:

```json
{
  "amount": 100.00,
  "currency": "PI",
  "payment_type": "deposit",
  "meta_data": "{\"order_id\": \"123\", \"product_type\": \"subscription\", \"plan\": \"pro\"}"
}
```

---

## Fee Options

Choose who pays the transaction fee:

```json
{
  "amount": 100.00,
  "currency": "PI",
  "payment_type": "deposit",
  "fee_payer": "customer"
}
```

| Option | Description |
|:-------|:------------|
| `merchant` | You pay the fee (default). Customer pays $100, you receive $99 |
| `customer` | Customer pays the fee. Customer pays $101, you receive $100 |

---

## Response

```json
{
  "message": "Payment created",
  "data": {
    "PaymentData": {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "payment_reference": "PAY_abc123xyz",
      "amount": 100.00,
      "currency": "PI",
      "payment_status": "unlinked",
      "payment_type": "deposit",
      "created_at": "2026-02-06T10:30:00Z"
    },
    "PaymentURL": "https://pay.coinskro.com?reference=PAY_abc123xyz&app=123..."
  }
}
```

**What to do with the response:**

1. Save the `payment_reference` in your database
2. Redirect your customer to `PaymentURL`
3. Wait for the [webhook](/webhooks) to confirm payment

---

## Check Payment Status

Verify a payment's status anytime:

```bash
curl https://api.coinskro.com/payment/info/PAY_abc123xyz
```

**Response:**

```json
{
  "message": "Payment information",
  "data": {
    "payment_reference": "PAY_abc123xyz",
    "amount": 100.00,
    "currency": "PI",
    "payment_status": "completed",
    "created_at": "2026-02-06T10:30:00Z",
    "updated_at": "2026-02-06T10:35:00Z"
  }
}
```

---

## Payment Statuses

| Status | Meaning | Action |
|:-------|:--------|:-------|
| `unlinked` | Payment created, waiting for customer | Customer should complete payment |
| `linked` | Customer started the payment | Payment in progress |
| `pending` | Payment being processed | Wait for completion |
| `completed` | Payment successful ✅ | Fulfill the order |
| `failed` | Payment failed | Ask customer to retry |
| `cancelled` | Payment cancelled | No action needed |
| `in-appeal` | Payment disputed | Check appeals dashboard |

---

## Supported Currencies

| Code | Currency | Min Amount |
|:-----|:---------|:-----------|
| `PI` | PI Coin | $1.00 |
| `USDC` | USD Coin | $1.00 |
| `USDT` | Tether | $5.00 |
| `CSK` | Coinskro Token | $5.00 |

---

## Complete Example

Here's a full example creating a payment for an e-commerce order:

```json
{
  "amount": 299.99,
  "currency": "PI",
  "payment_type": "deposit",
  "payment_reference": "ORD_20260206_001",
  "customer_email": "buyer@example.com",
  "customer_reference": "USER_789",
  "success_url": "https://mystore.com/checkout/success?order=ORD_20260206_001",
  "failure_url": "https://mystore.com/checkout/failed?order=ORD_20260206_001",
  "payment_note": "Order #ORD_20260206_001",
  "fee_payer": "merchant",
  "item_list": {
    "items": [
      {
        "name": "Wireless Headphones",
        "price": 249.99,
        "quantity": 1
      },
      {
        "name": "Shipping",
        "price": 50.00,
        "quantity": 1
      }
    ]
  },
  "meta_data": "{\"shipping_address\": \"123 Main St\", \"coupon\": \"SAVE10\"}"
}
```
