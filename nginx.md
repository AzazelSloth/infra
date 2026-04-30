# Ubuntu installation

## Initialize

Install the prerequisites:

```bash
sudo apt install curl gnupg2 ca-certificates lsb-release ubuntu-keyring
```

Import an official nginx signing key so apt could verify the packages authenticity. Fetch the key:

```bash
curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor \
    | sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg >/dev/null
```

Verify that the downloaded file contains the proper key:

```bash
gpg --dry-run --quiet --no-keyring --import --import-options import-show /usr/share/keyrings/nginx-archive-keyring.gpg
```

The output should contain the full fingerprint `573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62` as follows:

```bash
pub   rsa2048 2011-08-19 [SC] [expires: 2027-05-24]
      573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62
uid                      nginx signing key <signing-key@nginx.com>
```

Note that the output can contain other keys used to sign the packages.

## Repository setup

To set up the apt repository for stable nginx packages, run the following command:

```bash
echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \
https://nginx.org/packages/ubuntu `lsb_release -cs` nginx" \
    | sudo tee /etc/apt/sources.list.d/nginx.list
```

If you would like to use mainline nginx packages, run the following command instead:

```bash
echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \
https://nginx.org/packages/mainline/ubuntu `lsb_release -cs` nginx" \
    | sudo tee /etc/apt/sources.list.d/nginx.list
```

Set up repository pinning to prefer our packages over distribution-provided ones:

```bash
echo -e "Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900\n" \
    | sudo tee /etc/apt/preferences.d/99nginx
```

## Installation

To install nginx, run the following commands:

```bash
sudo apt update
sudo apt install nginx
```

Doc at: [Install Nginx on Ubuntu](https://nginx.org/en/linux_packages.html#Ubuntu)

---

# Config

Edit `apache-proxy.conf` file by running command:

```bash
nano /etc/nginx/conf.d/rps.conf
```

Then copy the script bellow:

```nginx
upstream rps_backend {
    server 127.0.0.1:3000;
    keepalive 32;
}

upstream rps_frontend {
    server 127.0.0.1:3001;
    keepalive 32;
}

upstream rps_n8n {
    server 127.0.0.1:5678;
    keepalive 16;
}

server {
    listen 127.0.0.1:8786;
    server_name appli.laroche360.ca automation.laroche360.ca;
    server_name_in_redirect off;
    port_in_redirect off;
    absolute_redirect off;
    real_ip_header X-Forwarded-For;
    set_real_ip_from 127.0.0.1;

    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;

    location /.well-known/acme-challenge/ {
        root /var/www/html;
        default_type "text/plain";
        allow all;
    }

    # ========================================
    # n8n UI + API via /n8n/
    # ========================================

    # Normalise /n8n -> /n8n/
    location = /n8n {
        return 301 /n8n/;
    }

    # n8n (editor, rest api, webhook, websocket)
    # Important: ce bloc doit être AVANT location /
    location ^~ /n8n/ {
        proxy_pass http://rps_n8n/;
        proxy_http_version 1.1;

        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $http_x_forwarded_proto;

        proxy_buffering off;
        proxy_request_buffering off;

        proxy_connect_timeout 60s;
        proxy_send_timeout 3600s;
        proxy_read_timeout 3600s;
    }

    # ========================================
    # Swagger UI - Documentation API
    # ========================================
    location ^~ /api-docs {
        proxy_pass http://rps_backend/api-docs;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $http_x_forwarded_proto;
        proxy_buffering off;
        proxy_request_buffering off;
        proxy_read_timeout 60s;
        proxy_connect_timeout 60s;
    }

    # OpenAPI JSON spec
    location /api-docs-json {
        proxy_pass http://rps_backend/api-docs-json;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $http_x_forwarded_proto;
        proxy_buffering off;
        proxy_request_buffering off;
    }

    # Next.js tRPC endpoint must stay on frontend server
    location /api/trpc/ {
        proxy_pass http://rps_frontend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $http_x_forwarded_proto;
        proxy_buffering off;
        proxy_request_buffering off;
        proxy_read_timeout 86400;
    }

    # ========================================
    # Special case: let the frontend handle the temporary-access helper
    # ========================================
    # This must come BEFORE the generic /api/ backend proxy
    location = /api/auth/temporary-access {
        proxy_pass http://rps_frontend;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $http_x_forwarded_proto;
        proxy_buffering off;
        proxy_request_buffering off;
        proxy_read_timeout 60s;
        proxy_connect_timeout 10s;
    }

    # API Backend (NestJS)
    location ^~ /api/ {
        proxy_pass http://rps_backend;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $http_x_forwarded_proto;
        proxy_buffering off;
        proxy_request_buffering off;
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }

    location /health {
        proxy_pass http://rps_backend/api/health;
        access_log off;
    }

    location /_next/static/ {
        proxy_pass http://rps_frontend;
        proxy_cache_valid 30d;
        add_header Cache-Control "public, immutable";
        expires 30d;
        access_log off;
    }

    location ~* \.(jpg|jpeg|png|gif|ico|css|js|svg|woff|woff2|ttf|eot)$ {
        proxy_pass http://rps_frontend;
        proxy_cache_valid 7d;
        add_header Cache-Control "public, max-age=604800";
        expires 7d;
    }

    # API Frontend (Next.js)
    location / {
        proxy_pass http://rps_frontend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $http_x_forwarded_proto;
        proxy_buffering off;
        proxy_request_buffering off;
        proxy_read_timeout 86400;
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
    }

    access_log /var/log/nginx/rps_access.log;
    error_log /var/log/nginx/rps_error.log warn;
}
```
