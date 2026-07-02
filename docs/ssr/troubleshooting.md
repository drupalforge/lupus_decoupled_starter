# Troubleshooting

## Lupus Decoupled Settings Not Visible In Admin

The settings form for these docs is at:

- `/admin/config/system/lupus-decoupled/settings`
- Admin UI: **Configuration -> System -> Lupus Decoupled Drupal**

If you cannot access it:

```bash
# Verify route and module status.
drush pml --status=enabled --type=module | grep lupus_decoupled_ce_api
drush php:eval 'echo \Drupal::service("router.route_provider")->getRouteByName("lupus_decoupled_ce_api.settings_form")->getPath();'

# Rebuild routes/cache.
drush cr
```

If you still do not see the menu item, check user permissions for `administer site configuration`.

## Routes Not Redirecting

**Check settings:**
```bash
drush cget lupus_decoupled_ce_api.settings frontend_base_url
drush cget lupus_decoupled_ce_api.settings frontend_routes_redirect
```

**Clear cache:**
```bash
drush cr
```

**Verify route marking:**
```bash
drush php:eval '$r=\Drupal::service("router.route_provider")->getRouteByName("entity.node.canonical");var_export($r->getOption("_lupus_frontend"));'
```
Expected: `true`

## Redirect Goes To Wrong Host Or Port

If Drupal redirects to `http://localhost:3000` but Nuxt runs in a container, that URL may not be reachable from your browser.

Set `frontend_base_url` to a browser-reachable URL:

- `http://localhost:<mapped-port>` (published port on the host)
- `https://<frontend-host>` (reverse proxy)

```bash
drush cset lupus_decoupled_ce_api.settings frontend_base_url 'http://localhost:<mapped-port>' -y
drush cr
drush cget lupus_decoupled_ce_api.settings frontend_base_url
```

Find mapped ports using your runtime tooling, for example:

```bash
docker port <nuxt-container> 3000
# or with DDEV
ddev describe
```

## Nuxt Can't Reach Drupal

If `frontend` does not exist yet, create a Nuxt app first:

```bash
mkdir -p frontend
test -f frontend/package.json || npx nuxi@latest init frontend
cd frontend && npm install
```

**Check .env:**
```bash
cat frontend/.env
```

**Test connectivity:**
```bash
curl -H "Accept: application/json" http://localhost/ce-api/
```

**Check Nuxt logs:**
```bash
npm run build && PORT=3000 HOST=0.0.0.0 node .output/server/index.mjs  # Look for fetch errors
```

## Infinite Redirect Loop

- Ensure Nuxt has `ssr: true` in `nuxt.config.ts`
- Verify `frontend_base_url` is not pointing to Drupal
- Check catch-all route in `pages/[...slug].vue` exists

## Assets Not Loading

**Check setting:**
```bash
drush cget lupus_decoupled_ce_api.settings absolute_file_urls
```

Should be `true`.
