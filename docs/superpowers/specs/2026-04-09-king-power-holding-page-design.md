# King Power Holding Page — Spec

**Date:** 2026-04-09
**Status:** Approved

---

## 1. Overview

A lightweight, static holding/redirect page built with Astro. Displays a branded SVG loading screen for 5 seconds, then silently redirects to `https://www.kingpower.com` with UTM parameters preserved. Built for deployment on **Cloudflare Pages** (swappable to GCS or S3 by replacing `wrangler.toml` with the respective cloud's static hosting config).

---

## 2. Framework & Architecture

- **Framework**: Astro 4.x with `output: 'static'`
- **Languages**: Astro components (zero client-side JS by default)
- **Build artifact**: Flat HTML/CSS/JS in `dist/` — deployment-agnostic
- **Deployment target**: Cloudflare Pages (swappable)

---

## 3. Configuration

### Environment Variables

| Variable | Default | Purpose |
|---|---|---|
| `PUBLIC_REDIRECT_URL` | `https://www.kingpower.com` | Redirect destination |
| `PUBLIC_GTM_ID` | `''` | Google Tag Manager ID |
| `PUBLIC_SITE_TITLE` | `King Power` | Page title |
| `PUBLIC_SITE_DESCRIPTION` | `Redirecting to King Power...` | Meta description |

### Deployment-Specific Files

| File | Purpose | Swappable? |
|---|---|---|
| `wrangler.toml` | Cloudflare Pages project config | Yes — replace with GCS/S3 equivalent |
| `.env` | Default values (committed) | No |
| `.env.production` | Production overrides (gitignored) | No |

---

## 4. Redirect Mechanism

**Primary:** Meta refresh in `<head>`:
```html
<meta http-equiv="refresh" content="5;url={REDIRECT_URL}?{utm_params}">
```

**Fallback:** Inline `<script>` as backup for browsers that block meta refresh:
```js
setTimeout(() => { window.location.href = '{REDIRECT_URL}?{utm_params}'; }, 5000);
```

**UTM Passthrough:** Any UTM params from the holding page URL (`?utm_source=foo`) are extracted and appended to the redirect URL.

---

## 5. Analytics (GTM)

- **Config file**: `src/config/analytics.ts`
  ```ts
  export const GTM_ID = import.meta.env.PUBLIC_GTM_ID ?? '';
  ```
- **Injection**: `src/components/Analytics.astro` injects GTM `<head>` snippet + `<noscript>` redirect
- **No tracking on the holding page itself** — GTM fires on the destination page

---

## 6. SEO / Meta Tags

- **Component**: `src/components/SeoMeta.astro`
- Handles: `<title>`, `<meta name="description">`, Open Graph tags, Twitter Card tags, canonical URL
- All values driven by env vars with sensible defaults
- Favicon: placeholder SVG in `public/favicon.svg`

---

## 7. File Structure

```
/
├── public/
│   └── favicon.svg
├── src/
│   ├── components/
│   │   ├── Analytics.astro    # GTM injection
│   │   └── SeoMeta.astro      # OG tags, meta
│   ├── config/
│   │   └── analytics.ts       # GTM_ID export
│   ├── layouts/
│   │   └── Base.astro         # HTML shell
│   └── pages/
│       └── index.astro        # SVG loading page + redirect
├── astro.config.mjs
├── wrangler.toml              # Cloudflare Pages config
├── .env                       # defaults
├── .env.production            # production overrides
└── package.json
```

---

## 8. Dependencies

Minimal Astro setup only:
- `astro`
- `@astrojs/cloudflare` (for Cloudflare Pages adapter if needed)
- `wrangler` (dev dependency for deployment)

---

## 9. Build & Deploy

```bash
# Development
npm run dev

# Build
npm run build

# Deploy to Cloudflare Pages
npx wrangler pages deploy dist/
```

---

## 10. Portability Note

The `dist/` output is **pure static files**. To migrate from Cloudflare Pages to GCS or S3:
1. Remove/replace `wrangler.toml`
2. Upload `dist/` contents to the new bucket
3. Configure the bucket for static website hosting (point to `index.html`)
4. No code changes needed
