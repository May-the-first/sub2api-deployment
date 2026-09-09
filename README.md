# Sub2API cloud deployment

A sanitized, reproducible Docker deployment for a public Linux cloud server, including Tencent Cloud, Alibaba Cloud, and other VPS providers. It follows the official [Sub2API](https://github.com/Wei-Shaw/sub2api) release image `weishaw/sub2api:0.2.4`.

This is not a backup of the running instance. It intentionally excludes OAuth tokens, account cookies, user API keys, passwords, database dumps, Redis data, browser profiles, proxy credentials, and the real `.env` file.

## Architecture

- Caddy is the only public service: ports `80` and `443`.
- Sub2API, PostgreSQL, and Redis communicate only on the Docker network.
- Persistent directories are `data/`, `postgres_data/`, and `redis_data/`. Do not commit them.
- The server must have a compliant, stable egress path for required upstream services. Do not copy this machine's proxy configuration or credentials.

## Deploy

1. Create a Linux server with Docker Compose, a public IPv4 address, and a domain.
2. In the cloud security group, allow inbound TCP `80` and `443` only. Keep SSH restricted to your own IP where possible. Do not open `5432`, `6379`, or `8080`.
3. Point the domain A record to the server public IPv4.
4. Copy `.env.example` to `.env` and replace every password and generated-secret placeholder. Generate `JWT_SECRET` and `TOTP_ENCRYPTION_KEY` with `openssl rand -hex 32`. Set the administrator email and a unique password.
5. Create the persistent directories and validate the stack:

```sh
mkdir -p data postgres_data redis_data
cp .env.example .env
# Edit .env locally on the server.
docker compose config
docker compose up -d
docker compose ps
curl -fsS https://your-domain.example/health
```

Caddy obtains and renews HTTPS certificates automatically after DNS and ports `80/443` are reachable.

## Migration

For a real migration, export PostgreSQL and copy the application `data/` directory by a secure channel outside GitHub. Restore to the new server only after first validating this clean stack. OAuth material, user keys, and account data are sensitive production data: transfer them only when you accept the operational and provider-policy responsibility for storing them on that server.

## Backup

```sh
set -a && . ./.env && set +a
BACKUP_DIR=./backups sh scripts/backup.sh
```

Encrypt backup archives before transferring them. Periodically test a restore on a separate server; a backup is not verified until it restores.
