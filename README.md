# yesapi-relay

A lightweight relay/proxy service for forwarding requests to [yesapi.online](https://yesapi.online), with request signing, retry logic, and response caching built in.

## Why a relay?

Calling an upstream API directly from client-side code or from many small internal services often leads to duplicated auth handling, inconsistent retry/backoff behavior, and no shared caching layer. This relay centralizes all of that in one place so downstream consumers only need to talk to a single, predictable endpoint.

## Features

- **Request signing** — attaches the required auth headers to every outbound call so upstream credentials never need to be distributed to client apps.
- **Automatic retries** — exponential backoff on `5xx` and network-level failures.
- **Response caching** — optional in-memory/Redis cache for idempotent `GET` endpoints, configurable per route.
- **Rate limiting** — per-client token bucket to stay within upstream quota.
- **Structured logging** — every proxied request/response pair is logged with a correlation ID.

## Getting started

```bash
git clone https://github.com/tjinyong/yesapi-relay.git
cd yesapi-relay
npm install
cp .env.example .env
npm start
```

## Configuration

All configuration is provided via environment variables (see `.env.example`):

| Variable | Description | Default |
|---|---|---|
| `YESAPI_BASE_URL` | Upstream base URL | `https://yesapi.online/api` |
| `YESAPI_KEY` | Upstream API key used for signing requests | — |
| `PORT` | Port this relay listens on | `8787` |
| `CACHE_TTL_SECONDS` | Default cache TTL for cacheable routes | `60` |
| `RATE_LIMIT_PER_MIN` | Max requests per client per minute | `120` |

## Usage

Point your application at the relay instead of the upstream API directly:

```bash
curl http://localhost:8787/v1/status
```

The relay forwards the request to `https://yesapi.online/api/v1/status`, injecting the configured auth headers, and returns the upstream response unmodified (unless caching is enabled for that route).

See [`docs/api.md`](docs/api.md) for the full list of proxied endpoints and [`docs/architecture.md`](docs/architecture.md) for how requests flow through the service.

## License

MIT
