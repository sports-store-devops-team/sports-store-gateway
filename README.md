# Sports Store Local Gateway

This NGINX image is a local-development-only adapter. Docker Compose uses it to preserve same-origin `/api` access; AWS/EKS and Minikube route directly from Ingress to the frontend and backend Services. This image is validated in CI but is not published to AWS ECR.

## Routes

| Route | Upstream |
| --- | --- |
| `/api/auth/` | `auth-service:8001` |
| `/api/products` | `catalog-service:8002` |
| `/api/internal/` | `catalog-service:8002` |
| `/api/cart` | `cart-service:8003` |
| `/api/orders` | `order-service:8004` |
| `/api/payments` | `payment-service:8005` |
| All other routes | `frontend:8080` |

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

Pull requests and pushes targeting `main`, plus manual dispatches, lint the Dockerfile, build the local-only image, run `nginx -t`, and scan the filesystem and image with Trivy. The workflow never publishes this image.
