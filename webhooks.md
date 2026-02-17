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

1. Go to your [Coinskro Dashboard](https://coinskro.com)
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

When an event occurs, we POST a flat JSON body to your URL:

```json
{
  "event_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "payment_id": "123e4567-e89b-12d3-a456-426614174000",
  "currency": "PI",
  "amount": 100.00,
  "payment_reference": "PAY_abc123xyz",
  "payment_status": "completed",
  "service_fee": 0.50,
  "customer_email": "customer@example.com",
  "customer_reference": "CUST_12345",
  "integration_id": "789e0123-e89b-12d3-a456-426614174000",
  "event_type": "payment_completed",
  "timestamp": 1738838100
}
```

| Field | Type | Description |
|:------|:-----|:------------|
| `event_id` | string (UUID) | Unique ID for this event — use as an **idempotency key** |
| `payment_id` | string (UUID) | The payment's unique ID |
| `currency` | string | Currency code (e.g., `PI`, `USDT`) |
| `amount` | number | Payment amount |
| `payment_reference` | string | Your payment reference |
| `payment_status` | string | `completed`, `abandoned`, `canceled`, etc. |
| `service_fee` | number | Transaction fee charged |
| `customer_email` | string | Customer's email (if provided) |
| `customer_reference` | string | Your customer reference (if provided) |
| `integration_id` | string (UUID) | Your integration's ID |
| `event_type` | string | Event code (e.g., `payment_completed`) |
| `timestamp` | number | Unix timestamp of the event |

### Webhook Headers

Each webhook request includes these headers:

| Header | Description |
|:-------|:------------|
| `Content-Type` | `application/json` |
| `X-Signature` | HMAC-SHA256 signature of the payload (base64-encoded) |
| `X-Event-Id` | The event's unique ID (same as `event_id` in the body) |

---

## Verifying Webhook Signatures

{: .important }
> **Always verify the `X-Signature` header** before processing a webhook. This ensures the request actually came from Coinskro and hasn't been tampered with.

We sign every webhook payload using **HMAC-SHA256** with your **Secret Key** and encode the result in **base64**. The signature is sent in the `X-Signature` header.

**How verification works:**

1. Read the raw request body (don't parse it first)
2. Compute HMAC-SHA256 of the raw body using your Secret Key
3. Base64-encode the result
4. Compare with the `X-Signature` header value

{: .warning }
> Use a **constant-time comparison** function to prevent timing attacks. Most languages provide one (e.g., `crypto.timingSafeEqual` in Node.js, `hmac.compare_digest` in Python).

### Node.js

```javascript
const crypto = require('crypto');

function verifySignature(secretKey, rawBody, signature) {
  const expectedSignature = crypto
    .createHmac('sha256', secretKey)
    .update(rawBody)
    .digest('base64');

  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(expectedSignature)
  );
}
```

### Python

```python
import hmac
import hashlib
import base64

def verify_signature(secret_key: str, raw_body: bytes, signature: str) -> bool:
    expected = base64.b64encode(
        hmac.new(secret_key.encode(), raw_body, hashlib.sha256).digest()
    ).decode()
    return hmac.compare_digest(signature, expected)
```

### PHP

```php
function verifySignature(string $secretKey, string $rawBody, string $signature): bool {
    $expected = base64_encode(hash_hmac('sha256', $rawBody, $secretKey, true));
    return hash_equals($expected, $signature);
}
```

### Go

```go
import (
    "crypto/hmac"
    "crypto/sha256"
    "crypto/subtle"
    "encoding/base64"
)

func verifySignature(secretKey string, rawBody []byte, signature string) bool {
    mac := hmac.New(sha256.New, []byte(secretKey))
    mac.Write(rawBody)
    expected := base64.StdEncoding.EncodeToString(mac.Sum(nil))
    return subtle.ConstantTimeCompare([]byte(expected), []byte(signature)) == 1
}
```

---

## Handling Webhooks

### Node.js / Express

```javascript
const express = require('express');
const crypto = require('crypto');
const app = express();

const SECRET_KEY = process.env.COINSKRO_SECRET_KEY;

// Use raw body for signature verification
app.post('/webhooks/coinskro', express.raw({ type: 'application/json' }), (req, res) => {
  const signature = req.headers['x-signature'];
  const rawBody = req.body;

  // 1. Verify the signature
  const expectedSignature = crypto
    .createHmac('sha256', SECRET_KEY)
    .update(rawBody)
    .digest('base64');

  if (!crypto.timingSafeEqual(Buffer.from(signature), Buffer.from(expectedSignature))) {
    console.error('Invalid webhook signature');
    return res.status(401).json({ error: 'Invalid signature' });
  }

  // 2. Parse the verified payload
  const payload = JSON.parse(rawBody);
  const { event_type, event_id, payment_reference, amount } = payload;

  if (event_type === 'payment_completed') {
    // 3. Check if not already processed (use event_id for idempotency)
    // 4. Fulfill the order

    console.log(`Payment ${payment_reference} completed for $${amount}`);
  }

  // Always respond with 200 OK
  res.status(200).json({ received: true });
});

app.listen(3000);
```

### Python / Flask

```python
import hmac
import hashlib
import base64
import json
from flask import Flask, request, jsonify

app = Flask(__name__)

SECRET_KEY = os.environ.get('COINSKRO_SECRET_KEY')

def verify_signature(secret_key, raw_body, signature):
    expected = base64.b64encode(
        hmac.new(secret_key.encode(), raw_body, hashlib.sha256).digest()
    ).decode()
    return hmac.compare_digest(signature, expected)

@app.route('/webhooks/coinskro', methods=['POST'])
def coinskro_webhook():
    signature = request.headers.get('X-Signature', '')
    raw_body = request.get_data()

    # 1. Verify the signature
    if not verify_signature(SECRET_KEY, raw_body, signature):
        return jsonify({'error': 'Invalid signature'}), 401

    # 2. Parse the verified payload
    payload = json.loads(raw_body)
    event_type = payload.get('event_type')
    payment_reference = payload.get('payment_reference')
    amount = payload.get('amount')

    if event_type == 'payment_completed':
        # 3. Check if not already processed (use event_id for idempotency)
        # 4. Fulfill the order
        print(f"Payment {payment_reference} completed for ${amount}")

    return jsonify({'received': True}), 200

if __name__ == '__main__':
    app.run(port=3000)
```

### PHP

```php
<?php
$secretKey = getenv('COINSKRO_SECRET_KEY');
$rawBody = file_get_contents('php://input');
$signature = $_SERVER['HTTP_X_SIGNATURE'] ?? '';

// 1. Verify the signature
$expectedSignature = base64_encode(hash_hmac('sha256', $rawBody, $secretKey, true));

if (!hash_equals($expectedSignature, $signature)) {
    http_response_code(401);
    echo json_encode(['error' => 'Invalid signature']);
    exit;
}

// 2. Parse the verified payload
$payload = json_decode($rawBody, true);
$eventType = $payload['event_type'];
$paymentReference = $payload['payment_reference'];
$amount = $payload['amount'];

if ($eventType === 'payment_completed') {
    // 3. Check if not already processed (use event_id for idempotency)
    // 4. Fulfill the order
    error_log("Payment {$paymentReference} completed for \${$amount}");
}

// Always respond with 200 OK
http_response_code(200);
echo json_encode(['received' => true]);
```

### Go

```go
package main

import (
    "crypto/hmac"
    "crypto/sha256"
    "crypto/subtle"
    "encoding/base64"
    "encoding/json"
    "fmt"
    "io"
    "net/http"
    "os"
)

var secretKey = os.Getenv("COINSKRO_SECRET_KEY")

type WebhookPayload struct {
    EventID          string  `json:"event_id"`
    PaymentID        string  `json:"payment_id"`
    Currency         string  `json:"currency"`
    Amount           float64 `json:"amount"`
    PaymentReference string  `json:"payment_reference"`
    PaymentStatus    string  `json:"payment_status"`
    ServiceFee       float64 `json:"service_fee"`
    CustomerEmail    string  `json:"customer_email"`
    EventType        string  `json:"event_type"`
    Timestamp        int64   `json:"timestamp"`
}

func verifySignature(secret string, body []byte, signature string) bool {
    mac := hmac.New(sha256.New, []byte(secret))
    mac.Write(body)
    expected := base64.StdEncoding.EncodeToString(mac.Sum(nil))
    return subtle.ConstantTimeCompare([]byte(expected), []byte(signature)) == 1
}

func webhookHandler(w http.ResponseWriter, r *http.Request) {
    // 1. Read raw body
    rawBody, err := io.ReadAll(r.Body)
    if err != nil {
        http.Error(w, "Failed to read body", http.StatusBadRequest)
        return
    }
    defer r.Body.Close()

    // 2. Verify the signature
    signature := r.Header.Get("X-Signature")
    if !verifySignature(secretKey, rawBody, signature) {
        http.Error(w, "Invalid signature", http.StatusUnauthorized)
        return
    }

    // 3. Parse the verified payload
    var payload WebhookPayload
    json.Unmarshal(rawBody, &payload)

    if payload.EventType == "payment_completed" {
        fmt.Printf("Payment %s completed for $%.2f\n",
            payload.PaymentReference, payload.Amount)

        // 4. Check if not already processed (use EventID for idempotency)
        // 5. Fulfill the order
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

Webhooks may be sent multiple times. Use the `event_id` field as an idempotency key:

```javascript
app.post('/webhooks/coinskro', express.raw({ type: 'application/json' }), (req, res) => {
  // ... verify signature first ...
  const payload = JSON.parse(req.body);
  const { event_id, payment_reference } = payload;
  
  // Check if already processed using event_id
  const existing = await db.webhookEvents.findOne({ event_id });
  if (existing) {
    return res.status(200).json({ received: true }); // Already handled
  }
  
  // Record this event before processing
  await db.webhookEvents.create({ event_id, processed_at: new Date() });
  
  // Process the payment...
});
```

### 3. Verify the Payment

Before fulfilling an order, verify the payment details:

```javascript
// Verify amount matches what you expected
const expectedAmount = order.total;
if (payload.amount !== expectedAmount) {
  console.error('Amount mismatch!');
  return;
}

// Verify currency matches
if (payload.currency !== order.currency) {
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

If your endpoint doesn't return a `2xx` status code, we'll retry up to **10 times**:

| Attempt | Delay |
|:--------|:------|
| 1st retry | 1 minute |
| 2nd retry | 5 minutes |
| 3rd retry | 30 minutes |
| 4th retry | 2 hours |
| 5th retry | 24 hours |
| 6th–10th | Periodic retries |

After 10 failed attempts, the webhook is marked as failed. You can manually retry from the dashboard.

{: .note }
> If your endpoint returns `410 Gone`, the event is immediately marked as failed (no further retries).

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
