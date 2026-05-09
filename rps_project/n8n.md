# n8n

Pour cette stack, `n8n` doit être prepare comme un service interne du projet, hébergé sur le VPS, exposé uniquement en local sur `127.0.0.1:5678`, puis publié via Nginx et Apache sur `https://automation.laroche360.ca/n8n/`.

L'objectif de cette doc est de partir d'un VPS fraichement reinitialise et de lister uniquement les commandes, manipulations et fichiers necessaires pour remettre `n8n` en place proprement.

## Choix d'arborescence retenu

On se base ici sur la structure suivante :

- dépôt Git du projet : `/srv/rps`
- service `n8n` isolé hors dépôt : `/srv/n8n`
- données persistantes `n8n` : `/srv/n8n/data`

Ce choix permet :

- de garder le code du projet dans `/srv/rps`
- d'isoler le runtime `n8n` de l'arbre Git
- d'éviter de mélanger données applicatives et données d'automatisation

## Organisation recommandée

Stockage conseillé :

- code du projet : `/srv/rps`
- dossier runtime `n8n` : `/srv/n8n`
- variables chargées par Docker Compose : `/srv/n8n/.env`
- compose du service : `/srv/n8n/docker-compose.yml`
- données persistantes `n8n` : `/srv/n8n/data`
- unité systemd du service interne : `/etc/systemd/system/rps-n8n.service`

## Secrets GitHub déjà existants

Les secrets suivants sont déjà renseignés dans GitHub :

- `N8N_BASIC_AUTH_ACTIVE`
- `N8N_BASIC_AUTH_PASSWORD`
- `N8N_BASIC_AUTH_USER`
- `DB_HOST`
- `DB_NAME_N8N`
- `DB_PASSWORD`
- `DB_PORT`
- `DB_USER`
- `N8N_ENCRYPTION_KEY`
- `N8N_HOST`
- `N8N_PORT`
- `N8N_PROTOCOL`
- `N8N_SECURE_COOKIE`
- `N8N_USER_FOLDER`

Le guide doit donc s'aligner sur cette logique :

- ces variables doivent venir en priorité des secrets GitHub
- le fichier local `/srv/n8n/.env` doit être reconstruit à partir de ces secrets lors du déploiement
- `N8N_USER_FOLDER` doit correspondre au dossier utilisateur utilisé dans le conteneur
- les variables PostgreSQL doivent elles aussi venir des secrets GitHub
- seules les variables non couvertes par GitHub doivent être completées en plus

## Preconfiguration sur VPS neuf

Avant de configurer `n8n`, il faut déjà avoir :

- Docker installé
- PostgreSQL installé
- le dossier projet `/srv/rps`
- l'utilisateur de travail `devlaroche360`

Références : [docker.md](./docker.md), [postgresql.md](./postgresql.md), [setup_sequence.md](./setup_sequence.md)

## Etape 1 - Créer l'arborescence du service

Commandes :

```bash
sudo mkdir -p /srv/n8n/data
sudo chown -R devlaroche360:devlaroche360 /srv/n8n
chmod 755 /srv/n8n
chmod 700 /srv/n8n/data
```

Fichiers ou dossiers concernés :

- `/srv/n8n/`
- `/srv/n8n/data/`

## Etape 2 - Préparer la base PostgreSQL pour n8n

Si vous dédiez une base à `n8n` :

```bash
sudo -u postgres psql
```

```sql
CREATE DATABASE rps_automation OWNER postgres;
```

La connexion sera ensuite stockée dans :

- `/srv/n8n/.env`

Variables de base attendues, alimentées par les secrets GitHub :

```text
DB_TYPE=postgresdb
DB_POSTGRESDB_HOST=${DB_HOST}
DB_POSTGRESDB_PORT=${DB_PORT}
DB_POSTGRESDB_DATABASE=${DB_NAME_N8N}
DB_POSTGRESDB_USER=${DB_USER}
DB_POSTGRESDB_PASSWORD=${DB_PASSWORD}
```

## Etape 3 - Vérifier la clé de chiffrement existante

`n8n` doit garder une clé stable pour chiffrer ses credentials. Comme `N8N_ENCRYPTION_KEY` existe déjà dans les secrets GitHub, il faut réutiliser cette valeur et ne pas la regénérer pendant une simple réinstallation.

Si vous devez la créer une première fois :

```bash
openssl rand -hex 32
```

La valeur produite doit être rangée dans :

- le secret GitHub `N8N_ENCRYPTION_KEY`

Puis injectée dans :

- `/srv/n8n/.env`

## Etape 4 - Definir la correspondance pour `N8N_USER_FOLDER`

Le secret GitHub `N8N_USER_FOLDER` doit être aligné avec le chemin interne du conteneur.

Valeur recommandée :

```env
N8N_USER_FOLDER=/home/node/.n8n
```

Correspondance de volume côté VPS :

- `/srv/n8n/data` sur l'hôte
- `/home/node/.n8n` dans le conteneur

Autrement dit :

- le secret GitHub garde la valeur interne `N8N_USER_FOLDER=/home/node/.n8n`
- le `docker-compose.yml` monte `/srv/n8n/data` vers `/home/node/.n8n`

## Etape 5 - Générer le fichier d'environnement à partir des secrets GitHub

Le fichier local du VPS reste nécessaire pour `docker compose`, mais il doit être alimenté à partir des secrets déjà présents dans GitHub.

Créer le fichier :

```bash
nano /srv/n8n/.env
```

Structure recommandée :

```env
N8N_HOST=${N8N_HOST}
N8N_PORT=${N8N_PORT}
N8N_PROTOCOL=${N8N_PROTOCOL}
N8N_SECURE_COOKIE=${N8N_SECURE_COOKIE}
N8N_BASIC_AUTH_ACTIVE=${N8N_BASIC_AUTH_ACTIVE}
N8N_BASIC_AUTH_USER=${N8N_BASIC_AUTH_USER}
N8N_BASIC_AUTH_PASSWORD=${N8N_BASIC_AUTH_PASSWORD}
N8N_ENCRYPTION_KEY=${N8N_ENCRYPTION_KEY}
N8N_USER_FOLDER=${N8N_USER_FOLDER}

N8N_PATH=/n8n/
N8N_EDITOR_BASE_URL=https://automation.laroche360.ca/n8n/
WEBHOOK_URL=https://automation.laroche360.ca/n8n/
N8N_LISTEN_ADDRESS=0.0.0.0
TZ=Indian/Antananarivo
GENERIC_TIMEZONE=Indian/Antananarivo

DB_TYPE=postgresdb
DB_POSTGRESDB_HOST=${DB_HOST}
DB_POSTGRESDB_PORT=${DB_PORT}
DB_POSTGRESDB_DATABASE=${DB_NAME_N8N}
DB_POSTGRESDB_USER=${DB_USER}
DB_POSTGRESDB_PASSWORD=${DB_PASSWORD}
```

Fichier concerné :

- `/srv/n8n/.env`

Points d'attention :

- `N8N_BASIC_AUTH_*`, `N8N_ENCRYPTION_KEY`, `N8N_HOST`, `N8N_PORT`, `N8N_PROTOCOL`, `N8N_SECURE_COOKIE` et `N8N_USER_FOLDER` doivent correspondre aux secrets GitHub
- `DB_HOST`, `DB_NAME_N8N`, `DB_PORT`, `DB_PASSWORD` et `DB_USER` doivent correspondre aux secrets GitHub
- gardez le slash final dans `N8N_PATH` et `N8N_EDITOR_BASE_URL`
- `WEBHOOK_URL` doit toujours pointer vers l'URL publique finale
- `N8N_USER_FOLDER` doit rester cohérent avec le volume monté dans Docker

## Etape 6 - Exemple de génération non interactive du `.env`

Si le déploiement passe par GitHub Actions, vous pouvez écrire `/srv/n8n/.env` depuis les secrets et les variables projet.

Exemple de contenu cible :

```env
N8N_HOST=${N8N_HOST}
N8N_PORT=${N8N_PORT}
N8N_PROTOCOL=${N8N_PROTOCOL}
N8N_SECURE_COOKIE=${N8N_SECURE_COOKIE}
N8N_BASIC_AUTH_ACTIVE=${N8N_BASIC_AUTH_ACTIVE}
N8N_BASIC_AUTH_USER=${N8N_BASIC_AUTH_USER}
N8N_BASIC_AUTH_PASSWORD=${N8N_BASIC_AUTH_PASSWORD}
N8N_ENCRYPTION_KEY=${N8N_ENCRYPTION_KEY}
N8N_USER_FOLDER=${N8N_USER_FOLDER}
N8N_PATH=/n8n/
N8N_EDITOR_BASE_URL=https://automation.laroche360.ca/n8n/
WEBHOOK_URL=https://automation.laroche360.ca/n8n/
N8N_LISTEN_ADDRESS=0.0.0.0
TZ=Indian/Antananarivo
GENERIC_TIMEZONE=Indian/Antananarivo
DB_TYPE=postgresdb
DB_POSTGRESDB_HOST=${DB_HOST}
DB_POSTGRESDB_PORT=${DB_PORT}
DB_POSTGRESDB_DATABASE=${DB_NAME_N8N}
DB_POSTGRESDB_USER=${DB_USER}
DB_POSTGRESDB_PASSWORD=${DB_PASSWORD}
```

Dans cette logique :

- les secrets GitHub alimentent les variables `N8N_*` déjà normalisées
- les variables de base de données viennent elles aussi des secrets GitHub
- les URLs publiques sous `/n8n/` restent des valeurs de configuration du projet

## Etape 7 - Créer le fichier Docker Compose

Créer le fichier :

```bash
nano /srv/n8n/docker-compose.yml
```

Contenu recommandé :

```yaml
services:
  n8n:
    image: docker.n8n.io/n8nio/n8n:latest
    container_name: rps-n8n
    restart: unless-stopped
    ports:
      - "127.0.0.1:5678:5678"
    env_file:
      - .env
    volumes:
      - ./data:/home/node/.n8n
    healthcheck:
      test: ["CMD-SHELL", "wget -q --spider http://127.0.0.1:5678/healthz || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 30s
```

Fichier concerné :

- `/srv/n8n/docker-compose.yml`

Point d'alignement important :

- le volume `./data:/home/node/.n8n` doit rester cohérent avec `N8N_USER_FOLDER=/home/node/.n8n`

## Etape 8 - Vérifier manuellement le service avant automatisation

Depuis le dossier du service :

```bash
cd /srv/n8n
docker compose up -d
docker compose logs -f n8n
```

Vérifications :

```bash
docker compose ps
curl -I http://127.0.0.1:5678
```

Si tout est bon, quittez les logs avec `Ctrl+C`.

## Etape 9 - Declarer n8n comme service interne du projet

Pour que `n8n` redémarre avec le serveur sans dépendre d'une session shell, créez une unité `systemd`.

Créer le fichier :

```bash
sudo nano /etc/systemd/system/rps-n8n.service
```

Contenu recommandé :

```ini
[Unit]
Description=Service interne n8n du projet RPS
Requires=docker.service
After=docker.service network-online.target

[Service]
Type=oneshot
RemainAfterExit=yes
User=devlaroche360
Group=devlaroche360
WorkingDirectory=/srv/n8n
ExecStart=/usr/bin/docker compose up -d
ExecStop=/usr/bin/docker compose down
TimeoutStartSec=0

[Install]
WantedBy=multi-user.target
```

Fichier concerne :

- `/etc/systemd/system/rps-n8n.service`

Pourquoi ce choix :

- `n8n` reste rattache au projet au niveau exploitation
- le fichier de definition vit dans le systeme
- le runtime et les données restent dans `/srv/n8n`
- le dépôt Git dans `/srv/rps` reste propre et separe
- le service se pilote avec `systemctl`, comme le reste du serveur

## Etape 10 - Activer le service

Commandes :

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now rps-n8n
sudo systemctl status rps-n8n
```

Vérifications utiles :

```bash
docker ps --filter name=rps-n8n
curl -I http://127.0.0.1:5678
sudo journalctl -u rps-n8n -n 100 --no-pager
```

## Etape 11 - Validation derrière le reverse proxy

Une fois Nginx et Apache configurés :

```bash
curl -I https://automation.laroche360.ca/n8n/
```

Resultat attendu :

- réponse `200`, `301` ou `302`
- interface `n8n` chargée sous `/n8n/`
- webhooks générés avec `https://automation.laroche360.ca/n8n/`

## Résumé des fichiers à conserver

- `/srv/n8n/.env` : variables finales chargées par Docker Compose
- `/srv/n8n/docker-compose.yml` : définition du conteneur `n8n`
- `/srv/n8n/data/` : données persistantes, credentials, état interne
- `/etc/systemd/system/rps-n8n.service` : service système qui pilote le conteneur

## Commandes de maintenance

Redémarrer le service :

```bash
sudo systemctl restart rps-n8n
```

Voir les logs du service systemd :

```bash
sudo journalctl -u rps-n8n -f
```

Voir les logs du conteneur :

```bash
cd /srv/n8n
docker compose logs -f n8n
```

Mettre à jour l'image :

```bash
cd /srv/n8n
docker compose pull
sudo systemctl restart rps-n8n
```

## Points d'attention

- ne publiez pas `5678` publiquement, gardez `127.0.0.1:5678:5678`
- sauvegardez le dossier `/srv/n8n/data/`
- sauvegardez aussi `/srv/n8n/.env`, sinon vous perdez la clé de chiffrement et la configuration active
- si vous changez `N8N_ENCRYPTION_KEY` aprèss coup, les anciens credentials risquent de devenir illisibles
- si vous changez `N8N_USER_FOLDER`, adaptez aussi le montage de volume dans `docker-compose.yml`
