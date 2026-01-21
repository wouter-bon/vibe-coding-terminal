# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Vibe Coding Terminal is a Docker-based infrastructure project that deploys a secure, browser-accessible VS Code IDE (code-server) with GitHub Copilot support. This is a configuration-driven infrastructure project, not an application codebase.

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Docker Host                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │   Traefik    │──│ OAuth2-Proxy │──│ code-server  │  │
│  │  (SSL/7443)  │  │   (GitHub)   │  │  (VS Code)   │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
└─────────────────────────────────────────────────────────┘
```

**Services:**
- **Traefik v3.0**: Reverse proxy with Let's Encrypt SSL/TLS on ports 80 and 7443
- **OAuth2-Proxy v7.6.0**: GitHub OAuth authentication middleware (internal port 4180)
- **code-server**: Browser-based VS Code with GitHub Copilot pre-configured (internal port 8080)

All services communicate on a custom bridge network (`vibe-coding-network`). Only Traefik is exposed externally.

## Common Commands

```bash
# Start all services
docker-compose up -d

# Stop services
docker-compose down

# Stop and remove all data
docker-compose down -v

# View logs
docker-compose logs -f

# Check container status
docker-compose ps

# Rebuild after changes
docker-compose up -d --build

# Pull latest images
docker-compose pull

# Backup project files
docker run --rm -v vibe-code-server-project:/data -v $(pwd):/backup alpine tar czf /backup/project-backup.tar.gz -C /data .

# Backup VS Code settings
docker run --rm -v vibe-code-server-data:/data -v $(pwd):/backup alpine tar czf /backup/vscode-backup.tar.gz -C /data .
```

## Configuration

- **docker-compose.yml**: Main orchestration file defining all services, networks, and volumes
- **traefik/traefik.yml**: Traefik static configuration for entrypoints and certificate resolvers
- **code-server/Dockerfile**: Custom code-server image with Copilot extensions pre-installed
- **.env**: Environment variables for secrets (use `.env.example` as template)

## Key Patterns

- Environment variables use `${VAR_NAME}` interpolation in docker-compose.yml
- All container names and volumes are prefixed with `vibe-`
- Traefik dynamic configuration uses Docker labels for service discovery
- OAuth2-Proxy middleware protects access to internal services via forward authentication
- code-server runs as non-root `coder` user

## Persistent Volumes

- `vibe-traefik-letsencrypt`: SSL certificates
- `vibe-code-server-data`: VS Code settings and extensions
- `vibe-code-server-project`: Project workspace files
