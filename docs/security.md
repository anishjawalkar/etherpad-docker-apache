# Security Design and Controls

This document outlines the security model used in this Etherpad deployment, including TLS, authentication, network isolation, and Apache DBD-based authentication.

---

## Security Philosophy

The system follows a **defense-in-depth** approach:
- Minimize exposed services
- Enforce security at the edge
- Keep application containers simple and isolated
- Avoid embedding credentials into application code

---

## Network Isolation

- Etherpad is bound to `127.0.0.1`
- PostgreSQL is accessible only within Docker
- No backend service is directly exposed to the internet
- Apache is the single entry point

This prevents:
- Direct container access
- Accidental public exposure
- Bypassing authentication controls

---

## TLS and HTTPS

TLS is terminated at Apache using Let’s Encrypt certificates.

Key properties:
- HTTPS enforced for all client connections
- Certificates managed via Certbot
- Backend traffic remains unencrypted but private

TLS termination at the proxy:
- Simplifies container configuration
- Centralizes certificate management
- Matches common enterprise deployments

---

## Proxy Trust Model

Etherpad is configured to trust proxy headers:

TRUST_PROXY=true


Apache injects:


X-Forwarded-Proto: https


This ensures:
- Correct URL generation
- Secure cookies
- Accurate client protocol detection

---

## Authentication Strategy

Authentication is implemented at the **Apache layer**, not inside Etherpad.

Advantages:
- No modification to application code
- Centralized access control
- Ability to swap authentication backends

---

## Apache Basic Authentication (Example)

```apache
<Location />
    AuthType Basic
    AuthName "Etherpad Access"
    AuthBasicProvider file
    AuthUserFile /etc/apache2/etherpad-users
    Require valid-user
</Location>
```

This method is simple and suitable for:

Internal tools

Small teams

Testing environments

Apache DBD Authentication (Database-Backed)

For production-style authentication, Apache DBD can be used.

Enable Required Modules
```
a2enmod dbd authn_dbd
systemctl reload apache2
```
Example DBD Configuration (Sanitized)
```
DBDriver pgsql
DBDParams "host=db.example.internal port=5432 dbname=authdb user=authuser password=example"
DBDMin 4
DBDKeep 8
DBDMax 20
DBDExptime 300

Authentication Query Example
<Location />
    AuthType Basic
    AuthName "Etherpad (DB Auth)"
    AuthBasicProvider dbd

    AuthDBDUserPWQuery \
    "SELECT password_hash FROM users WHERE username = %s AND active = true"

    Require valid-user
</Location>
```

Notes:

Passwords should be stored hashed

Query is executed per authentication request

Database access is limited to auth use only

Why Use Apache DBD?

Integrates with existing databases

Centralized identity management

No duplication of user stores

Works well with legacy and internal systems

Credential Handling

No credentials are hardcoded in this repository

All secrets are represented as placeholders

Environment variables are preferred

Real credentials must be injected at deploy time

Logging and Auditing

Apache access logs capture authentication events

Separate logs for SSL and application traffic

Container logs available via Docker

This supports:

Debugging

Access auditing

Incident analysis

Limitations

Basic authentication is not encrypted beyond TLS

No rate limiting implemented

No MFA or OAuth integration

These can be added depending on deployment needs.
