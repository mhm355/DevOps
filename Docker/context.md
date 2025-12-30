# Docker Context

Manage multiple Docker environments (local, remote, cloud).

## What is Docker Context?

Docker context allows you to switch between different Docker hosts without changing environment variables. Useful for:

- Managing dev/staging/production environments
- Connecting to remote Docker hosts
- Deploying to cloud services (ECS, ACI)

---

## Basic Commands

```bash
docker context ls                      # List all contexts
docker context use <context_name>      # Switch context
docker context inspect <context_name>  # View context details
docker context create <context_name>   # Create new context
docker context rm <context_name>       # Remove context
docker context show                    # Show current context name
docker context update <context_name>   # Update existing context
```

---

## Command Options Reference

### `docker context create` Options

| Flag                           | Description                                                          |
| ------------------------------ | -------------------------------------------------------------------- |
| `--docker`                     | Docker endpoint config (`host=`, `ca=`, `cert=`, `key=`, `skip-tls-verify=`) |
| `--description`                | Human-readable description                                           |
| `--from`                       | Create from existing context                                         |
| `--default-stack-orchestrator` | Default orchestrator: `swarm`, `kubernetes`, `all`                   |

### `docker context ls` Options

| Flag           | Description                                   |
| -------------- | --------------------------------------------- |
| `--format`     | Format output (e.g., `--format "{{.Name}}"`)  |
| `-q, --quiet`  | Only show context names                       |

### `docker context inspect` Options

| Flag           | Description                      |
| -------------- | -------------------------------- |
| `--format, -f` | Format output using Go template  |

### `docker context update` Options

| Flag            | Description              |
| --------------- | ------------------------ |
| `--docker`      | Update Docker endpoint   |
| `--description` | Update description       |

### `docker context rm` Options

| Flag         | Description                  |
| ------------ | ---------------------------- |
| `-f, --force` | Force remove (even if in use) |

### `docker context export` Options

| Flag           | Description               |
| -------------- | ------------------------- |
| `--kubeconfig` | Export as kubeconfig file |

### Docker Endpoint Configuration

The `--docker` flag accepts these parameters:

| Parameter          | Description             | Example                                   |
| ------------------ | ----------------------- | ----------------------------------------- |
| `host`             | Docker daemon address   | `ssh://user@host`, `tcp://host:2376`      |
| `ca`               | CA certificate path     | `/certs/ca.pem`                           |
| `cert`             | Client certificate path | `/certs/cert.pem`                         |
| `key`              | Client key path         | `/certs/key.pem`                          |
| `skip-tls-verify`  | Skip TLS verification   | `true` (not recommended)                  |

---

## Create Remote Context

### SSH Connection (Recommended)

```bash
docker context create remote-server \
  --docker "host=ssh://user@192.168.1.100" \
  --description "Production server"
```

### TCP Connection

```bash
docker context create remote-tcp \
  --docker "host=tcp://192.168.1.100:2375" \
  --description "Dev server (insecure)"
```

### TCP with TLS (Secure)

```bash
docker context create secure-remote \
  --docker "host=tcp://192.168.1.100:2376,ca=/certs/ca.pem,cert=/certs/cert.pem,key=/certs/key.pem" \
  --description "Production (TLS secured)"
```

---

## Cloud Contexts

### AWS ECS

```bash
# Create ECS context (requires AWS credentials)
docker context create ecs myecs

# Or from environment variables
docker context create ecs myecs --from-env
```

### Azure ACI

```bash
docker context create aci myaci
```

### Use with Compose

```bash
docker --context ecs compose up
docker --context aci compose up
```

---

## Environment Variables

| Variable         | Purpose                                |
| ---------------- | -------------------------------------- |
| `DOCKER_CONTEXT` | Set default context                    |
| `DOCKER_HOST`    | Override host (alternative to context) |

```bash
export DOCKER_CONTEXT=production
docker ps  # Runs on production context
```

---

## Export & Import Contexts

Share contexts between machines:

```bash
# Export context to file
docker context export mycontext > mycontext.dockercontext

# Import on another machine
docker context import mycontext mycontext.dockercontext
```

---

## Using Context with Commands

### Per-Command Override

```bash
docker --context production ps
docker --context staging logs myapp
```

### With Docker Compose

```bash
docker --context production compose up -d
docker --context staging compose down
```

---

## Practical Example: Multi-Environment Setup

```bash
# Create contexts for different environments
docker context create dev --docker "host=ssh://dev@dev-server"
docker context create staging --docker "host=ssh://deploy@staging-server"
docker context create production --docker "host=ssh://deploy@prod-server"

# Switch between environments
docker context use dev
docker ps  # Shows dev containers

docker context use production
docker ps  # Shows production containers
```

---

## Troubleshooting

```bash
# Check current context
docker context show

# Test connection
docker --context remote-server info

# Debug SSH issues
docker --context remote-server system info --debug
```
