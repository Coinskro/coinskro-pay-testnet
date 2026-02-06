---
layout: default
title: Error Handling
nav_order: 6
permalink: /errors
---

# Error Handling
{: .no_toc }

Handle errors gracefully in your integration.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Error Response Format

All API errors return this format:

```json
{
  "message": "Error description",
  "data": null,
  "code": 1001
}
```

---

## HTTP Status Codes

| Status | Meaning | What to Do |
|:-------|:--------|:-----------|
| `200` | Success | Process the response |
| `400` | Bad Request | Check your request parameters |
| `401` | Unauthorized | Check your API key |
| `403` | Forbidden | Check integration status or permissions |
| `404` | Not Found | Check the resource ID/reference |
| `500` | Server Error | Retry or contact support |

---

## Common Errors

### Authentication Errors

**401 - Missing or Invalid API Key**

```json
{
  "message": "Unauthorized: Missing secret key"
}
```

**Fix:** Add the Authorization header:

```bash
-H "Authorization: Bearer sk_live_xxxxxxxxxxxx"
```

---

### Validation Errors

**400 - Currency not supported**

```json
{
  "message": "Currency not supported"
}
```

**Fix:** Use a supported currency: `USDT`, `BTC`, `ETH`, `USDC`

---

**400 - Payment type is required**

```json
{
  "message": "Payment type is required"
}
```

**Fix:** Include `payment_type` in your request:

```json
{
  "payment_type": "deposit"
}
```

---

**400 - Payment type not supported**

```json
{
  "message": "Payment type not supported"
}
```

**Fix:** Use a valid payment type: `deposit`, `withdrawal`, `debit`, `credit`

---

### Permission Errors

**403 - Integration not approved**

```json
{
  "message": "Integration is not yet approved"
}
```

**Fix:** Wait for your integration to be approved, or use test mode with `sk_test_` keys.

---

**403 - Insufficient balance**

```json
{
  "message": "Insufficient balance"
}
```

**Fix:** For withdrawals, ensure you have enough balance in your wallet.

---

### Not Found Errors

**404 - Payment not found**

```json
{
  "message": "Payment information not found"
}
```

**Fix:** Verify the payment reference is correct.

---

## Handling Errors in Code

### Node.js

```javascript
try {
  const response = await axios.put(
    'https://api.coinskro.com/payment/create',
    payload,
    { headers }
  );
  
  return response.data;
  
} catch (error) {
  if (error.response) {
    // API returned an error
    const { status, data } = error.response;
    
    switch (status) {
      case 400:
        console.error('Invalid request:', data.message);
        throw new Error('Please check your payment details');
        
      case 401:
        console.error('Authentication failed');
        throw new Error('Payment service unavailable');
        
      case 403:
        console.error('Permission denied:', data.message);
        throw new Error('Payment cannot be processed');
        
      case 500:
        console.error('Server error');
        throw new Error('Please try again later');
        
      default:
        throw new Error('Payment failed');
    }
  } else {
    // Network error
    console.error('Network error:', error.message);
    throw new Error('Please check your connection');
  }
}
```

### Python

```python
import requests

try:
    response = requests.put(
        'https://api.coinskro.com/payment/create',
        json=payload,
        headers=headers
    )
    response.raise_for_status()
    return response.json()
    
except requests.exceptions.HTTPError as e:
    status = e.response.status_code
    data = e.response.json()
    
    if status == 400:
        raise ValueError(f"Invalid request: {data['message']}")
    elif status == 401:
        raise PermissionError("Authentication failed")
    elif status == 403:
        raise PermissionError(f"Permission denied: {data['message']}")
    elif status == 500:
        raise Exception("Server error, please try again")
    else:
        raise Exception("Payment failed")
        
except requests.exceptions.ConnectionError:
    raise Exception("Network error, please check your connection")
```

### PHP

```php
try {
    $response = Http::withHeaders($headers)->put($url, $payload);
    
    if ($response->successful()) {
        return $response->json();
    }
    
    $status = $response->status();
    $data = $response->json();
    
    switch ($status) {
        case 400:
            throw new \InvalidArgumentException("Invalid request: " . $data['message']);
        case 401:
            throw new \Exception("Authentication failed");
        case 403:
            throw new \Exception("Permission denied: " . $data['message']);
        case 500:
            throw new \Exception("Server error, please try again");
        default:
            throw new \Exception("Payment failed");
    }
    
} catch (\Illuminate\Http\Client\ConnectionException $e) {
    throw new \Exception("Network error, please check your connection");
}
```

---

## User-Friendly Error Messages

Don't show raw API errors to users. Map them to friendly messages:

| API Error | User Message |
|:----------|:-------------|
| Currency not supported | "This currency is not available" |
| Unauthorized | "Payment service is temporarily unavailable" |
| Integration not approved | "Payments are not yet enabled" |
| Insufficient balance | "Unable to process withdrawal" |
| Server error | "Something went wrong. Please try again." |

---

## Retry Strategy

For `500` errors, implement retries with exponential backoff:

```javascript
async function createPaymentWithRetry(payload, maxRetries = 3) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await createPayment(payload);
    } catch (error) {
      if (error.response?.status === 500 && attempt < maxRetries) {
        // Wait before retrying: 1s, 2s, 4s
        const delay = Math.pow(2, attempt - 1) * 1000;
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }
      throw error;
    }
  }
}
```

---

## Logging Best Practices

Log errors for debugging, but sanitize sensitive data:

```javascript
function logError(error, context) {
  console.error({
    timestamp: new Date().toISOString(),
    error: error.message,
    status: error.response?.status,
    // Don't log full request/response - may contain sensitive data
    context: {
      payment_reference: context.payment_reference,
      amount: context.amount,
      // Don't log: api_key, customer_email
    }
  });
}
```
