# SSR with Nuxt for Drupal CMS - Quick Start

Welcome! This page has everything you need to enable server-side rendering for your Drupal CMS site using Nuxt 3.

## TL;DR (3-Minute Setup)

```bash
# 1. Create the frontend app (if it does not exist yet)
mkdir -p frontend
test -f frontend/package.json || npx nuxi@latest init frontend
cd frontend && npm install && npm run build && PORT=3000 HOST=0.0.0.0 node .output/server/index.mjs

# 2. Find the browser-reachable frontend URL
# docker port <nuxt-container> 3000
# ddev describe

# 3. Configure Drupal SSR settings
drush cset lupus_decoupled_ce_api.settings frontend_base_url 'http://localhost:<mapped-port>' -y
drush cset lupus_decoupled_ce_api.settings frontend_routes_redirect true -y
drush cset lupus_decoupled_ce_api.settings absolute_file_urls true -y
drush cr

# 4. Visit Drupal and click a page link
# → Should redirect to your SSR URL
```

**Done!** Public pages now render on your Nuxt SSR frontend while Drupal handles content management.

---

## What Is This?

- **SSR (Server-Side Rendering)**: Pages render on a server, improving SEO and performance.
- **Decoupled**: Drupal is your content API; Nuxt is your frontend.
- **Drupal CMS**: Built on [Lupus Decoupled](https://www.drupal.org/project/lupus_decoupled).
- **Nuxt 3**: Modern Vue-based framework with SSR support.

## Key Pages

| Page | Purpose |
|------|---------|
| **[Setup Guide](./ssr/setup-guide.md)** | Complete step-by-step instructions |
| **[Quick Start](./ssr/quick-start.md)** | 3-5 minute setup for experienced users |
| **[Checklist](./ssr/checklist.md)** | Validation & testing steps |
| **[Troubleshooting](./ssr/troubleshooting.md)** | Common issues & fixes |
| **[Production Deployment](./ssr/production-deployment.md)** | Nginx, PM2, SSL, and scaling |

## Installation Paths

## Notes Before You Start

- `frontend` is the recommended location used in this documentation. It is not created automatically in this repository.
- Drupal settings are at `/admin/config/system/lupus-decoupled/settings` (Admin UI: **Configuration -> System -> Lupus Decoupled Drupal**).
- If you run Nuxt in a container, do not assume `http://localhost:3000` will be reachable from your browser or other containers. Use a browser-reachable URL for `frontend_base_url`:
	- mapped host port: `http://localhost:<mapped-port>`
	- reverse proxy URL: `https://<frontend-host>`
- Find the mapped port after the frontend is running (for example `docker port <nuxt-container> 3000`, or `ddev describe` when using DDEV).
- Run the frontend with `npm run build && PORT=3000 HOST=0.0.0.0 node .output/server/index.mjs`.

### Manual Setup (Recommended)

```bash
# Nuxt
mkdir -p frontend
test -f frontend/package.json || npx nuxi@latest init frontend
cd frontend && npm install && npm run build && PORT=3000 HOST=0.0.0.0 node .output/server/index.mjs

# Discover browser-reachable SSR URL
# docker port <nuxt-container> 3000
# ddev describe

# Drupal
drush cset lupus_decoupled_ce_api.settings frontend_base_url 'http://localhost:<mapped-port>' -y
drush cset lupus_decoupled_ce_api.settings frontend_routes_redirect true -y
drush cset lupus_decoupled_ce_api.settings absolute_file_urls true -y
drush cr
```

Self-contained setup with no repo-local helper scripts required.

This repository already has `lupus_decoupled_ce_api` and `lupus_decoupled_cors` enabled.

### Minimal Update (If module is already enabled)

```bash
# Nuxt
cd frontend && npm install && npm run build && PORT=3000 HOST=0.0.0.0 node .output/server/index.mjs

# Discover browser-reachable SSR URL
# docker port <nuxt-container> 3000
# ddev describe

# Drupal
drush cset lupus_decoupled_ce_api.settings frontend_base_url 'http://localhost:<mapped-port>' -y
drush cset lupus_decoupled_ce_api.settings frontend_routes_redirect true -y
drush cr
```

See **[Setup Guide](./ssr/setup-guide.md)** for details.

## How It Works (30 Seconds)

1. Browser requests `/node/1`
2. Drupal redirects to Nuxt SSR server
3. Nuxt fetches content from Drupal CE API
4. Nuxt renders page on server
5. Browser gets complete HTML + interactive JS

[See detailed architecture ->](./ssr/setup-guide.md#architecture)

## FAQ

**Q: Do I rewrite my Drupal code?**
A: No. Drupal works unchanged. You only add a Nuxt frontend.

**Q: Will admin pages redirect to SSR?**
A: No. Admin routes stay on Drupal automatically.

**Q: Can I use this in production?**
A: Yes. See [Production Deployment](./ssr/production-deployment.md) guide.
