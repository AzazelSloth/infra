# Certbot

Dans cette architecture, Apache2 est le point d'entree public. Il est donc plus logique d'emettre les certificats avec le plugin Apache plutot que `--nginx`.

## Installation

```bash
sudo apt-get remove certbot
sudo snap install core
sudo snap refresh core
sudo snap install --classic certbot
sudo ln -sf /snap/bin/certbot /usr/local/bin/certbot
```

## Emission du certificat

Pour couvrir les deux noms utilises par la stack :

```bash
sudo certbot certonly --apache \
  -d appli.laroche360.ca \
  -d automation.laroche360.ca
```

Les certificats seront ensuite reutilises par Apache2 :

```text
/etc/letsencrypt/live/appli.laroche360.ca/fullchain.pem
/etc/letsencrypt/live/appli.laroche360.ca/privkey.pem
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
