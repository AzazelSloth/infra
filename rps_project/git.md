# Git

## Installation

```bash
sudo apt update
sudo apt install git
```

## Configuration initiale

```bash
git config --global user.name "Votre Nom"
git config --global user.email "vous@example.com"
git config --global init.defaultBranch main
git config --global pull.rebase false
git config --global core.editor nano
```

## Cle SSH de deploiement

Sur le VPS, il est preferable d'utiliser une cle dediee au deploiement :

```bash
ssh-keygen -t ed25519 -C "deploy@laroche360" -f ~/.ssh/id_ed25519
cat ~/.ssh/id_ed25519.pub
```

Ajoutez la cle publique comme deploy key ou dans votre compte GitHub.

## Test de connexion GitHub

```bash
ssh -T git@github.com
```

## Clone recommande

```bash
sudo mkdir -p /srv/rps
sudo chown $USER:$USER /srv/rps
git clone git@github.com:OWNER/REPO.git /srv/rps
```

## Verification

```bash
git config --list
git remote -v
```

## Recommandations

- Evitez de deployer en `root` si un utilisateur applicatif est possible.
- Utilisez SSH plutot que HTTPS pour simplifier les automatisations CI/CD.
