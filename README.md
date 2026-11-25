# WordPress + MariaDB (Docker Compose)

A local **development** stack for WordPress backed by MariaDB, with WP-CLI, Xdebug and Adminer.
WordPress code lives on your disk (`./src`), configuration lives in `.env`, and the database is persisted in a named volume.

> This setup is meant for development. Before using it on a public server, review the
> [Production notes](#production-notes).

---

## Features

- WordPress (PHP + Apache) container built from `docker/wordpress/Dockerfile`
- MariaDB with a healthcheck (WordPress starts only after the DB is ready)
- WP-CLI and Xdebug preinstalled
- Adminer for browser-based DB access
- Configuration via `.env`
- Database persisted in the named volume `wpdb`; WordPress files in `./src`
- All published ports are bound to `127.0.0.1` (not reachable from other machines)

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/) (v2, the `docker compose` command)

## Quick Start

1. **Clone the repository**

   ```bash
   git clone https://github.com/a-sabagh/wordpress-mariadb-docker.git
   cd wordpress-mariadb-docker
   ```

2. **Create your environment file**

   ```bash
   cp .env.example .env
   ```

   Edit `.env`: change the passwords, and set `UID` / `GID` to the output of `id -u` and `id -g`
   so files in `./src` are owned by you, not root.

3. **Start the stack**

   ```bash
   docker compose up -d --build
   ```

4. **Open WordPress**: http://localhost:8080 (or your `WORDPRESS_PORT`).
   **Adminer**: http://localhost:8082 (or your `ADMINER_PORT`).
   In Adminer choose System `MySQL`, Server `mariadb`, and the user/password from `.env`.

5. **Stop the stack**

   ```bash
   docker compose down
   ```

## Configuration

`.env.example` contains every variable:

```ini
# MariaDB
MARIADB_USER=wpmari_user
MARIADB_PASSWORD=change_me
MARIADB_DATABASE=wpmari_db
MARIADB_ROOT_PASSWORD=change_me_root

# WordPress
WORDPRESS_DEBUG=1
WORDPRESS_PORT=8080

# Adminer
ADMINER_PORT=8082

# Host user
UID=1000
GID=1000
```

> Warning: MariaDB reads the passwords only when the database is first created.
> Changing them in `.env` later does **not** change the existing database users.

## Project Structure

```
.
├── compose.yaml
├── .env.example
├── .gitignore
├── docker/
│   └── wordpress/
│       ├── Dockerfile
│       └── php.ini
├── src/        # WordPress files (created on first start, git-ignored)
└── backups/    # your backups (git-ignored)
```

## Data Persistence

- **Database**: named volume `wpdb`.
- **WordPress files** (core, plugins, themes, uploads): bind mount `./src`.

Remove containers but keep data:

```bash
docker compose down
```

Remove containers **and the database volume**:

```bash
docker compose down -v
```

> This deletes the database. Files in `./src` are not touched by `down -v`; delete that folder manually if you want a clean start.

## Debugging with Xdebug

Xdebug is configured in `docker/wordpress/php.ini` (client host `host.docker.internal`, port `9003`).

1. Set your IDE to listen on port `9003`.
2. Enable the Xdebug helper extension in your browser, or send the `XDEBUG_TRIGGER` cookie/parameter.
3. For WP-CLI: `docker compose exec web env XDEBUG_TRIGGER=1 wp <command>`

## Troubleshooting

- **WordPress can't connect to the database**
  - The DB host must be the service name `mariadb` (set automatically in `compose.yaml`).
  - Check the logs: `docker compose logs mariadb`.
  - If you changed the passwords in `.env` after the first start, the database still has the old ones.
- **Port already in use**
  - Change `WORDPRESS_PORT` (or `ADMINER_PORT`) in `.env`, then `docker compose up -d`.
- **Permission issues in `./src`**
  - Make sure `UID` and `GID` in `.env` match `id -u` and `id -g`.
  - If files were already created as root, fix only that folder: `sudo chown -R "$(id -u):$(id -g)" src`

## Maintenance

Update images (after bumping the pinned versions in `compose.yaml` / `Dockerfile`):

```bash
docker compose pull
docker compose build --pull
docker compose up -d
```

> Always back up before upgrading MariaDB or WordPress. A MariaDB data directory cannot be downgraded.

View logs:

```bash
docker compose logs -f
```

## Backup & Restore

Create the folder once: `mkdir -p backups`

**Backup the database**

```bash
docker compose exec -T mariadb sh -c \
  'exec mariadb-dump --single-transaction -u"$MARIADB_USER" -p"$MARIADB_PASSWORD" "$MARIADB_DATABASE"' \
  > backups/db-$(date +%F).sql
```

**Restore the database**

```bash
docker compose exec -T mariadb sh -c \
  'exec mariadb -u"$MARIADB_USER" -p"$MARIADB_PASSWORD" "$MARIADB_DATABASE"' \
  < backups/db-YYYY-MM-DD.sql
```

**Backup WordPress files** (includes uploads, plugins and themes)

```bash
tar czf backups/src-$(date +%F).tar.gz src
```

## Production notes

This repository is a development workspace. For a public server you would additionally need:
a production Dockerfile target without Xdebug, `display_errors=0` / `log_errors=1`,
no bind mount of source code, a reverse proxy with HTTPS, no Adminer, strong secrets, and regular off-host backups.

## License

This project currently does not specify a license. If you want others to reuse it, add one (e.g. MIT or Apache-2.0).

## Contributing

Contributions are welcome. Please open an issue or a pull request with a clear description and testing steps.

## References

- [WordPress (Docker Hub)](https://hub.docker.com/_/wordpress)
- [MariaDB (Docker Hub)](https://hub.docker.com/_/mariadb)
- [Adminer (Docker Hub)](https://hub.docker.com/_/adminer)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Xdebug documentation](https://xdebug.org/docs/)
