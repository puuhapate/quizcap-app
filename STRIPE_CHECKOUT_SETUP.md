# Stripe Checkout - Setup Complete ✅

## What Was Fixed

### 1. **Added Authorization Headers**
The frontend wasn't sending the `userId` in the Authorization header. Updated all API calls in `src/lib/subscriptionService.ts` to include:
```javascript
headers: {
  "Authorization": `Bearer ${userId}`,
}
```

This allows the backend to authenticate requests properly.

### 2. **Created .env File**
Created `.env` file with test Stripe keys. **IMPORTANT**: The price IDs are currently placeholders and need to be replaced with actual Stripe price IDs.

### 3. **Backend Already Configured**
The backend endpoint `POST /api/checkout/create-session` was already properly implemented in `server/routes/checkout.js` and correctly:
- Creates Stripe checkout sessions
- Attaches user ID to metadata
- Returns session URL for redirect

## What You Need to Do

### Create Actual Stripe Price IDs

The current `.env` file has placeholder price IDs. You need to create **actual price IDs** in your Stripe Dashboard:

1. **Go to Stripe Dashboard**: https://dashboard.stripe.com/test/products
2. **Create a Product** (if you haven't already):
   - Name: "QuizCap Premium"
   - Description: "Unlimited quiz generation"

3. **Create Two Prices**:

   **Monthly Price:**
   - Amount: €4.99
   - Billing period: Monthly
   - Copy the **Price ID** (starts with `price_`, NOT `prod_`)

   **Yearly Price:**
   - Amount: €29.00
   - Billing period: Yearly
   - Copy the **Price ID** (starts with `price_`, NOT `prod_`)

4. **Update .env File**:
   Replace these lines in `.env`:
   ```bash
   VITE_STRIPE_MONTHLY_PRICE_ID=price_monthly_placeholder
   VITE_STRIPE_YEARLY_PRICE_ID=price_yearly_placeholder
   ```

   With your actual price IDs:
   ```bash
   VITE_STRIPE_MONTHLY_PRICE_ID=price_1ABC123def456GHI789
   VITE_STRIPE_YEARLY_PRICE_ID=price_1XYZ987uvw654RST321
   ```

### Verify Stripe Keys

Make sure the Stripe keys in `.env` are correct for your account:
- `VITE_STRIPE_PUBLISHABLE_KEY` - Your publishable key (starts with `pk_test_`)
- `STRIPE_SECRET_KEY` - Your secret key (starts with `sk_test_`)

Get these from: https://dashboard.stripe.com/test/apikeys

## How It Works Now

1. User clicks "Upgrade to Premium" button
2. Frontend calls `redirectToCheckout(userId, planId)`
3. Function makes API call to `POST /api/checkout/create-session` with Authorization header
4. Backend creates Stripe checkout session with the price ID
5. Backend returns session URL
6. Frontend redirects user to Stripe checkout page
7. User completes payment on Stripe
8. Stripe redirects back to success/cancel URL

## Testing

Once you've added real price IDs:

1. **Start the backend server**:
   ```bash
   cd server
   npm install
   npm start
   ```

2. **Start the frontend** (in E2B, it's already running)

3. **Click "Upgrade to Premium"**:
   - Should redirect to actual Stripe checkout page
   - No alert popups
   - No mock/placeholder behavior

## Important Notes

- ✅ Backend endpoint is working
- ✅ Frontend makes proper API calls with auth
- ✅ Stripe SDK properly initialized
- ⚠️ **NEED**: Real Stripe price IDs (currently placeholders)
- ⚠️ **NEED**: Backend server must be running on port 3001

## Current Product vs Price ID Issue

The old `.env.example` had **product IDs** (`prod_UFuM...`) instead of **price IDs**. Stripe checkout sessions require **price IDs** (they start with `price_`, not `prod_`). This has been corrected in the new `.env` file, but you need to replace the placeholders with actual price IDs from your Stripe dashboard.
