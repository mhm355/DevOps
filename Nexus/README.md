# Run Nexus Using Docker Compose

## 1. Create a directory for Nexus

```bash
mkdir -p nexus/nexus-data

sudo chown -R 200:200 nexus/nexus-data
```

## 2. Create a `docker-compose-nexus.yml` file inside the `nexus` directory
```yaml
services:
  nexus:
    image: sonatype/nexus3:latest
    container_name: nexus
    ports:
      - 8081:8081
    volumes:
      - ./nexus-data:/nexus-data
    restart: unless-stopped
    networks:
      - devops
    # Uncomment and configure the following if you want to set resource limits
    # deploy:
    #   resources:
    #     limits:
    #       cpus: '1'
    #       memory: 1g
    #     reservations:
    #       cpus: '0.5'
    #       memory: 512m
    # environment:
    #   - INSTALL4J_ADD_VM_PARAMS=-Xms1g -Xmx2g -XX:MaxDirectMemorySize=3g -Djava.util.prefs.userRoot=/nexus-data/javaprefs

networks:
  devops:
    name: devops
    driver: bridge
```

## 3. Build and run the Nexus container

```bash
docker-compose -f docker-compose-nexus.yml up -d

docker logs -f nexus
```

## 4. Access Nexus

- Wait 2–3 minutes after starting the container.

- Open: `http://localhost:8081`  

  Default username: `admin`  

- Retrieve the admin password:

```bash
docker-compose exec nexus cat /nexus-data/admin.password
```

## 5. Push Docker images to Nexus

### Step 1: Create a Docker repository in Nexus

1. Go to **Settings** (gear icon) → **Repositories**  

2. Click **Create repository**  

3. Select **docker (hosted)**  

4. Set:

   - **Repository name:** `docker-hosted`  

   - **HTTP connector port:** `8082`  

   - Enable **Docker Bearer Token Realm** if you want anonymous access  

5. Click **Create repository**

### Step 2: Configure Docker client for Nexus

```bash

# Edit Docker daemon configuration

sudo mkdir -p /etc/docker

sudo tee /etc/docker/daemon.json <<EOF
{
  "insecure-registries": ["localhost:8082"]
}
EOF

# Restart Docker daemon
sudo systemctl restart docker

# Tag the image for Nexus registry
docker tag image_name localhost:8082/image_name:tag

# Login to Nexus registry
docker login localhost:8082
# Username: admin
# Password: (Nexus admin password)

# Push the image to Nexus
docker push localhost:8082/image_name:tag
```