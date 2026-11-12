BLUEFM — COMPLETE WEBSITE SOURCE ZIP

File: bluefm-final-complete.zip
Size: 26,884,134 bytes
SHA-256: de7385370213269ff88558e4861f849cff5ac9b2361b61922ef8aaf1e38c4852
Project files: 346 (including the current production build)

The ZIP extracts into one `bluefm/` folder. It combines the full BLUEFM website source with the approved payment, Cloudflare upload/delivery, safe Hindi story-ID, policy/domain, service-worker/cache, and Download App changes. The build in `dist/client` was regenerated.

Validation completed: `npm run build` PASS; Worker tests 16/16 PASS; cloud tests 22/22 PASS (after `npm run fb:prepare`). Package integrity check PASS.

Primary domain: https://bluefm.live. The header APK link points to the BLUEFM GitHub Release.

Security/data note: `.env`, live keys, service-account credentials, `node_modules`, production Firestore records, and production B2 uploads are not included. Bundled local sample media is included. The APK itself remains a separate GitHub Release asset. Read `bluefm/HANDOFF-STATUS.md` for current state and known follow-ups.
