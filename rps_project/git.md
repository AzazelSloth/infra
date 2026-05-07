# Git

Ce guide part du principe que vous travaillez avec l'utilisateur `devlaroche360`, pas avec `root`.

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

La cle SSH GitHub est documentee dans [ssh.md](./ssh.md).

## Test de connexion GitHub

```bash
ssh -T git@github.com
```

## Clone recommande

```bash
sudo mkdir -p /srv/rps
sudo chown -R devlaroche360:devlaroche360 /srv/rps
git clone git@github.com:OWNER/REPO.git /srv/rps
```

## Verification

```bash
git config --list
git remote -v
ssh -T git@github.com
```

## Recommandations

- Travaillez en `devlaroche360` pour Git, SSH, PM2 et les fichiers du projet.
- Reservez `sudo` aux installations systeme et aux services comme `nginx`, `httpd` et `postgresql`.
- Utilisez SSH plutot que HTTPS pour simplifier les automatisations CI/CD.
