# 02 — Environment Variables and Ports

## Environment variables

Real values live only in `.env` on the server (gitignored). `.env.example` lists the names with placeholder values.

### Backend (EpicBook Node.js app)

| Variable | Purpose | Notes |
|---|---|---|
| `NODE_ENV` | Selects the Sequelize config block in `config/config.json` | Must be `production` in Docker, so the app uses `JAWSDB_URL` instead of the hard-coded `127.0.0.1` development settings |
| `PORT` | Port the Express server listens on | Defaults to `8080` (`server.js`) |
| `JAWSDB_URL` | Full MySQL connection string used in production | Format: `mysql://<user>:<password>@database:3306/bookstore` — `database` is the Compose service name |

### Database (MySQL container)

| Variable | Purpose |
|---|---|
| `MYSQL_ROOT_PASSWORD` | Password for the MySQL root (admin) account |
| `MYSQL_DATABASE` | Database created on first start (`bookstore`) |
| `MYSQL_USER` | Application user created on first start |
| `MYSQL_PASSWORD` | Password for the application user (must match the password inside `JAWSDB_URL`) |

## Ports

| Service | Internal port | Published to host? | Reached by |
|---|---|---|---|
| `reverse-proxy` (Nginx) | 80 | **Yes — 80:80** (only public port) | Internet users |
| `frontend` (Nginx, static files) | 80 | No | reverse-proxy, on `front-tier` |
| `backend` (Node.js/Express) | 8080 | No | reverse-proxy, on `front-tier` |
| `database` (MySQL 8) | 3306 | No | backend only, on `back-tier` |

VM inbound rules: TCP 80 from Anywhere; TCP 22 from my IP only.

## Persistent data

| Item | Details |
|---|---|
| Named volume | `db_data` |
| Mounted at | `/var/lib/mysql` in the `database` container |
| Contains | All MySQL data files (books, authors, cart, orders) |
| Survives | `docker compose down` / `up`, container rebuilds |
| Deleted by | `docker compose down -v` (full reset only) |
| Backups | `mysqldump` files written to a host `backups/` directory |

## Health-check methods

| Service | Health check |
|---|---|
| `database` | `mysqladmin ping` inside the container |
| `backend` | HTTP `GET /health` on port 8080 |
| `frontend` | HTTP request to its Nginx on port 80 |
| `reverse-proxy` | HTTP `GET /health` through the proxy |
