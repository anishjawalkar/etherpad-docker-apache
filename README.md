# Etherpad Deployment (Docker + Apache Reverse Proxy)

Production-style Etherpad deployment using Docker, PostgreSQL, and Apache reverse proxy with HTTPS, authentication, and persistent storage.

## Features
- Dockerized Etherpad with PostgreSQL backend
- Apache reverse proxy with WebSocket support
- HTTPS via Let's Encrypt (Certbot)
- Authentication at proxy layer
- Persistent plugin and data storage
- Production-ready port binding (localhost only)

## Architecture
Internet → Apache (HTTPS) → Docker Host → Etherpad (9001)

## Status
This repository is intended for learning, self-hosting, and DevOps practice.
