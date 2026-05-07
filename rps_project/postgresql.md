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

## Restriction au localhost

Verifier ou adapter :

```bash
sudo nano /etc/postgresql/*/main/postgresql.conf
sudo nano /etc/postgresql/*/main/pg_hba.conf
```

Valeurs recommandees :

```conf
listen_addresses = 'localhost'
password_encryption = scram-sha-256
```

Ajoutez si necessaire dans `pg_hba.conf` :

```conf
host    all             all             127.0.0.1/32            scram-sha-256
host    all             all             ::1/128                 scram-sha-256
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

## Sauvegarde simple

```bash
pg_dump -U rps_app -h 127.0.0.1 rps_app > rps_app.sql
pg_dump -U n8n_user -h 127.0.0.1 n8n > n8n.sql
```
