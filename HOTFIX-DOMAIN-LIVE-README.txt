# BLUE FM — bluefm.live domain update

Primary website: **https://bluefm.live**
Allowed browser origins: `https://bluefm.live`, `https://www.bluefm.live`, `https://bluefm.vercel.app` (fallback).

This update changes website/payment-return/CORS settings only. **Do not replace working Cashfree, B2, Firebase or Cloudflare keys.** The media Worker stays at `https://bluefm-stream.singhvikas21571.workers.dev`; `media.bluefm.live` in older docs is an optional future CDN hostname, not required for this update.

## What changed

- Cashfree new-order return: `https://bluefm.live/payment/return?order_id={order_id}`
- Cashfree new-order webhook: `https://bluefm.live/api/payments/webhooks/cashfree`
- Worker upload/stream CORS: exact origin allowlist above (no wildcard domains).
- Privacy, Terms and Contact show the new website.
- `npm run fb:prepare` refreshes the Functions source mirror and updates only `APP_URL`, `WEB_RETURN_BASE_URL`, `ALLOWED_ORIGINS` in `firebase/functions-web/.env`. Other env lines are preserved; no `.env` or secret is included in the hotfix ZIP.
- Latest Hindi-title fix is included: the Hindi title stays unchanged; its internal story/storage id becomes ASCII-safe.

## Before deploying

1. Attach `bluefm.live` and `www.bluefm.live` to the **existing** Vercel project. Copy the exact current DNS records shown by Vercel into your registrar; do not reuse an old guide's IP/verification token. Wait until Vercel shows valid HTTPS for both hosts.
2. In Cashfree's domain/URL whitelist, request approval for `https://bluefm.live` (and `https://www.bluefm.live` if you use www). This is a dashboard action; code cannot approve it.
3. Extract the hotfix into the existing project root, replacing matching files. Keep `node_modules`, local credentials and unrelated files.

## Deploy one layer at a time (PowerShell)

Run from `C:\Users\User\Downloads\bluefm-fixed\bluefm`. Wait for each command to finish successfully before the next.

```powershell
npm run cf:deploy
```

```powershell
npm run fb:prepare
```

```powershell
cd firebase; firebase deploy --only functions:web --project nodio-575bf; cd ..
```

```powershell
vercel --prod
```

The Functions command must use **functions:web**, not `functions:webApi`. The prepare step must run before Functions deploy, otherwise an old source mirror/.env can ship the previous domain.

## Verify with a fresh request

- Open `https://bluefm.live` and hard-refresh once. `/privacy.html`, `/terms.html`, `/refund.html`, `/contact.html` should load.
- Sign in to the live admin panel, create a **new** story (Hindi title is fine), upload its cover and MP3, publish it, then play it. Do not reuse a failed upload/order.
- Create a **fresh** purchase order after all deploys. Its Cashfree return should land on `bluefm.live`; access must still be granted only after server verification.
- Browser DevTools → Network: Worker `/upload` preflight should allow `https://bluefm.live`; authorized MP3 response should include `X-Stream-Provider: cloudflare+b2`.

Existing orders keep their original return URLs. A domain change does not retroactively rewrite them or whitelist the new domain in Cashfree.
