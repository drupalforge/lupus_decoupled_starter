# Setup-Checklist

Use this checklist to verify your SSR setup is complete and working.

## Pre-Setup

- [ ] Drupal 10 or 11 installed
- [ ] PHP 8.2+ available
- [ ] Lupus Decoupled module installed (`composer require drupal/lupus_decoupled`)
- [ ] Node.js 18+ installed
- [ ] npm installed
- [ ] Drush CLI working (`drush status`)

## Drupal Configuration

- [ ] `lupus_decoupled_ce_api` and `lupus_decoupled_cors` are already enabled in this repository
- [ ] Nuxt SSR app is running before setting `frontend_base_url`
- [ ] Configure manually:
  - [ ] `drush cset lupus_decoupled_ce_api.settings frontend_base_url 'http://localhost:<mapped-port>' -y` (or your SSR proxy URL)
  - [ ] `drush cset lupus_decoupled_ce_api.settings frontend_routes_redirect true -y`
  - [ ] `drush cset lupus_decoupled_ce_api.settings absolute_file_urls true -y`
  - [ ] `drush cr` (clear cache)
- [ ] Settings form is reachable at `/admin/config/system/lupus-decoupled/settings`

## Drupal Validation

- [ ] `drush cget lupus_decoupled_ce_api.settings frontend_base_url` returns your SSR URL
- [ ] `drush cget lupus_decoupled_ce_api.settings frontend_routes_redirect` returns `true`
- [ ] Node route marked as frontend
- [ ] Admin route NOT marked as frontend

## Nuxt Setup

- [ ] `frontend/` directory exists (create with `mkdir -p frontend` if needed)
- [ ] Nuxt project exists at `frontend/` (initialize with `npx nuxi@latest init frontend` if missing)
- [ ] `npm install` completed in Nuxt directory
- [ ] `.env` file created (`cp .env.example .env` if present, otherwise create `.env`)
- [ ] `.env` has correct `NUXT_PUBLIC_DRUPAL_BASE_URL`
- [ ] `npm run build && PORT=3000 HOST=0.0.0.0 node .output/server/index.mjs` starts without errors
- [ ] Mapped host port (or proxy URL) identified after Nuxt starts (`docker port <nuxt-container> 3000` or `ddev describe`)
- [ ] SSR URL used in `frontend_base_url` is reachable from browser (mapped port or proxy)

## Functional Testing

- [ ] Test CE API: `curl -H "Accept: application/json" http://localhost/ce-api/`
- [ ] Test Nuxt SSR: `curl http://localhost:<mapped-port>/node/1` (or proxy URL) returns HTML
- [ ] Browser test: `/node/1` redirects to your configured SSR URL
- [ ] Admin pages don't redirect: `/admin/content` stays on Drupal

## Production Readiness

- [ ] Reverse proxy config planned
- [ ] Process manager chosen (PM2 or systemd)
- [ ] Domain/SSL plan documented
- [ ] Cache/revalidation strategy planned
