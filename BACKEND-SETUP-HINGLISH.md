# BLUEFM Backend Setup — Ekdum Aasan Guide (Hinglish)

> Ye guide `bluefm` (2nd website) ka backend live karne ke liye hai.
> Har step ke baad **CHECK** likha hai — wo pass ho tohi aage badho.
> Kahi atko toh screenshot/error Arena chat me bhejo, saath me karenge.

---

## Samjho: backend me 4 log hain, kaam kya karte hain

| Service | Kaam | Account chahiye |
|---------|------|-----------------|
| **Firebase** (`nodio-575bf`) | Database + API (dimaag) | Google account |
| **Backblaze B2** | Audio files ka godown | Backblaze account |
| **Cashfree** | Payments (₹2/₹14/₹60) | Cashfree merchant account |
| **Cloudflare** | Audio free me fast deliver | Cloudflare account (baad me) |

**Order zaroori hai:** pehle Firebase (API zinda karo) → phir data → phir B2 (audio) → phir Cashfree (paisa) → Cloudflare sabse last (optimization).

---

## STEP 0 — PC pe tools (5 min)

PowerShell kholo (Start → "PowerShell"):

```powershell
node -v          # v20+ hona chahiye
npm -v
```

Node nahi hai toh https://nodejs.org se LTS install karo. Phir:

```powershell
npm i -g firebase-tools
firebase --version
```

**CHECK:** `firebase --version` number dikhaye (e.g. 14.x). ✅

---

## STEP 1 — Fixed code lo (2 min)

Ye wali zip use karo: **`bluefm-fixed.zip`** (purani `bluefm (1).zip` me 2 bugs the jo deploy crash karte the — isme fix hain).

```powershell
cd C:\bluefm            # jahan unzip kiya, us folder me jao
npm install
```

**CHECK:** `npm install` bina error khatam ho. ✅

Local test (optional par recommended):

```powershell
npm start
```

Browser me `http://localhost:8080` kholo — app dikhni chahiye. Band karne ko `Ctrl+C`.

---

## STEP 2 — Firebase login + project (5 min)

```powershell
firebase login          # browser khulega, Google se login karo
firebase projects:list  # nodio-575bf dikhna chahiye
cd firebase
firebase use nodio-575bf
cd ..
```

> Agar `nodio-575bf` nahi dikha toh console.firebase.google.com pe jao →
> project banao (naam kuch bhi, ID `nodio-575bf` agar free ho) → Blaze plan
> (pay-as-you-go) enable karo. **Cloud Functions ke liye Blaze zaroori hai.**

**CHECK:** `firebase projects:list` me `nodio-575bf (current)` dikhe. ✅

---

## STEP 3 — Firestore + Auth ON karo (5 min, browser me)

console.firebase.google.com → project `nodio-575bf`:

1. **Build → Firestore Database → Create database**
   - Location: `asia-south1 (Mumbai)`
   - Mode: **Production mode**
2. **Build → Authentication → Get started → Email/Password → Enable**

**CHECK:** Firestore me "(default)" database dikhe, Auth me Email/Password "Enabled" ho. ✅

---

## STEP 4 — Secrets daalo (10 min)

Secrets = passwords/keys jo code me nahi likhte. Ek-ek karke:

```powershell
cd firebase
firebase functions:secrets:set SESSION_SECRET
firebase functions:secrets:set AUDIO_SIGNING_SECRET
firebase functions:secrets:set STREAM_TOKEN_SECRET
firebase functions:secrets:set ADMIN_PASSWORD
```

Har command value puchegi. **Random value** ke liye PowerShell me:

```powershell
[Convert]::ToBase64String((1..32 | ForEach-Object { Get-Random -Max 256 }))
```

4 baar chalao, 4 alag values banao, paste karo. `ADMIN_PASSWORD` me apna strong password.

> Baaki secrets (B2, Cashfree) abhi khaali chhod sakte ho — agle steps me bharenge.
> Lekin deploy ke liye ye 4 + neeche wale placeholders set hone chahiye.

Abhi ke liye inko bhi set kar do (baad me real values se update karenge):

```powershell
firebase functions:secrets:set CASHFREE_CLIENT_ID      # abhi: placeholder
firebase functions:secrets:set CASHFREE_CLIENT_SECRET  # abhi: placeholder
firebase functions:secrets:set CASHFREE_ENV            # value: sandbox
firebase functions:secrets:set B2_KEY_ID               # abhi: placeholder
firebase functions:secrets:set B2_APP_KEY              # abhi: placeholder
firebase functions:secrets:set B2_BUCKET_ID            # abhi: placeholder
cd ..
```

**CHECK:** `firebase functions:secrets:access SESSION_SECRET` value dikhaye. ✅

---

## STEP 5 — Deploy! 🚀 (5 min)

```powershell
npm run fb:prepare
cd firebase
firebase deploy --only functions:web
```

3-6 min lagenge. Success pe URL milega:

```
Function URL (webApi): https://asia-south1-nodio-575bf.cloudfunctions.net/webApi
```

**CHECK:** browser me ye kholo:
```
https://asia-south1-nodio-575bf.cloudfunctions.net/webApi/api/health
```
`{"ok":true,...}` aana chahiye. 🎉 **Backend zinda hai!**

> 500 aaye toh: `firebase functions:log --only webApi` — error chat me bhejo.

---

## STEP 6 — Data bharo (stories/episodes) (5 min)

Service account key chahiye:
Console → Project settings (⚙️) → **Service accounts → Generate new private key** → JSON download.

```powershell
$env:GOOGLE_APPLICATION_CREDENTIALS="C:\path\to\service-account.json"
node scripts/migrate-to-firebase.js --apply
node scripts/migrate-to-firebase.js --verify
```

**CHECK:** `--verify` bole `✓ OK — the web app will render this catalogue` (10 stories, 67 episodes). ✅

> Note: B2 keys abhi placeholder hain toh audio upload skip ho sakta hai —
> metadata Firestore me aa jayega. Audio STEP 7 ke baad dobara `--apply` se chadh jayegi.

---

## STEP 7 — Backblaze B2 (audio) (10 min)

backblaze.com → login → **B2 Cloud Storage**:

1. Bucket `bluefm-web-audio` pehle se hai (67 files). Naya banana ho toh **Private** bucket.
2. **Application Keys → Add a New Application Key**:
   - name: `bluefm-web`
   - bucket: **sirf `bluefm-web-audio`** (bucket-scoped!)
   - capabilities: `readFiles, writeFiles, listFiles`
3. keyID aur applicationKey copy karo.

Secrets update:

```powershell
cd firebase
firebase functions:secrets:set B2_KEY_ID       # real keyID
firebase functions:secrets:set B2_APP_KEY      # real appKey
firebase functions:secrets:set B2_BUCKET_ID    # bucket ka ID
cd ..
```

Audio upload (agar pehle nahi chadhi):

```powershell
$env:B2_KEY_ID="<keyID>"; $env:B2_APP_KEY="<appKey>"
$env:B2_BUCKET_ID="<bucketID>"; $env:B2_BUCKET_NAME="bluefm-web-audio"
node scripts/migrate-to-firebase.js --apply
```

Redeploy: `cd firebase; firebase deploy --only functions:web; cd ..`

**CHECK:** site pe koi story kholo, pass lo (sandbox), play karo — audio chale. ✅

---

## STEP 8 — Cashfree (payments) (10 min)

> **Pehle payment hotfix lagao** (agar `bluefm-fixed.zip` naya extract kiya hai to skip):
> `bluefm-payment-hotfix.zip` ko `bluefm` folder me extract karo (Replace All), phir
> `npm run fb:prepare`. Ye client me asli Cashfree checkout + phone number + return
> page add karta hai. Purana code sirf sandbox simulation jaanta tha, isliye real
> payment pe 404 aata tha.

cashfree.com → merchant dashboard → **Payment Gateway → Developers**:

1. **API Keys** se Client ID + Client Secret lo. Test ke liye **Sandbox**, real paise
   ke liye **Production** keys (KYC/activation complete hona chahiye).
2. **Website/return domain** me `bluefm.qzz.io` add karo (aapne ye already kiya hai).
3. Secrets update:

```powershell
cd firebase
firebase functions:secrets:set CASHFREE_CLIENT_ID      # Client ID
firebase functions:secrets:set CASHFREE_CLIENT_SECRET  # Client Secret
firebase functions:secrets:set CASHFREE_ENV            # sandbox  (real ke liye: production)
cd ..
```

4. **Webhooks** me ye URL add karo (Express API wala — callable `webCashfreeWebhook`
   NAHI, wo alag `web_*` collections use karta hai):
```
https://bluefm.qzz.io/api/payments/webhooks/cashfree
```
   (Vercel `/api/*` ko webApi pe bhejta hai, isliye ye domain wala URL sahi hai.)
5. Redeploy: `cd firebase; firebase deploy --only functions:web; cd ..`
   Frontend bhi: `vercel --prod`

**Phone number:** Cashfree har payment ke liye customer ka 10-digit mobile maangta
hai. Checkout sheet me ab phone field aata hai; ek baar bharo, account me save ho
jaata hai, agli baar pre-filled. Galat/missing number pe server order banata hi nahi.

**CHECK (sandbox):** test card se ₹2 pass kharido → Cashfree page khule → wapas
`/payment/return` pe aaye → "Payment successful" → access milna chahiye.
`firebase functions:log --only webApi` me `payment verified` dikhe. ✅

**Real (production) pe jane se pehle:**
- `CASHFREE_ENV` = `production`, aur Production Client ID/Secret set karo.
- Cashfree dashboard me Production keys + website `bluefm.qzz.io` approved ho.
- Ek chhota ₹2 real payment khud karke verify karo, phir users ko kholo.

---

## STEP 9 — Frontend Vercel pe (10 min)

```powershell
npm i -g vercel
vercel login
vercel --prod
```

Vercel → Project → Settings → **Environment Variables** (Firebase Console →
Project settings → Your apps → Web app se config lo):

| Name | Value |
|------|-------|
| `VITE_FIREBASE_API_KEY` | `AIza...` |
| `VITE_FIREBASE_AUTH_DOMAIN` | `nodio-575bf.firebaseapp.com` |
| `VITE_FIREBASE_PROJECT_ID` | `nodio-575bf` |
| `VITE_FIREBASE_APP_ID` | `1:...:web:...` |

Phir `vercel --prod` dobara. Domain `bluefm.qzz.io` add karo (Settings → Domains).

**CHECK:** `bluefm.qzz.io` pe site khule, login ho, stories dikhein. ✅

---

## STEP 10 — Cloudflare (FREE audio delivery) — sabse last

Ye optimization hai, baad me karo. `cloudflare/README.md` follow karo:
- B2 bucket ka custom domain `media.bluefm.site` (Cloudflare proxy ON)
- Worker deploy (`wrangler deploy`)
- Server me `B2_DOWNLOAD_URL=https://media.bluefm.site`

Isse B2 egress **free** ho jata hai (Bandwidth Alliance).

---

## Admin banana

Site pe signup karo, phir:

```powershell
node scripts/make-admin.js aapka@email.com --project=nodio-575bf
```

`/admin` khul jayega.

---

## Atko toh?

| Symptom | Fix |
|---------|-----|
| deploy fail | `firebase functions:log --only webApi` ka error bhejo |
| `Container Healthcheck failed` + logs me `Named export 'logger' not found` | Functions SDK purana 6.x hai. `firebase/functions-web/package.json` me `firebase-functions` `^7.4.0` karo, `npm run fb:prepare`, phir redeploy |
| health 500 | koi secret missing — STEP 4 dobara |
| stories nahi dikh rahi | `node scripts/migrate-to-firebase.js --verify` |
| login ke baad logout | cookie issue — `ALLOWED_ORIGINS` me domain hai? |
| audio 401 | `STREAM_TOKEN_SECRET` server/worker same hai? |
| payment ke baad access nahi | Cashfree webhook URL + secret check |

**Koi bhi error aaye — screenshot ya text Arena chat me bhejo, main saath me fix karunga.** 💪