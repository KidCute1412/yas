# Install Docker

Step 1: Create /tool directory

```
sudo mkdir -p /tool
sudo chown -R $USER:$USER /tool
ls -la /tool
```

Step 2: Create the Docker install script

```
nano /tool/install-docker.sh
```

Paste the full content below:

```
#!/usr/bin/env bash
set -euo pipefail

echo "===== Remove old Docker-related packages ====="
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do
  sudo apt-get remove -y "$pkg" || true
done

echo "===== Update apt and install dependencies ====="
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg git conntrack

echo "===== Add Docker official GPG key ====="
sudo install -m 0755 -d /etc/apt/keyrings

if [ ! -f /etc/apt/keyrings/docker.gpg ]; then
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
    sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
fi

sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo "===== Add Docker apt repository ====="
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

echo "===== Install Docker Engine ====="
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

echo "===== Enable Docker service ====="
sudo systemctl enable --now docker

echo "===== Add current user to docker group ====="
sudo usermod -aG docker "$USER"

echo "===== Verify Docker installation with sudo ====="
sudo docker version
sudo docker compose version
sudo docker buildx version

echo "===== Docker installation completed ====="
echo "IMPORTANT: run: newgrp docker"
echo "Then test with: docker run hello-world"
```

Save the file in nano:

Ctrl + O
Enter
Ctrl + X

Step 3: Grant execute permission

```
chmod +x /tool/install-docker.sh
```

Step 4: Run the Docker install script

```
/tool/install-docker.sh
```

Step 5: Verify

```
newgrp docker
docker run hello-world
```
