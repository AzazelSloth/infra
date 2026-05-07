# Preparation et configuration du VPS

Ce document donne l'ordre recommande pour utiliser les fichiers `.md` deja presents dans ce dossier, en tenant compte de l'environnement actuel :

- utilisateur applicatif : `devlaroche360`
- acces GitHub via SSH deja prepare sous `devlaroche360`
- repertoire projet : `/srv/rps`
- services systeme administres avec `sudo`
- reverse proxy public via `httpd`
- reverse proxy interne via `nginx`

## Vue d'ensemble

Sequence recommandee :

1. `ssh.md`
2. `git.md`
3. `node.md`
4. `docker.md`
5. `postgresql.md`
6. `n8n.md`
7. `pm2.md`
8. `nginx.md`
9. `apache2.md`
10. `certbot.md`
11. `ci_cd.md`

`ufw.md` n'est pas inclus ici car vous avez demande de l'exclure.

## Etape 1 - Verifier l'utilisateur de travail

Objectif : s'assurer que vous travaillez bien en `devlaroche360` pour Git, SSH, PM2 et les fichiers projet.

Commandes :

```bash
whoami
echo $HOME
pwd
```

Resultat attendu :

- `whoami` doit retourner `devlaroche360`
- `echo $HOME` doit retourner `/home/devlaroche360`

Ne faites pas les commandes GitHub en `root`.

Reference : [ssh.md](./ssh.md)

## Etape 2 - Verifier SSH GitHub

Objectif : confirmer que le serveur peut parler a GitHub via la cle de `devlaroche360`.

Commandes :

```bash
ls -la /home/devlaroche360/.ssh
ssh -T git@github.com
```

Resultat attendu :

```text
Hi <username>! You've successfully authenticated, but GitHub does not provide shell access.
```

Reference : [ssh.md](./ssh.md)

## Etape 3 - Preparer le repertoire projet

Objectif : preparer `/srv/rps` et s'assurer qu'il appartient a `devlaroche360`.

Commandes :

```bash
sudo mkdir -p /srv/rps
sudo chown -R devlaroche360:devlaroche360 /srv/rps
ls -ld /srv/rps
```

Resultat attendu :

- le proprietaire doit etre `devlaroche360`

Reference : [git.md](./git.md)

## Etape 4 - Cloner ou reconnecter le depot Git

### Cas A : le dossier `/srv/rps` est vide

```bash
git clone git@github.com:OWNER/REPO.git /srv/rps
```

### Cas B : le projet existe deja

```bash
cd /srv/rps
git remote -v
git remote set-url origin git@github.com:OWNER/REPO.git
git fetch
git pull
```

Verification :

```bash
cd /srv/rps
git remote -v
git branch
```

Reference : [git.md](./git.md), [ssh.md](./ssh.md)

## Etape 5 - Installer Node.js et npm

Objectif : preparer l'environnement Node pour le frontend, le backend et PM2.

Commandes :

```bash
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs
node -v
npm -v
```

Reference : [node.md](./node.md)

## Etape 6 - Installer PM2

Objectif : gerer les processus Node en production sous `devlaroche360`.

Commandes :

```bash
sudo npm install -g pm2
pm2 -v
```

Reference : [pm2.md](./pm2.md)

## Etape 7 - Installer Docker et Docker Compose

Objectif : preparer les services conteneurises, notamment `n8n`.

Commandes :

```bash
sudo apt update
sudo apt install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo \"$VERSION_CODENAME\") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl enable --now docker
sudo usermod -aG docker devlaroche360
```

Puis ouvrez une nouvelle session shell et verifiez :

```bash
docker --version
docker compose version
```

Reference : [docker.md](./docker.md)

## Etape 8 - Installer et configurer PostgreSQL

Objectif : preparer la base pour l'application et eventuellement pour `n8n`.

Commandes :

```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
sudo systemctl enable --now postgresql
sudo -u postgres psql
```

Exemple de creation :

```sql
CREATE ROLE rps_app WITH LOGIN PASSWORD 'change-me';
CREATE DATABASE rps_app OWNER rps_app;

CREATE ROLE n8n_user WITH LOGIN PASSWORD 'change-me-too';
CREATE DATABASE n8n OWNER n8n_user;
```

Puis :

```bash
pg_isready
sudo -u postgres psql -l
```

Reference : [postgresql.md](./postgresql.md)

## Etape 9 - Installer et configurer n8n

Objectif : faire tourner `n8n` derriere Nginx et `httpd`, en local sur `127.0.0.1:5678`.

Placez-vous dans le projet :

```bash
cd /srv/rps
```

Preparez vos variables `.env` selon la doc, puis lancez :

```bash
docker compose up -d
docker compose logs -f n8n
```

Verification :

```bash
curl -I http://127.0.0.1:5678
```

Reference : [n8n.md](./n8n.md), [docker.md](./docker.md)

## Etape 10 - Installer les dependances du projet

Objectif : preparer frontend et backend pour le build et l'execution.

Depuis le repertoire projet :

```bash
cd /srv/rps
npm ci
npm run build --if-present
```

Si le projet est separe en plusieurs sous-applications, executez ces commandes dans les dossiers concernes.

Reference : [node.md](./node.md)

## Etape 11 - Configurer PM2

Objectif : lancer les applications Node sur les ports locaux attendus par Nginx.

Le fichier `ecosystem.config.js` doit etre adapte a votre projet avec :

- backend sur `3000`
- frontend sur `3001`

Puis :

```bash
cd /srv/rps
pm2 start ecosystem.config.js
pm2 status
pm2 save
pm2 startup systemd -u $USER --hp $HOME
```

Reference : [pm2.md](./pm2.md)

## Etape 12 - Configurer Nginx

Objectif : faire le reverse proxy interne entre `httpd` et les services applicatifs.

Creer le fichier :

```bash
sudo nano /etc/nginx/conf.d/rps.conf
```

Puis verifier :

```bash
sudo nginx -t
sudo systemctl enable --now nginx
sudo systemctl reload nginx
ss -tulpn | grep -E ':8786|:3000|:3001|:5678'
```

Reference : [nginx.md](./nginx.md)

## Etape 13 - Configurer httpd

Objectif : exposer les domaines publics en `80/443` et relayer vers Nginx.

Selon votre environnement, utilisez :

- soit `/etc/httpd/conf.d/rps.conf`
- soit les `userdata includes` cPanel si `httpd` est gere par WHM

Verification :

```bash
sudo apachectl configtest
sudo httpd -M | grep -E 'ssl|proxy|proxy_http|proxy_wstunnel|rewrite|headers|remoteip|http2'
sudo systemctl restart httpd
```

Reference : [apache2.md](./apache2.md)

## Etape 14 - Installer le certificat TLS

Objectif : activer HTTPS sur `appli.laroche360.ca` et `automation.laroche360.ca`.

Commandes :

```bash
sudo snap install core
sudo snap refresh core
sudo snap install --classic certbot
sudo ln -sf /snap/bin/certbot /usr/local/bin/certbot
sudo certbot certonly --apache -d appli.laroche360.ca -d automation.laroche360.ca
sudo certbot certificates
sudo apachectl configtest
sudo systemctl reload httpd
```

Reference : [certbot.md](./certbot.md)

## Etape 15 - Verifications finales

Commandes :

```bash
pm2 status
docker compose -f /srv/rps/docker-compose.yml ps
sudo nginx -t
sudo apachectl configtest
curl -I https://appli.laroche360.ca
curl -I https://automation.laroche360.ca/n8n/
```

Resultats attendus :

- PM2 actif
- `n8n` actif
- Nginx valide
- `httpd` valide
- reponses HTTP 200/301/302 selon les routes

## Etape 16 - Mettre en place le CI/CD

Objectif : deployer depuis GitHub Actions vers le VPS via SSH.

Secrets minimaux :

- `VPS_HOST`
- `VPS_PORT`
- `VPS_USER`
- `VPS_SSH_KEY`
- `VPS_DEPLOY_PATH`

Valeur recommandee pour `VPS_DEPLOY_PATH` :

```text
/srv/rps
```

Reference : [ci_cd.md](./ci_cd.md)

## Resume rapide

Si vous voulez l'ordre minimal a suivre :

1. verifier `devlaroche360`
2. verifier SSH GitHub
3. preparer `/srv/rps`
4. cloner le depot
5. installer Node + PM2
6. installer Docker
7. installer PostgreSQL
8. configurer `n8n`
9. configurer PM2
10. configurer Nginx
11. configurer `httpd`
12. installer Certbot
13. verifier la stack
14. brancher le CI/CD
