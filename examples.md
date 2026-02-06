---
layout: default
title: Code Examples
nav_order: 4
permalink: /examples
---

# Code Examples
{: .no_toc }

Ready-to-use code snippets for popular languages and frameworks.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Node.js / Express

### Complete Integration

```javascript
const express = require('express');
const axios = require('axios');
const app = express();

app.use(express.json());

const COINSKRO_SECRET_KEY = process.env.COINSKRO_SECRET_KEY;
const COINSKRO_API = 'https://api.coinskro.com';

// Create a payment
app.post('/checkout', async (req, res) => {
  try {
    const { amount, email, orderId } = req.body;
    
    const response = await axios.put(
      `${COINSKRO_API}/payment/create`,
      {
        amount: amount,
        currency: 'USDT',
        payment_type: 'deposit',
        customer_email: email,
        payment_reference: `ORDER_${orderId}`,
        success_url: `https://yoursite.com/order/${orderId}/success`,
        failure_url: `https://yoursite.com/order/${orderId}/failed`,
        meta_data: JSON.stringify({ order_id: orderId })
      },
      {
        headers: {
          'Authorization': `Bearer ${COINSKRO_SECRET_KEY}`,
          'Content-Type': 'application/json'
        }
      }
    );
    
    // Save payment reference to your database
    // await db.orders.update(orderId, { payment_ref: response.data.data.PaymentData.payment_reference });
    
    // Return payment URL to frontend
    res.json({ 
      paymentUrl: response.data.data.PaymentURL 
    });
    
  } catch (error) {
    console.error('Payment creation failed:', error.response?.data);
    res.status(500).json({ error: 'Failed to create payment' });
  }
});

// Handle webhook
app.post('/webhooks/coinskro', async (req, res) => {
  res.status(200).json({ received: true });
  
  const { event, data } = req.body;
  
  if (event === 'payment_completed') {
    const orderId = data.payment_reference.replace('ORDER_', '');
    
    // Update order status
    // await db.orders.update(orderId, { status: 'paid', paid_at: new Date() });
    
    // Send confirmation email
    // await sendOrderConfirmation(data.customer_email, orderId);
    
    console.log(`Order ${orderId} paid successfully!`);
  }
});

app.listen(3000, () => console.log('Server running on port 3000'));
```

---

## Python / Django

### views.py

```python
import json
import requests
from django.http import JsonResponse
from django.views.decorators.csrf import csrf_exempt
from django.conf import settings

COINSKRO_SECRET_KEY = settings.COINSKRO_SECRET_KEY
COINSKRO_API = 'https://api.coinskro.com'

def create_payment(request):
    if request.method != 'POST':
        return JsonResponse({'error': 'Method not allowed'}, status=405)
    
    data = json.loads(request.body)
    
    payload = {
        'amount': data['amount'],
        'currency': 'USDT',
        'payment_type': 'deposit',
        'customer_email': data['email'],
        'payment_reference': f"ORDER_{data['order_id']}",
        'success_url': f"https://yoursite.com/order/{data['order_id']}/success",
        'failure_url': f"https://yoursite.com/order/{data['order_id']}/failed",
    }
    
    headers = {
        'Authorization': f'Bearer {COINSKRO_SECRET_KEY}',
        'Content-Type': 'application/json'
    }
    
    response = requests.put(
        f'{COINSKRO_API}/payment/create',
        json=payload,
        headers=headers
    )
    
    if response.status_code == 200:
        result = response.json()
        return JsonResponse({
            'paymentUrl': result['data']['PaymentURL']
        })
    else:
        return JsonResponse({'error': 'Payment creation failed'}, status=500)


@csrf_exempt
def coinskro_webhook(request):
    if request.method != 'POST':
        return JsonResponse({'error': 'Method not allowed'}, status=405)
    
    payload = json.loads(request.body)
    event = payload.get('event')
    data = payload.get('data')
    
    if event == 'payment_completed':
        payment_ref = data['payment_reference']
        order_id = payment_ref.replace('ORDER_', '')
        
        # Update order in database
        # Order.objects.filter(id=order_id).update(status='paid')
        
        print(f"Order {order_id} paid successfully!")
    
    return JsonResponse({'received': True})
```

### urls.py

```python
from django.urls import path
from . import views

urlpatterns = [
    path('checkout/', views.create_payment, name='create_payment'),
    path('webhooks/coinskro/', views.coinskro_webhook, name='coinskro_webhook'),
]
```

---

## PHP / Laravel

### PaymentController.php

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Http;

class PaymentController extends Controller
{
    private $apiUrl = 'https://api.coinskro.com';
    
    public function createPayment(Request $request)
    {
        $request->validate([
            'amount' => 'required|numeric|min:1',
            'email' => 'required|email',
            'order_id' => 'required|string',
        ]);
        
        $response = Http::withHeaders([
            'Authorization' => 'Bearer ' . env('COINSKRO_SECRET_KEY'),
            'Content-Type' => 'application/json',
        ])->put($this->apiUrl . '/payment/create', [
            'amount' => $request->amount,
            'currency' => 'USDT',
            'payment_type' => 'deposit',
            'customer_email' => $request->email,
            'payment_reference' => 'ORDER_' . $request->order_id,
            'success_url' => url('/order/' . $request->order_id . '/success'),
            'failure_url' => url('/order/' . $request->order_id . '/failed'),
        ]);
        
        if ($response->successful()) {
            $data = $response->json();
            return response()->json([
                'paymentUrl' => $data['data']['PaymentURL']
            ]);
        }
        
        return response()->json(['error' => 'Payment creation failed'], 500);
    }
    
    public function webhook(Request $request)
    {
        $payload = $request->all();
        $event = $payload['event'] ?? null;
        $data = $payload['data'] ?? [];
        
        if ($event === 'payment_completed') {
            $paymentRef = $data['payment_reference'];
            $orderId = str_replace('ORDER_', '', $paymentRef);
            
            // Update order status
            // Order::where('id', $orderId)->update(['status' => 'paid']);
            
            \Log::info("Order {$orderId} paid successfully!");
        }
        
        return response()->json(['received' => true]);
    }
}
```

### routes/web.php

```php
use App\Http\Controllers\PaymentController;

Route::post('/checkout', [PaymentController::class, 'createPayment']);
Route::post('/webhooks/coinskro', [PaymentController::class, 'webhook'])
    ->withoutMiddleware([\App\Http\Middleware\VerifyCsrfToken::class]);
```

---

## Go

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "io"
    "net/http"
    "os"
)

const coinskroAPI = "https://api.coinskro.com"

type PaymentRequest struct {
    Amount           float64 `json:"amount"`
    Currency         string  `json:"currency"`
    PaymentType      string  `json:"payment_type"`
    CustomerEmail    string  `json:"customer_email"`
    PaymentReference string  `json:"payment_reference"`
    SuccessURL       string  `json:"success_url"`
    FailureURL       string  `json:"failure_url"`
}

type PaymentResponse struct {
    Message string `json:"message"`
    Data    struct {
        PaymentData struct {
            PaymentReference string `json:"payment_reference"`
        } `json:"PaymentData"`
        PaymentURL string `json:"PaymentURL"`
    } `json:"data"`
}

type WebhookPayload struct {
    Event string `json:"event"`
    Data  struct {
        PaymentReference string  `json:"payment_reference"`
        Amount           float64 `json:"amount"`
        CustomerEmail    string  `json:"customer_email"`
    } `json:"data"`
}

func createPayment(w http.ResponseWriter, r *http.Request) {
    var req struct {
        Amount  float64 `json:"amount"`
        Email   string  `json:"email"`
        OrderID string  `json:"order_id"`
    }
    json.NewDecoder(r.Body).Decode(&req)

    payload := PaymentRequest{
        Amount:           req.Amount,
        Currency:         "USDT",
        PaymentType:      "deposit",
        CustomerEmail:    req.Email,
        PaymentReference: "ORDER_" + req.OrderID,
        SuccessURL:       fmt.Sprintf("https://yoursite.com/order/%s/success", req.OrderID),
        FailureURL:       fmt.Sprintf("https://yoursite.com/order/%s/failed", req.OrderID),
    }

    body, _ := json.Marshal(payload)
    
    client := &http.Client{}
    httpReq, _ := http.NewRequest("PUT", coinskroAPI+"/payment/create", bytes.NewBuffer(body))
    httpReq.Header.Set("Authorization", "Bearer "+os.Getenv("COINSKRO_SECRET_KEY"))
    httpReq.Header.Set("Content-Type", "application/json")

    resp, err := client.Do(httpReq)
    if err != nil {
        http.Error(w, "Payment creation failed", 500)
        return
    }
    defer resp.Body.Close()

    respBody, _ := io.ReadAll(resp.Body)
    var paymentResp PaymentResponse
    json.Unmarshal(respBody, &paymentResp)

    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(map[string]string{
        "paymentUrl": paymentResp.Data.PaymentURL,
    })
}

func webhookHandler(w http.ResponseWriter, r *http.Request) {
    var payload WebhookPayload
    json.NewDecoder(r.Body).Decode(&payload)

    if payload.Event == "payment_completed" {
        fmt.Printf("Payment %s completed for $%.2f\n",
            payload.Data.PaymentReference,
            payload.Data.Amount)
        
        // Update order in database
    }

    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(map[string]bool{"received": true})
}

func main() {
    http.HandleFunc("/checkout", createPayment)
    http.HandleFunc("/webhooks/coinskro", webhookHandler)
    
    fmt.Println("Server running on :3000")
    http.ListenAndServe(":3000", nil)
}
```

---

## Frontend Examples

### React Checkout Button

```jsx
import { useState } from 'react';

function CheckoutButton({ amount, orderId }) {
  const [loading, setLoading] = useState(false);

  const handleCheckout = async () => {
    setLoading(true);
    
    try {
      const response = await fetch('/api/checkout', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          amount,
          email: 'customer@example.com',
          order_id: orderId
        })
      });
      
      const { paymentUrl } = await response.json();
      
      // Redirect to Coinskro payment page
      window.location.href = paymentUrl;
      
    } catch (error) {
      alert('Failed to create payment');
    } finally {
      setLoading(false);
    }
  };

  return (
    <button onClick={handleCheckout} disabled={loading}>
      {loading ? 'Processing...' : `Pay $${amount} with Crypto`}
    </button>
  );
}

export default CheckoutButton;
```

### Vanilla JavaScript

```html
<button id="pay-button">Pay $100 with Crypto</button>

<script>
document.getElementById('pay-button').addEventListener('click', async () => {
  const button = document.getElementById('pay-button');
  button.disabled = true;
  button.textContent = 'Processing...';
  
  try {
    const response = await fetch('/api/checkout', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        amount: 100,
        email: 'customer@example.com',
        order_id: 'ORD123'
      })
    });
    
    const { paymentUrl } = await response.json();
    window.location.href = paymentUrl;
    
  } catch (error) {
    alert('Payment failed. Please try again.');
    button.disabled = false;
    button.textContent = 'Pay $100 with Crypto';
  }
});
</script>
```
