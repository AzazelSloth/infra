# Apache2

Apache2 joue ici le role d'entree publique sur `80/443` et reverse-proxy vers Nginx en local sur `127.0.0.1:8786`.

## Modules a activer

```bash
sudo a2enmod ssl rewrite headers proxy proxy_http proxy_wstunnel remoteip http2
sudo systemctl restart apache2
```

## VirtualHost recommande

Creer le fichier `rps.conf` :

```bash
sudo nano /etc/apache2/sites-available/rps.conf
```

```apache
<VirtualHost *:80>
    ServerName appli.laroche360.ca
    ServerAlias automation.laroche360.ca

    RewriteEngine On
    RewriteCond %{REQUEST_URI} !^/\.well-known/acme-challenge/
    RewriteRule ^ https://%{HTTP_HOST}%{REQUEST_URI} [R=301,L]

    ErrorLog ${APACHE_LOG_DIR}/rps-http-error.log
    CustomLog ${APACHE_LOG_DIR}/rps-http-access.log combined
</VirtualHost>

<IfModule mod_ssl.c>
<VirtualHost *:443>
    ServerName appli.laroche360.ca
    ServerAlias automation.laroche360.ca

    Protocols h2 http/1.1
    SSLEngine on
    SSLProtocol TLSv1.2 TLSv1.3

    SSLCertificateFile /etc/letsencrypt/live/appli.laroche360.ca/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/appli.laroche360.ca/privkey.pem

    Header always set X-Content-Type-Options "nosniff"
    Header always set X-Frame-Options "SAMEORIGIN"
    Header always set Referrer-Policy "strict-origin-when-cross-origin"

    ProxyPreserveHost On
    ProxyRequests Off
    SSLProxyEngine On
    ProxyTimeout 3600
    Timeout 3600

    ProxyPass /.well-known/acme-challenge/ !

    RewriteEngine On
    RewriteCond %{HTTP:Upgrade} =websocket [NC]
    RewriteRule /(.*) ws://127.0.0.1:8786/$1 [P,L]

    ProxyPass / http://127.0.0.1:8786/
    ProxyPassReverse / http://127.0.0.1:8786/

    RequestHeader set X-Forwarded-Proto "https"
    RequestHeader append X-Forwarded-For %{REMOTE_ADDR}s
    RequestHeader set X-Real-IP %{REMOTE_ADDR}s
    RequestHeader set X-Forwarded-Host %{HTTP_HOST}s
    RequestHeader set X-Forwarded-Port %{SERVER_PORT}s

    ErrorLog ${APACHE_LOG_DIR}/rps-https-error.log
    CustomLog ${APACHE_LOG_DIR}/rps-https-access.log combined
</VirtualHost>
</IfModule>
```

## Activation

```bash
sudo a2ensite rps.conf
sudo apache2ctl configtest
sudo apache2ctl -M | grep -E 'ssl|proxy|proxy_http|proxy_wstunnel|rewrite|headers|remoteip|http2'
sudo systemctl reload apache2
```

## Points d'attention

- Apache doit rester le seul service expose sur `80` et `443`.
- Nginx doit ecouter uniquement sur `127.0.0.1:8786`.
- Si vous activez HSTS, faites-le seulement apres validation complete du HTTPS sur tous les sous-domaines.
