# n8n

Pour cette stack, la meilleure option est de faire tourner `n8n` derriere Nginx et Apache, sur `127.0.0.1:5678`, de preference via Docker.

## Variables importantes

Si `n8n` est publie sous `/n8n/`, alignez la configuration avec le reverse proxy :

```env
N8N_HOST=automation.laroche360.ca
N8N_PORT=5678
N8N_PROTOCOL=https
N8N_PATH=/n8n/
N8N_EDITOR_BASE_URL=https://automation.laroche360.ca/n8n/
WEBHOOK_URL=https://automation.laroche360.ca/n8n/
N8N_SECURE_COOKIE=true
N8N_LISTEN_ADDRESS=0.0.0.0
TZ=Indian/Antananarivo
GENERIC_TIMEZONE=Indian/Antananarivo
```

## Lancement via Docker Compose

```yaml
services:
  n8n:
    image: docker.n8n.io/n8nio/n8n:latest
    restart: unless-stopped
    ports:
      - "127.0.0.1:5678:5678"
    env_file:
      - .env
    volumes:
      - ./n8n_data:/home/node/.n8n
```

Puis :

```bash
docker compose up -d
docker compose logs -f n8n
```

## Si vous tenez a npm

```bash
npm install -g n8n
n8n start
```

Dans ce cas, gerez le processus via PM2 plutot qu'avec une session shell ouverte.

## Verification

```bash
curl -I http://127.0.0.1:5678
curl -I https://automation.laroche360.ca/n8n/
```

## Points d'attention

- `WEBHOOK_URL` doit pointer vers l'URL publique finale, sinon les webhooks seront invalides.
- Si vous gardez le sous-chemin `/n8n/`, le slash final doit rester coherent dans `N8N_PATH` et `N8N_EDITOR_BASE_URL`.
