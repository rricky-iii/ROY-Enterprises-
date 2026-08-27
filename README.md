# ROY Enterprises — storefront + custom job requests

A small e-commerce site: a product catalog with Stripe checkout, and a
custom-job request form. Both paths email you when something happens.

## What's in here

- `index.html` — the storefront (products, cart, checkout, job request form)
- `success.html` — shown after a successful Stripe payment
- `netlify/functions/create-checkout-session.js` — looks up real prices
  server-side and starts a Stripe Checkout session
- `netlify/functions/stripe-webhook.js` — Stripe calls this automatically
  when a payment completes; it emails you the order details
- `netlify/functions/submit-job-request.js` — handles the custom-job form
  and emails you the details (no payment involved)

## One-time setup (about 15–20 minutes)

### 1. Stripe account
1. Sign up at https://dashboard.stripe.com/register
2. Grab your **test** secret key from https://dashboard.stripe.com/test/apikeys
   (starts with `sk_test_...`) — use this while developing, switch to the
   live key only once you're ready to take real payments.

### 2. Resend account (sends your notification emails)
1. Sign up at https://resend.com
2. Verify a domain you own (or use their default for testing) — this becomes
   your `NOTIFY_FROM_EMAIL`
3. Grab an API key from https://resend.com/api-keys

### 3. Deploy to Netlify
1. Push this folder to a GitHub repo, then "Import Project" on
   https://app.netlify.com from that repo — this (not Drop) is the right
   path here because you need environment variables and serverless
   functions, not just a static file.
2. In Netlify: **Site settings → Environment variables**, add everything
   from `.env.example` with your real values (except `STRIPE_WEBHOOK_SECRET`
   — that comes in the next step).

### 4. Connect the Stripe webhook (the "notify me reliably" piece)
1. In Stripe: **Developers → Webhooks → Add endpoint**
2. Endpoint URL: `https://your-site-name.netlify.app/.netlify/functions/stripe-webhook`
3. Select event: `checkout.session.completed`
4. Stripe gives you a signing secret (`whsec_...`) — add that to Netlify as
   `STRIPE_WEBHOOK_SECRET`
5. Redeploy the site so the new environment variable takes effect

### 5. Test it
- Use Stripe's test card `4242 4242 4242 4242`, any future expiry, any CVC
- Place a test order → you should get an email within a few seconds
- Submit the custom job form → you should get a separate email

## Going live

Switch `STRIPE_SECRET_KEY` to your **live** key (not test), and create a
second webhook endpoint in Stripe's live mode pointing at the same URL
with its own live webhook secret.

## Customizing

- **Products & prices**: edit the `PRODUCTS` array in both
  `index.html` (what customers see) and
  `netlify/functions/create-checkout-session.js` (what actually gets
  charged) — keep these two in sync.
- **Shop name/branding**: search-and-replace "ROY Enterprises" in
  `index.html` and `success.html`.
- **Notification email format**: edit the HTML strings inside
  `stripe-webhook.js` and `submit-job-request.js`.
