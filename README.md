# Sub2API deployment

This repository contains deployment scaffolding only. It intentionally excludes all runtime data and secrets: OAuth tokens, user API keys, cookies, databases, backups, browser profiles, proxy credentials, and `.env`.

## Before deployment

1. Obtain an authorized Sub2API container image and set `SUB2API_IMAGE` in a local `.env` copied from `.env.example`.
2. Point `DOMAIN` to the server's public IPv4 address.
3. Generate unique PostgreSQL and Redis passwords.
4. Keep ports `5432`, `6379`, and `18080` private. Only Caddy exposes ports `80` and `443`.
5. Review the application image's required environment variables before starting the stack. The database and Redis URLs here are a template, not a substitute for the application's documentation.

## Local validation

```sh
cp .env.example .env
# Edit .env with non-production test values.
docker compose config
```

Do not run the stack against production OAuth credentials until the VPS has been hardened and the image source is confirmed.

## Backup

On the server, load the environment first and then run:

```sh
set -a && . ./.env && set +a
BACKUP_DIR=./backups sh scripts/backup.sh
```

Encrypt backups before moving them off the server. Backups contain account and user data.
