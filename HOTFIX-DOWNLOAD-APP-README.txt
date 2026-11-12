BLUEFM — Download App header button

This small website hotfix adds a persistent “Download App” button next to the
Sign in / profile action in the sticky top bar. It links directly to the BLUEFM
Android APK release:
https://github.com/bluefmtts/bluefm-/releases/latest/download/bluefm0.6.apk

Files in this archive (extract into the existing BLUEFM source folder, keeping
folders and overwriting the matching files):
- client/src/components/Layout.jsx
- client/src/styles/app.css

No backend, Firebase, Cloudflare Worker, Cashfree, or secrets changes are needed.

Windows deployment:
1. Extract this archive into C:\Users\User\Downloads\bluefm-fixed\bluefm and overwrite the two existing files.
2. Open PowerShell in that source folder and run: npm run build
3. If the build succeeds, run: vercel --prod
4. After Vercel reports Ready, hard-refresh https://bluefm.live (Ctrl+Shift+R).

On very narrow phones the top-bar search shortcut is hidden so the app-download
and sign-in buttons stay visible; search remains available from Explore.
