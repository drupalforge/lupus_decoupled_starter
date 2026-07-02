# Quick-Start

Quick setup for experienced developers.

## Important Assumptions

- `frontend` is the convention used in these docs, not a pre-existing folder.
- The settings form is at `/admin/config/system/lupus-decoupled/settings` (Configuration -> System -> Lupus Decoupled Drupal).
- Set `frontend_base_url` to a browser-reachable SSR URL. In containers, this is often a mapped host port (`http://localhost:<mapped-port>`) or a proxy URL.

## 4-Step Setup

### 1. Set Up Nuxt

```bash
mkdir -p frontend
test -f frontend/package.json || npx nuxi@latest init frontend
cd frontend
npm install
test -f .env.example && cp .env.example .env || touch .env
npm run build && PORT=3000 HOST=0.0.0.0 node .output/server/index.mjs
```

### 2. Find Browser-Reachable SSR URL

```bash
docker port <nuxt-container> 3000
# or with DDEV
ddev describe
```

Use the mapped host port or your proxy URL for the next step.

### 3. Configure Drupal

```bash
drush cset lupus_decoupled_ce_api.settings frontend_base_url 'http://localhost:<mapped-port>' -y
drush cset lupus_decoupled_ce_api.settings frontend_routes_redirect true -y
drush cr
```

### 4. Test

```bash
curl http://localhost:<mapped-port>/node/1  # Should return HTML
# In browser: http://localhost/node/1 should redirect to SSR
```
