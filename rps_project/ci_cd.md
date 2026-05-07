# CI/CD

Cette stack se prete bien a un deploiement GitHub Actions vers le VPS via SSH.

## Secrets GitHub recommandes

- `VPS_HOST`
- `VPS_PORT`
- `VPS_USER`
- `VPS_SSH_KEY`
- `VPS_DEPLOY_PATH`

## Strategie conseillee

- Build et tests dans GitHub Actions.
- Copie des artefacts ou du code sur le serveur.
- Redemarrage applicatif avec PM2 pour les apps Node.
- `docker compose up -d` seulement pour les services conteneurises, par exemple `n8n`.

Ne faites pas tourner la meme application a la fois sous PM2 et sous Docker.

## Exemple de workflow

```yaml
name: Deploy VPS

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test --if-present

      - name: Build
        run: npm run build --if-present

      - name: Sync project to server
        uses: appleboy/scp-action@v0.1.7
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USER }}
          key: ${{ secrets.VPS_SSH_KEY }}
          port: ${{ secrets.VPS_PORT }}
          source: "."
          target: ${{ secrets.VPS_DEPLOY_PATH }}

      - name: Restart services on server
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USER }}
          key: ${{ secrets.VPS_SSH_KEY }}
          port: ${{ secrets.VPS_PORT }}
          script: |
            cd ${{ secrets.VPS_DEPLOY_PATH }}
            npm ci --omit=dev
            npm run build --if-present
            pm2 startOrReload ecosystem.config.js --update-env
            docker compose up -d
```

## Cote serveur

- Le depot applicatif vit dans `/srv/rps`.
- Les services Node geres par PM2 doivent exposer seulement des ports locaux comme `3000` et `3001`.
- `n8n` peut rester en conteneur sur `127.0.0.1:5678`.
- Les commandes applicatives du pipeline doivent tourner avec l'utilisateur `devlaroche360`.

## Verification apres deploiement

```bash
pm2 status
docker compose ps
sudo systemctl reload nginx
sudo systemctl reload httpd
curl -I https://appli.laroche360.ca
```
