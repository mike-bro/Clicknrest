# Setting up real payments (Stripe)

Your site is currently just static files (HTML/CSS/JS) — files that a browser reads directly, with nothing running on a server. That's fine for browsing and searching hotels, but **taking real card payments always needs a small piece of code that runs on a server**, because that's the only safe place to keep your secret Stripe key. If that key sat in your JS files instead, anyone could view it and charge things to your account.

The good news: you don't need to rent or manage a real server. **Netlify Functions** let you drop a couple of small JavaScript files into a `netlify/functions` folder (already done — see `create-checkout-session.js` and `verify-session.js`), and Netlify runs them for you, on demand, for free at this scale. This project is already wired to call those two functions.

## How the payment flow works now

1. Guest fills in their name/email on `booking.html` and clicks **Continue to payment**.
2. The page calls your `create-checkout-session` function, which asks Stripe for a secure, Stripe-hosted payment page and sends the guest there.
3. Guest pays on Stripe's page (you never see or touch their card number).
4. Stripe redirects them back to `booking.html`, which calls `verify-session` to double check with Stripe (server-side) that the payment really went through, then shows the confirmation screen.

Note: this charges the card for the room total, but it doesn't yet automatically reserve the room with Liteapi — that step still needs to be added (or done manually for now, e.g. by checking paid bookings in your Stripe dashboard each day and booking the room yourself). Happy to wire that up too when you're ready.

## Step 1 — Create a Stripe account

1. Go to stripe.com and sign up (free).
2. Stay in **Test mode** (toggle top-right of the Stripe dashboard) while you're setting things up — test mode lets you "pay" with fake card numbers, no real money moves.
3. Go to **Developers → API keys** and copy the **Secret key** (starts with `sk_test_...`). Keep this private — don't paste it into any HTML/JS file.

## Step 2 — Put the site on Netlify

1. Go to netlify.com and sign up (free).
2. Easiest path: put this project folder in a GitHub repository, then in Netlify choose **Add new site → Import an existing project** and pick that repo. Netlify will detect `netlify.toml` and automatically set up both the static site and the two functions.
   - (Alternative if you don't want to use GitHub yet: install the Netlify CLI and run `netlify deploy --prod` from inside this folder — it uploads the functions too. Drag-and-drop deploys on the Netlify website do **not** include functions, so avoid that method.)
3. Once deployed, Netlify gives you a live URL like `https://your-site-name.netlify.app`.

## Step 3 — Add your Stripe key to Netlify

1. In your Netlify site, go to **Site configuration → Environment variables**.
2. Add a variable named exactly `STRIPE_SECRET_KEY` with the value being the `sk_test_...` key you copied earlier.
3. Redeploy the site (Netlify usually prompts you to, or trigger a redeploy from the **Deploys** tab) so the function can see the new variable.

## Step 4 — Test it

1. Visit your live site, search hotels, pick a room, and go through checkout.
2. On Stripe's payment page, use a test card: **4242 4242 4242 4242**, any future expiry date, any 3-digit CVC, any postcode.
3. You should land back on `booking.html` with the "Payment received" confirmation. You can see the test payment appear in your Stripe dashboard under **Payments**.

## Going live later

- When you're ready to take real money, switch Stripe out of Test mode, grab the **live** secret key (`sk_live_...`), and replace the `STRIPE_SECRET_KEY` value in Netlify with that.
- About using your father's payment setup instead: if he already has a Stripe account for his customers, you can point this same site at his account just by putting *his* secret key in `STRIPE_SECRET_KEY` — no code changes needed. (If he uses a different payment processor entirely, like PayPal or Square, that would need a different function — happy to build that version when the time comes.)
- Stripe takes a small percentage + fixed fee per transaction — check stripe.com/pricing for current rates in your country.
- Before relying on this for real bookings, close the gap mentioned above: have the booking actually get created in Liteapi after payment (via their prebook/book endpoints) instead of relying on manual follow-up.
