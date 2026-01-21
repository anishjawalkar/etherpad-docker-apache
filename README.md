# Etherpad Deployment (Docker + Apache Reverse Proxy)

Production-style deployment of **Etherpad** using Docker and PostgreSQL, fronted by an **Apache reverse proxy** with HTTPS, WebSocket support, authentication, and persistent storage.

This repository is intended to demonstrate **real-world DevOps practices** for self-hosted collaborative applications.

---

## Overview

This setup runs Etherpad inside Docker, binds it only to localhost, and exposes it securely to the internet through Apache.  
TLS termination, authentication, and access control are handled at the proxy layer, keeping the application container isolated.

Key goals of this setup:
- Secure public access
- Reproducible deployments
- Persistent data and plugins
- Proxy-aware application configuration

---

## Architecture

Internet
|
[ HTTPS : Apache ]
| (TLS, Auth, WebSockets)
|
[ Reverse Proxy ]
|
[ Docker Host ]
├── Etherpad (9001 → 8086)
└── PostgreSQL


- Etherpad listens on **port 9001** inside the container
- Docker publishes it to **127.0.0.1:8086**
- Apache proxies traffic to `localhost:8086`
- Backend services are **not exposed publicly**

---

## Components Used

- **Docker / Docker Compose** – container orchestration
- **Etherpad** – collaborative editor
- **PostgreSQL** – persistent datastore
- **Apache HTTP Server** – reverse proxy and authentication
- **Certbot (Let’s Encrypt)** – TLS certificates
- **ABIWord** – document export support

---

## Docker Deployment

Etherpad and PostgreSQL are deployed using Docker Compose.

Key characteristics:
- Persistent volumes for:
  - Plugins
  - Application data
  - Database storage
- Environment-based configuration
- Restart policies for resilience
- Proxy-aware configuration using `TRUST_PROXY=true`

The service is intentionally bound to `127.0.0.1` to ensure it is accessible **only through Apache**.

---

## Apache Reverse Proxy

Apache is responsible for:
- HTTPS termination
- WebSocket proxying (`/socket.io`)
- Forwarding headers (`X-Forwarded-Proto`)
- Optional authentication and access control

Why Apache?
- Mature and stable
- Excellent WebSocket support
- Flexible authentication modules
- Common in enterprise environments

The reverse proxy ensures:
- Etherpad never directly handles TLS
- Security controls are centralized
- Backend services remain isolated

---

## Authentication

Authentication is implemented **at the proxy layer**, not inside Etherpad.

Supported approaches:
- Basic authentication
- Database-backed authentication (Apache DBD)
- File-based authentication (for simpler setups)

This design keeps Etherpad stateless and avoids modifying application code.

---

## Plugin Management

Etherpad plugins are persisted using Docker volumes.

How it works:
- Plugins are installed inside the container
- The plugin directory is mounted to the host
- Plugins survive container restarts and upgrades

Benefits:
- No reinstallation after redeployments
- Easy plugin auditing
- Predictable behavior across environments

---

## Document Export (ABIWord)

ABIWord is installed inside the container to support:
- PDF export
- DOC/DOCX export
- ODT export

The installation script is included for reproducibility.

---

## Security Considerations

This setup follows basic production security practices:
- HTTPS enforced via Let’s Encrypt
- Backend ports bound to localhost only
- Authentication handled before traffic reaches Etherpad
- TLS termination at the proxy
- Credentials injected via environment variables (placeholders only)

⚠️ Note:  
All domains, credentials, and secrets in this repository are **examples only**.

---

## Tested Environment

- Debian-based Linux host
- Docker Engine
- Apache 2.4
- PostgreSQL 15
- Etherpad (official image)

---

## Intended Use

This repository is designed for:
- DevOps practice
- Self-hosted internal tools
- Learning reverse proxy and container patterns
- Demonstrating production-style deployments

It is **not** a managed SaaS setup.

---

## Notes

- EtherCalc is intentionally maintained in a separate repository
- This repository focuses exclusively on Etherpad
- Configuration files are simplified for clarity

---

## License

MIT License
