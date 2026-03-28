# CHANGES — Coolify-Compatible Stalwart Deployment

## What Changed

Updated `docker-compose.yaml` to be fully Coolify-ready with all configuration externalized as environment variables. Added OAuth2 Proxy hardening, health checks, logging limits, and security options.

### Changes from previous version

| Change | Before | After |
|--------|--------|-------|
| Ports 443/8080 | `ports:` (publicly exposed) | `expose:` (internal only, OAuth2 Proxy fronts) |
| OAuth2 Proxy image | `latest` (unpinned) | `v7.7.1` (pinned) |
| OAuth2 Proxy port conflict | Hardcoded `4180:4180` | Configurable `${OAUTH2_PROXY_PORT:-443}:4180` |
| Mail ports | Hardcoded | All configurable via env vars |
| Stalwart image | Hardcoded | `${STALWART_IMAGE:-stalwartlabs/stalwart:latest}` |
| Stalwart hostname | Not set | `${STALWART_HOSTNAME:-mail.example.com}` |
| Admin secret | Not configurable | `${ADMIN_SECRET:-}` (empty = auto-generate) |
| Timezone | Not set | `${TZ:-UTC}` |
| OIDC issuer/client | Required (`:?`) | Optional with defaults (`:-`) |
| OIDC scopes | Missing | `${OAUTH2_PROXY_SCOPE:-openid profile email}` |
| PKCE (code challenge) | Missing | `${OAUTH2_PROXY_CODE_CHALLENGE_METHOD:-S256}` |
| Cookie HTTPONLY | Missing | `"true"` |
| Email domains | Hardcoded `"*"` | `${OAUTH2_PROXY_EMAIL_DOMAINS:-*}` |
| Health check start_period | Missing | `30s` (mailserver), `10s` (proxy) |
| OAuth2 Proxy health check | Missing | `/ping` endpoint |
| Security options | Missing | `no-new-privileges:true` on both services |
| Logging | Missing | JSON driver, `max-size: 10m`, compressed |

---

## Environment Variable Reference

### Required

| Variable | Description |
|----------|-------------|
| `OIDC_CLIENT_SECRET` | OAuth2 client secret registered in your OIDC provider (e.g. Logto) |
| `OAUTH2_PROXY_COOKIE_SECRET` | 32-byte base64 secret for cookie encryption. Generate: `python3 -c 'import os,base64;print(base64.urlsafe_b64encode(os.urandom(32)).decode())'` |

### Optional — Stalwart

| Variable | Default | Description |
|----------|---------|-------------|
| `STALWART_IMAGE` | `stalwartlabs/stalwart:latest` | Stalwart Docker image and tag |
| `STALWART_HOSTNAME` | `mail.example.com` | Mail server FQDN (used for TLS, HELO, etc.) |
| `ADMIN_SECRET` | *(auto-generated)* | Fallback admin password. Leave empty to auto-generate on first run (check `docker logs stalwart-mailserver`) |
| `TZ` | `UTC` | Container timezone |

### Optional — OAuth2 Proxy / OIDC

| Variable | Default | Description |
|----------|---------|-------------|
| `OIDC_ISSUER_URL` | `https://logto.example.com/oidc` | OIDC issuer URL |
| `OIDC_CLIENT_ID` | `stalwart` | OIDC client ID |
| `OAUTH2_PROXY_REDIRECT_URL` | `https://mail-admin.example.com/oauth2/callback` | OAuth2 callback URL |
| `OAUTH2_PROXY_PORT` | `443` | Host port for the OAuth2 Proxy (web admin access) |
| `OAUTH2_PROXY_EMAIL_DOMAINS` | `*` | Restrict access by email domain (comma-separated, `*` = all) |
| `OAUTH2_PROXY_SCOPE` | `openid profile email` | OIDC scopes to request |
| `OAUTH2_PROXY_CODE_CHALLENGE_METHOD` | `S256` | PKCE code challenge method |

### Optional — Mail Protocol Ports

| Variable | Default | Description |
|----------|---------|-------------|
| `SMTP_PORT` | `25` | SMTP relay / incoming mail |
| `SUBMISSION_PORT` | `587` | Authenticated submission (STARTTLS) |
| `SMTPS_PORT` | `465` | Implicit TLS SMTP |
| `IMAP_PORT` | `143` | IMAP |
| `IMAPS_PORT` | `993` | IMAPS (implicit TLS) |
| `POP3_PORT` | `110` | POP3 |
| `POP3S_PORT` | `995` | POP3S (implicit TLS) |
| `MANAGESIEVE_PORT` | `4190` | ManageSieve filter management |

---

## Port Mapping Summary

| Host Port | Container Port | Protocol | Access |
|-----------|---------------|----------|--------|
| `${OAUTH2_PROXY_PORT:-443}` | 4180 | HTTPS | Public — Web admin UI via OAuth2 Proxy |
| `${SMTP_PORT:-25}` | 25 | SMTP | Public — Incoming mail |
| `${SUBMISSION_PORT:-587}` | 587 | SMTP | Public — Client submission |
| `${SMTPS_PORT:-465}` | 465 | SMTPS | Public — Implicit TLS submission |
| `${IMAP_PORT:-143}` | 143 | IMAP | Public — Mail retrieval |
| `${IMAPS_PORT:-993}` | 993 | IMAPS | Public — TLS mail retrieval |
| `${POP3_PORT:-110}` | 110 | POP3 | Public — POP3 retrieval |
| `${POP3S_PORT:-995}` | 995 | POP3S | Public — TLS POP3 retrieval |
| `${MANAGESIEVE_PORT:-4190}` | 4190 | ManageSieve | Public — Sieve filter management |
| *(internal)* | 443 | HTTPS | Internal — Stalwart web/JMAP (proxied) |
| *(internal)* | 8080 | HTTP | Internal — Stalwart admin API (proxied) |

---

## Post-Deployment Setup

1. **Get admin credentials**: `docker logs stalwart-mailserver` (shown on first run)
2. **Access admin UI**: Navigate to `https://<your-domain>:<OAUTH2_PROXY_PORT>` — authenticates via Logto SSO
3. **Configure mail domain**: Settings > Storage > add your domain
4. **Configure TLS for mail ports**: Settings > Server > TLS (use Let's Encrypt ACME or upload certificates)
5. **Create user accounts**: Management > Directory > Accounts
6. **Set up DNS records**:
   - `A` record: `mail.example.com` → server IP
   - `MX` record: `example.com` → `mail.example.com`
   - `SPF` TXT: `v=spf1 ip4:<server-ip> -all`
   - `DKIM` TXT: generate in Stalwart admin, add to DNS
   - `DMARC` TXT: `v=DMARC1; p=quarantine; rua=mailto:postmaster@example.com`

> **Note**: Most cloud providers block outbound port 25. You may need to request unblocking or configure an external SMTP relay.
