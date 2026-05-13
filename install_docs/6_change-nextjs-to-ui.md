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
