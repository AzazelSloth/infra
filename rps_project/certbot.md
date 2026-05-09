# Certbot

Dans cette architecture, Apache est le point d'entree public. Sur une installation standard, le plugin Apache de Certbot peut convenir, mais dans votre environnement actuel il peut echouer s'il cherche `apache2ctl` alors que seul `apachectl` est disponible.

La methode la plus robuste ici est donc `--standalone`, avec un arret temporaire d'Apache pendant l'emission du certificat.

## Prerequis

Avant de lancer Certbot, ne laissez pas de directives SSL actives si les fichiers de certificat n'existent pas encore.

Si votre vhost est dans `/etc/apache2/conf.d/rps.conf`, commencez par ne garder qu'un bloc HTTP en `:80`, ou commentez temporairement :

```apache
#SSLEngine on
#SSLCertificateFile /etc/letsencrypt/live/appli.laroche360.ca/fullchain.pem
#SSLCertificateKeyFile /etc/letsencrypt/live/appli.laroche360.ca/privkey.pem
```

Validez ensuite que la configuration Apache passe avant l'emission du certificat :

```bash
sudo apachectl configtest
```

Le domaine doit deja pointer vers le VPS et le port `80` doit etre joignable depuis Internet.

## Installation

```bash
sudo apt-get remove certbot
sudo snap install core
sudo snap refresh core
sudo snap install --classic certbot
sudo ln -sf /snap/bin/certbot /usr/local/bin/certbot
```

## Methode recommandee : emission avec `--standalone`

Si vous obtenez une erreur du type :

```text
NoInstallationError('Cannot find Apache executable apache2ctl')
```

contournez le plugin Apache et utilisez :

```bash
sudo apachectl configtest
sudo systemctl stop apache2 || sudo systemctl stop httpd
sudo certbot certonly --standalone \
  -d appli.laroche360.ca \
  -d automation.laroche360.ca
sudo systemctl start httpd
sudo certbot certificates
```

## Methode alternative : plugin Apache

Si votre environnement expose un binaire compatible avec le plugin Apache, vous pouvez utiliser :

```bash
sudo certbot certonly --apache \
  -d appli.laroche360.ca \
  -d automation.laroche360.ca
```

Si vous preferez laisser Certbot modifier automatiquement le vhost Apache quand l'environnement est propre, vous pouvez utiliser :

```bash
sudo certbot --apache \
  -d appli.laroche360.ca \
  -d automation.laroche360.ca
```

Les certificats seront ensuite reutilises par Apache :

```text
/etc/letsencrypt/live/appli.laroche360.ca/fullchain.pem
/etc/letsencrypt/live/appli.laroche360.ca/privkey.pem
```

## Rebrancher HTTPS dans Apache

Une fois le certificat emis, remettez les directives SSL dans `/etc/apache2/conf.d/rps.conf` ou dans votre include cPanel equivalent.

Exemple minimal :

```apache
<IfModule mod_ssl.c>
<VirtualHost *:443>
    ServerName appli.laroche360.ca
    ServerAlias automation.laroche360.ca

    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/appli.laroche360.ca/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/appli.laroche360.ca/privkey.pem

    ProxyPreserveHost On
    ProxyPass / http://127.0.0.1:8786/
    ProxyPassReverse / http://127.0.0.1:8786/
</VirtualHost>
</IfModule>
```

Puis testez et rechargez :

```bash
sudo apachectl configtest
sudo systemctl reload httpd
```

## Renouvellement

Verifier que le timer est actif :

```bash
systemctl list-timers | grep certbot
```

Tester un renouvellement a blanc :

```bash
sudo certbot renew --dry-run
```

## Hook utile

Si vous voulez vous assurer qu'Apache recharge toujours la nouvelle cle :

```bash
sudo certbot renew --deploy-hook "systemctl reload httpd"
```

## Verification

```bash
sudo certbot certificates
sudo apachectl configtest
sudo systemctl reload httpd
```

## Points d'attention

- Le port `80` doit rester joignable pour l'ACME challenge.
- Si un certificat unique est emis avec `appli.laroche360.ca` comme domaine principal, il peut quand meme couvrir `automation.laroche360.ca` en SAN.
- Si `apachectl configtest` echoue avant meme l'execution de Certbot, corrigez d'abord la configuration Apache en retirant toute reference a des fichiers SSL absents.
- Dans votre environnement actuel, utilisez `/var/log/apache2/` pour `ErrorLog` et `CustomLog`, pas `/var/log/httpd/`.
- Si `certbot --apache` echoue avec une erreur sur `apache2ctl`, utilisez `certbot certonly --standalone` a la place.
