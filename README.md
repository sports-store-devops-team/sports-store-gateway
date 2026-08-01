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
