BLUEFM — Cloudflare delivery + real upload + demo cleanup hotfix
================================================================

Is hotfix me kya hai:
1. Cloudflare Worker ab sirf stream nahi, admin upload + public covers bhi serve karta hai.
2. Admin panel se real MP3 + cover upload: browser -> Cloudflare Worker -> B2.
   Cloud Function me ffprobe/sharp ki zaroorat nahi.
3. Demo content safely hatane ka script: scripts/clear-demo.js
4. Firebase me B2_DOWNLOAD_URL secret bind hota hai (delivery ON switch).

Files (source folder me overwrite karo):
  cloudflare/worker-stream-auth.js
  cloudflare/wrangler.toml
  cloudflare/README.md
  server/lib/b2.js
  server/routes/admin.js
  server/services/catalog.js
  firebase-web/api.js
  client/src/lib/mediaUpload.js
  client/src/pages/Admin.jsx
  scripts/clear-demo.js
  package.json
  test/worker.test.mjs
  test/cloud.test.js

IMPORTANT:
- STREAM_TOKEN_SECRET Worker aur Firebase dono me SAME hona chahiye.
- B2 application key me readFiles + writeFiles capability honi chahiye.
- Pehle Worker deploy karo, phir uska URL Firebase secret B2_DOWNLOAD_URL me set karo.
- Deploy filter hamesha: functions:web
- Project flag hamesha: --project nodio-575bf

Step-by-step README cloudflare/README.md me hai.
