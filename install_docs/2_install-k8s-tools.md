# Install Kubernetes tools

Step 1: Create /tool/install-k8s-tools.sh script

```
cat > /tool/install-k8s-tools.sh <<'EOF'
#!/usr/bin/env bash
set -euo pipefail

echo "===== Install kubectl ====="
cd /tmp
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client

echo "===== Install Minikube ====="
cd /tmp
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
minikube version

echo "===== Install Helm ====="
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version

echo "===== Kubernetes tools installed successfully ====="
EOF
```

Step 2: Grant execute permission

```
chmod +x /tool/install-k8s-tools.sh
```

Step 3: Run the script

```
/tool/install-k8s-tools.sh
```
