BLUEFM — PAYMENT HOTFIX (Cashfree real checkout)
=================================================

Kya fix hua
-----------
Purana client sirf "sandbox simulation" jaanta tha. Live server pe provider
cashfree hai, isliye Pay dabate hi /api/payments/sandbox/simulate 404 deta tha
aur Cashfree checkout kabhi khulta hi nahi tha. Plus:
  - phone number collect nahi hota tha (dummy 9999999999 jata tha)
  - Cashfree ke return_url / notify_url localhost pe jaate the (APP_URL unset)
  - /payment/return route hi nahi tha -> NotFound

Is hotfix me:
  - client/src/lib/cashfree.js        : Cashfree SDK v3 loader + checkout
  - client/src/pages/Story.jsx        : phone field + real Cashfree redirect
  - client/src/pages/PaymentReturn.jsx: return page, server-side verify + retry
  - client/src/main.jsx               : /payment/return route
  - server/routes/payments.js         : phone validate/save, orderId-only verify,
                                        pending (402) instead of hard-fail
  - server/services/payments.js       : cashfree ke liye valid phone mandatory
  - server/payments/provider.js       : dummy phone hataya, notify_url override
  - firebase-web/api.js               : APP_URL default https://bluefm.qzz.io
  - test/cashfree.test.js             : nye tests C8-C11 (11/11 pass)

LAGAO KAISE (Windows PowerShell)
--------------------------------
1. bluefm-payment-hotfix.zip ko apne bluefm folder me extract karo:
   C:\Users\User\Downloads\bluefm-fixed\bluefm
   (Extract All -> destination upar wala folder -> Replace All)

2. Functions bundle sync + deploy:
   cd C:\Users\User\Downloads\bluefm-fixed\bluefm
   npm run fb:prepare
   cd firebase
   firebase deploy --only functions:web
   cd ..

3. Frontend deploy:
   vercel --prod

4. Cashfree dashboard:
   - Website/return domain: bluefm.qzz.io
   - Webhook URL: https://bluefm.qzz.io/api/payments/webhooks/cashfree
     (Express API wala. webCashfreeWebhook callable NAHI.)

5. Real payments ke liye secrets (firebase folder se):
   firebase functions:secrets:set CASHFREE_CLIENT_ID      # Production Client ID
   firebase functions:secrets:set CASHFREE_CLIENT_SECRET  # Production Secret
   firebase functions:secrets:set CASHFREE_ENV            # production
   firebase deploy --only functions:web

TEST
----
Sandbox keys + CASHFREE_ENV=sandbox me ₹2 pass kharido:
  Pay -> phone bharo -> Cashfree page -> wapas /payment/return ->
  "Payment successful" -> story unlock.

Real pe jane se pehle ek ₹2 real payment khud karke dekh lo.

NOTE: cache/SW/covers code is hotfix me chheda NAHI gaya.
