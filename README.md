# Valoria Docker

Docker Compose deployment for the Valoria OpenMU server, PostgreSQL database,
automatic backups, Kotlin API, and React frontend.

## Services

- `db`: PostgreSQL database.
- `openmu`: OpenMU server built from the configured repository and Git ref.
- `db-backup`: verified PostgreSQL backups with retention.
- `valoria-api`: account, ranking, VIP, notices, and backup API.
- `valoria-web`: public website frontend.

The PostgreSQL database, API, and administrative panel are not published
directly to the internet. The OpenMU administrative panel binds to
`127.0.0.1:${ADM_PORT}` and should be accessed through an SSH tunnel.

## Requirements

- Linux server with Docker Engine and Docker Compose.
- At least 4 GB RAM and 80 GB storage for the initial deployment.
- Public TCP ports `44406`, `55902`, `55904`, and `55906-55908`.
- Ports `80` and `443` when an HTTPS reverse proxy is added.

## Configuration

```bash
git clone https://github.com/MDalfre/ValoriaDocker.git
cd ValoriaDocker
cp .env.example .env
chmod 600 .env
```

Edit `.env` and replace every placeholder. In production:

- Set `RESOLVE_IP` to the server public static IP.
- Set `VALORIA_ORIGIN` to the final HTTPS website address.
- Keep `VALORIA_REQUIRE_HTTPS=true`.
- Generate `VALORIA_JWT_SECRET` with `openssl rand -base64 32`.
- Use a unique, strong value for `DB_PASS`.
- Without a domain and TLS, set `VALORIA_WEB_BIND=0.0.0.0` and
  `VALORIA_WEB_PORT=80` only for public, unauthenticated pages.

The real `.env` is intentionally ignored by Git.

## Start

```bash
docker compose config --quiet
docker compose build --pull
docker compose up -d
docker compose ps
```

The initial OpenMU image build can take several minutes.

## Update

```bash
git pull --ff-only
docker compose build --pull
docker compose up -d --remove-orphans
```

## Backups

Backups run every `BACKUP_INTERVAL` seconds and are written to `./backups`.
Each dump is validated with `pg_restore --list` before it is retained.

Create a backup immediately:

```bash
docker exec openmu-db-backup /usr/local/bin/backup.sh
```

List the latest backups:

```bash
ls -lh backups/*.dump.gz
```

Before restoring a database, stop the game server and API, create a fresh
backup, and verify the selected dump. Database restoration is intentionally
not automated in this repository.

## Administrative access

Create an SSH tunnel from the administrator machine:

```bash
ssh -L 29596:127.0.0.1:29596 ubuntu@SERVER_IP
```

Then open `http://127.0.0.1:29596`. Do not expose this port through the cloud
firewall.

## Security

- Never commit `.env`, database dumps, SSH keys, or downloaded backups.
- Do not publish PostgreSQL port `5432`.
- Keep the administrative and frontend test ports bound to loopback.
- Terminate public website traffic through HTTPS before enabling real logins.
- Do not enable account login or administration while serving the website over
  plain HTTP.
- Keep `VALORIA_BACKUP_RESTORE_ENABLED=false` until restore authorization and
  operational safeguards have been reviewed.
