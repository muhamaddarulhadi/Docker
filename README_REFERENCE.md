# 📚 Docker & Docker Compose — Complete Command Reference

> Service Management · Containers · Images · Compose · Windows Server

---

## 1. Start Docker Service

Before using Docker commands, the Docker daemon must be running. The method varies by operating system.

### 🐧 Linux — systemd (Modern)

```bash
sudo systemctl start docker       # Start Docker service
sudo systemctl stop docker        # Stop Docker service
sudo systemctl restart docker     # Restart Docker service
sudo systemctl status docker      # Check Docker service status
sudo systemctl enable docker      # Enable Docker to start on boot
sudo systemctl disable docker     # Disable Docker from starting on boot
```

### 🐧 Linux — service (Older Systems)

```bash
sudo service docker start         # Start Docker service
sudo service docker stop          # Stop Docker service
sudo service docker restart       # Restart Docker service
sudo service docker status        # Check Docker service status
```

### 🍎 macOS — Docker Desktop

```bash
open -a Docker                                  # Start Docker Desktop app
osascript -e 'quit app "Docker"'               # Stop Docker Desktop app
docker info                                     # Shows info if running, error if not
docker ps                                       # Quick check — works if daemon is up
```

### 🪟 Windows — Command Prompt (CMD)

```cmd
net start docker                                REM Start Docker service
net stop docker                                 REM Stop Docker service
sc query docker                                 REM Check Docker service status
sc config docker start= auto                    REM Set Docker to auto-start on boot
start "" "C:\Program Files\Docker\Docker\Docker Desktop.exe"   REM Start Docker Desktop
docker version
docker ps
```

### 🪟 Windows — PowerShell

```powershell
Start-Service docker                            # Start Docker service
Stop-Service docker                             # Stop Docker service
Restart-Service docker                          # Restart Docker service
Get-Service docker                              # Check Docker service status
Start-Process "C:\Program Files\Docker\Docker\Docker Desktop.exe"   # Start Docker Desktop
```

### ✅ Verify Docker is Running (All Platforms)

```bash
docker info       # Full daemon info
docker version    # Client & server version
docker ps         # List containers (confirms daemon is up)
```

> 💡 **Tip:** On Linux servers, always run `sudo systemctl enable docker` so Docker starts automatically after a reboot — critical for production environments.

---

## 2. Docker Commands

### Container Operations

**Running Containers**

```bash
docker run <image>                    # Run a container
docker run -d <image>                 # Run in detached (background) mode
docker run -it <image> bash           # Run interactively with shell
docker run -p 8080:80 <image>         # Map host port 8080 → container port 80
docker run -v /host:/container <image> # Mount a volume
docker run --name myapp <image>       # Assign a name
```

**Managing Containers**

```bash
docker ps                             # List running containers
docker ps -a                          # List all containers (including stopped)
docker stop <container>               # Stop a container
docker start <container>              # Start a stopped container
docker restart <container>            # Restart a container
docker rm <container>                 # Remove a container
docker rm -f <container>              # Force remove a running container
```

### Images

```bash
docker images                         # List local images
docker pull <image>                   # Pull image from registry
docker build -t myapp:1.0 .           # Build image from Dockerfile
docker push <image>                   # Push image to registry
docker rmi <image>                    # Remove an image
docker tag <image> new:tag            # Tag an image
```

### Logs & Inspection

```bash
docker logs <container>               # View container logs
docker logs -f <container>            # Follow/stream logs
docker exec -it <container> bash      # Shell into running container
docker inspect <container>            # Full container details (JSON)
docker stats                          # Live resource usage
docker top <container>                # Running processes inside container
```

### Cleanup

```bash
docker system prune                   # Remove all unused resources
docker system prune -a                # Also remove unused images
docker volume prune                   # Remove unused volumes
docker image prune                    # Remove dangling images
```

> 💡 **Tip:** Use `docker system prune -a` periodically to free up disk space on your system.

---

## 3. Docker Compose Commands

### Core Lifecycle

```bash
docker compose up                     # Start all services
docker compose up -d                  # Start in detached mode
docker compose up --build             # Rebuild images before starting
docker compose down                   # Stop and remove containers
docker compose down -v                # Also remove volumes
docker compose restart                # Restart all services
```

### Monitoring

```bash
docker compose ps                     # List service containers
docker compose logs                   # View all service logs
docker compose logs -f                # Follow logs
docker compose logs <service>         # Logs for a specific service
docker compose top                    # Running processes
```

### Running Commands

```bash
docker compose exec <service> bash    # Shell into a running service
docker compose run <service> bash     # Run a one-off command in new container
```

### Build & Config

```bash
docker compose build                  # Build all images
docker compose build <service>        # Build a specific service
docker compose config                 # Validate and view merged config
docker compose pull                   # Pull latest images
```

### Scaling

```bash
docker compose up -d --scale web=3   # Run 3 instances of 'web' service
```

---

## 4. Custom Named Compose File (`-f` flag)

When your compose file isn't named `docker-compose.yml`, use the `-f` flag to specify the filename.

### Lifecycle with Custom File

```bash
docker compose -f <file.yml> up           # Start with custom file
docker compose -f <file.yml> up -d        # Start in detached mode
docker compose -f <file.yml> up --build   # Rebuild before starting
docker compose -f <file.yml> down         # Stop and remove
docker compose -f <file.yml> down -v      # Also remove volumes
docker compose -f <file.yml> restart      # Restart services
```

### Monitoring & Inspection with Custom File

```bash
docker compose -f <file.yml> ps               # List containers
docker compose -f <file.yml> logs             # View logs
docker compose -f <file.yml> logs -f          # Follow logs
docker compose -f <file.yml> logs <service>   # Logs for specific service
docker compose -f <file.yml> top              # Running processes
```

### Build & Exec with Custom File

```bash
docker compose -f <file.yml> build              # Build all images
docker compose -f <file.yml> build <service>    # Build specific service
docker compose -f <file.yml> exec <service> bash  # Shell into service
docker compose -f <file.yml> run <service> bash   # One-off command
docker compose -f <file.yml> config             # Validate config
```

### Real-World Examples

```bash
docker compose -f docker-compose.prod.yml up -d
docker compose -f docker-compose.staging.yml up --build
docker compose -f infra/compose.yml down -v
```

---

## 5. Multiple / Merged Compose Files

You can stack multiple files — later files override earlier ones. This is useful for environment-specific configuration.

```bash
# Merge two files (later file overrides earlier)
docker compose -f docker-compose.yml -f docker-compose.override.yml up -d
docker compose -f docker-compose.yml -f docker-compose.prod.yml up --build
docker compose -f docker-compose.yml -f docker-compose.staging.yml down
```

### Common File Pattern

| File | Purpose |
|---|---|
| `docker-compose.yml` | Base config (shared across all environments) |
| `docker-compose.override.yml` | Local dev overrides (auto-loaded, no `-f` needed) |
| `docker-compose.prod.yml` | Production overrides |
| `docker-compose.staging.yml` | Staging overrides |

> 💡 **Tip:** `docker-compose.override.yml` is automatically merged with `docker-compose.yml` — no `-f` flag needed for that one.

---

## 6. Example docker-compose.yml

A typical compose file with a web application and database:

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
      DB_PORT: "5432"
      DB_NAME: "my_database"
      DB_USER: "app_user"
      DB_PASSWORD: "s3cur3p@ssword"
      TOKEN_SECRET: "my-jwt-secret-key"
      TOKEN_TTL_SECONDS: "3600"
    depends_on:
      - db

  db:
    image: postgres:16
    container_name: "my-postgres"
    environment:
      POSTGRES_DB: "my_database"
      POSTGRES_USER: "app_user"
      POSTGRES_PASSWORD: "s3cur3p@ssword"
    ports:
      - "5432:5432"
    volumes:
      - db_data:/var/lib/postgresql/data

volumes:
  db_data:
```

---

## 7. Quick Tips & Reminders

| Tip | Details |
|---|---|
| `docker compose` vs `docker-compose` | Use `docker compose` (space) for the modern CLI plugin; `docker-compose` (hyphen) for the older standalone version |
| Override file auto-load | `docker-compose.override.yml` is automatically merged with `docker-compose.yml` — no `-f` flag needed |
| Production detached mode | Always use `-d` (detached) in production so containers run in the background |
| Volume deletion warning | Use `down -v` carefully — it permanently deletes volumes and all stored data |
| Disk space cleanup | Run `docker system prune -a` periodically to reclaim disk space from unused images and containers |
| Linux server boot | Run `sudo systemctl enable docker` on Linux so Docker starts automatically after every reboot |
| Windows CMD note | Use `net start docker` and `net stop docker` in CMD; `sc query docker` to check status; comments use `REM` instead of `#` |
| Windows Server note | Docker Desktop is **NOT** supported on Windows Server — use Docker Engine directly via PowerShell or CMD |

---

*See [`README.md`](./README.md) for an overview | [`README_LINUX.md`](./README_LINUX.md) for Linux setup | [`README_WINDOWS.md`](./README_WINDOWS.md) for Windows Server setup*
