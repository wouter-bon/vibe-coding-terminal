# Vibe Coding Terminal

A web-based VS Code environment (code-server) with GitHub Copilot, secured with HTTPS (Let's Encrypt) and GitHub OAuth authentication.

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Docker Host                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │   Traefik    │  │ OAuth2-Proxy │  │ code-server  │  │
│  │  (SSL/7443)  │──│   (GitHub)   │──│  (VS Code)   │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
│         │                                               │
│    Let's Encrypt                                        │
└─────────────────────────────────────────────────────────┘
```

## Pre-installed Tools

The code-server container comes with a full development environment:

**Languages & Runtimes:**
- Node.js LTS (with npm)
- Python 3 (with pip and venv)
- Go 1.22
- Rust (with cargo)

**CLI Tools:**
- Docker CLI
- kubectl
- Terraform
- AWS CLI v2

**VS Code Extensions:**
- GitHub Copilot
- GitHub Copilot Chat

## Prerequisites

- Docker and Docker Compose installed
- A domain name pointing to your server
- Port 80 and 7443 accessible from the internet
- A GitHub account

## Setup Instructions

### 1. Create a GitHub OAuth App

1. Go to [GitHub Developer Settings](https://github.com/settings/developers)
2. Click **New OAuth App**
3. Fill in the application details:
   - **Application name**: Vibe Coding Terminal
   - **Homepage URL**: `https://your-domain.com:7443`
   - **Authorization callback URL**: `https://your-domain.com:7443/oauth2/callback`
4. Click **Register application**
5. Copy the **Client ID**
6. Click **Generate a new client secret** and copy the secret

### 2. Configure DNS

Add an A record pointing your domain to your server's IP address:

```
Type: A
Name: your-subdomain (or @ for root domain)
Value: YOUR_SERVER_IP
TTL: 300
```

### 3. Configure Environment Variables

```bash
# Copy the example environment file
cp .env.example .env

# Edit the .env file with your values
nano .env
```

Fill in the required values:

```bash
DOMAIN=your-domain.com
EMAIL=your-email@example.com
GITHUB_CLIENT_ID=<from step 1>
GITHUB_CLIENT_SECRET=<from step 1>
OAUTH2_PROXY_COOKIE_SECRET=<generate below>
```

Generate a cookie secret:

```bash
openssl rand -base64 32 | tr -- '+/' '-_'
```

### 4. Deploy

**Option A: Use the deployment script (recommended)**

```bash
chmod +x deploy.sh
./deploy.sh
```

The script will:
- Check prerequisites
- Configure environment interactively
- Generate cookie secret
- Build and start all services

**Option B: Manual deployment**

```bash
# Build and start all services
docker-compose up -d

# View logs
docker-compose logs -f

# Check status
docker-compose ps
```

**Deployment script commands:**

```bash
./deploy.sh           # Full deployment
./deploy.sh --status  # Show status
./deploy.sh --logs    # View logs
./deploy.sh --restart # Restart services
./deploy.sh --stop    # Stop services
```

### 5. Access the Terminal

1. Navigate to `https://your-domain.com:7443`
2. You'll be redirected to GitHub for authentication
3. Authorize the OAuth app
4. VS Code will load in your browser

### 6. Set Up GitHub Copilot

1. Click the **Copilot icon** in the VS Code sidebar (or bottom status bar)
2. Click **Sign in to GitHub**
3. Complete the device authorization flow
4. Start coding with Copilot suggestions

## Usage

### Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `` Ctrl+` `` | Open/close terminal |
| `Ctrl+Shift+P` | Command palette |
| `Ctrl+P` | Quick file open |
| `Tab` | Accept Copilot suggestion |
| `Esc` | Dismiss Copilot suggestion |
| `Alt+]` / `Alt+[` | Next/previous Copilot suggestion |

### Persistent Storage

The following data is persisted across container restarts:

- **Project files**: `/home/coder/project` → `vibe-code-server-project` volume
- **VS Code settings/extensions**: `vibe-code-server-data` volume
- **SSL certificates**: `vibe-traefik-letsencrypt` volume

### Mount Local Directory

To work with files from your host machine, edit `docker-compose.yml`:

```yaml
code-server:
  volumes:
    - ./workspace:/home/coder/project
```

## Restricting Access

### Limit to Specific GitHub Users

Edit `.env` and uncomment:

```bash
GITHUB_ALLOWED_USERS=user1,user2,user3
```

### Limit to GitHub Organization Members

Edit `.env` and uncomment:

```bash
GITHUB_ALLOWED_ORG=your-organization
```

## Troubleshooting

### Certificate Issues

If Let's Encrypt fails to issue a certificate:

1. Ensure port 80 is accessible from the internet
2. Verify DNS is properly configured
3. Check Traefik logs: `docker-compose logs traefik`

### OAuth Errors

If GitHub authentication fails:

1. Verify the callback URL matches exactly: `https://your-domain.com:7443/oauth2/callback`
2. Check OAuth2-proxy logs: `docker-compose logs oauth2-proxy`
3. Ensure the cookie secret is properly base64 encoded

### code-server Not Loading

1. Check code-server logs: `docker-compose logs code-server`
2. Verify the container is running: `docker-compose ps`
3. Rebuild the image: `docker-compose build code-server`

## Maintenance

### Update Images

```bash
docker-compose pull
docker-compose up -d --build
```

### Backup Data

```bash
# Backup project files
docker run --rm -v vibe-code-server-project:/data -v $(pwd):/backup alpine tar czf /backup/project-backup.tar.gz -C /data .

# Backup VS Code settings
docker run --rm -v vibe-code-server-data:/data -v $(pwd):/backup alpine tar czf /backup/vscode-backup.tar.gz -C /data .
```

### Stop Services

```bash
docker-compose down
```

### Remove All Data

```bash
docker-compose down -v
```

## Security Notes

- All traffic is encrypted via TLS 1.2+
- GitHub OAuth ensures only authorized users can access
- code-server runs as non-root user (`coder`)
- Secrets are managed via environment variables (never commit `.env`)
- The Traefik dashboard is protected by OAuth (optional)

## License

MIT
