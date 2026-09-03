# gst-law

Over-the-air **data** package for the GST law reader app. Static JSON only — no
code. The app ships every dataset bundled (the offline floor); this repo lets the
refreshable ones (CBIC notifications / circulars / instructions / orders, GSTN
advisories, portal FAQs, forms, GST rates, the editorial commentary, and the
derived search indexes) update without an app-store release.

## Layout

```
v1/
  manifest.json              # schemaVersion, per-dataset {version, sha256, bytes, url}, changelog
  <dataset>.<version>.json   # immutable, content-addressed by the sha256 in the manifest
```

`v1` is the data-schema namespace (`schemaVersion` in the manifest). A future
breaking shape change starts a `v2/` and older app builds keep reading `v1/`.

## How the app uses it

`https://shailendra-mestry.github.io/gst-law/v1/manifest.json` is baked into
the app (`app.json` → `expo.extra.dataManifestUrl`). On launch the app fetches
the manifest, downloads only the datasets whose `version` + `sha256` changed,
**verifies each file against its SHA-256 before use**, and swaps them in on the
next launch. Nothing executable is ever fetched.

## Publishing an update

From the app repo (`~/Claude/gst-reader`):

```bash
npm run data                 # or a targeted npm run data:advisories etc.
python3 scripts/publish_data.py \
  --base https://shailendra-mestry.github.io/gst-law/v1 \
  --out  ~/Claude/gst-law/v1 \
  --note "What changed in this package"
```

Then here:

```bash
git add -A && git commit -m "data: <what changed>" && git push
```

GitHub Pages redeploys in ~1 min. Installed apps pick it up on their next launch.
`data/published-manifest.json` in the app repo is the version ledger — commit it
there each time.

## Settings

GitHub Pages must be enabled: **Settings → Pages → Source: Deploy from a branch →
`main` / `/ (root)`**. `.nojekyll` keeps Pages from running Jekyll over the files.
