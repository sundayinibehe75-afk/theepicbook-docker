# Reverse Proxy Routing and CORS

## Selected reverse proxy
Nginx (`nginx:alpine`) is the only public entry point. It is the only service that publishes a
port (80). The frontend, backend and database are only reachable inside the Docker networks.

## Routes

| Path | Goes to | Purpose |
|---|---|---|
| `= /health` | `backend:8080` | Health endpoint for monitoring and the proxy health check |
| `/api/` | `backend:8080` | API requests (e.g. `/api/cart`) |
| `/assets/` | `frontend:80` | Static files (CSS, JS, images) served by the frontend Nginx |
| `/` | `backend:8080` | Application pages, rendered by the Node.js backend |

Nginx picks the most specific match first, so `/health`, `/api/` and `/assets/` are handled
before the catch-all `/` route.

The proxy passes the `Host`, `X-Real-IP`, `X-Forwarded-For` and `X-Forwarded-Proto` headers, so the
backend sees the real client details instead of the proxy's.

## Was CORS required?
No. The browser loads the pages, static assets and API from the same origin
(`http://<VM_PUBLIC_IP>`), all through the reverse proxy on port 80. CORS only applies when a page
calls a different origin (another domain, port or protocol), so the browser never performs a CORS check here.

CORS headers are still set on `/api/` with a single allowed origin (the VM public IP, not `*`) as a
safeguard, in case the frontend is ever served from a separate origin. They are not needed for the
current setup.
