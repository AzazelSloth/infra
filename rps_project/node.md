# Node.js et npm

Le plus simple sur Ubuntu 22.04 est d'installer la LTS depuis NodeSource pour obtenir une version recente de `node` et `npm`.

## Installation Node.js LTS

```bash
# Download and install nvm:
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.4/install.sh | bash

# in lieu of restarting the shell
\. "$HOME/.nvm/nvm.sh"

# Download and install Node.js:
nvm install 24

# Verify the Node.js version:
node -v # Should print "v24.15.0".

# Verify npm version:
npm -v # Should print "11.12.1".
```

## Verification

```bash
node -v
npm -v
```

## Outils globaux utiles

```bash
sudo npm install -g pm2
pm2 -v
```

## Recommandations npm

```bash
npm config set fund false
npm config set audit true
```

## Emplacements applicatifs conseilles

- Frontend Next.js : port local `3001`
- Backend API : port local `3000`
- Les processus Node doivent etre redemarres par PM2, pas par une session SSH ouverte.
