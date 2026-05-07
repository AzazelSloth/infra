# Node.js et npm

Le plus simple sur Ubuntu 22.04 est d'installer la LTS depuis NodeSource pour obtenir une version recente de `node` et `npm`.

## Installation Node.js LTS

```bash
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs
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
