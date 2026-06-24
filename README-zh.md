# MinIO Loong64

<p align="center"><a href="README.md">English</a> | <a href="README-zh.md">中文</a></p>

[MinIO](https://github.com/minio/minio) 是一款高性能对象存储，使用 [GNU AGPL v3](https://www.gnu.org/licenses/agpl-3.0.en.html) 许可证发布。本仓库提供 MinIO Server 和 [MinIO Client (mc)](https://github.com/minio/mc) 的 **LoongArch (loong64)** 架构构建。

## 概述

上游 MinIO 不提供官方的 loong64 二进制文件或 Docker 镜像。本项目弥补了这一缺口，通过以下方式：

- 克隆上游 MinIO 和 mc 源代码
- 应用最小化补丁以支持 loong64 构建
- 交叉编译原生 `linux/loong64` 二进制文件
- 构建多基础镜像的 Docker 镜像
- 发布带 GPG 签名的正式版本

## 特性

| 产物         | 描述                                                      |
|------------|---------------------------------------------------------|
| `minio`    | MinIO 服务器二进制文件 — `linux/loong64`                        |
| `mc`       | MinIO Client 二进制文件 — `linux/loong64`                    |
| Docker 镜像  | 多基础镜像（Anolis OS / Debian / Debian Slim），`linux/loong64` |
| GPG 签名     | 所有产物均附带分离式 `.asc` 签名                                    |
| Docker Hub | 发布时自动推送镜像至 `kubernetesloong64/minio-loong64`            |

## 分支命名规则

分支遵循 `loong64-<MINIO_RELEASE_DATE>` 格式，其中 `MINIO_RELEASE_DATE` 是上游 MinIO 发布标签的日期，采用 ISO 8601 格式：

```
loong64-YYYY-MM-DDThh-mm-ssZ
```

| 分支                             | MinIO 发布日期  |
|--------------------------------|-------------|
| `loong64-2025-04-22T22-12-26Z` | 2025-04-22  |
| `loong64-2026-02-12T20-18-48Z` | 2026-02-12  |

2026+ 分支使用固定提交引用并进行日期校验，以确保构建可复现。

> **注意：** `2025-04-22T22-12-26Z` 是上游 MinIO 最后一个支持 **Access Keys** 的版本。

## CI/CD

构建的触发条件：

- **推送**到任意 `loong64-*` 分支 → 执行 `build` 任务
- **推送** `release-loong64-*` 标签 → 执行 `build` + `release`（签名、推送 Docker 镜像、创建 GitHub Release）
- **拉取请求**目标为 `loong64-*` → 执行 `build` 任务

CI 流水线运行在 GitHub Actions 的 `debian:13` 容器中，通过 QEMU binfmt 交叉编译 `linux/loong64`。

## 产物

每个发布版本提供：

### 二进制文件

| 文件      | 描述                 |
|---------|--------------------|
| `minio` | MinIO 服务器二进制文件     |
| `mc`    | MinIO Client 二进制文件 |

### Docker 镜像

| 标签后缀           | 基础镜像               | 描述                  |
|----------------|--------------------|---------------------|
| `-anolis`      | Anolis OS 23.4     | 针对 LoongArch 优化（龙蜥） |
| `-debian`      | Debian 14 (Trixie) | 完整 Debian 环境        |
| `-debian-slim` | Debian 14 Slim     | 最小化 Debian 环境       |

## 下载与使用

### 加载 Docker 镜像

```shell
docker load -i minio-loong64-<VERSION>-anolis.tar
docker load -i minio-loong64-<VERSION>-debian.tar
docker load -i minio-loong64-<VERSION>-debian-slim.tar
```

### 从 Docker Hub 拉取

```shell
docker pull kubernetesloong64/minio-loong64:<VERSION>-anolis
docker pull kubernetesloong64/minio-loong64:<VERSION>-debian
docker pull kubernetesloong64/minio-loong64:<VERSION>-debian-slim
```

### 运行 Docker 容器

```shell
docker run -p 9000:9000 -p 9001:9001 \
  -e MINIO_ROOT_USER=admin \
  -e MINIO_ROOT_PASSWORD=password \
  kubernetesloong64/minio-loong64:<VERSION>-debian server /data --console-address ":9001"
```

## 验证发布

- 发布文件使用 GPG 签名。
- 从 [keys.openpgp.org](https://keys.openpgp.org) 下载公钥。
- 指纹：[FCF8724722CCBF9F51B1FBE376532BE7E3013105](https://keys.openpgp.org/debug?q=FCF8724722CCBF9F51B1FBE376532BE7E3013105)
- [手动下载](https://keys.openpgp.org/vks/v1/by-fingerprint/FCF8724722CCBF9F51B1FBE376532BE7E3013105)

```shell
gpg --keyserver keys.openpgp.org --recv-keys FCF8724722CCBF9F51B1FBE376532BE7E3013105
echo "FCF8724722CCBF9F51B1FBE376532BE7E3013105:6:" | gpg --import-ownertrust
```

或者，手动下载公钥文件后导入：

```shell
gpg --import /tmp/xxx
```

## 许可证

[Apache License 2.0](LICENSE)
