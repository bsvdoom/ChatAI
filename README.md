# ChatAI

Simple local AI chat stack using **LobeChat** and **Ollama** with optional **NVIDIA GPU acceleration**.

## Components

- LobeChat web UI
- Ollama local model runtime
- Docker Compose
- Optional NVIDIA GPU acceleration

LobeChat supports self-hosted Docker deployment and Ollama integration:
- https://lobehub.com/docs/self-hosting/platform/docker
- https://lobehub.com/docs/usage/providers/ollama

Ollama provides an official Docker image and supports NVIDIA GPU acceleration:
- https://docs.ollama.com/docker
- https://github.com/ollama/ollama

## Requirements

- Docker Engine
- Docker Compose plugin
- NVIDIA driver + NVIDIA Container Toolkit if GPU acceleration is used

Docker Compose installation:
- https://docs.docker.com/compose/install/

NVIDIA Container Toolkit installation:
- https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html

## Configuration

Copy the example environment file:

```bash
cp .env.example .env
```

Edit `.env` and set the required values.

Example:

```env
OPENAI_API_KEY=
OPENAI_PROXY_URL=
LOBE_ACCESS_CODE=
```

`LOBE_ACCESS_CODE` is not used ATM.

Docker Compose supports `.env` files for variable interpolation:
- https://docs.docker.com/compose/how-tos/environment-variables/variable-interpolation/

Do not commit the real `.env` file if it contains credentials or API keys.

## Start

```bash
docker compose up -d
```

Check container status:

```bash
docker compose ps
```

Follow logs:

```bash
docker compose logs -f
```

Open LobeChat in the browser:

```text
http://localhost:3210
```

Docker Compose command reference:
- https://docs.docker.com/reference/cli/docker/compose/

## Ollama configuration

Inside the Docker network, LobeChat can reach Ollama by its Compose service name:

```text
http://ollama:11434
```

Docker Compose provides DNS-based service discovery between containers on the same Compose network:
- https://docs.docker.com/compose/how-tos/networking/

In LobeChat, use:

```text
Interface proxy address: http://ollama:11434
```

When using the Docker-internal hostname above, keep **Client-Side Fetching Mode disabled**, because the hostname `ollama` is resolvable inside the Docker network rather than directly by the host browser.

## Check installed Ollama models

```bash
docker compose exec ollama ollama list
```

The Ollama CLI supports listing locally installed models:
- https://docs.ollama.com/cli

To pull a model:

```bash
docker compose exec ollama ollama pull llama3.1:8b
```

Use the exact installed model name/tag shown by `ollama list`.

## NVIDIA GPU acceleration

### Host prerequisites

Install a supported NVIDIA driver first, then install the NVIDIA Container Toolkit.

On Debian/Ubuntu-based systems, after configuring NVIDIA's repository, the toolkit package can be installed with:

```bash
sudo apt-get install -y nvidia-container-toolkit
```

Configure the Docker runtime:

```bash
>>>>>>> 66b14be (finalize readme)
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```

<<<<<<< HEAD
### DISABLE NVIDIA GPU ACCELERATION

Comment out the `runtime: nvidia` line from `docker-compose.yml`
=======
Official NVIDIA instructions:
- https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html

### Verify GPU access

On the host:

```bash
nvidia-smi
```

Inside the Ollama container:

```bash
docker compose exec ollama nvidia-smi
```

You can also inspect currently loaded Ollama models:

```bash
docker compose exec ollama ollama ps
```

Ollama documents NVIDIA GPU support for Docker:
- https://docs.ollama.com/docker

### Disable NVIDIA GPU acceleration

If `compose.yml` contains:

```yaml
runtime: nvidia
```

remove or comment out that line, then recreate the containers:

```bash
docker compose down
docker compose up -d
```

Docker also supports GPU device reservations in Compose:
- https://docs.docker.com/compose/how-tos/gpu-support/

## Useful commands

Restart the stack:

```bash
docker compose restart
```

Stop the stack:

```bash
docker compose down
```

Pull newer container images:

```bash
docker compose pull
docker compose up -d
```

Show Ollama logs:

```bash
docker compose logs -f ollama
```

Show LobeChat logs:

```bash
docker compose logs -f lobe-chat
```

## Persistent Ollama data

A typical Ollama volume mapping is:

```yaml
volumes:
  - ./ollama:/root/.ollama
```

Ollama's official Docker documentation persists data under `/root/.ollama`:
- https://docs.ollama.com/docker

The local `ollama/` directory should normally be excluded from Git because it may contain large downloaded model files.

## Security notes

Keep secrets such as API keys and access codes outside the committed Compose file where possible.

Recommended files to commit:

```text
compose.yml
.env.example
.gitignore
README.md
```

Recommended files/directories to ignore:

```text
.env
ollama/
```

Docker documentation covers environment-variable handling and secrets:
- https://docs.docker.com/compose/how-tos/environment-variables/best-practices/
- https://docs.docker.com/compose/how-tos/use-secrets/

## Update

```bash
docker compose pull
docker compose up -d
```

Optionally remove unused Docker images:

```bash
docker image prune
```

Docker image management documentation:
- https://docs.docker.com/reference/cli/docker/image/prune/

---

## References

- LobeChat self-hosting: https://lobehub.com/docs/self-hosting/platform/docker
- LobeChat Ollama integration: https://lobehub.com/docs/usage/providers/ollama
- Ollama Docker: https://docs.ollama.com/docker
- Ollama CLI: https://docs.ollama.com/cli
- Docker Compose networking: https://docs.docker.com/compose/how-tos/networking/
- Docker Compose GPU support: https://docs.docker.com/compose/how-tos/gpu-support/
- NVIDIA Container Toolkit: https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html
