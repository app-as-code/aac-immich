<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/icon.svg">
    <img src="assets/icon.svg" alt="Immich logo" width="128">
  </picture>
</p>

<h1 align="center">aac-immich</h1>

<p align="center">Self-hosted deployment definition for Immich, a self-hosted photo and video management solution.</p>

<p align="center">
  <a href="https://github.com/immich-app/immich"><img alt="Upstream" src="https://img.shields.io/badge/upstream-immich--app%2Fimmich-blue?logo=github"></a>
  <img alt="Image" src="https://img.shields.io/badge/image-ghcr.io%2Fimmich--app%2Fimmich--server%3Av3.2.2-2496ED?logo=docker&logoColor=white">
  <a href="https://github.com/immich-app/immich/blob/main/LICENSE"><img alt="License" src="https://img.shields.io/badge/license-AGPL--3.0-green"></a>
</p>

## About

Immich is a self-hosted photo and video management application. It backs up photos and videos
from mobile devices, organizes them into albums and timelines, and offers search over the
library, including machine-learning based object and face recognition. Everything runs on your
own hardware, so the media library stays under your control.

Key features (as provided by upstream):

- Automatic backup of photos and videos from mobile apps
- Timeline, albums, sharing and search
- Machine-learning features such as smart search and face recognition
- Web interface and mobile apps

## Upstream

- Project: <https://github.com/immich-app/immich>
- Documentation: <https://immich.app/docs>
- Container image: `ghcr.io/immich-app/immich-server:v3.2.2`
- Dependencies:
  - Machine learning: `ghcr.io/immich-app/immich-machine-learning:v3.2.2-cuda` (deployed as `aac-immich-machine-learning`)
  - Database: `ghcr.io/immich-app/postgres` with VectorChord (deployed as `aac-immich-db`)
  - Cache: `docker.io/valkey/valkey:8` (deployed as `aac-immich-redis`)
- License: AGPL-3.0 (upstream project)

## Quick start (Docker Compose)

The Compose file is rendered from `service.yml` by the App-as-Code template engine. For a
manual run, the external networks used for routing must exist first:

```bash
docker network create services-secured
docker network create services-exposed

# Create stack.env with the secrets listed under "Configuration" (see below)
docker compose up -d
```

The machine-learning container requires an NVIDIA GPU and the NVIDIA container runtime, as
defined in `service.yml`.

## Configuration

Secrets are not stored in this repository. They are provided by the deployment pipeline or
must be written to `stack.env` by you. Do not commit them.

Environment variables (defaults from `service.yml`; `CHANGE_ME` must be set per deployment):

| Variable | Default | Description |
|---|---|---|
| `DB_PASSWORD` | `CHANGE_ME` (secret) | Database password used by Immich; must match `POSTGRES_PASSWORD` |
| `POSTGRES_PASSWORD` | `CHANGE_ME` (secret) | Password of the PostgreSQL superuser |
| `HOMEPAGE_WIDGET_KEY` | `CHANGE_ME` (secret) | API key for the Homepage dashboard widget |
| `IMMICH_VERSION` | `release` | Immich version tag used for the server image |
| `DB_HOSTNAME` | `aac-immich-db` | Hostname of the database container |
| `DB_PORT` | `5432` | Database port |
| `DB_USERNAME` | `postgres` | Database user |
| `DB_DATABASE_NAME` | `immich` | Database name |
| `REDIS_HOSTNAME` | `aac-immich-redis` | Hostname of the Redis (Valkey) container |
| `POSTGRES_DB` | `immich` | Database created by the PostgreSQL container |
| `POSTGRES_USER` | `postgres` | PostgreSQL superuser name |
| `POSTGRES_INITDB_ARGS` | `--data-checksums` | Arguments passed to `initdb` |
| `MACHINE_LEARNING_WORKERS` | `2` | Number of machine-learning workers |
| `VIPS_NOVECTOR` | `1` | Disable SIMD vector instructions in libvips |
| `IMMICH_IGNORE_MOUNT_CHECK_ERRORS` | `true` | Continue startup when mount checks fail (for network mounts) |
| `PUID` / `PGID` | `1000` | User and group ID the containers run as |
| `TZ` | `Europe/Berlin` | Time zone |

Ports:

| Port | Protocol | Purpose |
|---|---|---|
| `2283` | TCP | Web interface and API (`web`) |

Volumes:

| Mount | Description |
|---|---|
| `appdata` -> `/data` | Immich application data |
| Bind mount -> `/data/library` | Immich media library (host path set per deployment) |
| Bind mount -> `/mnt/familie` | Additional photo folder (host path set per deployment) |
| Bind mount -> additional photo folder | Second additional photo folder (host and container paths set in `service.yml`) |
| Bind mount -> `/etc/localtime` | Host time zone information (read from the host) |
| `model-cache` -> `/cache` | Machine-learning model cache |
| `db` -> `/var/lib/postgresql/data` | PostgreSQL data (database service) |
| `redis` -> `/data` | Redis data (cache service) |

The bind-mount host paths and the container paths for the additional photo folders are defined in `service.yml` and differ per deployment.

## Deploy with App-as-Code

1. Edit `service.yml`. It is the single source of truth for image, ports, environment,
   volumes, dependencies and per-stage overrides. Do not edit the rendered output.
2. Pushing to `main` runs the service pipeline, which renders the manifests with
   `aac-template-engine` and triggers the deployment.
3. Releases of this definition are published under <https://github.com/app-as-code/aac-immich>.

## Credits and license

Immich is developed by the Immich project and licensed under AGPL-3.0. This repository only
contains the deployment definition for it and is not a fork of the application. The logo is
provided by the Immich project; see [`assets/ICON_SOURCE.md`](assets/ICON_SOURCE.md) for the
icon source and license.
