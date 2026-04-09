# King Power Holding Page

A lightweight, static holding/redirect page built with Astro. Displays a branded SVG loading screen for 5 seconds, then silently redirects to the configured destination URL with UTM parameters preserved.

## Quick Start

```bash
npm install
npm run dev      # Development server at http://localhost:4321
npm run build    # Build static files to dist/
```

## Configuration

All configuration is via environment variables:

| Variable | Default | Description |
|---|---|---|
| `PUBLIC_REDIRECT_URL` | `https://www.kingpower.com` | Redirect destination |
| `PUBLIC_GTM_ID` | `''` | Google Tag Manager ID |
| `PUBLIC_SITE_TITLE` | `King Power` | Page title |
| `PUBLIC_SITE_DESCRIPTION` | `Redirecting to King Power...` | Meta description |

### Environment Files

- `.env` — Default values (committed to repo)
- `.env.production` — Production overrides (gitignored)

## Deploy to Cloudflare Pages

```bash
npm run build
npx wrangler pages deploy dist/
```

Or connect your GitHub repo to Cloudflare Pages for automatic deployments.

---

## Switching to S3 or GCS

The `dist/` folder is pure static HTML/CSS/JS — no code changes needed.

### AWS S3 + CloudFront

1. Build the project: `npm run build`
2. Upload `dist/` contents to an S3 bucket configured for static hosting
3. Example `wrangler.toml` replacement:

```toml
# s3-website.toml (use with aws-cli or terraform)
bucket = "your-bucket-name"
region = "ap-southeast-1"
```

4. Configure your CloudFront distribution to point to the S3 bucket

### Google Cloud Storage (GCS)

1. Build the project: `npm run build`
2. Upload `dist/` contents to a GCS bucket

```bash
gsutil -m rsync -r ./dist gs://your-bucket-name
```

3. Enable static website hosting in GCS bucket permissions
4. Set up Cloud CDN or Load Balancer as needed

### Netlify / Vercel / Other Static Hosts

1. Change the deploy command in your CI/CD:
   - **Netlify**: `npm run build && netlify deploy --prod`
   - **Vercel**: `npm run build && vercel --prod`

No code changes required — only the deployment method changes.

---

## Docker Deployment

A Dockerfile with Nginx is provided for containerized hosting.

### Build & Run

```bash
docker build -t kingpower-holding-page .
docker run -p 8080:80 kingpower-holding-page
```

### Build Arguments

| Argument | Default | Description |
|---|---|---|
| `NGINX_CONF` | `default.conf` | Nginx config file to use |

### Using a Custom Nginx Config

```bash
docker build -t kingpower-holding-page --build-arg NGINX_CONF=custom.conf .
```

### Push to Container Registry

```bash
docker tag kingpower-holding-page:latest your-registry/kingpower-holding-page:latest
docker push your-registry/kingpower-holding-page:latest
```

### Kubernetes / Cloud Run Deployment

For Cloud Run:
```bash
docker build -t gcr.io/your-project/kingpower-holding-page .
docker push gcr.io/your-project/kingpower-holding-page:latest
gcloud run deploy kingpower-holding-page \
  --image gcr.io/your-project/kingpower-holding-page:latest \
  --port 80
```

---

## Project Structure

```
/
├── public/
│   └── favicon.svg          # SVG loading graphic
├── src/
│   ├── components/
│   │   ├── Analytics.astro  # GTM injection
│   │   └── SeoMeta.astro    # OG tags, meta
│   ├── config/
│   │   └── analytics.ts     # GTM ID config
│   ├── layouts/
│   │   └── Base.astro       # HTML shell
│   └── pages/
│       └── index.astro       # Landing page + redirect
├── .env                     # Default config
├── .env.production          # Production config (gitignored)
├── wrangler.toml            # Cloudflare Pages config
└── Dockerfile               # Docker + Nginx
```

---

## Redirect Behavior

1. Page displays SVG loading graphic
2. After 5 seconds, redirects to `PUBLIC_REDIRECT_URL`
3. UTM parameters from the holding page URL are preserved:
   - `?utm_source=foo&utm_medium=bar` → forwarded to destination
4. Primary redirect: HTML `<meta http-equiv="refresh">`
5. Fallback: JavaScript `window.location` for browsers that block meta refresh
