# Apache HTTP Server (`httpd`)

Oui, la meme logique de reverse proxy s'applique a `httpd`. Ce qui change surtout, ce sont les commandes, l'emplacement des fichiers et la facon d'activer les modules.

Dans cette architecture, `httpd` reste l'entree publique sur `80/443` et reverse-proxy vers Nginx en local sur `127.0.0.1:8786`.

## Ce qui change par rapport a `apache2`

- Service : `httpd` au lieu de `apache2`
- Test de configuration : `apachectl configtest` ou `httpd -t`
- Liste des modules : `httpd -M`
- Pas de `a2enmod`
- Pas de `sites-available` / `a2ensite`
- Fichier de vhost generalement dans `/etc/httpd/conf.d/`

Dans votre environnement actuel, la variante observee est differente :

- binaire de controle : `httpd` et `apachectl`
- vhost charge depuis `/etc/apache2/conf.d/rps.conf`
- logs Apache dans `/var/log/apache2/`

Les exemples ci-dessous utilisent donc cette variante, qui correspond a ce que vous avez sur le VPS.

## Modules a verifier

Avec `httpd`, les modules sont en general deja charges via `conf.modules.d`. Verifiez simplement leur presence :

```bash
sudo httpd -M | grep -E 'ssl|proxy|proxy_http|proxy_wstunnel|rewrite|headers|remoteip|http2'
```

Si un module manque, il faut l'activer dans la config du systeme ou installer le paquet correspondant selon votre distribution.

## VirtualHost recommande

Dans votre environnement actuel, creez :

```bash
sudo nano /etc/apache2/conf.d/rps.conf
```

## Configuration HTTP seulement

Utilisez cette version tant que les certificats SSL/TLS n'ont pas encore ete emis. Elle permet :

- de valider Apache sans erreur de certificat manquant
- de laisser le port `80` disponible pour Certbot
- de tester le reverse proxy avant d'activer HTTPS

```apache
<VirtualHost *:80>
    ServerName appli.laroche360.ca
    ServerAlias automation.laroche360.ca

    ProxyPreserveHost On
    ProxyRequests Off
    ProxyTimeout 3600
    Timeout 3600

    RewriteEngine On
    RewriteCond %{HTTP:Upgrade} =websocket [NC]
    RewriteRule /(.*) ws://127.0.0.1:8786/$1 [P,L]

    ProxyPass / http://127.0.0.1:8786/
    ProxyPassReverse / http://127.0.0.1:8786/

    RequestHeader set X-Forwarded-Proto "http"
    RequestHeader append X-Forwarded-For %{REMOTE_ADDR}s
    RequestHeader set X-Real-IP %{REMOTE_ADDR}s
    RequestHeader set X-Forwarded-Host %{HTTP_HOST}s
    RequestHeader set X-Forwarded-Port %{SERVER_PORT}s

    ErrorLog /var/log/apache2/rps-http-error.log
    CustomLog /var/log/apache2/rps-http-access.log combined
</VirtualHost>
```

Verification :

```bash
sudo apachectl configtest
sudo systemctl restart httpd
```

## Configuration HTTP + HTTPS

Une fois les certificats presents dans `/etc/letsencrypt/live/appli.laroche360.ca/`, remplacez le contenu precedent par cette version.

Le bloc `:80` garde l'acces au challenge ACME puis redirige le reste vers HTTPS.

```apache
<VirtualHost *:80>
    ServerName appli.laroche360.ca
    ServerAlias automation.laroche360.ca

    RewriteEngine On
    RewriteCond %{REQUEST_URI} !^/\.well-known/acme-challenge/
    RewriteRule ^ https://%{HTTP_HOST}%{REQUEST_URI} [R=301,L]

    ErrorLog /var/log/apache2/rps-http-error.log
    CustomLog /var/log/apache2/rps-http-access.log combined
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

    ErrorLog /var/log/apache2/rps-https-error.log
    CustomLog /var/log/apache2/rps-https-access.log combined
</VirtualHost>
</IfModule>
```

Verification :

```bash
sudo httpd -t
sudo systemctl restart httpd
```

## Activation

```bash
sudo httpd -t
sudo systemctl restart httpd
sudo systemctl status httpd
```

## Cas special cPanel / WHM

Si votre serveur WHM/cPanel pilote Apache, n'editez pas le vhost principal a la main dans `/etc/httpd/conf.d/` ou `/etc/apache2/conf/httpd.conf`, car cPanel peut le regenerer.

Dans ce cas, utilisez plutot un include de vhost. Les chemins typiques sont :

- HTTP : `/etc/apache2/conf.d/userdata/std/2_4/<user>/<domaine>/rps.conf`
- HTTPS : `/etc/apache2/conf.d/userdata/ssl/2_4/<user>/<domaine>/rps.conf`

Exemple :

```bash
sudo mkdir -p /etc/apache2/conf.d/userdata/std/2_4/USER/appli.laroche360.ca
sudo mkdir -p /etc/apache2/conf.d/userdata/ssl/2_4/USER/appli.laroche360.ca
sudo nano /etc/apache2/conf.d/userdata/std/2_4/USER/appli.laroche360.ca/rps.conf
sudo nano /etc/apache2/conf.d/userdata/ssl/2_4/USER/appli.laroche360.ca/rps.conf
```

Apres modification :

```bash
sudo /usr/local/cpanel/scripts/rebuildhttpdconf
sudo apachectl configtest
sudo systemctl restart httpd
```

Si vous exposez aussi `automation.laroche360.ca` comme sous-domaine separe dans cPanel, prevoyez egalement les includes correspondants pour ce vhost.

## Points d'attention

- `httpd` doit rester le seul service expose sur `80` et `443`.
- Nginx doit ecouter uniquement sur `127.0.0.1:8786`.
- Si vous etes sous cPanel, privilegiez les `userdata includes` plutot qu'une edition directe du fichier principal.
- Si vous activez HSTS, faites-le seulement apres validation complete du HTTPS sur tous les sous-domaines.
