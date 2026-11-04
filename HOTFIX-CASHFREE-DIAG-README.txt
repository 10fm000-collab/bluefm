BLUEFM — CASHFREE 401 DIAGNOSTIC + SECRET CLEANUP (backend-only)
================================================================

Kya karta hai
-------------
1. Secret Manager se aayi CASHFREE values ke aage-piche ki whitespace,
   newline, BOM aur wrapping quotes hata deta hai (ye sab 401 dete hain).
2. CASHFREE_ENV ko normalise karta hai: "Production"/"prod"/"live" -> production.
3. Startup pe ek SAFE diagnostic log chhapta hai (secret kabhi print nahi hota):
     [BLUEFM] Cashfree diag: env=... endpoint=... clientId=CFxx…xxxx (len N)
       secret=cfsk…xxxx (len N) secretLooks=PRODUCTION|SANDBOX|unknown
   Agar env=production par secret SANDBOX nikla -> "KEY/ENV MISMATCH" warning.
4. 401 error message me endpoint + env + hint add karta hai.

LAGAO (PowerShell) — sirf backend, Vercel ki zaroorat NAHI
----------------------------------------------------------
cd C:\Users\User\Downloads\bluefm-fixed\bluefm
Expand-Archive -Force "$env:USERPROFILE\Downloads\bluefm-cashfree-diag-hotfix.zip" .
npm run fb:prepare
cd firebase
firebase deploy --only functions:web --project nodio-575bf
firebase functions:log --only webApi -n 20 --project nodio-575bf

CHECK: log me "Cashfree diag" line dhundo aur wo line chat me paste karo.
Usse pakka pata chalega keys production ki hain ya sandbox ki, aur endpoint
sahi hai ya nahi. Secret ki value mat bhejna — sirf diag line.

NOTE: cache/SW/covers code chheda nahi gaya.
