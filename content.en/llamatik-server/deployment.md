---
title: "Deployment"
weight: 40
---

# Deployment

## Docker

### Build the image

```bash
git clone https://github.com/ferranpons/Llamatik-Server.git
cd Llamatik-Server
docker build -t llamatik-server .
```

### Run the container

```bash
docker run -d \
  --name llamatik \
  -p 8080:8080 \
  -v /srv/models:/models \
  -e JWT_SECRET="change-me" \
  -e JDBC_DRIVER="org.h2.Driver" \
  -e JDBC_DATABASE_URL="jdbc:h2:mem:llamatik;DB_CLOSE_DELAY=-1" \
  -e MODELS_DIR=/models \
  llamatik-server
```

Mount your model directory to `/models` inside the container and set `MODELS_DIR=/models` so path validation works correctly.

### Docker Compose (with PostgreSQL)

```yaml
version: "3.9"

services:
  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: llamatik
      POSTGRES_USER: llamatik
      POSTGRES_PASSWORD: secret
    volumes:
      - pgdata:/var/lib/postgresql/data

  llamatik:
    image: llamatik-server:latest
    ports:
      - "8080:8080"
    volumes:
      - /srv/models:/models
    environment:
      JWT_SECRET: "change-me-in-production"
      JDBC_DRIVER: "org.postgresql.Driver"
      JDBC_DATABASE_URL: "jdbc:postgresql://db:5432/llamatik"
      DB_USER: llamatik
      DB_PASSWORD: secret
      MODELS_DIR: /models
      SESSIONS_DIR: /models/sessions
    depends_on:
      - db

volumes:
  pgdata:
```

---

## systemd (Linux)

Create `/etc/systemd/system/llamatik.service`:

```ini
[Unit]
Description=Llamatik Server
After=network.target

[Service]
Type=simple
User=llamatik
WorkingDirectory=/opt/llamatik-server
EnvironmentFile=/etc/llamatik/env
ExecStart=/usr/bin/java -jar /opt/llamatik-server/llamatik-server.jar
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Create `/etc/llamatik/env`:

```
JWT_SECRET=change-me
JDBC_DRIVER=org.postgresql.Driver
JDBC_DATABASE_URL=jdbc:postgresql://localhost:5432/llamatik
DB_USER=llamatik
DB_PASSWORD=secret
MODELS_DIR=/srv/models
SESSIONS_DIR=/srv/models/sessions
REQUIRE_INFERENCE_AUTH=true
```

Enable and start:

```bash
sudo systemctl daemon-reload
sudo systemctl enable llamatik
sudo systemctl start llamatik
```

---

## HTTPS with a reverse proxy

Llamatik Server speaks plain HTTP on port 8080. Terminate TLS at a reverse proxy.

> Until built-in encryption is available, traffic between the Llamatik app and the server is **not encrypted**. On networks you do not fully trust, put the server behind a TLS reverse proxy and avoid sending sensitive data.

### Browser (Wasm/JS) clients

The server enables CORS for any origin and allows the `Authorization` and `Content-Type` headers. If you proxy the server, make sure `OPTIONS` preflight requests reach it (or are answered by the proxy with the same headers); otherwise browser clients fail with a CORS error. Pages served over HTTPS can only call HTTPS servers.

### Caddy (recommended for simplicity)

```
inference.example.com {
    reverse_proxy localhost:8080
}
```

Caddy handles certificate provisioning automatically from Let's Encrypt.

### Nginx

```nginx
server {
    listen 443 ssl;
    server_name inference.example.com;

    ssl_certificate     /etc/letsencrypt/live/inference.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/inference.example.com/privkey.pem;

    # Required for SSE streaming endpoints
    proxy_buffering     off;
    proxy_cache         off;
    chunked_transfer_encoding on;

    location / {
        proxy_pass         http://localhost:8080;
        proxy_set_header   Host $host;
        proxy_set_header   X-Real-IP $remote_addr;
        proxy_read_timeout 600s;
    }
}
```

The `proxy_buffering off` setting is required for SSE endpoints (`/stream`, `/continueStream`, `/jsonStream`, `/analyzeStream`) — without it Nginx will buffer the event stream and clients will not receive tokens until the buffer fills.

---

## Production checklist

- [ ] `JWT_SECRET` is a long random string (at least 32 bytes)
- [ ] `REQUIRE_INFERENCE_AUTH=true` if the server is reachable from the internet
- [ ] `MODELS_DIR` is set to a directory containing only model files you trust callers to use
- [ ] TLS is terminated at a reverse proxy
- [ ] The server process runs as a non-root user with read-only access to the model directory
- [ ] `ENABLE_SHUTDOWN_URL` is not set (or `false`) on internet-facing deployments
- [ ] Database credentials are stored in a secrets manager or environment file not committed to version control
- [ ] Regular backups of the PostgreSQL database (user accounts and profiles)

---

## GPU acceleration

Model inference runs on CPU by default. To offload layers to a GPU:

1. Build the native runtimes with CUDA or Metal support (see [Building from source](../../guides/building-from-source/)).
2. Set `gpuLayers` when calling `POST /v1/generation/params` after loading the model:

```json
{ "gpuLayers": -1 }
```

`-1` offloads all layers. Start with a moderate value (e.g. `20`) if you run out of VRAM.
