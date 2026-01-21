# Architecture Overview

This document describes the architecture of the Etherpad deployment used in this repository, focusing on container isolation, reverse proxying, and security boundaries.

---

## High-Level Design

The system is designed around a **reverse-proxy-first** model where all external traffic is terminated and controlled by Apache before reaching the application container.

Client (Browser)
|
HTTPS (443)
|
[ Apache Reverse Proxy ]
|
HTTP / WebSocket
|
[ Docker Host ]
├── Etherpad Container (9001)
└── PostgreSQL Container (5432)
---

## Component Responsibilities

### Apache (Edge Layer)
- Terminates TLS (HTTPS)
- Handles authentication and access control
- Proxies HTTP and WebSocket traffic
- Injects proxy headers for backend awareness
- Acts as the only public-facing service

### Docker Host (Application Layer)
- Runs Etherpad and PostgreSQL containers
- Exposes services only to localhost
- Manages persistent storage using volumes
- Isolates application runtime from the host OS

### Etherpad
- Runs inside a container on port `9001`
- Receives traffic only from Apache
- Trusts proxy headers (`TRUST_PROXY=true`)
- Handles collaborative editing logic

### PostgreSQL
- Provides persistent storage for Etherpad data
- Runs in a private Docker network
- Is not exposed to the host or internet

---

## Port Mapping Strategy

| Component | Internal Port | Host Binding |
|---------|---------------|--------------|
| Etherpad | 9001 | 127.0.0.1:8086 |
| PostgreSQL | 5432 | Docker internal only |
| Apache | 443 | Public |

This ensures:
- No direct public access to containers
- All traffic flows through Apache
- Reduced attack surface

---

## WebSocket Handling

Etherpad uses `socket.io` for real-time collaboration.

Apache explicitly proxies WebSocket traffic:

/socket.io/ → ws://127.0.0.1:8086/socket.io/

yaml
Copy code

This allows:
- Real-time editing
- Compatibility with HTTPS
- Proxy-level authentication

---

## Plugin Architecture

Plugins are stored on the host and mounted into the container.

Host Directory
└── plugins/
├── ep_author_hover
├── ep_adminpads
└── ep_comments

yaml
Copy code

Benefits:
- Plugins persist across container restarts
- Easy plugin inspection and auditing
- Predictable deployment behavior

---

## Document Export Flow

ABIWord is installed inside the Etherpad container to support document export formats.

Flow:
User → Etherpad → ABIWord → Exported File

yaml
Copy code

This keeps export tooling isolated inside the container.

---

## Failure Isolation

- Apache failure does not corrupt application data
- Container restarts do not affect stored data
- Database persistence is independent of application lifecycle

---

## Design Goals

- Secure by default
- Minimal public exposure
- Reproducible deployments
- Separation of concerns
- Production-like behavior for learning and practice

---

## Non-Goals

- High availability (HA)
- Horizontal scaling
- Multi-tenant SaaS hosting

These can be added later but are intentionally excluded here.
