# Architecture

## Request flow

```
Client
  │
  │  HTTP request (no upstream credentials attached)
  ▼
Relay (this service)
  │
  ├─ 1. Optional auth check (RELAY_ACCESS_TOKEN)
  ├─ 2. Rate limit check (per client, token bucket)
  ├─ 3. Cache lookup (for cacheable GET routes)
  │       └─ cache hit → return cached response
  ├─ 4. Attach YESAPI_KEY / signing headers
  ├─ 5. Forward request to yesapi.online, with retry/backoff
  ▼
yesapi.online (upstream)
  │
  ▼
Relay
  │
  ├─ 6. Store in cache (if cacheable)
  ├─ 7. Log request/response with correlation ID
  ▼
Client (receives upstream response)
```

## Components

- **Router** — maps incoming paths to the corresponding upstream route and per-route policy (cacheable, retryable, rate-limit tier).
- **Signer** — attaches the `Authorization` header and any additional signing metadata required by the upstream API.
- **Cache** — pluggable backend (in-memory by default, Redis for multi-instance deployments) keyed by method + path + normalized query string.
- **Retrier** — wraps the outbound HTTP call with exponential backoff, retrying only on `5xx` and network errors, never on `4xx`.
- **Rate limiter** — token bucket per client identifier (IP address, or `RELAY_ACCESS_TOKEN` if configured).
- **Logger** — structured JSON logs, one line per proxied request, including status code, latency, cache hit/miss, and retry count.

## Why centralize the upstream key here

Distributing the upstream `YESAPI_KEY` to every client that needs to call `yesapi.online` makes key rotation and revocation hard, and increases the chance of the key leaking (committed to a repo, embedded in a mobile app, etc.). By putting the relay in front, only this service holds the key, and clients authenticate against the relay using a separate, easily-rotatable token if needed.

## Scaling

The relay is stateless aside from the cache. Running multiple instances behind a load balancer works as long as the cache backend is shared (Redis) rather than in-memory — otherwise cache hit rates drop and rate limiting becomes per-instance instead of global.
