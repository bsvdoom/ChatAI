# LobeHub + Ollama Docker Stack

Self-hosted LobeHub setup with local Ollama models.

## Stack

- LobeHub
- Ollama + NVIDIA GPU
- PostgreSQL
- Redis
- RustFS S3-compatible storage
- RustFS CLI for bucket initialization

Server-side database + S3 storage is required for full file upload support in LobeHub.

## Start

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

Generate safe secrets/passwords with:

```bash
openssl rand -hex 24
openssl rand -base64 32
```

Avoid special URI characters in `POSTGRES_PASSWORD`, or percent-encode them, because the password is part of `DATABASE_URL`.

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

File uploads from LobeHub require CORS to be enabled for the `lobe` bucket in RustFS.

Open the RustFS Console (`http://<host>:9001`), select the `lobe` bucket, enable **Bucket CORS**, and allow the LobeHub origin (for example `http://192.168.100.22:3210`) with `GET`, `POST`, `PUT`, `DELETE`, and `HEAD`.

You can verify it with:

```bash
curl -i \
  -H "Origin: http://192.168.1.22:3210" \
  http://192.168.1.22:9000/
```

The response should include Access-Control-Allow-Origin for the LobeHub origin.

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
