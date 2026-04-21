# Hash Lock OIDC Demo — Rationale

## What this app demonstrates

The minimal compose footprint to protect any HTTP backend with OIDC authentication
using `nginx-hash-lock` v1.0.7 and the PCS's built-in Authelia + `auth-registrar`.
The backend is `traefik/whoami`, so after logging in you see the request headers
hash-lock forwarded — visible proof that the session cookie + proxy chain work.

## Why it's in AppStoreLab

Reference deployment for app developers adding OIDC to their own Yundera apps.
Copy the compose, swap `whoami` for your backend, done. The `OIDC_REGISTRAR_URL`
env var is the whole auth configuration — no client IDs, no secrets, nothing to
inject at install time.

## Deployment prerequisites

This app requires a PCS stack with:
- `authelia` container running (Yundera's OIDC identity provider)
- `auth-registrar` container running on the `pcs` network

Both are provisioned by the current `template-root`. If they're not up, the
app will fail at first login with `ENOTFOUND auth-registrar` in the hash-lock
container logs. Check with:

```bash
docker ps --filter name=auth-registrar --filter name=authelia
```

## Caddy label collision warning

Do **not** install on a PCS where another app claims `auth-${APP_DOMAIN}` in its
Caddy labels. An `auth-*` collision on the `pcs` network causes caddy-docker-proxy
to round-robin between both Authelia instances — requests landing on the wrong
one return `invalid_client` intermittently. (Observed in the wild with MetaCore's
`meta-authelia` service.)

## Container-naming constraint

The `container_name: hashlockdemo` is load-bearing:

- Mesh-router routes `hashlockdemo-<user>.${APP_DOMAIN}` → container named `hashlockdemo`.
- `auth-registrar` derives the OIDC `client_id` from the caller's container name
  via PTR lookup on the `pcs` network, so `hashlockdemo` is also the `client_id`
  that gets written to `clients.d/hashlockdemo.yml`.

If you fork this app, keep `name:`, the service name, `container_name`, and
`store_app_id` all matching (lowercase alnum + `-`).

## Required assets

Before publishing, drop in:

| File | Size | Description |
|------|------|-------------|
| `icon.png` | 192x192 px | App icon (transparent background) |
| `screenshot-1.png` | 1280x720 px | The whoami plaintext output after a successful login |
