# MinIO Loong64

<p align="center"><a href="README.md">English</a> | <a href="README-zh.md">中文</a></p>

[MinIO](https://github.com/minio/minio) is a High Performance Object Storage released under the [GNU AGPL v3](https://www.gnu.org/licenses/agpl-3.0.en.html). This repository provides **LoongArch (loong64)** architecture builds of both MinIO Server and [MinIO Client (mc)](https://github.com/minio/mc).

## Overview

Upstream MinIO does not provide official loong64 binaries or Docker images. This project bridges that gap by:

- Cloning the upstream MinIO and mc source code
- Applying minimal patches for loong64 build compatibility
- Cross-compiling native `linux/loong64` binaries
- Building multi-base Docker images
- Publishing signed releases with GPG verification

## Features

| Artifact       | Description                                                             |
|----------------|-------------------------------------------------------------------------|
| `minio`        | MinIO server binary — `linux/loong64`                                   |
| `mc`           | MinIO Client binary — `linux/loong64`                                   |
| Docker images  | Multi-base images (Anolis OS / Debian / Debian Slim), `linux/loong64`   |
| GPG signatures | All artifacts are signed with detached `.asc` signatures                |
| Docker Hub     | Images pushed to `kubernetesloong64/minio-loong64` on release           |

## Branch Naming Convention

Branches follow the pattern `loong64-<MINIO_RELEASE_DATE>` where `MINIO_RELEASE_DATE` is the upstream MinIO release tag date in ISO 8601 format:

```
loong64-YYYY-MM-DDThh-mm-ssZ
```

| Branch                                 | MinIO Release Date |
|----------------------------------------|--------------------|
| `loong64-2025-04-22T22-12-26Z`         | 2025-04-22         |
| `loong64-2026-02-12T20-18-48Z`         | 2026-02-12         |

The 2026+ branches use fixed commit references with date validation to ensure build reproducibility.

> **Note:** `2025-04-22T22-12-26Z` is the last upstream MinIO release that supports **Access Keys**.

## CI/CD

Builds are triggered by:

- **Push** to any `loong64-*` branch → runs the `build` job
- **Push** of a `release-loong64-*` tag → runs `build` + `release` (signs, pushes Docker images, creates GitHub Release)
- **Pull Request** targeting `loong64-*` → runs the `build` job

The CI pipeline runs on GitHub Actions in a `debian:13` container and cross-compiles for `linux/loong64` using QEMU binfmt.

## Artifacts

Each release provides:

### Binaries

| File    | Description         |
|---------|---------------------|
| `minio` | MinIO server binary |
| `mc`    | MinIO Client binary |

### Docker Images

| Tag Suffix     | Base Image         | Description                      |
|----------------|--------------------|----------------------------------|
| `-anolis`      | Anolis OS 23.4     | Optimized for LoongArch (Anolis) |
| `-debian`      | Debian 14 (Trixie) | Full Debian environment          |
| `-debian-slim` | Debian 14 Slim     | Minimal Debian footprint         |

## Download & Usage

### Load Docker Images

```shell
docker load -i minio-loong64-<VERSION>-anolis.tar
docker load -i minio-loong64-<VERSION>-debian.tar
docker load -i minio-loong64-<VERSION>-debian-slim.tar
```

### Pull from Docker Hub

```shell
docker pull kubernetesloong64/minio-loong64:<VERSION>-anolis
docker pull kubernetesloong64/minio-loong64:<VERSION>-debian
docker pull kubernetesloong64/minio-loong64:<VERSION>-debian-slim
```

### Run Docker Container

```shell
docker run -p 9000:9000 -p 9001:9001 \
  -e MINIO_ROOT_USER=admin \
  -e MINIO_ROOT_PASSWORD=password \
  kubernetesloong64/minio-loong64:<VERSION>-debian server /data --console-address ":9001"
```

## Verifying Releases

- Releases are signed with GPG.
- Download the public key from [keys.openpgp.org](https://keys.openpgp.org).
- Fingerprint: [FCF8724722CCBF9F51B1FBE376532BE7E3013105](https://keys.openpgp.org/debug?q=FCF8724722CCBF9F51B1FBE376532BE7E3013105)
- [Manual download](https://keys.openpgp.org/vks/v1/by-fingerprint/FCF8724722CCBF9F51B1FBE376532BE7E3013105)

```shell
gpg --keyserver keys.openpgp.org --recv-keys FCF8724722CCBF9F51B1FBE376532BE7E3013105
echo "FCF8724722CCBF9F51B1FBE376532BE7E3013105:6:" | gpg --import-ownertrust
```

Or download the key file manually and import it:

```shell
gpg --import /tmp/xxx
```

## License

[Apache License 2.0](LICENSE)
