---
layout: default
title: Testing
nav_order: 5
permalink: /testing
---

# Testing Your Integration
{: .no_toc }

Test your integration before going live.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Test Mode vs Live Mode

Coinskro Pay provides two environments:

| Mode | API Key Prefix | Real Money? | Use For |
|:-----|:---------------|:------------|:--------|
| **Test** | `sk_test_` | No | Development & testing |
| **Live** | `sk_live_` | Yes | Production |

{: .important }
> Always use test keys during development. Test payments don't involve real cryptocurrency.

---

## Getting Test Keys

1. Log in to your [Dashboard](https://coinskro.com)
2. Select your integration
3. Toggle to **Test Mode** (top-right switch)
4. Copy your **Test Secret Key**

---

## Making Test Payments

### Create a Test Payment

Use your test secret key:

```bash
curl -X PUT https://api.coinskro.com/payment/create \
  -H "Authorization: Bearer sk_test_xxxxxxxxxxxx" \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 50.00,
    "currency": "USDT",
    "payment_type": "deposit",
    "customer_email": "test@example.com"
  }'
```

### Complete the Test Payment

When redirected to the payment page in test mode, you'll see a **"Simulate Payment"** button. Click it to simulate:

- ✅ **Successful payment** - Triggers `payment_completed` webhook
- ❌ **Failed payment** - Triggers `payment_failed` webhook
- ⏰ **Abandoned payment** - Close the page (triggers `payment_abandoned` after timeout)

---

## Testing Webhooks

### Local Development

Use [ngrok](https://ngrok.com) to receive webhooks on your local machine:

```bash
# Terminal 1: Start your server
npm start  # or python app.py, go run main.go, etc.

# Terminal 2: Start ngrok
ngrok http 3000
```

Copy the ngrok URL and set it as your webhook URL in the dashboard:

```
https://abc123.ngrok.io/webhooks/coinskro
```

### Webhook Testing Checklist

Test these scenarios:

- [ ] Payment completed - order marked as paid
- [ ] Payment failed - user shown error message
- [ ] Payment abandoned - order remains pending
- [ ] Duplicate webhook - order not processed twice
- [ ] Invalid webhook - request rejected gracefully

### Manual Webhook Testing

Test your webhook endpoint with curl:

```bash
curl -X POST https://yoursite.com/webhooks/coinskro \
  -H "Content-Type: application/json" \
  -d '{
    "event": "payment_completed",
    "timestamp": "2026-02-06T10:35:00Z",
    "data": {
      "payment_reference": "ORDER_TEST_001",
      "integration_id": "test-integration-id",
      "amount": 50.00,
      "currency": "USDT",
      "status": "completed",
      "customer_email": "test@example.com"
    }
  }'
```

---

## Test Scenarios

### Scenario 1: Successful Payment

1. Create a payment
2. Redirect to payment URL
3. Click "Simulate Successful Payment"
4. Verify:
   - Webhook received with `payment_completed`
   - Order marked as paid in your database
   - Customer redirected to `success_url`

### Scenario 2: Failed Payment

1. Create a payment
2. Redirect to payment URL
3. Click "Simulate Failed Payment"
4. Verify:
   - Customer redirected to `failure_url`
   - Order remains unpaid

### Scenario 3: Abandoned Payment

1. Create a payment
2. Redirect to payment URL
3. Close the browser tab
4. Wait 10 minutes (test mode timeout)
5. Verify:
   - Webhook received with `payment_abandoned`
   - Order remains pending

### Scenario 4: Duplicate Webhooks

1. Complete a test payment
2. Manually retry the webhook from dashboard
3. Verify:
   - Second webhook processed without errors
   - Order not fulfilled twice
   - No duplicate emails sent

---

## Integration Checklist

Before going live, verify:

### Payment Flow
- [ ] Payments created successfully
- [ ] Customers redirected to payment page
- [ ] Success URL receives customer after payment
- [ ] Failure URL receives customer on failure

### Webhooks
- [ ] Webhook endpoint accessible from internet
- [ ] Webhook returns 200 OK quickly
- [ ] `payment_completed` webhook fulfills orders
- [ ] Duplicate webhooks handled (idempotency)
- [ ] Webhook errors logged for debugging

### Error Handling
- [ ] Invalid API key shows proper error
- [ ] Invalid amount shows proper error
- [ ] Network errors handled gracefully
- [ ] User sees friendly error messages

### Security
- [ ] Secret key stored securely (environment variable)
- [ ] Secret key never exposed in frontend code
- [ ] HTTPS used for all API calls
- [ ] Webhook endpoint validates requests

---

## Going Live

Once testing is complete:

1. Switch to **Live Mode** in dashboard
2. Copy your **Live Secret Key**
3. Update your environment variable:

```bash
# Development
COINSKRO_SECRET_KEY=sk_test_xxxxxxxxxxxx

# Production
COINSKRO_SECRET_KEY=sk_live_xxxxxxxxxxxx
```

4. Deploy your application
5. Make a small real payment to verify

{: .warning }
> Double-check you're using the live key in production. Test payments won't work with live keys.

---

## Debugging Tips

### Payment Not Created

```
401 Unauthorized: Missing secret key
```
→ Check Authorization header format: `Bearer sk_xxx`

```
403 Integration is not yet approved
```
→ Wait for your integration to be approved, or use test mode

### Webhook Not Received

1. Check webhook is **activated** in dashboard
2. Verify URL is publicly accessible
3. Check server logs for incoming requests
4. Try the webhook test tool in dashboard

### Payment Status Not Updating

1. Check webhook logs in dashboard
2. Verify webhook returns 200 OK
3. Check your server logs for errors
4. Ensure database updates are working
