# RLM-jupyter

Multi-user JupyterHub deployment for a hands-on **Recursive Language Models** tutorial.
Clone this repo, run one script, and your users can log in — no passwords, no accounts to provision.

## What's Inside

```
.
├── docker-compose.yml          # Orchestrates JupyterHub + single-user containers
├── hub/
│   ├── Dockerfile              # JupyterHub image (with DockerSpawner)
│   └── jupyterhub_config.py    # Hub config: auth, spawner, resource limits
├── singleuser/
│   ├── Dockerfile              # Per-user notebook image
│   ├── requirements.txt        # Python packages (PyTorch, Transformers, etc.)
│   └── notebooks/
│       └── Recursive_Language_Models_Tutorial.ipynb
├── setup.sh                    # One-command deploy
├── teardown.sh                 # Stop / purge
├── .env.example                # Default resource limits
└── README.md
```

## Quick Start

### Prerequisites

- A Linux box with Docker and Docker Compose installed
- Sufficient RAM for your expected users (each user gets a configurable slice)

### Deploy

```bash
git clone <this-repo-url> && cd RLM-jupyter
./setup.sh
```

That's it. JupyterHub will be available at **http://your-host:8000**.

### Logging In

- Enter **any username** (e.g., your name)
- Leave the password field **blank** (or type anything)
- Click "Sign in"

Each user gets their own isolated Jupyter container with the tutorial notebook pre-loaded.

## Configuration

Copy `.env.example` to `.env` (the setup script does this automatically) and edit:

| Variable | Default | Description |
|----------|---------|-------------|
| `MEM_LIMIT` | `512M` | RAM per user container (`256M`, `1G`, `2G`, etc.) |
| `CPU_LIMIT` | `1.0` | CPU cores per user container (fractional OK: `0.5`) |
| `COMPOSE_PROJECT_NAME` | `rlm-jupyter` | Docker Compose project name |

### Sizing Guide

| Users | RAM per user | Total RAM needed |
|-------|-------------|-----------------|
| 10    | 512M        | ~6 GB (hub + 10 users) |
| 25    | 512M        | ~14 GB |
| 50    | 256M        | ~14 GB |
| 100   | 256M        | ~27 GB |

The hub itself uses ~500MB. Budget approximately `(users × MEM_LIMIT) + 1GB` for the host.

## The Tutorial Notebook

The included notebook covers:

1. **Background** — why recursive structure matters for language
2. **Recursive Neural Networks (TreeRNNs)** — the simplest recursive model, built from scratch in PyTorch
3. **Training** — training on a synthetic sentiment treebank
4. **Tree-LSTMs** — gated recursive composition
5. **Recursive Transformers** — blending attention with tree structure
6. **Visualization** — t-SNE of learned representations, per-node sentiment visualization
7. **Exercises** — negation handling, depth analysis, real SST data

## Operations

```bash
# View hub logs
docker compose logs -f jupyterhub

# Stop everything (user data preserved)
./teardown.sh

# Stop everything AND delete all user data
./teardown.sh --purge

# Restart after config change
./teardown.sh && ./setup.sh

# See running user containers
docker ps --filter "name=jupyter-"
```

## Idle Culling

User servers that are idle for **1 hour** are automatically shut down to free resources. The idle culler checks every 5 minutes. Users can restart their server by logging in again — their work is persisted in Docker volumes.

## Architecture

```
┌─────────────────────────────────────────────────┐
│                  Host Machine                    │
│                                                  │
│  ┌──────────────┐                                │
│  │  JupyterHub  │ :8000                          │
│  │  (container) │                                │
│  └──────┬───────┘                                │
│         │ DockerSpawner                          │
│         │ (talks to Docker socket)               │
│         │                                        │
│  ┌──────┴───────┐  ┌──────────────┐              │
│  │  jupyter-    │  │  jupyter-    │  ...          │
│  │  alice       │  │  bob         │              │
│  │  (512M RAM)  │  │  (512M RAM)  │              │
│  └──────────────┘  └──────────────┘              │
│                                                  │
│  Docker volumes: jupyterhub-user-alice, etc.     │
└─────────────────────────────────────────────────┘
```

## Troubleshooting

**Hub won't start**: Check `docker compose logs jupyterhub`. The most common issue is the Docker socket not being accessible — make sure `/var/run/docker.sock` exists and your user has permission.

**User container fails to spawn**: Make sure the `rlm-singleuser:latest` image was built successfully. Run `docker images | grep rlm-singleuser` to verify.

**Out of memory**: Reduce `MEM_LIMIT` in `.env` or add swap space to the host.

**Port 8000 in use**: Change the port mapping in `docker-compose.yml` (e.g., `"9000:8000"`).

## License

See [LICENSE](LICENSE).
