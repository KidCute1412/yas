# Workspace setup

You should create a separate folder for source code and avoid cloning into /tool, because /tool should be reserved for install scripts like Docker, Minikube, and Helm.

Recommended layout:

```
/tool                 # install scripts
/workspace/yas        # YAS source code
/workspace/yas-gitops # future GitOps repo for ArgoCD
```

Step 1: Update Docker Hub image repositories

Set the image repository for each service to the Docker Hub repo below:

- backoffice-bff -> thanhtienntt/yas-backoffice-bff
- backoffice-ui -> thanhtienntt/yas-backoffice-ui
- storefront-bff -> thanhtienntt/yas-storefront-bff
- storefront-ui -> thanhtienntt/yas-storefront-ui
- cart -> thanhtienntt/yas-cart
- customer -> thanhtienntt/yas-customer
- inventory -> thanhtienntt/yas-inventory
- location -> thanhtienntt/yas-location
- media -> thanhtienntt/yas-media
- order -> thanhtienntt/yas-order
- payment -> thanhtienntt/yas-payment
- promotion -> thanhtienntt/yas-promotion
- product -> thanhtienntt/yas-product
- rating -> thanhtienntt/yas-rating
- recommendation -> thanhtienntt/yas-recommendation
- sampledata -> thanhtienntt/yas-sampledata
- search -> thanhtienntt/yas-search
- tax -> thanhtienntt/yas-tax
- webhook -> thanhtienntt/yas-webhook

Step 2: Create the workspace folder

Run on the VM:

```
sudo mkdir -p /workspace
sudo chown -R $USER:$USER /workspace
cd /workspace
```

Step 3: Clone your YAS repo

Your Jenkins uses this repo:

https://github.com/KidCute1412/yas.git

So clone it:

```
git clone https://github.com/KidCute1412/yas.git
```

Then go into the folder:

```
cd /workspace/yas
```
