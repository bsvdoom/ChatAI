# LobeHub + Ollama Docker Stack

Self-hosted LobeHub setup with local Ollama models, intended for a personal home server on a trusted LAN.

This configuration intentionally:

- publishes LobeHub on port `3210`;
- publishes the RustFS S3 API and console on ports `9000` and `9001`;
- publishes the Ollama API on port `11434` for direct LAN access;
- allows anyone who can reach LobeHub to create an account.

The default setup is not intended for direct exposure to the public Internet. Docker port mappings without a host address normally listen on all host interfaces; they are not automatically limited to the LAN. Use host and router firewall rules appropriate for your network, and do not forward these ports directly from the Internet. An Internet-facing deployment needs additional controls such as HTTPS, a reverse proxy, and deliberate access restrictions.

## Stack

- LobeHub
- Ollama + NVIDIA GPU
- PostgreSQL
- Redis
- RustFS S3-compatible storage
- RustFS CLI for bucket initialization

Server-side database and S3 storage are required for full file upload support in LobeHub.

## Prerequisites

- Docker Engine with Docker Compose v2
- A supported NVIDIA GPU and working NVIDIA driver
- NVIDIA Container Toolkit installed and configured for Docker
- The `nvidia` container runtime registered with Docker

The current Compose file uses `runtime: nvidia`. It does not use Compose GPU device reservations, so Docker must recognize a runtime named `nvidia`; otherwise the Ollama container will not start. Follow the NVIDIA Container Toolkit installation instructions for your operating system, run the toolkit's Docker runtime configuration step, and restart Docker if those instructions require it.

Docker's general Compose GPU documentation is available at <https://docs.docker.com/compose/how-tos/gpu-support/>.

## Start

First create the local environment file:

```bash
cp .env.example .env
```

Edit `.env` before starting the stack. Replace every `MY_...` example value with a real value; the Compose required-variable checks reject missing or empty values, but they cannot detect a non-empty placeholder such as `MY_AUTH_SECRET`.

After completing the required settings, optionally pull the current images and start the stack:

```bash
docker compose pull
docker compose up -d
```

Check status:

```bash
docker compose ps
```

Logs:

```bash
docker compose logs --tail=100 lobe-chat
```

## Environment

Secrets and machine-specific settings are stored in `.env`.

Generate safe secrets and passwords with:

```bash
openssl rand -hex 24
openssl rand -base64 32
```

Avoid special URI characters in `POSTGRES_PASSWORD`, or percent-encode them, because the password is part of `DATABASE_URL`.

### Host IP address

The examples use `192.168.1.22` as the Docker host's LAN address. Replace that address in `.env` with the actual LAN address of your machine in all of these values:

- `APP_URL`
- `OLLAMA_ORIGINS`
- `RUSTFS_CORS_ALLOWED_ORIGINS`
- `S3_ENDPOINT`
- `S3_PUBLIC_DOMAIN`

Keep `OLLAMA_PROXY_URL=http://ollama:11434`: `ollama` is the service name used inside the Compose network, not a host address for browsers.

The URL in `APP_URL` is also the LobeHub origin used by the Ollama and RustFS CORS settings. An origin consists of the scheme, host, and port, so `http://192.168.1.22:3210` and `http://localhost:3210` are different origins.

### Required settings

The current base configuration requires these non-empty values:

- `APP_URL`: browser-visible LobeHub URL
- `LOBE_DB_NAME`: PostgreSQL database name
- `POSTGRES_PASSWORD`: PostgreSQL password
- `KEY_VAULTS_SECRET`: LobeHub key-vault encryption secret
- `AUTH_SECRET`: LobeHub authentication/session secret
- `OLLAMA_PROXY_URL`: Ollama URL used by LobeHub inside Compose
- `OLLAMA_ORIGINS`: browser origin allowed by Ollama
- `RUSTFS_ACCESS_KEY`: RustFS access key
- `RUSTFS_SECRET_KEY`: RustFS secret key
- `RUSTFS_CORS_ALLOWED_ORIGINS`: LobeHub origin allowed by the RustFS S3 listener
- `S3_ENDPOINT`: RustFS endpoint reachable using the configured host address
- `S3_PUBLIC_DOMAIN`: browser-visible RustFS address

### Optional settings

OpenAI integration is optional. To use only local Ollama models, leave `OPENAI_API_KEY` and `OPENAI_PROXY_URL` empty or comment them out in `.env`. To enable OpenAI, replace the example API key and keep or adjust the proxy URL as required.

`OLLAMA_KEEP_ALIVE` is also optional. If omitted, the Compose file uses `5m`; `.env.example` selects `2m` for this setup.

## Network and registration behavior

Ports `3210`, `9000`, `9001`, and `11434` are intentionally published by this Compose file. Without an explicit bind address, Docker normally publishes them on all host addresses, including addresses that may be reachable beyond the trusted LAN depending on firewall, routing, and IPv6 configuration.

LobeHub registration is intentionally open: anyone who can reach port `3210` can create an account. The Ollama API on port `11434` is also intentionally reachable directly from the LAN. These choices assume a trusted home network and are not suitable defaults for an untrusted or public network.

See Docker's port-publishing documentation for the exact host exposure behavior: <https://docs.docker.com/engine/network/port-publishing/>.

## SSRF trade-off

The LobeHub container currently uses:

```yaml
SSRF_ALLOW_PRIVATE_IP_ADDRESS: "1"
```

This allows LobeHub to connect to private-network services used by this stack, but it also relaxes LobeHub's protection against server-side requests to private IP addresses. Keep this setting only in the intended trusted-LAN deployment and do not expose the stack directly to untrusted networks.

## Ollama models

Pull a model manually:

```bash
docker exec -it ollama ollama pull qwen3:8b
```

List installed models:

```bash
docker exec -it ollama ollama list
```

Check currently loaded models:

```bash
docker exec -it ollama ollama ps
```

Enabling a model in LobeHub does not automatically mean it exists in the local Ollama instance. A `404 model not found` error usually means the model still needs to be pulled.

See `MODELS.md` for a quick model selection guide.

## VRAM unloading

Ollama keeps models loaded for a short period after the last request.

Configured through `.env`:

```env
OLLAMA_KEEP_ALIVE=2m
```

and passed to the Ollama container:

```yaml
environment:
  OLLAMA_KEEP_ALIVE: ${OLLAMA_KEEP_ALIVE:-5m}
```

Examples:

```text
1m   unload after 1 minute
2m   unload after 2 minutes
5m   Ollama default
0    unload immediately
```

For a 12 GB GPU with several local models, `2m` is a reasonable compromise between fast model reuse and freeing VRAM.

## Installation notes / issues encountered

### RustFS CORS

File uploads from LobeHub require CORS to be enabled manually for the `lobe` bucket. The `rustfs-init` service creates the bucket but does not configure its bucket CORS policy.

After the stack is running:

1. Open the RustFS Console at `http://192.168.1.22:9001`, replacing the example address with your Docker host's LAN IP.
2. Sign in with `RUSTFS_ACCESS_KEY` and `RUSTFS_SECRET_KEY` from your local `.env`.
3. Select the `lobe` bucket.
4. Enable **Bucket CORS**.
5. Add the exact LobeHub origin from `APP_URL`, for example `http://192.168.1.22:3210`.
6. Allow `GET`, `POST`, `PUT`, `DELETE`, and `HEAD`.

The allowed origin must identify the LobeHub page making the browser request, not the RustFS address. Keep `RUSTFS_CORS_ALLOWED_ORIGINS` aligned with the same LobeHub origin. Do not use a path or trailing route in the origin value.

You can inspect the preflight response with:

```bash
curl -i -X OPTIONS \
  -H "Origin: http://192.168.1.22:3210" \
  -H "Access-Control-Request-Method: PUT" \
  http://192.168.1.22:9000/lobe/test
```

For that configured origin, the response should include an appropriate `Access-Control-Allow-Origin` header and allow the requested method. This command is a diagnostic example; it does not configure CORS.

RustFS CORS documentation: <https://docs.rustfs.com/en/administration/cors>.

### LobeHub database migration failed with `Invalid URL`

Example:

```text
Database migrate failed
TypeError: Invalid URL
```

The PostgreSQL password contained characters that made the generated `DATABASE_URL` invalid.

Fix: use a URL-safe password, for example:

```bash
openssl rand -hex 24
```

For a brand-new database volume, recreate the stack after changing the initial PostgreSQL password:

```bash
docker compose down -v
docker compose up -d
```

Do not use `down -v` on an existing installation unless deleting the database/storage volumes is intentional.

### Ollama returned `model ... not found`

Models must be downloaded into Ollama separately:

```bash
docker exec -it ollama ollama pull MODEL_NAME
```

Example:

```bash
docker exec -it ollama ollama pull qwen3:14b
```

## Useful commands

```bash
# Stack status
docker compose ps

# LobeHub logs
docker compose logs -f lobe-chat

# Ollama logs
docker compose logs -f ollama

# Installed models
docker exec -it ollama ollama list

# Loaded models / VRAM usage
docker exec -it ollama ollama ps

# Restart the stack
docker compose restart
```
