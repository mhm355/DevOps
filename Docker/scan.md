# Docker Security Scanning Tools

Scan Docker images, Dockerfiles, and docker-compose files using these security tools.

---

## 1. Dockle - Container Image Linter

Checks best practices for container images.

### Option A: Install Locally
```bash
VERSION=$(curl -s "https://api.github.com/repos/goodwithtech/dockle/releases/latest" | grep '"tag_name":' | sed -E 's/.*"v([^"]+)".*/\1/')
curl -L -o dockle.deb https://github.com/goodwithtech/dockle/releases/download/v${VERSION}/dockle_${VERSION}_Linux-64bit.deb
sudo dpkg -i dockle.deb

# Usage
dockle <image-name>
```

### Option B: Run via Docker
```bash
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  goodwithtech/dockle:latest <image-name>
```

---

## 2. Hadolint - Dockerfile Linter

Validates Dockerfile syntax and best practices.

### Option A: Install Locally
```bash
wget -O hadolint https://github.com/hadolint/hadolint/releases/latest/download/hadolint-Linux-x86_64
chmod +x hadolint
sudo mv hadolint /usr/local/bin/

# Usage
hadolint Dockerfile
```

### Option B: Run via Docker
```bash
docker run --rm -i hadolint/hadolint < Dockerfile

# Or mount the file
docker run --rm -v $(pwd):/app hadolint/hadolint hadolint /app/Dockerfile
```

---

## 3. Trivy - Vulnerability Scanner

Scans images, Dockerfiles, and IaC files for vulnerabilities.

### Option A: Install Locally
```bash
sudo apt-get install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/trivy.list
sudo apt-get update && sudo apt-get install trivy

# Usage
trivy image <image-name>
trivy config Dockerfile
trivy config docker-compose.yml
```

### Option B: Run via Docker
```bash
# Scan image
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy:latest image <image-name>

# Scan Dockerfile
docker run --rm -v $(pwd):/app aquasec/trivy:latest config /app/Dockerfile

# Scan docker-compose.yml
docker run --rm -v $(pwd):/app aquasec/trivy:latest config /app/docker-compose.yml
```

---

## 4. Checkov - IaC Security Scanner

Scans Dockerfiles and docker-compose files for misconfigurations.

### Option A: Install Locally
```bash
pip install checkov

# Usage
checkov -f Dockerfile
checkov -f docker-compose.yml
checkov -d .  # Scan entire directory
```

### Option B: Run via Docker
```bash
# Scan Dockerfile
docker run --rm -v $(pwd):/app bridgecrew/checkov -f /app/Dockerfile

# Scan docker-compose.yml
docker run --rm -v $(pwd):/app bridgecrew/checkov -f /app/docker-compose.yml

# Scan entire directory
docker run --rm -v $(pwd):/app bridgecrew/checkov -d /app
```

---

## 5. Docker Scout - Built-in Docker Security

Official Docker security scanning tool (no installation needed).

### Usage (Native - included with Docker)
```bash
docker scout cves <image-name>
docker scout quickview <image-name>
docker scout recommendations <image-name>
```

