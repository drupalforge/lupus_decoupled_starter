# Production-Deployment

## Nginx Reverse Proxy

```nginx
upstream drupal {
    server localhost:8000;
}

upstream nuxt {
    server localhost:3000;
}

server {
    listen 80;
    server_name example.com www.example.com;

    location ~ ^/(ce-api|admin|user) {
        proxy_pass http://drupal;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location / {
        proxy_pass http://nuxt;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## PM2 Process Manager

Create `ecosystem.config.js`:

```javascript
module.exports = {
  apps: [{
    name: "nuxt-ssr",
    script: "./.output/server/index.mjs",
    env: {
      NODE_ENV: "production",
      NUXT_PUBLIC_DRUPAL_BASE_URL: "https://example.com",
      PORT: 3000,
    },
    instances: 2,
    exec_mode: "cluster",
  }],
};
```

Start:
```bash
pm2 start ecosystem.config.js
```

## Systemd Service

Create `/etc/systemd/system/nuxt-ssr.service`:

```ini
[Unit]
Description=Nuxt SSR Frontend
After=network.target

[Service]
Type=simple
User=www-data
WorkingDirectory=/var/www/html/frontend
Environment="NUXT_PUBLIC_DRUPAL_BASE_URL=https://example.com"
ExecStart=/usr/bin/node ./.output/server/index.mjs
Restart=always

[Install]
WantedBy=multi-user.target
```

Enable:
```bash
sudo systemctl enable nuxt-ssr
sudo systemctl start nuxt-ssr
```

## SSL/TLS

```bash
sudo certbot certonly --webroot -w /var/www/letsencrypt -d example.com
```

Update Nginx config to use certificates.
