---
title: "Configuration"
weight: 20
---

# Configuration

Llamatik Server is configured entirely through environment variables — no config files.

## Environment variables

### Required

| Variable | Description |
|---|---|
| `JWT_SECRET` | HMAC-512 secret used to sign and verify JWT tokens. Use a long random string in production. |
| `JDBC_DRIVER` | JDBC driver class, e.g. `org.postgresql.Driver` or `org.h2.Driver` |
| `JDBC_DATABASE_URL` | Full JDBC connection URL, e.g. `jdbc:postgresql://db:5432/llamatik` |

### Database (optional)

| Variable | Default | Description |
|---|---|---|
| `DB_USER` | — | Database username |
| `DB_PASSWORD` | — | Database password |
| `DB_MAX_POOL_SIZE` | `3` | HikariCP connection pool size |

### Security (optional but recommended)

| Variable | Default | Description |
|---|---|---|
| `REQUIRE_INFERENCE_AUTH` | `false` | Set to `"true"` to require a valid JWT `Bearer` token on all inference endpoints (`/v1/generation`, `/v1/embeddings`, `/v1/whisper`, `/v1/stablediffusion`, `/v1/multimodal`). **Always enable this for internet-facing deployments.** |
| `MODELS_DIR` | — | Absolute path to the directory containing model files, WAV files, and companion files (e.g. `/srv/models`). When set, all caller-supplied file paths must resolve inside this directory. Prevents path traversal. **Strongly recommended for all deployments.** |
| `SESSIONS_DIR` | `$MODELS_DIR/sessions` | Absolute path for KV-session save/load files. Defaults to a `sessions/` subdirectory of `MODELS_DIR` when that variable is set. Session save/load is unconstrained when neither is configured. |

### Administration (optional)

| Variable | Default | Description |
|---|---|---|
| `ENABLE_SHUTDOWN_URL` | `false` | Set to `"true"` to enable `POST /ktor/application/shutdown` for graceful remote shutdown. Do not enable on internet-facing deployments. |

## Security model

### LAN-only (home / office network)

No authentication is required for LAN-only deployments. Path containment is still strongly recommended to prevent accidental access outside the model directory:

```bash
export JWT_SECRET="..."
export MODELS_DIR=/srv/models
```

### Internet-facing

Always enable authentication and path containment together:

```bash
export JWT_SECRET="..."
export REQUIRE_INFERENCE_AUTH=true
export MODELS_DIR=/srv/models
export SESSIONS_DIR=/srv/models/sessions
```

With `REQUIRE_INFERENCE_AUTH=true`, clients must pass a valid JWT in the `Authorization` header:

```
Authorization: Bearer <token>
```

Obtain a token by calling `POST /v1/users/login`. See the [Auth & Profiles](#auth--profiles) section below.

### Path containment

When `MODELS_DIR` is set, Llamatik Server validates every caller-supplied file path (`modelPath`, `vaePath`, `t5xxlPath`, `clipVisionPath`, `wavPath`, session `path`) against that directory before passing it to the native runtime. Paths that resolve outside the directory return HTTP 400.

Paths containing null bytes are always rejected, regardless of `MODELS_DIR`.

### Error message sanitization

JNI error messages from the native runtimes are sanitized before being sent to HTTP callers. Absolute filesystem paths in error strings are replaced with `[path]`, so internal host layout is never exposed in API responses.

## Auth & Profiles

User registration and login are always available regardless of `REQUIRE_INFERENCE_AUTH`:

| Method | Path | Description |
|---|---|---|
| POST | `/v1/users/create` | Register a new user, returns JWT |
| POST | `/v1/users/login` | Log in, returns JWT |
| POST | `/v1/profile/create` | Create user profile (JWT required) |
| GET | `/v1/profile` | Fetch user profile (JWT required) |
| PATCH | `/v1/profile/update` | Update user profile (JWT required) |

## Interactive API docs

The server bundles an OpenAPI 3.1.1 spec and a Swagger UI:

```
http://localhost:8080/openapi
```
