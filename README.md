# A-Life Medica — Performance Reports

Monthly social-media + paid-ads performance reports for the A-Life Medica brand portfolio (BIOS Line, LYSI), built by **Polar Bear Agency**.

## Navigation

- `/` → year picker
- `/2026/` → month picker
- `/2026/april/` → brand picker (BIOS Line / LYSI)
- `/2026/april/bios-line/` → BIOS Line April 2026 report
- `/2026/april/lysi/` → LYSI April 2026 report

## Adding a new month

```
mkdir -p 2026/<month>/<brand>
# copy index.html, charts and thumbs/ into the brand folder
# update 2026/index.html and 2026/<month>/index.html to link to it
```

## Deploy
Static site on Vercel — no build step. Pushes to `main` auto-deploy.
