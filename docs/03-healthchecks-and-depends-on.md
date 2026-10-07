# Health Checks and Startup Order

## Health-check method per service

| Service | What is checked | Command |
|---|---|---|
| database (MySQL 8.0) | MySQL is accepting connections | `mysqladmin ping` |
| backend (Node.js) | App responds on its health endpoint | `curl -f http://localhost:8080/health` |
| frontend (Nginx) | Nginx is serving the static files | `wget --spider http://localhost:80` |
| reverse-proxy (Nginx) | Proxy is up and can reach the backend | `wget --spider http://127.0.0.1/health` |

All checks run every 10s with a 5s timeout and 5 retries. Each service has a `start_period`
(database 20s, backend 10s, frontend and proxy 5s), so slow startups are not counted as failures.

## Startup dependency order

1. **database** and **frontend** start first. They do not depend on any other service.
2. **backend** waits until the database is `healthy` (`condition: service_healthy`),
   because it needs a working MySQL connection when it starts.
3. **reverse-proxy** starts last. It waits until both the frontend and the backend are `healthy`,
   so users never reach the proxy while its upstreams are down.

## Why this matters

Without health checks, `depends_on` only waits for a container to start, not for it to be ready.
MySQL needs several seconds before it accepts connections. With `service_healthy`,
the backend does not start until the database can actually serve queries.
