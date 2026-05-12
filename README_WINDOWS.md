# 🪟 Windows Server — Docker Setup Guide

> Docker Engine · Docker Compose · SQL Server Connectivity
> **Platform:** Windows Server (PowerShell)

---

## 📋 About This Guide

This guide covers installing Docker Engine and Docker Compose on **Windows Server** using PowerShell. Docker Desktop is a GUI application built exclusively for developer workstations (Windows 10/11) and is **not supported on Windows Server**. Windows Server uses Docker Engine installed directly via PowerShell — no GUI involved.

---

## Platform Overview

| Feature | Windows 10/11 (PC) | Windows Server |
|---|---|---|
| Docker Desktop | ✅ Supported | ❌ Not Supported |
| Docker Engine | Via Desktop | ✅ Natively Supported |
| Linux Containers | Via WSL2 / Hyper-V | Via Hyper-V (limited) |
| Windows Containers | ✅ Supported | ✅ Supported |

---

## 1. Install Docker Engine

Open **PowerShell as Administrator** and run the following steps:

### Step 1 — Enable Windows Containers Feature

```powershell
Install-WindowsFeature -Name Containers -Restart
```

> ⚠️ The server will **reboot** after this step. Re-open PowerShell as Administrator after restart.

### Step 2 — Install Docker Engine

After reboot, install Docker Engine using the official script:

```powershell
Invoke-WebRequest -UseBasicParsing "https://raw.githubusercontent.com/microsoft/Windows-Containers/Main/helpful_tools/Install-DockerCE/install-docker-ce.ps1" -OutFile install-docker-ce.ps1

.\install-docker-ce.ps1
```

### Step 3 — Start Docker and Set to Auto-Start

```powershell
# Start the Docker service
Start-Service docker

# Configure Docker to start automatically on boot
Set-Service -Name docker -StartupType Automatic
```

### Step 4 — Verify Installation

```powershell
docker version
docker ps
```

---

## 2. Install Docker Compose

### Step 1 — Create the CLI Plugins Directory

```powershell
mkdir "$env:ProgramFiles\Docker\cli-plugins" -Force
```

### Step 2 — Download the Latest Docker Compose Binary

```powershell
Invoke-WebRequest -UseBasicParsing "https://github.com/docker/compose/releases/latest/download/docker-compose-windows-x86_64.exe" -OutFile "$env:ProgramFiles\Docker\cli-plugins\docker-compose.exe"
```

### Step 3 — Verify Installation

```powershell
docker compose version
```

---

## 3. Managing Docker on Windows Server

### PowerShell Commands

```powershell
Start-Service docker                                    # Start Docker service
Stop-Service docker                                     # Stop Docker service
Restart-Service docker                                  # Restart Docker service
Get-Service docker                                      # Check Docker service status
Set-Service -Name docker -StartupType Automatic         # Enable on boot
```

### Command Prompt (CMD) Commands

```cmd
net start docker          REM Start Docker service
net stop docker           REM Stop Docker service
sc query docker           REM Check Docker service status
sc config docker start= auto    REM Enable Docker to start on boot
sc config docker start= demand  REM Disable Docker from starting on boot
docker version            REM Verify Docker is running
docker ps
```

---

## 4. Linux Containers on Windows Server

Windows Server does not support WSL2 natively. Running Linux containers requires Hyper-V:

```powershell
# Enable Hyper-V feature
Install-WindowsFeature -Name Hyper-V -IncludeManagementTools -Restart

# Switch Docker to Linux container mode
& "C:\Program Files\Docker\Docker\DockerCli.exe" -SwitchLinuxEngine
```

> ⚠️ **Warning:** Linux containers on Windows Server via Hyper-V has **limited support** and is **not recommended for production**. For Linux containers in production, use a Linux server (Ubuntu, RHEL, etc.) instead.

---

## 5. Check SQL Server Connectivity via pyodbc in a Container

### SQL Server 2012 and Above

```powershell
docker exec -it <container_name> python -c "import pyodbc; conn = pyodbc.connect('DRIVER={SQL Server};SERVER=<ip_server_db>,1433;DATABASE=your_db;UID=your_user;PWD=your_password;'); print('Connected!')"
```

### SQL Server 2008 (Legacy)

For SQL Server 2008, the `ADDRESS` parameter must be included in the connection string:

```powershell
docker exec -it <container_name> python -c "import pyodbc; conn = pyodbc.connect('DRIVER={SQL Server};SERVER=.;ADDRESS=<ip_server_db>,1433;DATABASE=your_db;UID=your_user;PWD=your_password;'); print('Connected!')"
```

---

## 6. ODBC Driver Support — Windows

The Microsoft ODBC Driver for SQL Server is available as a standalone installer for Windows. The latest version is **18.6.2.1 (GA)**. All major versions can be installed **side by side**.

> Source: [Microsoft Learn — Download ODBC Driver for SQL Server](https://learn.microsoft.com/en-us/sql/connect/odbc/download-odbc-driver-for-sql-server?view=sql-server-ver16)

### Supported Windows Versions

| Windows Platform | Driver 18.x | Driver 17.x | Driver 13.x |
|---|:---:|:---:|:---:|
| Windows 11 (x64 / ARM64) | ✅ | ✅ | ❌ |
| Windows 10 (x64 / x86) | ✅ | ✅ | ✅ |
| Windows Server 2022 | ✅ | ✅ | ❌ |
| Windows Server 2019 | ✅ | ✅ | ❌ |
| Windows Server 2016 | ✅ | ✅ | ✅ |
| Windows Server 2012 R2 | ❌ | ✅ | ✅ |
| Windows Server 2008 R2 | ❌ | ❌ | ✅ |

### Available Architectures

| Architecture | Installer |
|---|---|
| x64 (64-bit) | `msodbcsql18-x64.msi` — installs both 64-bit and 32-bit drivers |
| x86 (32-bit only) | `msodbcsql18-x86.msi` |
| ARM64 | `msodbcsql18-arm64.msi` — installs both ARM64 and 32-bit drivers |

### SQL Server Version Compatibility (Driver 18.x)

| SQL Server / Azure Version | Supported |
|---|:---:|
| Microsoft Fabric SQL Database | ✅ |
| Azure SQL Database | ✅ |
| Azure Synapse Analytics | ✅ |
| Azure SQL Managed Instance | ✅ |
| SQL Server 2025 | ✅ |
| SQL Server 2022 | ✅ |
| SQL Server 2019 | ✅ |
| SQL Server 2017 | ✅ |
| SQL Server 2016 | ✅ |
| SQL Server 2014 | ✅ |
| SQL Server 2012 | ✅ |
| SQL Server 2008 / 2008 R2 | ❌ (use Driver 17) |

> ⚠️ **Prerequisite:** The ODBC Driver for SQL Server on Windows requires the **Microsoft Visual C++ Redistributable**. Most Windows systems already have this installed. If driver installation fails, download the matching x64/x86/ARM64 version from Microsoft before retrying.

---

## 7. Recommendations

| Use Case | Recommendation |
|---|---|
| Development on PC | Docker Desktop (Windows 10/11) |
| Windows Server workloads | Docker Engine via PowerShell |
| Linux containers in production | Use a Linux server (Ubuntu / RHEL) |
| Mixed environments | Linux VM or WSL2 on Windows PC |

---

## 💡 Tips

- Use `sc query docker` (CMD) or `Get-Service docker` (PowerShell) to check the Docker service status at any time.
- In CMD, comments use `REM` instead of `#`.
- Always run PowerShell **as Administrator** when installing or managing Docker services.
- Docker Engine on Windows Server supports **Windows containers** natively — Linux containers require Hyper-V and have limited production support.

---

*See [`README.md`](./README.md) for an overview | [`README_REFERENCE.md`](./README_REFERENCE.md) for full command reference*
