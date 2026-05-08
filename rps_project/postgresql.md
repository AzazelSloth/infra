# PostgreSQL

PostgreSQL peut servir a la fois a l'application et a `n8n`. Sur ce VPS, il est preferable de le limiter a `localhost`.

## Installation

```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
sudo systemctl enable --now postgresql
```

## Creation des roles et bases

Exemple avec une base pour l'application et une pour `n8n` :

```bash
sudo -u postgres psql
```

```sql
CREATE ROLE rps_app WITH LOGIN PASSWORD 'change-me';
CREATE DATABASE rps_app OWNER rps_app;

CREATE ROLE n8n_user WITH LOGIN PASSWORD 'change-me-too';
CREATE DATABASE n8n OWNER n8n_user;
```

## Option compatible avec l'existant : utiliser l'utilisateur `postgres`

Si vous voulez minimiser les changements dans l'application et dans GitHub, vous pouvez garder l'utilisateur PostgreSQL `postgres` comme utilisateur de connexion, tout en lui ajoutant un mot de passe.

Important :

- `postgres` utilisateur Linux : sert a administrer PostgreSQL sur le serveur
- `postgres` role PostgreSQL : utilisateur de base de donnees

Pour définir un mot de passe sur le rôle PostgreSQL `postgres` :

```bash
sudo -u postgres psql
```

```sql
ALTER USER postgres WITH PASSWORD 'votre-mot-de-passe-fort';
```

Si votre application doit utiliser `postgres` comme utilisateur principal, vous pouvez aussi créer ou réutiliser une base dediée :

```sql
CREATE DATABASE rps_app OWNER postgres;
```

Exemple de chaine de connexion :

```env
DATABASE_URL=postgresql://postgres:VOTRE_MOT_DE_PASSE@127.0.0.1:5432/rps_app
```

## Restriction au localhost

Vérifier ou adapter :

```bash
sudo nano /etc/postgresql/*/main/postgresql.conf
sudo nano /etc/postgresql/*/main/pg_hba.conf
```

Valeurs recommandées :

```conf
listen_addresses = 'localhost'
password_encryption = scram-sha-256
```

Ajoutez si nécessaire dans `pg_hba.conf` :

```conf
host    all             all             127.0.0.1/32            scram-sha-256
host    all             all             ::1/128                 scram-sha-256
```

Si vous utilisez explicitement l'utilisateur `postgres`, vous pouvez aussi être plus precis avec :

```conf
host    all             postgres        127.0.0.1/32            scram-sha-256
host    all             postgres        ::1/128                 scram-sha-256
```

Puis :

```bash
sudo systemctl restart postgresql
```

## Verification

```bash
sudo -u postgres psql -l
pg_isready
```

Test de connexion avec mot de passe en tant que `postgres` :

```bash
psql -h 127.0.0.1 -U postgres -d postgres
```

Ou vers votre base applicative :

```bash
psql -h 127.0.0.1 -U postgres -d rps_app
```

## Sauvegarde simple

```bash
pg_dump -U rps_app -h 127.0.0.1 rps_app > rps_app.sql
pg_dump -U n8n_user -h 127.0.0.1 n8n > n8n.sql
```

Si vous utilisez `postgres` comme utilisateur principal :

```bash
pg_dump -U postgres -h 127.0.0.1 rps_app > rps_app.sql
```
