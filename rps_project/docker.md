# Docker et Docker Compose

Docker sert ici surtout a isoler les services auxiliaires comme `n8n`. La commande moderne a utiliser est `docker compose` via le plugin officiel.

## Installation

```bash
sudo apt update
sudo apt install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo \"$VERSION_CODENAME\") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## Post-installation

```bash
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
docker --version
docker compose version
```

Ouvrez une nouvelle session shell apres l'ajout au groupe `docker`.

## Exemple de structure

```text
/srv/rps/
  docker-compose.yml
  .env
  n8n_data/
```

## Exemple minimal pour n8n

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

## Commandes utiles

```bash
docker compose pull
docker compose up -d
docker compose logs -f
docker compose ps
docker compose down
```

## Recommandations

- Exposez les conteneurs sensibles seulement sur `127.0.0.1`.
- Laissez Apache2 et Nginx gerer l'acces public et le TLS.
- Activez toujours une politique de redemarrage comme `unless-stopped`.
