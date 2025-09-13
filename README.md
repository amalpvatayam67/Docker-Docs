
# Docker Setup Guide (Linux, Windows, macOS)

A concise, copy‑paste‑friendly README to install Docker and run your first container on **Linux**, **Windows**, and **macOS**.

---

## Table of Contents
- [Quick Checks](#quick-checks)
- [Windows (Docker Desktop + WSL 2)](#windows-docker-desktop--wsl-2)
- [macOS (Docker Desktop)](#macos-docker-desktop)
- [Linux (Docker Engine)](#linux-docker-engine)
  - [Ubuntu / Debian](#ubuntu--debian)
  - [Fedora / CentOS Stream / RHEL](#fedora--centos-stream--rhel)
  - [Arch Linux / Manjaro](#arch-linux--manjaro)
  - [openSUSE Leap / Tumbleweed](#opensuse-leap--tumbleweed)
- [Post‑Install (all OSes)](#postinstall-all-oses)
- [Docker Compose v2](#docker-compose-v2)
- [GPU Support (NVIDIA)](#gpu-support-nvidia)
- [HTTP Proxy Setup (Linux)](#http-proxy-setup-linux)
- [Optional Daemon Tunings](#optional-daemon-tunings)
- [Rootless Docker (Linux optional)](#rootless-docker-linux-optional)
- [Troubleshooting](#troubleshooting)
- [Uninstall](#uninstall)

---

## Quick Checks
- **Admin rights** on your machine.
- **Virtualization enabled** in BIOS/UEFI.
- Minimum free disk: **10 GB**.
- Ports 80/443 not blocked by other software.

To verify Docker after install (all OSes):
```bash
docker version
docker run --rm hello-world
docker info
```

---

## Windows (Docker Desktop + WSL 2)
> Works on Windows 10/11 64‑bit. Recommended path uses **WSL 2**.

1) **Enable WSL 2 & Virtual Machine Platform** (run PowerShell as Administrator):
```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
wsl --install
wsl --set-default-version 2
```
2) **Install Docker Desktop for Windows**  
   - Download & run the installer. During setup, keep **“Use WSL 2 instead of Hyper‑V”** checked.
3) **Start Docker Desktop** → Settings → **Resources ▸ WSL Integration** → enable your distro(s).
4) **Test** in Windows Terminal / PowerShell:
```powershell
docker run --rm hello-world
docker compose version
```
**Tips**
- If `docker` not found, sign out/in or ensure PATH integration is enabled in Docker Desktop settings.
- Reset stuck WSL: `wsl --shutdown` then relaunch Docker Desktop.

---

## macOS (Docker Desktop)
> Works on macOS 12+ (Monterey or newer), Apple Silicon (M‑series) or Intel.

**Install (choose one):**
- **Homebrew (recommended):**
```bash
brew install --cask docker
open -a Docker
```
- **DMG:** Download Docker.dmg → drag **Docker.app** into Applications → launch Docker.

**Test:**
```bash
docker run --rm hello-world
docker compose version
```

**Notes**
- On Apple Silicon, most images are multi‑arch. If you need x86_64, run with emulation: `--platform linux/amd64`.

---

## Linux (Docker Engine)

### Uninstall old Docker if present
```bash
sudo apt-get remove -y docker docker-engine docker.io containerd runc || true
sudo dnf remove -y docker docker-engine docker.io containerd runc || true
sudo zypper rm -y docker docker.io containerd runc || true
```

### Ubuntu / Debian
```bash
# 1) Prereqs
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg

# 2) Add Docker’s official GPG key
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# 3) Add repository
echo   "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")   $(. /etc/os-release; echo "$VERSION_CODENAME") stable" |   sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 4) Install
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 5) Enable + start
sudo systemctl enable --now docker

# 6) (Optional) Run as non-root
sudo usermod -aG docker $USER
newgrp docker  # or log out/in
```

### Fedora / CentOS Stream / RHEL
```bash
# 1) Prereqs
sudo dnf -y install dnf-plugins-core

# 2) Add repo
sudo dnf config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo || sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# 3) Install
sudo dnf -y install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 4) Enable + start
sudo systemctl enable --now docker

# 5) (Optional) Run as non-root
sudo usermod -aG docker $USER
newgrp docker
```

### Arch Linux / Manjaro
```bash
sudo pacman -Syu --noconfirm
sudo pacman -S --noconfirm docker docker-compose
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
newgrp docker
```

### openSUSE Leap / Tumbleweed
```bash
sudo zypper refresh
sudo zypper install -y docker docker-compose
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
newgrp docker
```

---

## Post‑Install (all OSes)
**Sanity check**
```bash
docker run --rm hello-world
docker pull nginx:latest
docker run -d -p 8080:80 --name web nginx
curl -I http://localhost:8080
docker ps
docker logs web --tail=20
docker stop web && docker rm web
```

---

## Docker Compose v2
Compose v2 is included as the `docker compose` subcommand (no hyphen).
```bash
docker compose version
# Quick sample:
mkdir myapp && cd myapp
cat > docker-compose.yml <<'YAML'
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
YAML
docker compose up -d
docker compose ps
docker compose down
```

---

## GPU Support (NVIDIA)
**Linux host**
```bash
# Install latest NVIDIA driver for your GPU
# Then install NVIDIA Container Toolkit:
distribution=$(. /etc/os-release;echo $ID$VERSION_ID) && curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
curl -fsSL https://nvidia.github.io/libnvidia-container/$distribution/libnvidia-container.list |   sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' |   sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker

# Test
docker run --rm --gpus all nvidia/cuda:12.4.1-base-ubuntu22.04 nvidia-smi
```
**Windows (WSL 2)**: Install recent NVIDIA drivers on Windows + enable GPU for WSL; install CUDA tools in the WSL distro; Docker Desktop will detect it.

**macOS**: Native NVIDIA GPU passthrough is not supported.

---

## HTTP Proxy Setup (Linux)
Create a systemd drop‑in file for Docker service:
```bash
sudo mkdir -p /etc/systemd/system/docker.service.d
cat | sudo tee /etc/systemd/system/docker.service.d/proxy.conf >/dev/null <<'EOF'
[Service]
Environment="HTTP_PROXY=http://proxy.example:8080"
Environment="HTTPS_PROXY=http://proxy.example:8080"
Environment="NO_PROXY=localhost,127.0.0.1,.local,.example.com"
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

---

## Optional Daemon Tunings
Create or edit `/etc/docker/daemon.json`:
```json
{
  "log-driver": "json-file",
  "log-opts": { "max-size": "10m", "max-file": "3" },
  "registry-mirrors": [],
  "insecure-registries": [],
  "default-address-pools": [
    { "base": "10.10.0.0/16", "size": 24 }
  ]
}
```
Apply:
```bash
sudo systemctl restart docker
```

---

## Rootless Docker (Linux optional)
Run Docker daemon as an unprivileged user.
```bash
sudo apt-get install -y uidmap dbus-user-session
dockerd-rootless-setuptool.sh install

# Environment (add to ~/.bashrc)
export PATH=/usr/bin:$PATH
export DOCKER_HOST=unix:///run/user/$(id -u)/docker.sock

# Start on login (systemd user service)
systemctl --user enable --now docker
```
Test:
```bash
docker run --rm hello-world
```

---

## Troubleshooting
- **`permission denied /var/run/docker.sock`**  
  Add your user to the `docker` group and re‑login:  
  `sudo usermod -aG docker $USER && newgrp docker`

- **Service won’t start (Linux)**  
  `sudo systemctl status docker -l` and `journalctl -u docker -b`

- **WSL issues (Windows)**  
  `wsl --shutdown` then restart Docker Desktop. Ensure your distro is set to WSL 2.

- **Port already in use**  
  Find the process: `sudo lsof -i :8080` (Linux/macOS) or `netstat -ab` (Windows PowerShell).

- **DNS problems inside containers**  
  Add `"dns": ["8.8.8.8","1.1.1.1"]` to `/etc/docker/daemon.json` (Linux), then restart Docker.

---

## Uninstall
**Windows/macOS**: Uninstall **Docker Desktop** from Apps/Applications.

**Ubuntu/Debian**
```bash
sudo apt-get purge -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo rm -rf /var/lib/docker /var/lib/containerd
```

**Fedora/CentOS/RHEL**
```bash
sudo dnf remove -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo rm -rf /var/lib/docker /var/lib/containerd
```

**Arch**
```bash
sudo pacman -Rns --noconfirm docker docker-compose
sudo rm -rf /var/lib/docker /var/lib/containerd
```

**openSUSE**
```bash
sudo zypper rm -y docker docker-compose
sudo rm -rf /var/lib/docker /var/lib/containerd
```

---

### You’re ready!
Spin up something real:
```bash
docker run -d --name redis -p 6379:6379 redis:alpine
docker logs -f redis
```
