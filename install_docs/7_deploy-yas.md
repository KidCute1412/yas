# Deploy YAS on the cluster

Step 1: Fix Postgres template syntax to avoid errors

Update the recommendation and webhook entries in this file:

k8s/deploy/postgres/postgresql/templates/postgresql.yaml

Make sure they use the correct Helm template syntax:

```yaml
recommendation: { { .Values.username } }
webhook: { { .Values.username } }
```

Step 2: Start Minikube and enable ingress

```bash
minikube start --driver=docker --cpus=4 --disk-size='40000mb' --memory='16g'
minikube addons enable ingress
kubectl get nodes -o wide
kubectl get pods -A
```

Step 3: Go to the deploy folder from the repository root

```bash
cd k8s/deploy
```

Step 4: Run these scripts in order

Note: If you are on a Unix-based system, make sure the scripts have execute permissions:

````bash
chmod +x *.sh

```bash
./setup-cluster.sh
./setup-keycloak.sh
./setup-redis.sh
./deploy-yas-configuration.sh
````

Step 5: Fix in-cluster DNS for identity.yas.local.com (CoreDNS static host)

```bash
kubectl -n kube-system edit configmap coredns
```

Step 6: In Corefile, add or update the hosts entry

Replace the IP if your minikube ip is different:

```txt
hosts {
  192.168.49.2 identity.yas.local.com
  fallthrough
}
```

Step 7: Restart CoreDNS

```bash
kubectl -n kube-system rollout restart deployment coredns
kubectl -n kube-system rollout status deployment coredns
```

Step 8: Deploy the YAS applications

```bash
./deploy-yas-applications.sh
```

Step 9: Deploy the API Ingress

```bash
kubectl apply -f setup-api-ingress.yaml
```

Step 10: Deploy the internal API Gateway

```bash

kubectl apply -f setup-internal-api-gateway.yaml
```
