# Stripe Backend API Documentation

This document describes the backend API endpoints required for the Stripe subscription integration in QuizCap.

## Overview

The frontend Stripe integration is complete and includes:
- ✅ Stripe.js and React Stripe.js installed
- ✅ Subscription service with checkout and portal redirects
- ✅ Upgrade to Premium component
- ✅ Manage Subscription component
- ✅ Premium access checking in quiz generation
- ✅ Daily usage limits (3 for free, unlimited for premium)

## Required Backend Endpoints

You need to implement the following backend API endpoints to complete the Stripe integration:

### 1. Create Checkout Session

**Endpoint:** `POST /api/checkout/create-session`

**Description:** Creates a Stripe Checkout session for subscribing to a plan.

**Request Body:**
```json
{
  "userId": "string",
  "planId": "monthly" | "yearly",
  "priceId": "price_xxx",
  "successUrl": "https://yourapp.com/subscription/success",
  "cancelUrl": "https://yourapp.com/subscription/cancel"
}
```

**Response:**
```json
{
  "sessionId": "cs_test_xxx",
  "url": "https://checkout.stripe.com/c/pay/cs_test_xxx"
}
```

**Backend Implementation Example (Node.js):**
```javascript
const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY);

app.post('/api/checkout/create-session', async (req, res) => {
  const { userId, priceId, successUrl, cancelUrl } = req.body;

  try {
    const session = await stripe.checkout.sessions.create({
      mode: 'subscription',
      payment_method_types: ['card'],
      line_items: [{
        price: priceId,
        quantity: 1,
      }],
      success_url: successUrl + '?session_id={CHECKOUT_SESSION_ID}',
      cancel_url: cancelUrl,
      client_reference_id: userId,
      metadata: { userId },
    });

    res.json({
      sessionId: session.id,
      url: session.url
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

### 2. Create Customer Portal Session

**Endpoint:** `POST /api/billing/create-portal-session`

**Description:** Creates a Stripe Customer Portal session for managing subscription.

**Request Body:**
```json
{
  "userId": "string",
  "returnUrl": "https://yourapp.com"
}
```

**Response:**
```json
{
  "url": "https://billing.stripe.com/p/session/test_xxx"
}
```

**Backend Implementation Example (Node.js):**
```javascript
app.post('/api/billing/create-portal-session', async (req, res) => {
  const { userId, returnUrl } = req.body;

  try {
    // Fetch user's Stripe customer ID from your database
    const user = await getUserFromDatabase(userId);

    if (!user.stripeCustomerId) {
      return res.status(400).json({ error: 'No subscription found' });
    }

    const session = await stripe.billingPortal.sessions.create({
      customer: user.stripeCustomerId,
      return_url: returnUrl,
    });

    res.json({ url: session.url });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

### 3. Get User Subscription

**Endpoint:** `GET /api/subscriptions/:userId`

**Description:** Retrieves the user's current subscription status.

**Response:**
```json
{
  "userId": "string",
  "tier": "free" | "premium",
  "status": "active" | "canceled" | "past_due" | "incomplete",
  "stripeCustomerId": "cus_xxx",
  "stripeSubscriptionId": "sub_xxx",
  "currentPeriodEnd": "2024-05-01T00:00:00.000Z",
  "cancelAtPeriodEnd": false,
  "createdAt": "2024-04-01T00:00:00.000Z",
  "updatedAt": "2024-04-01T00:00:00.000Z"
}
```

**Backend Implementation Example (Node.js):**
```javascript
app.get('/api/subscriptions/:userId', async (req, res) => {
  const { userId } = req.params;

  try {
    const subscription = await getSubscriptionFromDatabase(userId);

    if (!subscription) {
      return res.status(404).json({ error: 'Subscription not found' });
    }

    res.json(subscription);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

### 4. Cancel Subscription

**Endpoint:** `POST /api/subscriptions/cancel`

**Description:** Cancels a subscription at the end of the billing period.

**Request Body:**
```json
{
  "userId": "string"
}
```

**Response:**
```json
{
  "success": true
}
```

**Backend Implementation Example (Node.js):**
```javascript
app.post('/api/subscriptions/cancel', async (req, res) => {
  const { userId } = req.body;

  try {
    const user = await getUserFromDatabase(userId);

    if (!user.stripeSubscriptionId) {
      return res.status(400).json({ error: 'No active subscription' });
    }

    await stripe.subscriptions.update(user.stripeSubscriptionId, {
      cancel_at_period_end: true,
    });

    await updateSubscriptionInDatabase(userId, {
      cancelAtPeriodEnd: true,
    });

    res.json({ success: true });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

### 5. Stripe Webhook Handler

**Endpoint:** `POST /api/webhooks/stripe`

**Description:** Handles Stripe webhook events to keep subscription status in sync.

**Important Events to Handle:**
- `checkout.session.completed` - New subscription created
- `customer.subscription.updated` - Subscription status changed
- `customer.subscription.deleted` - Subscription canceled
- `invoice.payment_succeeded` - Payment successful
- `invoice.payment_failed` - Payment failed

**Backend Implementation Example (Node.js):**
```javascript
app.post('/api/webhooks/stripe', express.raw({type: 'application/json'}), async (req, res) => {
  const sig = req.headers['stripe-signature'];

  let event;

  try {
    event = stripe.webhooks.constructEvent(
      req.body,
      sig,
      process.env.STRIPE_WEBHOOK_SECRET
    );
  } catch (err) {
    return res.status(400).send(`Webhook Error: ${err.message}`);
  }

  switch (event.type) {
    case 'checkout.session.completed':
      const session = event.data.object;
      const userId = session.metadata.userId;
      const subscription = await stripe.subscriptions.retrieve(session.subscription);

      // Save subscription to database
      await saveSubscriptionToDatabase({
        userId,
        tier: 'premium',
        status: 'active',
        stripeCustomerId: session.customer,
        stripeSubscriptionId: subscription.id,
        currentPeriodEnd: new Date(subscription.current_period_end * 1000),
        cancelAtPeriodEnd: false,
      });
      break;

    case 'customer.subscription.updated':
      const updatedSub = event.data.object;
      await updateSubscriptionInDatabase(updatedSub.metadata.userId, {
        status: updatedSub.status,
        currentPeriodEnd: new Date(updatedSub.current_period_end * 1000),
        cancelAtPeriodEnd: updatedSub.cancel_at_period_end,
      });
      break;

    case 'customer.subscription.deleted':
      const deletedSub = event.data.object;
      await updateSubscriptionInDatabase(deletedSub.metadata.userId, {
        tier: 'free',
        status: 'canceled',
      });
      break;
  }

  res.json({ received: true });
});
```

## Database Schema

You'll need to store subscription data in your database. Here's a recommended schema:

### `user_subscriptions` table

```sql
CREATE TABLE user_subscriptions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id VARCHAR(255) NOT NULL UNIQUE,
  tier VARCHAR(20) NOT NULL DEFAULT 'free', -- 'free' or 'premium'
  status VARCHAR(20) NOT NULL DEFAULT 'active', -- 'active', 'canceled', 'past_due', 'incomplete'
  stripe_customer_id VARCHAR(255),
  stripe_subscription_id VARCHAR(255),
  current_period_end TIMESTAMP,
  cancel_at_period_end BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_user_id ON user_subscriptions(user_id);
CREATE INDEX idx_stripe_customer_id ON user_subscriptions(stripe_customer_id);
```

## Environment Variables

### Frontend (.env.local)
```bash
VITE_STRIPE_PUBLISHABLE_KEY=pk_test_xxx
VITE_STRIPE_MONTHLY_PRICE_ID=price_xxx
VITE_STRIPE_YEARLY_PRICE_ID=price_xxx
```

### Backend
```bash
STRIPE_SECRET_KEY=sk_test_xxx
STRIPE_WEBHOOK_SECRET=whsec_xxx
```

## Stripe Dashboard Setup

1. **Create Products:**
   - Go to https://dashboard.stripe.com/test/products
   - Create "Premium Monthly" product (€4.99/month)
   - Create "Premium Yearly" product (€29/year)
   - Copy the Price IDs and add them to your environment variables

2. **Set up Webhook:**
   - Go to https://dashboard.stripe.com/test/webhooks
   - Add endpoint: `https://yourbackend.com/api/webhooks/stripe`
   - Select events: `checkout.session.completed`, `customer.subscription.updated`, `customer.subscription.deleted`
   - Copy the webhook secret and add it to your backend environment variables

3. **Configure Customer Portal:**
   - Go to https://dashboard.stripe.com/test/settings/billing/portal
   - Enable subscription cancellation
   - Set your business information and branding

## Testing

Use Stripe's test cards:
- **Success:** 4242 4242 4242 4242
- **Decline:** 4000 0000 0000 0002
- **3D Secure:** 4000 0025 0000 3155

Any future date and any 3-digit CVC will work for test cards.

## Security Notes

- **NEVER** expose your Stripe Secret Key in the frontend
- **ALWAYS** verify webhook signatures
- Use HTTPS in production
- Implement rate limiting on API endpoints
- Validate user IDs match authenticated users
- Store Stripe customer IDs securely

## Next Steps

1. Implement the backend API endpoints listed above
2. Set up your database schema
3. Create products in Stripe Dashboard
4. Configure webhooks
5. Add your API keys to environment variables
6. Test the full flow in test mode
7. Switch to live keys when ready for production
