# Docker Buildx

Extended build capabilities with BuildKit.

## What is Buildx?

Buildx is a Docker CLI plugin that extends `docker build` with:

- Multi-platform image builds (ARM, AMD64, etc.)
- Advanced caching options
- Multiple builder instances
- Build attestations for supply chain security

---

## Basic Usage

### Create a Builder Instance

```bash
docker buildx create --name mybuilder --driver docker-container --bootstrap --use
```

#### `docker buildx create` Options

| Flag              | Description                                              |
| ----------------- | -------------------------------------------------------- |
| `--name`          | Builder instance name                                    |
| `--driver`        | Driver: `docker`, `docker-container`, `kubernetes`, `remote` |
| `--driver-opt`    | Driver-specific options                                  |
| `--platform`      | Fixed platforms for builder                              |
| `--bootstrap`     | Boot builder after creation                              |
| `--use`           | Set as current builder                                   |
| `--node`          | Create/modify node for multi-node builder                |
| `--buildkitd-flags` | Flags for BuildKit daemon                              |

### List & Inspect Builders

```bash
docker buildx ls
docker buildx inspect mybuilder
docker buildx inspect --bootstrap  # Start builder if stopped
```

---

## `docker buildx build` Options

| Flag              | Description                                              |
| ----------------- | -------------------------------------------------------- |
| `--platform`      | Target platforms (e.g., `linux/amd64,linux/arm64`)       |
| `--tag, -t`       | Image name and tag                                       |
| `--file, -f`      | Dockerfile path                                          |
| `--target`        | Build specific stage from multi-stage Dockerfile         |
| `--push`          | Push image to registry                                   |
| `--load`          | Load image into local Docker                             |
| `--no-cache`      | Do not use cache                                         |
| `--pull`          | Always pull base images                                  |
| `--build-arg`     | Set build-time variables                                 |
| `--secret`        | Mount secret file (not stored in image)                  |
| `--ssh`           | SSH agent socket or keys                                 |
| `--cache-from`    | External cache sources                                   |
| `--cache-to`      | Cache export destinations                                |
| `--output, -o`    | Output destination (`type=local,dest=path`)              |
| `--progress`      | Progress output: `auto`, `plain`, `tty`                  |
| `--sbom`          | Generate SBOM attestation                                |
| `--provenance`    | Generate provenance attestation                          |
| `--metadata-file` | Write build metadata to file                             |
| `--attest`        | Attestation parameters                                   |
| `--builder`       | Use specific builder instance                            |

---

## `docker buildx bake` Options

| Flag              | Description                                              |
| ----------------- | -------------------------------------------------------- |
| `-f, --file`      | Config file path                                         |
| `--set`           | Override target value (e.g., `--set target.args.FOO=bar`) |
| `--push`          | Push to registry                                         |
| `--load`          | Load into local Docker                                   |
| `--print`         | Print resolved config without building                   |
| `--no-cache`      | Do not use cache                                         |
| `--pull`          | Always pull images                                       |
| `--progress`      | Progress output type                                     |
| `--metadata-file` | Write build metadata to file                             |

---

## `docker buildx imagetools` Options

```bash
docker buildx imagetools inspect user/app:latest  # Inspect multi-platform image
docker buildx imagetools create --tag user/app:v2 user/app:v1  # Create from existing
```

| Flag          | Description                 |
| ------------- | --------------------------- |
| `--dry-run`   | Show what would be created  |
| `--tag`       | Set reference for new image |
| `--append`    | Append to existing manifest |
| `--format`    | Format output               |

---

## Multi-Platform Builds

Build for multiple architectures simultaneously:

```bash
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  --tag user/app:v1 \
  --push .
```

| Flag         | Description                                   |
| ------------ | --------------------------------------------- |
| `--platform` | Target platforms (comma-separated)            |
| `--push`     | Push to registry after build                  |
| `--load`     | Load into local Docker (single platform only) |

---

## Build Drivers

| Driver             | Use Case                               |
| ------------------ | -------------------------------------- |
| `docker`           | Default, single platform only          |
| `docker-container` | Multi-platform, full BuildKit features |
| `kubernetes`       | Build on Kubernetes pods               |
| `remote`           | Connect to remote BuildKit daemon      |

```bash
docker buildx create --name k8s-builder --driver kubernetes
```

---

## Caching

### Registry Cache

```bash
docker buildx build \
  --cache-to type=registry,ref=user/app:cache,mode=max \
  --cache-from type=registry,ref=user/app:cache \
  --push .
```

### Local Cache

```bash
docker buildx build \
  --cache-to type=local,dest=/tmp/buildcache \
  --cache-from type=local,src=/tmp/buildcache \
  .
```

### GitHub Actions Cache

```bash
docker buildx build \
  --cache-to type=gha,mode=max \
  --cache-from type=gha \
  .
```

### S3 Cache (AWS)

```bash
docker buildx build \
  --cache-to type=s3,region=us-east-1,bucket=my-cache \
  --cache-from type=s3,region=us-east-1,bucket=my-cache \
  .
```

| Cache Mode   | Description                                    |
| ------------ | ---------------------------------------------- |
| `mode=min`   | Cache only final build layers                  |
| `mode=max`   | Cache all intermediate layers (larger, faster) |

---

## BuildKit Features

### Secrets Mounting

Mount secrets without baking into image:

```bash
docker buildx build --secret id=mysecret,src=secret.txt .
```

In Dockerfile:

```dockerfile
RUN --mount=type=secret,id=mysecret cat /run/secrets/mysecret
```

### SSH Forwarding

Forward SSH agent for private repo access:

```bash
docker buildx build --ssh default .
```

In Dockerfile:

```dockerfile
RUN --mount=type=ssh git clone git@github.com:private/repo.git
```

### Build Attestations (Supply Chain Security)

```bash
docker buildx build \
  --sbom=true \
  --provenance=true \
  --push .
```

---

## Bake (Multi-Target Builds)

Build multiple targets from a config file.

### docker-bake.hcl

```hcl
group "default" {
  targets = ["backend", "frontend"]
}

target "base" {
  context = "."
  platforms = ["linux/amd64", "linux/arm64"]
}

target "backend" {
  inherits = ["base"]
  dockerfile = "backend.Dockerfile"
  tags = ["user/backend:latest"]
  cache-to = ["type=registry,ref=user/backend:cache,mode=max"]
  cache-from = ["type=registry,ref=user/backend:cache"]
}

target "frontend" {
  inherits = ["base"]
  dockerfile = "frontend.Dockerfile"
  tags = ["user/frontend:latest"]
  args = {
    REACT_APP_ENV = "production"
  }
  cache-to = ["type=registry,ref=user/frontend:cache,mode=max"]
  cache-from = ["type=registry,ref=user/frontend:cache"]
}
```

### Run Bake

```bash
docker buildx bake                    # Build all targets in group "default"
docker buildx bake backend            # Build specific target
docker buildx bake -f docker-bake.hcl # Specify config file
docker buildx bake --push             # Build and push all
```

Supported config files: `docker-bake.hcl`, `docker-bake.json`, `docker-compose.yml`

---

## Debugging Builds

```bash
# Verbose output
docker buildx build --progress=plain .

# Interactive debug on failure
docker buildx debug --on=error .

# Print build info without building
docker buildx bake --print
```
