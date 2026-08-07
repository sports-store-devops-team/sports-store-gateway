# Sports Store Gateway

The only externally exposed application component. This standalone NGINX image routes browser traffic to the internal frontend and backend services while preserving same-origin `/api` access.

## Routes

| Route | Upstream |
| --- | --- |
| `/api/auth/` | `auth-service:8001` |
| `/api/products` | `catalog-service:8002` |
| `/api/internal/` | `catalog-service:8002` |
| `/api/cart` | `cart-service:8003` |
| `/api/orders` | `order-service:8004` |
| `/api/payments` | `payment-service:8005` |
| All other routes | `frontend:80` |

`GET /health` returns `200` directly from NGINX and has no upstream dependency. Forwarded host, client IP, protocol, and `Authorization` headers are preserved. The `proxy_pass` directives have no URI suffix, so the original FastAPI `/api` paths remain intact.

## Docker

```sh
docker build -t sports-store/gateway:0.2.0 .
```

The upstream DNS names must be resolvable on the container network; use the sibling `sports-store-local` Compose project for a complete local stack.

## Observability

NGINX writes minimal one-line JSON access logs to stdout without URLs, query
strings, headers, cookies, or bodies. A loopback-only `stub_status` listener is
available to the Kubernetes exporter sidecar and is never proxied publicly.

## Continuous integration

Pull requests targeting `main` build the container and run `nginx -t` with local host mappings, without starting backend services or publishing an image. Pushes to `main` repeat validation, authenticate to AWS through GitHub OIDC, and publish exactly one immutable ECR image tagged `<VERSION>-<7-character-git-hash>`.

`VERSION` is the semantic-version source and is changed deliberately through a pull request. Configure the Actions variables `AWS_REGION` and `AWS_ECR_PUBLISH_ROLE_ARN` at repository or organization scope. The role ARN is configuration, not a secret; no static AWS credentials are stored. CI publishes only to ECR and does not deploy to EKS. Deployment is handled later through Argo CD.
