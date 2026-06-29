# Mongike Payment Integration

## Overview
Mongike is a payment aggregator that handles all local Tanzanian mobile money payments through a single, unified API. This integration enables your app to accept payments via:

- **M-Pesa** (Vodacom)
- **Airtel Money** (Airtel)
- **Mixx by Yas** (Tigo)
- **Halo Pesa** (Halotel)

## Setup

### 1. Get Mongike Credentials
1. Sign up at [https://mongike.com](https://mongike.com)
2. Navigate to Dashboard → API Keys
3. Get your:
   - `API Key`
   - `Secret Key`
   - `Merchant ID`

### 2. Configure Environment Variables

Add these to your `.env` file in the `backend/` directory:

```bash
# MONGIKE - Payment Aggregator (Local TZ Mobile Money)
MONGIKE_API_KEY=your_mongike_api_key
MONGIKE_SECRET_KEY=your_mongike_secret_key
MONGIKE_MERCHANT_ID=your_mongike_merchant_id
```

## API Endpoints

### 1. Process Payment via Mongike

**POST** `/api/payment/mongike/process`

Initiates a push payment (customer sends money to merchant).

```json
{
  "paymentId": "payment_123",
  "phoneNumber": "+255XXXXXXXXX",
  "paymentMethod": "MPESA",
  "callbackUrl": "https://your-backend.com/api/payment/mongike/callback"
}
```

**Supported Payment Methods:**
- `MPESA` - M-Pesa (Vodacom)
- `AIRTEL` - Airtel Money
- `MIXX` - Mixx by Yas
- `HALOTEL` - Halo Pesa

**Response:**
```json
{
  "success": true,
  "data": {
    "paymentId": "payment_123",
    "transactionId": "mongike_txn_123",
    "status": "completed",
    "tokens": 10000,
    "message": "Payment processed successfully"
  }
}
```

### 2. Check Transaction Status

**GET** `/api/payment/mongike/status/:transactionId`

Check the status of a Mongike transaction.

**Response:**
```json
{
  "success": true,
  "data": {
    "transactionId": "mongike_txn_123",
    "status": "SUCCESS",
    "message": "Transaction successful"
  }
}
```

**Possible Statuses:**
- `PENDING` - Payment is being processed
- `SUCCESS` - Payment completed successfully
- `FAILED` - Payment failed
- `EXPIRED` - Payment request expired

### 3. Refund Payment

**POST** `/api/payment/mongike/refund/:paymentId`

Refund/reverse a Mongike payment.

**Response:**
```json
{
  "success": true,
  "data": {
    "paymentId": "payment_123",
    "status": "refunded",
    "message": "Payment refunded successfully"
  }
}
```

### 4. Webhook Callback

**POST** `/api/payment/mongike/callback`

Mongike sends real-time payment updates to this endpoint. Configure this URL in your Mongike dashboard:

```
https://your-backend.com/api/payment/mongike/callback
```

**Payload from Mongike:**
```json
{
  "transactionId": "mongike_txn_123",
  "status": "SUCCESS",
  "paymentId": "payment_123",
  "amount": 26360,
  "phone": "+255XXXXXXXXX",
  "paymentMethod": "MPESA",
  "timestamp": 1234567890
}
```

## Implementation Files

### Backend
- **`src/lib/mongike.ts`** - Mongike API client
  - `MongieClient` class with all Mongike API methods
  - Phone number validation and carrier detection
  - Signature generation for request authentication
  - Transaction polling and refund handling

- **`src/services/payment.service.ts`** - Payment service integration
  - `processMongiePayment()` - Process payment via Mongike
  - `checkMongieTransactionStatus()` - Check transaction status
  - `refundMongieTransaction()` - Refund a payment

- **`src/routes/payment.ts`** - Payment API routes
  - `POST /api/payment/mongike/process` - Process payment
  - `GET /api/payment/mongike/status/:transactionId` - Check status
  - `POST /api/payment/mongike/refund/:paymentId` - Refund payment
  - `POST /api/payment/mongike/callback` - Webhook endpoint

## Phone Number Formats

The system accepts Tanzanian phone numbers in multiple formats:

✅ Valid formats:
- `255XXXXXXXXX` (international without +)
- `+255XXXXXXXXX` (international with +)
- `0XXXXXXXXX` (local format)
- `XXXXXXXXX` (9 digits)

❌ Invalid:
- Country codes other than 255
- Invalid mobile network prefixes

## Carrier Detection

The system automatically detects which carrier a phone number belongs to:

| Prefix | Carrier | Networks |
|--------|---------|----------|
| 255-6x | Vodacom | M-Pesa, Mixx |
| 255-7x | Airtel | Airtel Money, Mixx |
| 255-8x | Tigo | Mixx |
| 255-9x | Halotel | Halo Pesa, Mixx |

## Security

- All requests to Mongike are signed with HMAC-SHA256
- Signatures use your `SECRET_KEY` for authentication
- Never expose API keys or secret keys in client-side code
- Always use HTTPS for payment operations

## Testing

### 1. Get Mongike Test Credentials
- Mongike provides test credentials for sandbox testing
- Use these for development and testing

### 2. Test Phone Numbers
Mongike provides test phone numbers for each payment method. Check your Mongike documentation for:
- M-Pesa test numbers
- Airtel Money test numbers
- Mixx test numbers
- Halo Pesa test numbers

### 3. Test Payment Flow
```bash
curl -X POST http://localhost:3000/api/payment/mongike/process \
  -H "Content-Type: application/json" \
  -d '{
    "paymentId": "test_payment_123",
    "phoneNumber": "+255XXXXXXXXX",
    "paymentMethod": "MPESA",
    "callbackUrl": "http://localhost:3000/api/payment/mongike/callback"
  }'
```

## Product Pricing

The app includes pre-configured subscription plans and token packages:

### Subscription Plans (Monthly)
| Tier | Tokens | Price TZS | Price USD |
|------|--------|-----------|-----------|
| Bronze | 10,000 | 26,360 | $10 |
| Silver | 20,000 | 52,720 | $20 |
| Gold | 50,000 | 131,800 | $50 |
| Platinum | 100,000 | 263,600 | $100 |
| Diamond | 200,000 | 527,200 | $200 |

### Token Packages (One-time)
| Pack | Tokens | Price TZS | Price USD |
|------|--------|-----------|-----------|
| Crush | 500 | 1,318 | $0.52 |
| Love | 1,000 | 2,636 | $1.03 |
| Heart | 2,000 | 5,272 | $2.07 |
| Passion | 5,000 | 13,180 | $5.17 |
| Devotion | 10,000 | 26,360 | $10.33 |

## Production Deployment

### Before Going Live:

1. **Get Production Credentials**
   - Contact Mongike sales
   - Get production API keys and merchant ID

2. **Update Environment Variables**
   ```bash
   MONGIKE_API_KEY=prod_api_key
   MONGIKE_SECRET_KEY=prod_secret_key
   MONGIKE_MERCHANT_ID=prod_merchant_id
   ```

3. **Configure Webhook URL**
   - Update Mongike dashboard with your production callback URL
   - Example: `https://yourdomain.com/api/payment/mongike/callback`

4. **Test with Real Transactions**
   - Process small test payments with real accounts
   - Verify webhook callbacks are received
   - Test refund functionality

5. **Set Correct Pricing**
   - Verify all pricing in `SUBSCRIPTION_PLANS` and `TOKEN_PACKAGES`
   - Update exchange rates if needed

## Troubleshooting

### Payment Fails: "Invalid phone number"
- Ensure phone is in format: `+255XXXXXXXXX`
- Check it starts with valid Tanzania prefix (6, 7, 8, or 9)

### Payment Fails: "Mongike is not configured"
- Verify `MONGIKE_API_KEY`, `MONGIKE_SECRET_KEY`, `MONGIKE_MERCHANT_ID` are set in `.env`
- Restart the backend server

### Transaction Status Shows "PENDING"
- Wait a few seconds and check again
- Mongike processes payments asynchronously
- Check webhook logs for actual status

### Webhook Not Receiving Callbacks
- Verify callback URL is publicly accessible
- Check firewall/network settings
- Verify URL matches what's configured in Mongike dashboard
- Check server logs for webhook processing errors

## Support

For Mongike-specific issues:
- Visit: https://mongike.com
- Contact support via dashboard
- Check API documentation at: https://api.mongike.com/docs

For app-specific issues:
- Check backend logs: `backend/server.log`
- Review error messages in payment records
- Check database for payment status
