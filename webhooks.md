---
layout: default
title: Webhooks
nav_order: 3
permalink: /webhooks
---

# Webhooks
{: .no_toc }

Get notified instantly when payments are completed.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Why Use Webhooks?

{: .important }
> **Always use webhooks to confirm payments.** Don't rely only on redirect URLs - customers might close their browser before being redirected.

Webhooks are HTTP callbacks that notify your server when events happen (like a payment completing). They're the most reliable way to know when to fulfill orders.

---

## Setup Webhooks

1. Go to your [Coinskro Dashboard](https://dashboard.coinskro.com)
2. Select your integration
3. Click **Webhooks** → **Add Webhook**
4. Enter your webhook URL (e.g., `https://yoursite.com/webhooks/coinskro`)
5. Select events to subscribe to
6. Click **Save** and **Activate**

---

## Webhook Events

Subscribe to the events you need:

| Event | When it fires |
|:------|:--------------|
| `payment_linked` | Customer started the payment process |
| `payment_completed` | Payment was successful ✅ |
| `payment_abandoned` | Payment expired without completion |
| `payment_canceled` | Payment was cancelled |

{: .note }
> Most integrations only need `payment_completed`.

---

## Webhook Payload

When an event occurs, we POST to your URL:

```json
{
  "event": "payment_completed",
  "timestamp": "2026-02-06T10:35:00Z",
  "data": {
    "payment_reference": "PAY_abc123xyz",
    "integration_id": "123e4567-e89b-12d3-a456-426614174000",
    "amount": 100.00,
    "currency": "PI",
    "status": "completed",
    "customer_email": "customer@example.com",
    "customer_reference": "CUST_12345",
    "meta_data": "{\"order_id\": \"123\"}"
  }
}
```

---

## Handling Webhooks

### Node.js / Express

```javascript
const express = require('express');
const app = express();

app.use(express.json());

app.post('/webhooks/coinskro', (req, res) => {
  const { event, data } = req.body;
  
  if (event === 'payment_completed') {
    const { payment_reference, amount, customer_email } = data;
    
    // 1. Verify this payment exists in your database
    // 2. Check if not already processed (idempotency)
    // 3. Fulfill the order
    
    console.log(`Payment ${payment_reference} completed for $${amount}`);
    
    // Mark order as paid in your database
    // await db.orders.update({ payment_reference }, { status: 'paid' });
    
    // Send confirmation email
    // await sendEmail(customer_email, 'Order Confirmed!');
  }
  
  // Always respond with 200 OK
  res.status(200).json({ received: true });
});

app.listen(3000);
```

### Python / Flask

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/webhooks/coinskro', methods=['POST'])
def coinskro_webhook():
    payload = request.json
    event = payload.get('event')
    data = payload.get('data')
    
    if event == 'payment_completed':
        payment_reference = data['payment_reference']
        amount = data['amount']
        
        # 1. Verify this payment exists in your database
        # 2. Check if not already processed (idempotency)
        # 3. Fulfill the order
        
        print(f"Payment {payment_reference} completed for ${amount}")
        
        # Mark order as paid
        # db.orders.update(payment_reference, status='paid')
    
    return jsonify({'received': True}), 200

if __name__ == '__main__':
    app.run(port=3000)
```

### PHP

```php
<?php
$payload = json_decode(file_get_contents('php://input'), true);

$event = $payload['event'];
$data = $payload['data'];

if ($event === 'payment_completed') {
    $payment_reference = $data['payment_reference'];
    $amount = $data['amount'];
    
    // 1. Verify this payment exists in your database
    // 2. Check if not already processed (idempotency)
    // 3. Fulfill the order
    
    error_log("Payment {$payment_reference} completed for \${$amount}");
    
    // Mark order as paid in your database
    // $pdo->prepare("UPDATE orders SET status = 'paid' WHERE payment_reference = ?")->execute([$payment_reference]);
}

// Always respond with 200 OK
http_response_code(200);
echo json_encode(['received' => true]);
```

### Go

```go
package main

import (
    "encoding/json"
    "fmt"
    "net/http"
)

type WebhookPayload struct {
    Event string `json:"event"`
    Data  struct {
        PaymentReference string  `json:"payment_reference"`
        Amount           float64 `json:"amount"`
        Currency         string  `json:"currency"`
        CustomerEmail    string  `json:"customer_email"`
    } `json:"data"`
}

func webhookHandler(w http.ResponseWriter, r *http.Request) {
    var payload WebhookPayload
    json.NewDecoder(r.Body).Decode(&payload)
    
    if payload.Event == "payment_completed" {
        fmt.Printf("Payment %s completed for $%.2f\n", 
            payload.Data.PaymentReference, 
            payload.Data.Amount)
        
        // 1. Verify this payment exists in your database
        // 2. Check if not already processed (idempotency)  
        // 3. Fulfill the order
    }
    
    w.WriteHeader(http.StatusOK)
    json.NewEncoder(w).Encode(map[string]bool{"received": true})
}

func main() {
    http.HandleFunc("/webhooks/coinskro", webhookHandler)
    http.ListenAndServe(":3000", nil)
}
```

---

## Best Practices

### 1. Respond Quickly

Return `200 OK` within **5 seconds**. Do heavy processing asynchronously:

```javascript
app.post('/webhooks/coinskro', async (req, res) => {
  // Respond immediately
  res.status(200).json({ received: true });
  
  // Process in background
  processWebhook(req.body).catch(console.error);
});

async function processWebhook(payload) {
  // Heavy processing here (emails, database, etc.)
}
```

### 2. Handle Duplicates (Idempotency)

Webhooks may be sent multiple times. Always check if you've already processed a payment:

```javascript
app.post('/webhooks/coinskro', async (req, res) => {
  const { payment_reference } = req.body.data;
  
  // Check if already processed
  const order = await db.orders.findOne({ payment_reference });
  if (order.status === 'paid') {
    return res.status(200).json({ received: true }); // Already handled
  }
  
  // Process the payment...
});
```

### 3. Verify the Payment

Before fulfilling an order, verify the payment details:

```javascript
// Verify amount matches what you expected
const expectedAmount = order.total;
if (data.amount !== expectedAmount) {
  console.error('Amount mismatch!');
  return;
}

// Verify currency matches
if (data.currency !== order.currency) {
  console.error('Currency mismatch!');
  return;
}
```

### 4. Log Everything

Keep records for debugging:

```javascript
app.post('/webhooks/coinskro', async (req, res) => {
  // Log incoming webhook
  await db.webhookLogs.create({
    payload: req.body,
    received_at: new Date()
  });
  
  // Process...
});
```

---

## Retry Policy

If your endpoint doesn't return `200 OK`, we'll retry:

| Attempt | Delay |
|:--------|:------|
| 1st retry | 1 minute |
| 2nd retry | 5 minutes |
| 3rd retry | 30 minutes |
| 4th retry | 2 hours |
| 5th retry | 24 hours |

After 5 failed attempts, the webhook is marked as failed. You can manually retry from the dashboard.

---

## Testing Webhooks Locally

Use [ngrok](https://ngrok.com) to test webhooks on your local machine:

```bash
# Start your local server
npm start  # Running on localhost:3000

# In another terminal, start ngrok
ngrok http 3000
```

Copy the ngrok URL (e.g., `https://abc123.ngrok.io`) and add it as your webhook URL in the dashboard.

---

## Troubleshooting

| Issue | Solution |
|:------|:---------|
| Not receiving webhooks | Check webhook is **activated** in dashboard |
| Getting 404 errors | Verify your webhook URL is correct |
| Getting 500 errors | Check your server logs for errors |
| Duplicate events | Implement idempotency checks |
| Delayed webhooks | Check if previous webhooks are failing (causing retries) |
