# Nginx

Nginx sert ici de reverse proxy interne entre Apache2 et les services Node/n8n.

## Installation

```bash
sudo apt update
sudo apt install nginx
```

## Configuration

Creer le fichier :

```bash
sudo nano /etc/nginx/conf.d/rps.conf
```

```nginx
map $http_upgrade $connection_upgrade {
    default upgrade;
    '' close;
}

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
    client_max_body_size 20m;

    set_real_ip_from 127.0.0.1;
    real_ip_header X-Forwarded-For;

    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;

    location /.well-known/acme-challenge/ {
        root /var/www/html;
        default_type text/plain;
        allow all;
    }

    location = /n8n {
        return 301 /n8n/;
    }

    location ^~ /n8n/ {
        proxy_pass http://rps_n8n;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_buffering off;
        proxy_request_buffering off;
        proxy_read_timeout 3600;
        proxy_send_timeout 3600;
    }

    location ^~ /api-docs {
        proxy_pass http://rps_backend/api-docs;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_buffering off;
    }

    location /api-docs-json {
        proxy_pass http://rps_backend/api-docs-json;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_buffering off;
    }

    location /api/trpc/ {
        proxy_pass http://rps_frontend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_buffering off;
        proxy_read_timeout 86400;
    }

    location = /api/auth/temporary-access {
        proxy_pass http://rps_frontend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_buffering off;
    }

    location ^~ /api/ {
        proxy_pass http://rps_backend;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_buffering off;
        proxy_read_timeout 60s;
        proxy_send_timeout 60s;
    }

    location /health {
        proxy_pass http://rps_backend/api/health;
        access_log off;
    }

    location /_next/static/ {
        proxy_pass http://rps_frontend;
        add_header Cache-Control "public, immutable";
        expires 30d;
        access_log off;
    }

    location ~* \.(jpg|jpeg|png|gif|ico|css|js|svg|woff|woff2|ttf|eot)$ {
        proxy_pass http://rps_frontend;
        add_header Cache-Control "public, max-age=604800";
        expires 7d;
        access_log off;
    }

    location / {
        proxy_pass http://rps_frontend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_buffering off;
        proxy_read_timeout 86400;
    }

    access_log /var/log/nginx/rps_access.log;
    error_log /var/log/nginx/rps_error.log warn;
}
```

## Validation

```bash
sudo nginx -t
sudo systemctl enable --now nginx
sudo systemctl reload nginx
ss -tulpn | grep -E ':8786|:3000|:3001|:5678'
```

## Recommandations

- Gardez Nginx en ecoute locale seulement sur `127.0.0.1:8786`.
- Les applications backend/frontend doivent aussi n'ecouter qu'en local si possible.
