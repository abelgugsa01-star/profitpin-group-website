# ProfitPin launch-day runbook

All four products launch together, unsigned (Windows SmartScreen shows
"protected your PC" on first run; every download page tells customers to choose
More info → Run anyway). Re-sign and re-release when Microsoft ever approves.

Order for each product: **card test → open → verify**. About 10 minutes each.
Do them one at a time so a problem is easy to attribute.

## 0. Before starting (founder)

- Have the founder's own card ready. Every test is a real live-mode checkout;
  nothing is charged during the 30-day trial, and the test subscription is
  cancelled from the Stripe portal right after.
- Stripe dashboard signed in (Browser pane) to watch Customers / Events.

## 1. SaleLedger

1. Open sign-ups: `python scripts/set_signups.py open` in `saleledger/`, commit
   and push `getsaleledger.com`; Render `saleledger-license` → Environment →
   `SIGNUPS_OPEN` = `1` → Save (redeploys ~1 min).
2. Card test: getsaleledger.com → Start free month → checkout with the founder's
   card → welcome page shows the `SL-…` key → download exe → Overview/activation
   accepts the key → app runs. In Stripe: customer + subscription `trialing`;
   webhook `saleledger-license` shows one successful delivery.
3. Cancel the test subscription from the welcome page's "Manage subscription"
   (Stripe portal). Keep the customer record; it costs nothing.
4. If anything failed: `set_signups.py closed` + `SIGNUPS_OPEN=0` and fix first.

## 2. SubLimit

1. Open: Render `sublimit-licensing-test` → Environment →
   `SUBLIMIT_CUSTOMER_API_ENABLED` = `1` (leave `SUBLIMIT_RECEIVER_PAUSED` = `0`)
   → Save. Site: the Start page already says "not open to everyone yet"; edit
   `website/start.html` in `sublimit/` (both the HTML and the inline RSC copy of
   the sentence) or leave it and rely on the in-app flow. Mirror to
   `getsublimit.com` and push.
2. Card test: install `SubLimit-0.7.2-Setup.exe` (getsublimit.com/start) →
   Overview → Activation → sign in with the founder's purchase email (code
   arrives from support@getsublimit.com, a Neo mailbox) → Continue to secure
   checkout → card → app shows "Access active until …". Stripe: subscription
   trialing; webhook `SubLimit production licensing` delivered.
3. Cancel via Manage or cancel subscription in the app (Stripe portal).
4. Rollback: `SUBLIMIT_CUSTOMER_API_ENABLED=0`.

## 3. StockStart

1. Open: `python scripts/set_signups.py open` in `stockstart/`, commit and push
   `getstockstart.com`; Render `stockstart-app` → `SIGNUPS_OPEN` = `1` → Save
   (rebuild ~4 min).
2. Card test: app.getstockstart.com/signup → create account → Subscribe → card →
   returns to the app with access → upload one sample invoice. Stripe:
   subscription trialing; webhook `stockstart-app` delivered (this webhook is
   what flips the account to active — if the app still says "subscribe", check
   the webhook delivery first).
3. Cancel via the app's billing portal.
4. Rollback: `set_signups.py closed` + `SIGNUPS_OPEN=0`.

## 4. LeafLedger

1. Open: `python scripts/set_signups.py open` in `leafledger/`, commit and push
   `getleafledger.com`; Render `leafledger-license` → `SIGNUPS_OPEN` = `1`.
2. Card test: getleafledger.com → Start free month → checkout → welcome page
   shows the `LL-…` key → download `LeafLedger.exe` (v0.77.0) → enter the key in
   the app → runs. Stripe: subscription trialing; webhook `leafledger-license`
   delivered.
3. Cancel via Manage subscription on the welcome page.
4. Rollback: `set_signups.py closed` + `SIGNUPS_OPEN=0`.

## 5. After all four

- profitpingroup.com: no change needed (already lists all products).
- Watch for a day: Stripe → Developers → Webhooks (any failed deliveries), and
  the four support mailboxes.
- Known follow-ups: code signing when approved; SubLimit checkout runs with
  Stripe Managed Payments off (Profitpin is the seller, no automatic tax) while
  the other three run with it on — align when convenient.
