# PayPal Token Purchase Integration

This guide explains how to set up PayPal payment processing for token purchases.

## Overview

Users can now purchase tokens via PayPal with the following features:
- **Token Packages**: Crush (500), Love (1000), Heart (2000), Passion (5000), Devotion (10000)
- **Automatic Token Award**: Tokens are instantly credited after payment
- **Payment Tracking**: All transactions are recorded in TokenTransaction history
- **Secure Processing**: Server-side order creation and capture

## Backend Setup

### 1. Get PayPal Credentials

1. Go to [PayPal Developer Dashboard](https://developer.paypal.com)
2. Sign in with your PayPal account (create one if needed)
3. Create or select a Sandbox app
4. Copy the **Sandbox Client ID** and **Sandbox Client Secret**

### 2. Configure Environment Variables

Add the following to `backend/.env`:

```env
PAYPAL_CLIENT_ID=your_sandbox_client_id
PAYPAL_CLIENT_SECRET=your_sandbox_client_secret
```

### 3. Testing

The integration is set up in Sandbox mode by default. To test:

```bash
# The backend is already running
# PayPal endpoints are available at:
# POST /api/payment/paypal/create-order
# POST /api/payment/paypal/capture-order
```

## Mobile UI

### Buy Tokens Screen

Users can access the token purchase screen at:
```
/buy-tokens
```

The screen shows:
- Available token packages with prices
- PayPal payment button
- Transaction success/error messages

### Flow

1. User selects a token package
2. Clicks "Pay with PayPal"
3. PayPal order is created on the backend
4. User approves payment (via PayPal)
5. Backend captures the order
6. Tokens are awarded and transaction is recorded

## API Endpoints

### Create PayPal Order

```
POST /api/payment/paypal/create-order

Request:
{
  "userId": "user_id",
  "productId": "love",  // crush, love, heart, passion, devotion
  "purchaseType": "token_pack",
  "amount": "1.03",  // USD
  "currency": "USD"
}

Response:
{
  "success": true,
  "data": {
    "paymentId": "payment_id",
    "orderId": "paypal_order_id",
    "approvalUrl": "https://sandbox.paypal.com/..."
  }
}
```

### Capture PayPal Order

```
POST /api/payment/paypal/capture-order

Request:
{
  "paymentId": "payment_id",
  "orderId": "paypal_order_id"
}

Response:
{
  "success": true,
  "data": {
    "paymentId": "payment_id",
    "status": "completed",
    "tokensAwarded": 1000,
    "newBalance": 1500
  }
}
```

## Database Records

When a payment is completed:

1. **Payment Record** - Created with:
   - Status: `completed`
   - PayPal Order ID: Stored in `paypalOrderId`
   - Amount and currency converted to USD

2. **TokenTransaction Record** - Created with:
   - Type: `purchase`
   - Amount: Number of tokens awarded
   - Description: Product name and PayPal reference
   - Balance before/after

3. **Subscription Updated** - User's token balance increased

## Security Considerations

- **Server-Side Processing**: All PayPal operations happen on the backend
- **Credentials**: PayPal credentials stored in environment variables only
- **Webhook Validation**: Ready for webhook signature verification (TODO)
- **Amount Validation**: Amounts verified before creating orders
- **User Verification**: User ID validated before awarding tokens

## Next Steps (Optional Enhancements)

1. **Webhook Handling**: Implement PayPal webhook signature verification
2. **Production Mode**: Switch to Production environment credentials
3. **Refund Processing**: Handle payment refunds and token reversals
4. **Payment History**: Build UI to view payment history
5. **Subscription Purchases**: Extend to support subscription tier purchases
6. **Multiple Currencies**: Add support for non-USD currencies

## Troubleshooting

### "PayPal credentials not configured"

Make sure `PAYPAL_CLIENT_ID` and `PAYPAL_CLIENT_SECRET` are set in `backend/.env`

### "Failed to create PayPal order"

- Check PayPal credentials are correct
- Verify amount and currency are valid
- Check backend logs for PayPal API errors

### Tokens not awarded

- Verify payment status is `completed`
- Check TokenTransaction records in database
- Review backend logs for token award failures

## Test Credentials

Use these PayPal Sandbox test accounts for testing:

**Buyer Account:**
- Email: `buyer@sandbox.paypal.com`
- Password: `123456789`

**Seller Account:**
- Email: `merchant@sandbox.paypal.com`
- Password: `123456789`

Note: Actual credentials will be provided by PayPal when you create your account.
