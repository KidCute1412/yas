# Change Nextjs to UI in application-prod.yaml

## Goal

Update the `uri` of the `nextjs` route in `storefront-bff/src/main/resources/application-prod.yaml` from `storefront-nextjs` to `ui`.

## Steps

1. Open `storefront-bff/src/main/resources/application-prod.yaml`.
2. Find the route block with `id: nextjs`.
3. Change the line:

   ```yaml
   uri: http://storefront-nextjs:3000
   ```

   to:

   ```yaml
   uri: http://ui:3000
   ```

4. Save the file.

## Example result

```yaml
- id: nextjs
  uri: http://ui:3000
  predicates:
    - Path=/**
```

## Rebuild and redeploy (after the change)

### 1) Check current Java

Run:

```bash
which java
java -version
echo $JAVA_HOME
```

If `which java` returns nothing, or `JAVA_HOME` is empty/incorrect, continue to step 2.

### 2) Install JDK

Try OpenJDK 21 first:

```bash
sudo apt update
sudo apt install -y openjdk-21-jdk
```

Verify:

```bash
java -version
javac -version
```

### 3) Rebuild `storefront-bff`

```bash
cd /workspace/yas/storefront-bff

chmod +x mvnw
./mvnw clean package -DskipTests
```

If the build succeeds, verify the JAR:

```bash
ls -lh target/*.jar
```

Then rebuild and push the Docker image:

```bash
cd /workspace/yas

docker build -t thanhtienntt/yas-storefront-bff:latest ./storefront-bff
docker push thanhtienntt/yas-storefront-bff:latest
```

### 4) Force Kubernetes to pull the latest image

Because the `latest` tag can be cached, run:

```bash
kubectl patch deployment storefront-bff -n yas \
  -p '{"spec":{"template":{"spec":{"containers":[{"name":"storefront-bff","imagePullPolicy":"Always"}]}}}}'
```

If the container name is different, check it first:

```bash
kubectl get deploy -n yas storefront-bff \
  -o jsonpath='{.spec.template.spec.containers[*].name}{"\n"}'
```

Then replace the name in the patch command.

### 5) Restart the deployment

```bash
kubectl rollout restart deployment -n yas storefront-bff
```
