# 🐧 Linux Server Setup Guide

> Docker Engine · Docker Compose · MSSQL ODBC Driver 18
> **Platform:** Ubuntu / Debian

---

## 📋 About This Guide

This guide covers step-by-step installation of:
- **Docker Engine** — the core container runtime
- **Docker Compose** — multi-container orchestration via YAML
- **Microsoft ODBC Driver 18 for SQL Server** — enables Linux apps to connect to MSSQL / Azure SQL

---

## 1. Install Docker Engine

### 1.1 Prerequisites

Update your package index and install required dependencies:

```bash
# Update package index
sudo apt-get update

# Install required packages
sudo apt-get install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

### 1.2 Add Docker's Official GPG Key & Repository

```bash
# Create keyrings directory
sudo install -m 0755 -d /etc/apt/keyrings

# Download and save Docker's GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Set correct permissions on the key
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add Docker repository to apt sources
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Update package index with new repo
sudo apt-get update
```

> ℹ️ **Note:** For Debian, replace `ubuntu` with `debian` in the repository URL above.

### 1.3 Install Docker Engine

```bash
sudo apt-get install -y \
    docker-ce \
    docker-ce-cli \
    containerd.io \
    docker-buildx-plugin \
    docker-compose-plugin
```

### 1.4 Verify Installation

```bash
# Check Docker service is running
sudo systemctl status docker

# Run hello-world test container
sudo docker run hello-world

# Check Docker version
docker --version
```

### 1.5 (Optional) Run Docker Without `sudo`

Add your user to the `docker` group to avoid typing `sudo` each time:

```bash
# Add current user to the docker group
sudo usermod -aG docker $USER

# Apply group change (or log out and back in)
newgrp docker

# Verify — should show docker group
groups $USER
```

### Installation Summary

| Step | Action | Details |
|---|---|---|
| 1 | Update system | Run `apt-get update` to refresh package index |
| 2 | Install deps | Install `curl`, `gnupg`, `ca-certificates` |
| 3 | Add GPG key | Download and register Docker's official signing key |
| 4 | Add repo | Add Docker's stable apt repository |
| 5 | Install | Install `docker-ce` and associated plugins |
| 6 | Verify | Run hello-world container to confirm working install |

---

## 2. Install Docker Compose

Docker Compose V2 is included as a plugin when you install Docker Engine using the steps above (`docker-compose-plugin`). If you need to install or update it separately, follow the steps below.

### 2.1 Install via APT (Recommended)

If you already followed Section 1, Docker Compose is already available. Verify:

```bash
# Verify Docker Compose is available (V2 plugin syntax)
docker compose version

# Install separately if needed
sudo apt-get install -y docker-compose-plugin

# Confirm version
docker compose version
```

> ℹ️ **Note:** Docker Compose V2 uses `docker compose` (space, no hyphen). The older V1 used `docker-compose` (with hyphen). This guide uses V2 syntax.

### 2.2 Install Standalone Binary (Alternative)

If you prefer the standalone binary or are on an older system:

```bash
# Download the latest Docker Compose binary
COMPOSE_VERSION=$(curl -s https://api.github.com/repos/docker/compose/releases/latest \
  | grep '"tag_name"' | cut -d'"' -f4)

sudo curl -L "https://github.com/docker/compose/releases/download/${COMPOSE_VERSION}/docker-compose-$(uname -s)-$(uname -m)" \
  -o /usr/local/bin/docker-compose

# Make binary executable
sudo chmod +x /usr/local/bin/docker-compose

# Verify installation
docker-compose --version
```

---

## 3. Install ODBC Driver 18 for SQL Server

The Microsoft ODBC Driver 18 enables applications on Linux to connect to Microsoft SQL Server and Azure SQL Database.

### 3.1 Import Microsoft Signing Key

```bash
curl -fsSL https://packages.microsoft.com/keys/microsoft.asc \
  | sudo gpg --dearmor -o /usr/share/keyrings/microsoft-prod.gpg
```

### 3.2 Add Microsoft Package Repository

Choose the command matching your distribution:

**Ubuntu 22.04 (Jammy)**
```bash
curl https://packages.microsoft.com/config/ubuntu/22.04/prod.list \
  | sudo tee /etc/apt/sources.list.d/mssql-release.list
```

**Ubuntu 20.04 (Focal)**
```bash
curl https://packages.microsoft.com/config/ubuntu/20.04/prod.list \
  | sudo tee /etc/apt/sources.list.d/mssql-release.list
```

**Debian 12 (Bookworm)**
```bash
curl https://packages.microsoft.com/config/debian/12/prod.list \
  | sudo tee /etc/apt/sources.list.d/mssql-release.list
```

**Debian 11 (Bullseye)**
```bash
curl https://packages.microsoft.com/config/debian/11/prod.list \
  | sudo tee /etc/apt/sources.list.d/mssql-release.list
```

### 3.3 Install the Driver

```bash
# Update package list
sudo apt-get update

# Accept EULA and install ODBC Driver 18
sudo ACCEPT_EULA=Y apt-get install -y msodbcsql18

# (Optional) Install command-line tools (sqlcmd, bcp)
sudo ACCEPT_EULA=Y apt-get install -y mssql-tools18

# (Optional) Install unixODBC dev headers (for compiling apps)
sudo apt-get install -y unixodbc-dev
```

### 3.4 Add sqlcmd to PATH (Optional)

```bash
# Add mssql-tools18 to PATH permanently
echo 'export PATH="$PATH:/opt/mssql-tools18/bin"' >> ~/.bashrc
source ~/.bashrc

# Test connection to SQL Server
sqlcmd -S <server_name> -U <username> -P "<password>"
```

### 3.5 Verify the Driver is Registered

```bash
# List installed ODBC drivers
odbcinst -q -d

# Expected output:
# [ODBC Driver 18 for SQL Server]
```

### ODBC Installation Summary

| Step | Action | Details |
|---|---|---|
| 1 | Import GPG key | Download Microsoft's signing key and register it |
| 2 | Add repo | Add Microsoft's package repository for your distro/version |
| 3 | Update index | Run `apt-get update` to refresh with new repo |
| 4 | Install driver | Install `msodbcsql18` with `ACCEPT_EULA=Y` flag |
| 5 | Verify | Run `odbcinst -q -d` to confirm driver is listed |

### 3.6 ODBC Driver Support — Linux Distributions

The table below shows which Microsoft ODBC Driver version supports each Linux distribution (as of driver v18.6, the latest GA release). ✅ = Supported, ❌ = Not supported.

> Source: [Microsoft Learn — System Requirements (Linux and macOS)](https://learn.microsoft.com/en-us/sql/connect/odbc/linux-mac/system-requirements?view=sql-server-ver16)

| Linux Distribution | Driver 18.6 (Latest) | Driver 18.x | Driver 17.x |
|---|:---:|:---:|:---:|
| Ubuntu 22.04 (Jammy) | ✅ | ✅ | ✅ |
| Ubuntu 20.04 (Focal) | ✅ | ✅ | ✅ |
| Ubuntu 18.04 (Bionic) | ❌ | ✅ | ✅ |
| Ubuntu 16.04 (Xenial) | ❌ | ❌ | ✅ |
| Debian 13 | ✅ | ❌ | ❌ |
| Debian 12 (Bookworm) | ✅ | ✅ | ✅ (17.10 only) |
| Debian 11 (Bullseye) | ✅ | ✅ | ❌ |
| Debian 10 (Buster) | ❌ | ✅ | ✅ |
| Red Hat Enterprise Linux 10 | ✅ | ❌ | ❌ |
| Red Hat Enterprise Linux 9 | ✅ | ✅ | ✅ (17.9+) |
| Red Hat Enterprise Linux 8 | ✅ | ✅ | ✅ |
| Red Hat Enterprise Linux 7 | ❌ | ✅ | ✅ |
| SUSE Linux Enterprise Server 15 | ✅ | ✅ | ✅ |
| SUSE Linux Enterprise Server 12 | ❌ | ✅ | ✅ |
| Oracle Linux 10 | ✅ | ❌ | ❌ |
| Oracle Linux 9 | ✅ | ✅ | ❌ |
| Oracle Linux 8 | ✅ | ✅ | ✅ |
| Alpine Linux 3.20 / 3.21 / 3.22 | ✅ | ❌ | ❌ |
| Alpine Linux 3.18 / 3.19 | ❌ | ✅ | ❌ |
| Azure Linux 3.0 | ✅ | ✅ | ❌ |

> ⚠️ **ARM64 support** on Red Hat 8/9, Debian 11, and Ubuntu 20.04/22.04 requires **Driver 18.1 or later**.

---

## 4. Running Docker Compose YAML Files

### 4.1 Example docker-compose.yml

```yaml
services:
  web:
    build: .
    image: myapp/web:1.0.0
    ports:
      - "8080:80"
    container_name: "my-web-app"
    environment:
      APP_ENV: "production"
      APP_PORT: "8080"
      DB_HOST: "192.168.1.100"
      DB_PORT: "1433"
      DB_NAME: "my_database"
      DB_USER: "app_user"
      DB_PASSWORD: "s3cur3p@ssword"
      DB_DRIVER: "ODBC Driver 18 for SQL Server"
      TOKEN_SECRET: "my-jwt-secret-key"
      TOKEN_TTL_SECONDS: "3600"
    depends_on:
      - db

  db:
    image: mcr.microsoft.com/mssql/server:2022-latest
    container_name: "my-sql-server"
    environment:
      ACCEPT_EULA: "Y"
      SA_PASSWORD: "s3cur3p@ssword"
      MSSQL_PID: "Express"
    ports:
      - "1433:1433"
    volumes:
      - db_data:/var/opt/mssql

volumes:
  db_data:
```

### 4.2 Starting Services

```bash
# Navigate to directory containing docker-compose.yml
cd /path/to/your/project

# Start all services in detached (background) mode
docker compose up -d

# Start services and rebuild images if changed
docker compose up -d --build

# Start using a specific YML file (non-default name)
docker compose -f mystack.yml up -d
```

### 4.3 Managing Running Services

```bash
# Check status of all services
docker compose ps

# View live logs for all services
docker compose logs -f

# View logs for a specific service
docker compose logs -f webapp

# List all running containers
docker ps

# Execute a command inside a running container
docker compose exec webapp bash
```

### 4.4 Stopping & Removing Services

```bash
# Stop services (containers remain)
docker compose stop

# Stop and remove containers (volumes preserved)
docker compose down

# Stop, remove containers AND named volumes
docker compose down -v

# Stop, remove containers, volumes, and images
docker compose down -v --rmi all
```

### 4.5 Compose Command Reference

| Command | Description |
|---|---|
| `docker compose up -d` | Start all services in detached background mode |
| `docker compose up --build` | Rebuild images before starting services |
| `docker compose down` | Stop and remove containers |
| `docker compose down -v` | Stop, remove containers and volumes |
| `docker compose ps` | Show status of all services |
| `docker compose logs -f` | Tail live logs from all services |
| `docker compose logs -f <svc>` | Tail logs from a specific service |
| `docker compose stop` | Stop running containers without removing them |
| `docker compose start` | Restart stopped containers |
| `docker compose restart` | Restart all running services |
| `docker compose exec <svc> bash` | Open shell inside running container |
| `docker compose pull` | Pull latest images defined in YML |
| `docker compose config` | Validate and view merged compose config |
| `docker compose -f file.yml up -d` | Use a custom-named YML file |

---

## 5. Troubleshooting

### Docker Service Not Starting

```bash
# Check service status and recent logs
sudo systemctl status docker
sudo journalctl -u docker -n 50 --no-pager

# Restart Docker service
sudo systemctl restart docker

# Enable Docker to start on boot
sudo systemctl enable docker
```

### Permission Denied Running Docker

```bash
# If you see: permission denied while trying to connect to Docker socket
# Add user to docker group
sudo usermod -aG docker $USER

# Apply immediately (or log out and back in)
newgrp docker
```

### ODBC Driver Not Found

```bash
# Verify driver is installed
odbcinst -q -d

# Reinstall if missing
sudo ACCEPT_EULA=Y apt-get install --reinstall msodbcsql18

# Check driver file exists
ls /opt/microsoft/msodbcsql18/lib64/
```

### Container Keeps Restarting

```bash
# Check container exit code and logs
docker compose ps
docker compose logs <service_name>

# Inspect container details
docker inspect <container_id>
```

---

## ✅ Setup Complete

You now have Docker Engine, Docker Compose, and Microsoft ODBC Driver 18 for SQL Server installed on your Ubuntu/Debian Linux server. Use `docker compose up -d` from any directory containing a `docker-compose.yml` file to launch your stack.

---

*See [`README.md`](./README.md) for an overview | [`README_REFERENCE.md`](./README_REFERENCE.md) for full command reference*
