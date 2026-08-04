# API Reference

The relay mirrors the upstream [yesapi.online](https://yesapi.online) API surface under the same paths, prefixed by this service's base URL (default `http://localhost:8787`).

## Authentication

Clients calling the relay do **not** need an upstream API key. The relay injects the `Authorization` header itself using the `YESAPI_KEY` configured in its environment.

Optionally, you can require clients to authenticate to the relay by setting `RELAY_ACCESS_TOKEN`. When set, all requests must include:

```
Authorization: Bearer <RELAY_ACCESS_TOKEN>
```

## Endpoints

### `GET /v1/status`

Health/status passthrough. Returns upstream service status.

```json
{
  "status": "ok",
  "upstream_latency_ms": 42
}
```

### `GET /v1/models`

Lists available models/resources exposed by the upstream API. Cached for `CACHE_TTL_SECONDS` (default 60s).

### `POST /v1/query`

Forwards a query payload to the upstream API. Not cached.

**Request body**

```json
{
  "input": "string",
  "options": {
    "max_tokens": 512,
    "temperature": 0.7
  }
}
```

**Response**

```json
{
  "output": "string",
  "usage": {
    "input_tokens": 12,
    "output_tokens": 128
  }
}
```

### `GET /v1/usage`

Returns aggregate usage/quota information for the configured `YESAPI_KEY`. Not cached, rate-limited more aggressively than other routes.

## Error format

All errors follow a consistent shape regardless of whether they originate from the relay or are passed through from upstream:

```json
{
  "error": {
    "code": "UPSTREAM_TIMEOUT",
    "message": "Upstream request timed out after 3 retries",
    "request_id": "a1b2c3d4"
  }
}
```

| Code | Meaning |
|---|---|
| `UPSTREAM_TIMEOUT` | Upstream did not respond within the configured timeout after retries |
| `UPSTREAM_ERROR` | Upstream returned a non-2xx response |
| `RATE_LIMITED` | Client exceeded `RATE_LIMIT_PER_MIN` |
| `INVALID_REQUEST` | Request body failed validation before being forwarded |
| `UNAUTHORIZED` | Missing/invalid `RELAY_ACCESS_TOKEN` (when enabled) |
