# campaign-api-gateway

Minimal Railway-deployable API gateway using Caddy.

## Files

- `Dockerfile`: builds a lightweight Caddy container.
- `Caddyfile`: routing, CORS, preflight handling, and health endpoint.

## Behavior

- Listens on Railway dynamic port via `:{$PORT}`.
- Proxies all `/campaign-center-api/*` requests to:
  - `http://campaign-center-api.railway.internal:8080`
- Preserves the full prefix path while proxying.
- Supports CORS for:
  - `https://campaign-center-admin-web.vercel.app`
  - `http://localhost:3000`
- Handles CORS preflight `OPTIONS` requests.
- Exposes `GET /health` returning `ok`.

## Deploy on Railway

1. Push this repository to GitHub.
2. In Railway, create a new project and choose **Deploy from GitHub repo**.
3. Select this repository.
4. Railway will build from the `Dockerfile` and provide `PORT` automatically.
5. (Recommended) Ensure `campaign-center-api` service exists in the same Railway project so `campaign-center-api.railway.internal` resolves.

## Test with curl

Replace `$GATEWAY_URL` with your Railway service URL.

```bash
# Health check
curl -i "$GATEWAY_URL/health"

# Proxy path-preservation check
curl -i "$GATEWAY_URL/campaign-center-api/v1/swagger/doc.json"

# CORS preflight check (allowed origin)
curl -i -X OPTIONS "$GATEWAY_URL/campaign-center-api/v1/swagger/doc.json" \
  -H "Origin: https://campaign-center-admin-web.vercel.app" \
  -H "Access-Control-Request-Method: GET" \
  -H "Access-Control-Request-Headers: Authorization, Content-Type"
```

Expected preflight response includes:
- `204` status
- `Access-Control-Allow-Origin` (for allowed origins)
- `Access-Control-Allow-Methods`
- `Access-Control-Allow-Headers`
