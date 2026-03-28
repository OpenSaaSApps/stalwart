# CHANGES — Coolify-Compatible Stalwart Deployment

## What Changed

Simplified `docker-compose.yaml` to standalone Stalwart with built-in admin auth. No OAuth2 Proxy, no Logto dependency. Zero required env vars — deploys with sensible defaults.

### Key decisions
- **No OAuth2 Proxy** — Stalwart has its own admin authentication (`ADMIN_SECRET`)
- **No required env vars** — everything has defaults, deploys immediately
- **Port defaults avoid conflicts** — HTTPS on 4500, Submission on 5870 (avoids Nginx 443 and WhatsApp 587)
- **Health check uses `nc -z`** — lightweight TCP check, works in minimal containers

---

## Environment Variable Reference

### All Optional — sensible defaults provided

| Variable | Default | Description |
|----------|---------|-------------|
| `STALWART_IMAGE` | `stalwartlabs/stalwart:latest` | Stalwart Docker image and tag |
| `STALWART_HOSTNAME` | `mail.example.com` | Mail server FQDN (used for TLS, HELO, etc.) |
| `ADMIN_SECRET` | *(auto-generated)* | Fallback admin password. Leave empty to auto-generate on first run (check `docker logs stalwart-mailserver`) |
| `TZ` | `UTC` | Container timezone |
| `HTTPS_PORT` | `4500` | Host port for web admin / JMAP / REST API |
| `SMTP_PORT` | `25` | SMTP relay / incoming mail |
| `SUBMISSION_PORT` | `5870` | Authenticated submission (STARTTLS) |
| `SMTPS_PORT` | `465` | Implicit TLS SMTP |
| `IMAP_PORT` | `143` | IMAP |
| `IMAPS_PORT` | `993` | IMAPS (implicit TLS) |
| `POP3_PORT` | `110` | POP3 |
| `POP3S_PORT` | `995` | POP3S (implicit TLS) |
| `MANAGESIEVE_PORT` | `4190` | ManageSieve filter management |

---

## Port Mapping Summary

| Host Port | Container Port | Protocol | Purpose |
|-----------|---------------|----------|---------|
| `${HTTPS_PORT:-4500}` | 443 | HTTPS | Web admin UI, JMAP, REST API |
| `${SMTP_PORT:-25}` | 25 | SMTP | Incoming mail |
| `${SUBMISSION_PORT:-5870}` | 587 | SMTP | Client submission (STARTTLS) |
| `${SMTPS_PORT:-465}` | 465 | SMTPS | Implicit TLS submission |
| `${IMAP_PORT:-143}` | 143 | IMAP | Mail retrieval |
| `${IMAPS_PORT:-993}` | 993 | IMAPS | TLS mail retrieval |
| `${POP3_PORT:-110}` | 110 | POP3 | POP3 retrieval |
| `${POP3S_PORT:-995}` | 995 | POP3S | TLS POP3 retrieval |
| `${MANAGESIEVE_PORT:-4190}` | 4190 | ManageSieve | Sieve filter management |

> **Note**: Default ports 4500 (HTTPS) and 5870 (Submission) are chosen to avoid conflicts with Nginx (443) and WhatsApp Proxy (587) on the opensaasapps VPS.

---

## Post-Deployment Setup

1. **Get admin credentials**: `docker logs stalwart-mailserver` (shown on first run only)
2. **Access admin UI**: `https://<server-ip>:4500` (accepts self-signed cert warning)
3. **Configure mail domain**: Settings > Storage > add your domain
4. **Configure TLS**: Settings > Server > TLS (Let's Encrypt ACME or upload certs)
5. **Create user accounts**: Management > Directory > Accounts
6. **Set up DNS records**:
   - `A` record: `mail.example.com` → server IP
   - `MX` record: `example.com` → `mail.example.com`
   - `SPF` TXT: `v=spf1 ip4:<server-ip> -all`
   - `DKIM` TXT: generate in Stalwart admin, add to DNS
   - `DMARC` TXT: `v=DMARC1; p=quarantine; rua=mailto:postmaster@example.com`

> **Note**: Most cloud providers block outbound port 25. You may need to request unblocking or configure an external SMTP relay.
