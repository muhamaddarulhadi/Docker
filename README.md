# 🐳 Docker Documentation Hub

Docker is an open platform for developing, shipping, and running applications inside **containers** — lightweight, portable, self-sufficient environments that package your code along with all its dependencies. Containers ensure your app runs the same way regardless of where it's deployed: on your laptop, a staging server, or in production.

---

## 📖 What is Docker?

| Concept | Description |
|---|---|
| **Container** | A runnable instance of an image — isolated, lightweight, and portable |
| **Image** | A read-only blueprint used to create containers (like a class vs. an object) |
| **Docker Engine** | The core runtime daemon that builds and runs containers |
| **Docker Compose** | A tool for defining and running multi-container applications via a YAML file |
| **Registry** | A repository for Docker images (e.g. Docker Hub, private registries) |

Docker eliminates the classic *"it works on my machine"* problem. Once containerized, your application and its environment travel together.

---

## 📂 Documentation Index

| File | Description |
|---|---|
| [`README_LINUX.md`](./README_LINUX.md) | Install Docker Engine, Docker Compose, and MSSQL ODBC Driver on Ubuntu/Debian Linux |
| [`README_WINDOWS.md`](./README_WINDOWS.md) | Install Docker Engine and Docker Compose on Windows Server via PowerShell |
| [`README_REFERENCE.md`](./README_REFERENCE.md) | Complete command reference — containers, images, compose, service management |

---

## 🚀 Quick Start

Once Docker is installed (see platform guides above), verify it's working:

```bash
docker version          # Check client & server version
docker info             # Full daemon information
docker run hello-world  # Run a test container
```

To start a multi-container app defined in a `docker-compose.yml`:

```bash
docker compose up -d
```

---

## 🖥️ Platform Support

| Platform | Recommended Approach |
|---|---|
| Ubuntu / Debian Linux | Docker Engine via `apt` (see `README_LINUX.md`) |
| Windows Server | Docker Engine via PowerShell (see `README_WINDOWS.md`) |
| Windows 10/11 PC | Docker Desktop (GUI application) |
| macOS | Docker Desktop |

> ⚠️ **Note:** Docker Desktop is **not** supported on Windows Server. Use Docker Engine installed directly via PowerShell instead.

---

## 💡 Key Tips

- Always use `docker compose` (with a space) — this is the modern V2 CLI plugin. The older `docker-compose` (with hyphen) is the legacy V1 standalone binary.
- On Linux servers, run `sudo systemctl enable docker` so Docker starts automatically after every reboot.
- Use `docker system prune -a` periodically to reclaim disk space from unused images and containers.
- Use `-d` (detached) mode in production so containers run in the background.
- Be careful with `docker compose down -v` — it permanently deletes named volumes and all stored data.
