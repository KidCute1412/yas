# Upgrade Swagger UI image tag

## Purpose

Upgrade Swagger UI Docker image from:

```yaml
tag: "v4.16.0"
```

to:

```yaml
tag: "v5.17.14"
```

This changes the Docker image used by Kubernetes from:

swaggerapi/swagger-ui:v4.16.0

to:

swaggerapi/swagger-ui:v5.17.14

## Why this change is needed

The previous Swagger UI version was outdated and may not render some OpenAPI definitions correctly.

Upgrading to v5.17.14 helps to:

- use a newer Swagger UI version
- improve OpenAPI 3.x rendering compatibility
- fix potential UI or JavaScript rendering issues
- avoid relying on an old Swagger UI image
- keep the deployment pinned to a specific stable image tag
