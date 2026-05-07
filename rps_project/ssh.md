# SSH GitHub

Ce guide configure l'acces SSH GitHub pour l'utilisateur `devlaroche360` sur le serveur, afin de permettre :

- `git clone`
- `git pull`
- `git push`
- les scripts de deploiement bases sur Git via SSH

## Contexte

Dans votre cas :

- vous etes connecte comme `devlaroche360`
- la cle `id_deploy` a ete creee dans le dossier courant
- elle appartient actuellement a `root`

Il faut donc la deplacer dans `/home/devlaroche360/.ssh/` et corriger les permissions.

## 1. Creer le dossier `.ssh`

```bash
mkdir -p /home/devlaroche360/.ssh
chmod 700 /home/devlaroche360/.ssh
```

## 2. Deplacer la cle dans le bon dossier

```bash
sudo mv /home/devlaroche360/id_deploy /home/devlaroche360/.ssh/
sudo mv /home/devlaroche360/id_deploy.pub /home/devlaroche360/.ssh/
sudo chown devlaroche360:devlaroche360 /home/devlaroche360/.ssh/id_deploy /home/devlaroche360/.ssh/id_deploy.pub
chmod 600 /home/devlaroche360/.ssh/id_deploy
chmod 644 /home/devlaroche360/.ssh/id_deploy.pub
```

## 3. Creer la configuration SSH

```bash
nano /home/devlaroche360/.ssh/config
```

Ajoutez :

```sshconfig
Host github.com
  HostName github.com
  User git
  IdentityFile /home/devlaroche360/.ssh/id_deploy
  IdentitiesOnly yes
```

Puis :

```bash
chmod 600 /home/devlaroche360/.ssh/config
```

## 4. Ajouter GitHub dans `known_hosts`

```bash
ssh-keyscan github.com >> /home/devlaroche360/.ssh/known_hosts
chmod 644 /home/devlaroche360/.ssh/known_hosts
```

## 5. Afficher la cle publique

```bash
cat /home/devlaroche360/.ssh/id_deploy.pub
```

Copiez le contenu affiche.

## 6. Ajouter la cle dans GitHub

Deux options sont possibles :

### Option A : cle SSH dans le compte GitHub

Ajoutez la cle dans :

`GitHub > Settings > SSH and GPG keys`

Cette option permet d'utiliser tous les depots accessibles par votre compte.

### Option B : deploy key sur un depot

Ajoutez la cle dans :

`Repository > Settings > Deploy keys`

Si vous voulez faire des `push`, activez `Allow write access`.

## 7. Tester l'authentification SSH

```bash
ssh -T git@github.com
```

Vous devriez voir un message du type :

```text
Hi <username>! You've successfully authenticated...
```

## 8. Cloner un depot via SSH

Utilisez l'URL SSH, pas HTTPS :

```bash
git clone git@github.com:OWNER/REPO.git
```

Exemple :

```bash
git clone git@github.com:kajutokirigaya4/mon-repo.git /home/devlaroche360/mon-repo
```

## 9. Basculer un depot existant de HTTPS vers SSH

Dans le dépôt :

```bash
cd /chemin/du/projet
git remote -v
git remote set-url origin git@github.com:OWNER/REPO.git
git remote -v
```

## 10. Verifier `pull` et `push`

```bash
git fetch
git pull
```

Test de `push` :

```bash
git checkout -b test-ssh
touch ssh-ok.txt
git add ssh-ok.txt
git commit -m "Test SSH from server"
git push -u origin test-ssh
```

## 11. Verifications utiles

```bash
whoami
ls -la /home/devlaroche360/.ssh
ssh -vT git@github.com
git remote -v
```

## Points d'attention

- N'utilisez pas `/root/.ssh` pour `devlaroche360`.
- La cle privee doit rester en `600`.
- Le dossier `.ssh` doit rester en `700`.
- Si vous regenerez une cle plus tard, pensez a remplacer aussi la cle enregistree dans GitHub.
