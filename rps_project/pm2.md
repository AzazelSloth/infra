# PM2

PM2 est le gestionnaire de processus recommande pour les services Node du projet.

## Installation

```bash
sudo npm install -g pm2
pm2 -v
```

## Exemple d'ecosystem

Creer `ecosystem.config.js` dans le projet :

```js
module.exports = {
  apps: [
    {
      name: "rps-backend",
      cwd: "/srv/rps/backend",
      script: "dist/main.js",
      env: {
        NODE_ENV: "production",
        PORT: 3000,
      },
    },
    {
      name: "rps-frontend",
      cwd: "/srv/rps/frontend",
      script: "node_modules/next/dist/bin/next",
      args: "start -p 3001",
      env: {
        NODE_ENV: "production",
        PORT: 3001,
      },
    },
  ],
};
```

## Commandes utiles

```bash
cd /srv/rps
pm2 start ecosystem.config.js
pm2 status
pm2 logs
pm2 restart rps-backend
pm2 restart rps-frontend
pm2 save
pm2 startup systemd -u $USER --hp $HOME
```

## Bonnes pratiques

- Lancez PM2 en tant que `devlaroche360`, pas en `root`.
- Gardez les apps Node sur des ports locaux uniquement.
- Faites pointer Nginx vers `3000` et `3001`.
- Utilisez `pm2 startOrReload ecosystem.config.js --update-env` dans le pipeline CI/CD.
