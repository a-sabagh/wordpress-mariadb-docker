# WordPress + MariaDB (Docker Compose)

Spin up a production-ready WordPress stack backed by MariaDB using Docker Compose.  
This repository provides a minimal, configurable setup with environment-based configuration and persistent volumes.

---

## ✨ Features

- 🧩 **WordPress (PHP + Apache)** container  
- 🗄️ **MariaDB** database container  
- ⚙️ **Environment-based configuration** via `.env`  
- 💾 **Persistent volumes** for WordPress files and database data  
- 🚀 One-command **setup and teardown** with Docker Compose  

---

## 🧰 Prerequisites

Before you begin, make sure you have:

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/)

---

## 🚀 Quick Start

1. **Clone the repository**
   ```bash
   git clone https://github.com/a-sabagh/wordpress-mariadb-docker.git
   cd wordpress-mariadb-docker
   ```

2. **Create your environment file**
   ```bash
   cp .env.example .env
   ```
   > Edit `.env` to customize database credentials, ports, and other environment variables.

3. **Start the stack**
   ```bash
   docker compose up -d
   ```

4. **Access WordPress**
   Open your browser and go to:
   ```
   http://localhost:8080
   ```
   > (Replace `8080` with your `WORDPRESS_PORT` value in `.env`.)

5. **Stop the stack**
   ```bash
   docker compose down
   ```

---

## ⚙️ Configuration

All key parameters are defined in the `.env` file:

```dotenv
# WordPress
WORDPRESS_PORT=8080
WORDPRESS_DB_HOST=db
WORDPRESS_DB_NAME=wordpress
WORDPRESS_DB_USER=wp_user
WORDPRESS_DB_PASSWORD=change_me

# MariaDB
MARIADB_ROOT_PASSWORD=change_me_root
MARIADB_DATABASE=wordpress
MARIADB_USER=wp_user
MARIADB_PASSWORD=change_me
```

> 🔒 **Tip:** Always change default passwords before deploying publicly.

---

## 🗂️ Project Structure

```
.
├── compose.yaml
├── .env.example
└── docker/
    └── wordpress/
        └── (optional config or Dockerfile)
```

---

## 💾 Data Persistence

This setup uses **named Docker volumes** to persist WordPress uploads, plugins, and database data across restarts.

To completely remove containers and volumes:
```bash
docker compose down -v
```

> ⚠️ This will delete your database and uploaded files — use with caution.

---

## 🔍 Troubleshooting

- **WordPress can’t connect to the database**
  - Ensure `WORDPRESS_DB_HOST` matches the database service name in `compose.yaml` (usually `db`)
  - Confirm the credentials match in both `.env` and the Compose file

- **Port already in use**
  - Edit `WORDPRESS_PORT` in `.env` and restart with `docker compose up -d`

- **Permission issues**
  - On Linux, you may need to adjust folder permissions (e.g., `sudo chown -R $USER:$USER .`)

---

## 🧹 Maintenance

**Update images**
```bash
docker compose pull
docker compose up -d
```

**View logs**
```bash
docker compose logs -f
```

---

## 🧳 Backup & Restore

**Backup database**
```bash
DB=$(docker compose ps -q db)
docker exec -i "$DB" mysqldump -u"$MARIADB_USER" -p"$MARIADB_PASSWORD" "$MARIADB_DATABASE" > backup.sql
```

**Restore database**
```bash
DB=$(docker compose ps -q db)
docker exec -i "$DB" mysql -u"$MARIADB_USER" -p"$MARIADB_PASSWORD" "$MARIADB_DATABASE" < backup.sql
```

---

## 📝 License

This project currently does not specify a license.  
If you plan to share or contribute, consider adding one (e.g., MIT or Apache 2.0).

---

## 🙌 Contributing

Contributions are welcome!  
Please open an issue or submit a pull request with clear descriptions and testing steps.

---

## 📎 References

- [WordPress (Docker Hub)](https://hub.docker.com/_/wordpress)  
- [MariaDB (Docker Hub)](https://hub.docker.com/_/mariadb)  
- [Docker Compose Documentation](https://docs.docker.com/compose/)
