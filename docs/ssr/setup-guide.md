# Enabling Server-Side Rendering for Drupal CMS with Nuxt

This guide explains how to enable server-side rendering (SSR) for a Drupal CMS site powered by [Lupus Decoupled](https://www.drupal.org/project/lupus_decoupled) using a Nuxt 3 frontend. SSR improves SEO, performance, and user experience compared to client-side rendering.

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Architecture](#architecture)
4. [Quick Start](#quick-start)
5. [Manual Configuration](#manual-configuration)
6. [Nuxt SSR Frontend Setup](#nuxt-ssr-frontend-setup)
7. [Validation & Testing](#validation--testing)
8. [Production Deployment](#production-deployment)
9. [Customization](#customization)

---

## Overview

### What This Enables

- **Server-side rendering**: Pages render on the server before being sent to the browser.
- **SEO-friendly**: Search engines receive fully rendered HTML, improving crawlability and indexing.
- **Decoupled architecture**: Drupal handles content management and APIs; Nuxt handles user-facing rendering.
- **Unified Content**: Both frontend and backend consume the same CE API from Drupal.

### Rendering Modes Explained

| Mode | How It Works | Use Case |
|------|-------------|----------|
| **CSR (Client-Side)** | Browser downloads empty HTML + JS; JS renders pages | Development, modern browsers, real-time apps |
| **SSR (Server-Side)** | Server renders pages; sends complete HTML to browser | Production sites, SEO-heavy content, slower networks |
| **Hybrid** | SSR for initial load; CSR for subsequent navigation | Optimal performance + SEO |

This guide enables **SSR** mode, suitable for content sites and public-facing pages.

---

## Prerequisites

### System Requirements

- Drupal 10 or 11
- PHP 8.2+
- [Lupus Decoupled](https://www.drupal.org/project/lupus_decoupled) v1.5+
- Node.js 18+ (for running Nuxt SSR server)
- npm or yarn

### Project Structure

Your Drupal site should already have:
- `drupal/lupus_decoupled` and submodules installed
- `drupal/lupus_decoupled_recipe` available (if using recipes)
- Access to the Drupal admin panel
- Drush CLI configured

### Knowledge Assumptions

- Familiarity with Drupal site structure
- Basic understanding of REST APIs
- Command-line comfort

---

## Architecture

### Data Flow

```
┌─────────────────┐
│  Browser        │
│  (User)         │
└────────┬────────┘
         │ HTTP GET /node/1
         │
    ┌────v──────────────────┐
    │   Nuxt SSR Server     │
    │ (Port 3000, default)  │
    │                       │
    │ Fetch /ce-api/node/1  │
    └────┬──────────────────┘
         │
    ┌────v──────────────────┐
    │   Drupal Backend      │
    │   /ce-api endpoint    │
    │   (Returns JSON)      │
    └──────────────────────┘
```

### Request Flow

1. Browser requests `/node/1` to Drupal.
2. Drupal route is marked `_lupus_frontend: TRUE`.
3. Drupal redirects to your configured frontend URL (Nuxt SSR).
4. Nuxt intercepts the request on the server.
5. Nuxt fetches `/ce-api/node/1` from Drupal.
6. Nuxt renders the page server-side.
7. Browser receives fully rendered HTML + JavaScript for hydration.

### Key Configuration Files

| Component | Config | Purpose |
|-----------|--------|---------|
| Drupal | `lupus_decoupled_ce_api.settings` | Frontend URL, redirect behavior |
| Nuxt | `nuxt.config.ts` | SSR mode, runtime config |
| Nuxt | `.env` | Drupal base URL, CE API prefix |
| Routes | `lupus_decoupled_ce_api.frontend_paths` | Which paths redirect to frontend |

---

## Quick Start

Use this approach for this repository and most local environments.

### 1. Set Up the Nuxt Frontend

The `frontend` directory is a documented convention and may not exist yet.
Create it if needed, then clone or initialize a Nuxt 3 SSR app:

```bash
mkdir -p frontend

# If you have a starter repo:
git clone <your-nuxt-ssr-repo> frontend
cd frontend

# Or start fresh:
npx nuxi@latest init frontend
cd frontend
```

### 2. Configure Nuxt Environment

```bash
test -f .env.example && cp .env.example .env || touch .env
```

Edit `.env`:

```env
NUXT_PUBLIC_DRUPAL_BASE_URL=http://localhost
NUXT_PUBLIC_CE_API_PREFIX=/ce-api
```

### 3. Install & Run

```bash
npm install
npm run build && PORT=3000 HOST=0.0.0.0 node .output/server/index.mjs
```

### 4. Find Browser-Reachable Frontend URL

Pick a frontend URL that is reachable from the browser:

- Mapped host port: `http://localhost:<mapped-port>`
- Reverse proxy URL: `https://<frontend-host>`

Find the mapped port only after Nuxt is running:

```bash
docker port <nuxt-container> 3000
# or with DDEV
ddev describe
```

If Nuxt runs inside a container, `http://localhost:3000` is often not the correct browser URL.

### 5. Configure Drupal SSR Settings

```bash
drush cset lupus_decoupled_ce_api.settings frontend_base_url 'http://localhost:<mapped-port>' -y
drush cset lupus_decoupled_ce_api.settings frontend_routes_redirect true -y
drush cset lupus_decoupled_ce_api.settings absolute_file_urls true -y
drush cr
```

Drupal admin UI equivalent: `/admin/config/system/lupus-decoupled/settings`
(Configuration -> System -> Lupus Decoupled Drupal).

### 6. Export Configuration

```bash
drush cr
drush cex -y
```

This persists settings in `config/sync/` for version control.

### 7. Validate

Test these URLs in your browser:

- **Frontend content**: `http://default/node/1` → redirects to your configured SSR URL
- **Admin pages**: `http://default/admin/content` → stays on Drupal (no redirect)
- **API endpoint**: `http://default/ce-api/node/1` → returns JSON from Drupal

---

## Manual Configuration

Use this approach if you prefer explicit configuration or need to customize the setup.

### 1. Install Lupus Decoupled (if not present)

```bash
composer require drupal/lupus_decoupled
```

In this repository, `lupus_decoupled_ce_api` and `lupus_decoupled_cors` are already enabled.

### 2. Configure SSR Settings via Drush

Before setting `frontend_base_url`, make sure your frontend is running and use its browser-reachable URL (mapped host port or proxy URL).

```bash
# Set frontend URL
drush cset lupus_decoupled_ce_api.settings frontend_base_url 'http://localhost:<mapped-port>' -y

# Enable redirect to frontend
drush cset lupus_decoupled_ce_api.settings frontend_routes_redirect true -y

# Enable absolute file URLs (for frontend asset linking)
drush cset lupus_decoupled_ce_api.settings absolute_file_urls true -y

# Set preview provider for editor (optional)
drush cset lupus_decoupled_ce_api.settings preview_provider nuxt -y

# Clear cache
drush cr
```

### 3. Verify Configuration

```bash
drush cget lupus_decoupled_ce_api.settings frontend_base_url
drush cget lupus_decoupled_ce_api.settings frontend_routes_redirect
```

Expected output:

```
'lupus_decoupled_ce_api.settings:frontend_base_url': 'http://localhost:<mapped-port>'
'lupus_decoupled_ce_api.settings:frontend_routes_redirect': true
```

### 4. Check Route Marking

Verify that frontend routes are correctly marked:

```bash
drush php:eval '
$route = \Drupal::service("router.route_provider")->getRouteByName("entity.node.canonical");
print "Node route frontend flag: ";
var_export($route->getOption("_lupus_frontend"));
'
```

Expected output: `true`

### 5. Verify Non-Frontend Routes

```bash
drush php:eval '
$route = \Drupal::service("router.route_provider")->getRouteByName("system.admin_content");
print "Admin route frontend flag: ";
var_export($route->getOption("_lupus_frontend"));
'
```

Expected output: `NULL` (no redirect for admin routes)

---

## Nuxt SSR Frontend Setup

### Minimal Starter Structure

Create a `nuxt.config.ts` with SSR enabled:

```typescript
export default defineNuxtConfig({
  compatibilityDate: "2025-01-01",
  ssr: true,  // Enable server-side rendering
  devtools: { enabled: true },
  runtimeConfig: {
    public: {
      drupalBaseUrl: process.env.NUXT_PUBLIC_DRUPAL_BASE_URL || "http://localhost",
      ceApiPrefix: process.env.NUXT_PUBLIC_CE_API_PREFIX || "/ce-api",
    },
  },
});
```

### Catch-All Page Route

Create `pages/[...slug].vue`:

```vue
<script setup lang="ts">
const route = useRoute();
const runtimeConfig = useRuntimeConfig();

const normalizedPath = computed(() => {
  const path = route.path || "/";
  return path === "" ? "/" : path;
});

const ceApiUrl = computed(() => {
  const path = normalizedPath.value === "/" ? "/" : normalizedPath.value;
  return new URL(
    `${runtimeConfig.public.ceApiPrefix}${path}`,
    runtimeConfig.public.drupalBaseUrl
  ).toString();
});

const { data, error } = await useAsyncData(
  () => `ce-api:${normalizedPath.value}`,
  () =>
    $fetch(ceApiUrl.value, {
      headers: { accept: "application/json" },
    }),
  { watch: [normalizedPath] }
);

// Handle CE API redirects
if ((data.value as { redirect?: { url?: string } } | null)?.redirect?.url) {
  await navigateTo(
    (data.value as { redirect: { url: string } }).redirect.url,
    { external: true, redirectCode: 302 }
  );
}
</script>

<template>
  <main>
    <h1>Page Content</h1>
    <pre v-if="data">{{ JSON.stringify(data, null, 2) }}</pre>
    <p v-if="error">Error: {{ error.message }}</p>
  </main>
</template>
```

### Environment Configuration

Create `.env.example`:

```env
# Drupal backend URL
NUXT_PUBLIC_DRUPAL_BASE_URL=http://localhost

# CE API endpoint prefix
NUXT_PUBLIC_CE_API_PREFIX=/ce-api
```

Copy and customize for your environment:

```bash
cp .env.example .env
```

### Integrate Custom Elements Renderer

Replace the JSON `<pre>` renderer above with your production component renderer:

```vue
<script setup lang="ts">
// Import your CE renderer integration
import { CustomElementRenderer } from '@your-org/nuxt-ce-renderer';
import { defineAsyncComponent } from 'vue';

const CustomElements = defineAsyncComponent(() =>
  import('@your-org/nuxt-ce-renderer')
);
</script>

<template>
  <CustomElements :payload="data" v-if="data" />
  <p v-else-if="error">Error loading page: {{ error.message }}</p>
  <p v-else>Loading...</p>
</template>
```

---

## Validation & Testing

### 1. Check Drupal Configuration

```bash
drush cget lupus_decoupled_ce_api.settings
```

Should show:

```yaml
frontend_base_url: 'http://localhost:<mapped-port>'
frontend_routes_redirect: true
absolute_file_urls: true
preview_provider: nuxt
```

### 2. Verify Route Redirection (Internal Check)

```bash
drush php:eval '
$node = \Drupal::entityTypeManager()->getStorage("node")->load(1);
if ($node) {
  $url = $node->toUrl("canonical", ["absolute" => TRUE])->toString();
  print "Node URL: " . $url;
}
'
```

Expected output: `Node URL: http://localhost:<mapped-port>/node/1`

### 3. Verify Admin Routes Stay Local

```bash
drush php:eval '
$route = \Drupal::service("router.route_provider")->getRouteByName("user.login");
print "Login route frontend flag: ";
var_export($route->getOption("_lupus_frontend"));
'
```

Expected output: `NULL` (no frontend redirect)

### 4. Test CE API Directly

```bash
# From Drupal container/host
curl -H "Accept: application/json" http://localhost/ce-api/node/1 | jq .
```

Should return JSON custom elements payload.

### 5. Test SSR Rendering

```bash
# From Nuxt host
curl http://localhost:<mapped-port>/node/1
```

Should return fully rendered HTML with content from Drupal CE API.

### 6. Browser Test

1. Open `http://localhost/` in a browser.
2. Click a link to public content (e.g., a node).
3. Verify the URL redirects to your configured SSR URL.
4. Open browser DevTools → Network.
5. Check that the response includes server-rendered HTML (not just `<div id="app"></div>`).

---

## Production Deployment

### Infrastructure Setup

#### Option A: Separate Servers

```
┌──────────────────┐         ┌──────────────────┐
│  Drupal Server   │         │  Nuxt SSR Server │
│  (Port 80/443)   │◄───────►│  (Port 3000)     │
└──────────────────┘         └──────────────────┘
```

**Pros**: Easy to scale, separate concerns.
**Cons**: More infrastructure overhead.

#### Option B: Reverse Proxy (Same Host)

```
           Public IP
                │
        ┌───────┴────────┐
        │                │
    ┌───v─────┐      ┌──v────┐
    │Drupal   │      │Nuxt   │
    │:8000    │      │:3000  │
    └─────────┘      └───────┘
        ▲                ▲
        └────────┬───────┘
                 │
          Nginx/Caddy
          (Port 80/443)
```

**Pros**: Single server, easier to manage.
**Cons**: Need reverse proxy configuration.

### Nginx Configuration Example

```nginx
upstream drupal {
    server localhost:8000;
}

upstream nuxt {
    server localhost:3000;
}

server {
    listen 80;
    server_name example.com;

    # Redirect non-www to www (optional)
    if ($host !~* ^www\.) {
        return 301 https://www.$server_name$request_uri;
    }

    # API and admin routes → Drupal
    location ~ ^/(ce-api|admin|user/login|user/register) {
        proxy_pass http://drupal;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Everything else → Nuxt SSR
    location / {
        proxy_pass http://nuxt;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Drupal Backend URL Configuration

Update `lupus_decoupled_ce_api.settings` for production:

```bash
drush cset lupus_decoupled_ce_api.settings frontend_base_url 'https://example.com' -y
drush cr
drush cex -y
```

### Process Management for Nuxt

#### PM2 (Recommended)

Install PM2 globally:

```bash
npm install -g pm2
```

Create `ecosystem.config.js` in your Nuxt project root:

```javascript
module.exports = {
  apps: [
    {
      name: "nuxt-ssr",
      script: "./.output/server/index.mjs",
      env: {
        NODE_ENV: "production",
        NUXT_PUBLIC_DRUPAL_BASE_URL: "https://example.com",
        NUXT_PUBLIC_CE_API_PREFIX: "/ce-api",
        PORT: 3000,
      },
      instances: 2,
      exec_mode: "cluster",
      restart_delay: 5000,
      error_file: "./logs/err.log",
      out_file: "./logs/out.log",
    },
  ],
};
```

Start the app:

```bash
pm2 start ecosystem.config.js
pm2 save
pm2 startup
```

#### Systemd Service

Create `/etc/systemd/system/nuxt-ssr.service`:

```ini
[Unit]
Description=Nuxt SSR Frontend for Drupal
After=network.target

[Service]
Type=simple
User=www-data
WorkingDirectory=/var/www/html/frontend
Environment="NUXT_PUBLIC_DRUPAL_BASE_URL=https://example.com"
Environment="NUXT_PUBLIC_CE_API_PREFIX=/ce-api"
Environment="NODE_ENV=production"
ExecStart=/usr/bin/node ./.output/server/index.mjs
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

Enable and start:

```bash
sudo systemctl enable nuxt-ssr
sudo systemctl start nuxt-ssr
sudo systemctl status nuxt-ssr
```

### SSL/TLS

Use Let's Encrypt with Certbot:

```bash
sudo certbot certonly --webroot -w /var/www/letsencrypt -d example.com -d www.example.com
```

Update Nginx to use the certificate and configure SSL redirects.

---

## Customization

### Customizing Frontend Paths

By default, only specific paths redirect to Nuxt. To add or remove paths, modify the service parameter in `web/modules/contrib/lupus_decoupled/modules/lupus_decoupled_ce_api/lupus_decoupled_ce_api.services.yml`:

```yaml
parameters:
  lupus_decoupled_ce_api.frontend_paths:
    - '/'
    - '/node'
    - '/node/{node}'
    - '/custom-page'  # Add custom paths here
    - '/blog/{slug}'
```

Then rebuild routes:

```bash
drush cr
```

### Implementing Custom Component Rendering

Replace the JSON renderer in `pages/[...slug].vue` with your custom elements integration:

```vue
<script setup lang="ts">
import { renderCustomElements } from '@your-org/ce-renderer';

const { data } = await useAsyncData(() => fetchFromCeApi(...));
const renderedHtml = computed(() => 
  data.value ? renderCustomElements(data.value) : ''
);
</script>

<template>
  <div v-html="renderedHtml"></div>
</template>
```

### Adding Middleware for Authentication

Protect certain routes from SSR redirect:

```bash
drush cset lupus_decoupled_ce_api.settings frontend.keep_frontend_paths '[/user, /user/{user}]' -y
```

This prevents `/user` routes from being marked as frontend routes, keeping them on Drupal for server-side authentication.

### Incremental Static Regeneration (ISR)

Add revalidation endpoints to Nuxt for cache invalidation:

```typescript
// server/api/revalidate.ts
export default defineEventHandler(async (event) => {
  const secret = useRuntimeConfig().revalidateSecret;
  const query = getQuery(event);

  if (query.secret !== secret) {
    return { revalidated: false };
  }

  // Revalidate paths on Drupal content update
  return { revalidated: true };
});
```

Configure webhook in Drupal to call this endpoint on content save.

### Dark Mode / Multi-Region Support

Access Drupal settings in Nuxt:

```typescript
const { data: settings } = await useAsyncData(() =>
  $fetch('/ce-api/api/system-settings')
);

const isDarkMode = computed(() => settings.value?.theme?.dark);
const region = computed(() => settings.value?.region);
```

---

## Resources

- [Lupus Decoupled Documentation](https://lupus-decoupled.org/)
- [Nuxt 3 SSR Guide](https://nuxt.com/docs/guide/ssr/deployment)
- [Drupal Custom Elements](https://www.drupal.org/project/custom_elements)
- [REST API for Menus](https://www.drupal.org/project/rest_menu_items)

---

## Next Steps

1. **Customize Components**: Replace the JSON renderer with your production component library.
2. **Add Caching**: Implement ISR or static generation for high-traffic pages.
3. **Monitor Performance**: Use tools like Lighthouse, WebPageTest, and Drupal's built-in performance profiling.
4. **Set Up CI/CD**: Automate Drupal config export, Nuxt builds, and deployments.
5. **Content Preview**: Enable editor preview by configuring the preview provider in Canvas.
