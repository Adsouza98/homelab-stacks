# Homelab Stacks

A complete Docker-based homelab infrastructure for media management, web services, and system monitoring. This repository contains modular, self-contained Docker Compose stacks for organizing and running all homelab services on TrueNAS Scale with a focus on stability, performance, and data safety.

## Stack Overview

### [ARR Stack](./arr-stack/) — Media Automation
Complete media automation ecosystem with VPN-routed services for private downloading and indexing.

**Services**: Prowlarr (indexer), Radarr/Radarr-4K (movies), Sonarr/Sonarr-4K (TV), Bazarr (subtitles), Questarr (games), Deluge (torrents), SABnzbd (usenet), Flaresolverr (anti-captcha), Tautulli (Plex monitoring), Kometa (metadata)

**Key Features**:
- All download/indexing services route through VPN (Gluetun + ExpressVPN)
- Separate 4K quality profiles for movies and TV
- Automatic subtitle management
- Plex library monitoring and metadata automation

**Ports**: 9696 (Prowlarr), 7878 (Radarr), 8989 (Sonarr), 5000 (Questarr), 8112 (Deluge), 8080 (SABnzbd), 8181 (Tautulli), + more

[→ Read ARR Stack README](./arr-stack/README.md)

---

### [Website Stack](./website/) — Web Services & Dashboard
Complete web service stack featuring dashboard, request management, and reverse proxy with SSL.

**Services**: Organizr (dashboard), Ombi (request system), Nginx Proxy Manager (reverse proxy), MariaDB (database), MariaDB Client (utility)

**Key Features**:
- Centralized dashboard (Organizr) for accessing all services
- User request system for movies/TV (Ombi)
- Automated SSL certificate management (Let's Encrypt)
- Database backend for multiple services

**Ports**: 85 (Organizr), 3579 (Ombi), 80/82/443 (Nginx Proxy Manager)

[→ Read Website Stack README](./website/README.md)

---

### [Misc Stack](./misc/) — Utilities & Monitoring
Collection of utility and monitoring services for the homelab.

**Services**: Muse (Discord music bot), Scrutiny (disk health monitoring), Threadfin (IPTV streaming)

**Key Features**:
- Discord bot for music streaming (YouTube + Spotify)
- Real-time disk health monitoring for all 12 drives
- IPTV/M3U playlist server

**Ports**: 9090 (Scrutiny), 34400 (Threadfin)

[→ Read Misc Stack README](./misc/README.md)

---

## Quick Start

### Prerequisites
- Docker and Docker Compose installed on TrueNAS Scale
- Portainer (for stack management via UI)
- 32GB RAM, Haswell i7-4790K (or similar)
- 12+ TB storage for media

### Setup Steps

1. **Clone the repository to TrueNAS**:
   ```bash
   git clone <repo-url> /mnt/Starlink/Stacks/homelab-stacks
   cd /mnt/Starlink/Stacks/homelab-stacks
   ```

2. **Configure environment variables** for each stack:
   ```bash
   # ARR Stack
   cat > arr-stack/arr-stack.env << EOF
   OPENVPN_USER=your_expressvpn_username
   OPENVPN_PASSWORD=your_expressvpn_password
   EOF
   
   # Website Stack
   cat > website/website.env << EOF
   DB_NAME=SMXPROD
   DB_USER=JuniorChange
   DB_PASSWORD=your_secure_password
   DB_ROOT_PASSWORD=your_secure_root_password
   EOF
   
   # Misc Stack (if using Muse)
   cat > misc/misc.env << EOF
   DISCORD_TOKEN=your_discord_bot_token
   YOUTUBE_API_KEY=your_youtube_api_key
   SPOTIFY_CLIENT_ID=your_spotify_client_id
   SPOTIFY_CLIENT_SECRET=your_spotify_client_secret
   EOF
   ```

3. **Import stacks into Portainer**:
   - Open Portainer: `http://<NAS-IP>:9000`
   - Go to Stacks → Add Stack
   - Select "Git repository"
   - Add repository URL and select stack
   - Deploy each stack from the root directory

4. **Start individual stacks** (CLI):
   ```bash
   cd arr-stack && docker-compose up -d
   cd ../website && docker-compose up -d
   cd ../misc && docker-compose up -d
   ```

5. **Access the services**:
   - Dashboard: `http://<NAS-IP>:85` (Organizr)
   - Prowlarr: `http://<NAS-IP>:9696`
   - Radarr: `http://<NAS-IP>:7878`
   - Sonarr: `http://<NAS-IP>:8989`
   - Questarr: `http://<NAS-IP>:5000`
   - Nginx Proxy Manager: `http://<NAS-IP>:82`
   - Tautulli: `http://<NAS-IP>:8181`
   - Scrutiny: `http://<NAS-IP>:9090`

## Directory Structure

```
homelab-stacks/
├── README.md                          # This file
├── .gitignore                         # Excludes .env, configs/, etc.
├── .gitattributes                     # Line ending normalization (LF)
│
├── arr-stack/
│   ├── compose.yaml                   # Service definitions
│   ├── local.compose.yaml             # Local overrides (not committed)
│   ├── arr-stack.env                  # Credentials (not committed)
│   ├── README.md                      # Stack documentation
│   └── configs/                       # Service configs (not committed)
│
├── website/
│   ├── compose.yaml                   # Service definitions
│   ├── local.compose.yaml             # Local overrides (not committed)
│   ├── website.env                    # Credentials (not committed)
│   ├── README.md                      # Stack documentation
│   └── configs/                       # Service configs (not committed)
│
└── misc/
    ├── compose.yaml                   # Service definitions
    ├── local.compose.yaml             # Local overrides (not committed)
    ├── misc.env                       # Credentials (not committed)
    ├── README.md                      # Stack documentation
    └── configs/                       # Service configs (not committed)
```

## Environment Configuration

Each stack requires environment variables for credentials and API keys. These are **excluded from git** for security.

### Creating `.env` Files

1. Create `<stack>.env` in each stack directory
2. Add required variables (see stack-specific READMEs)
3. **Never commit `.env` files**

### Required Environment Variables

**ARR Stack** (`arr-stack.env`):
```env
OPENVPN_USER=expressvpn_username
OPENVPN_PASSWORD=expressvpn_password
```

**Website Stack** (`website.env`):
```env
DB_NAME=SMXPROD
DB_USER=JuniorChange
DB_PASSWORD=your_password
DB_ROOT_PASSWORD=your_root_password
```

**Misc Stack** (`misc.env`):
```env
DISCORD_TOKEN=discord_bot_token
YOUTUBE_API_KEY=youtube_api_key
SPOTIFY_CLIENT_ID=spotify_client_id
SPOTIFY_CLIENT_SECRET=spotify_client_secret
```

## Service Dependencies

```
Gluetun (VPN) ──┬─ Prowlarr ──┐
                ├─ Radarr    ├─ Download to /Downloads
                ├─ Sonarr    │
                ├─ Questarr  │
                ├─ Deluge ───┘
                └─ SABnzbd

MariaDB ────────┬─ Nginx Proxy Manager
                └─ Ombi

Plex (external) ┬─ Tautulli
                └─ Kometa
```

## Network Architecture

### VPN-Routed Services (Private IP via ExpressVPN/Toronto)
- Prowlarr, Radarr, Radarr-4K, Sonarr, Sonarr-4K
- Bazarr, Flaresolverr, Deluge, SABnzbd, Questarr
- All torrent/usenet traffic is private and anonymized

### Direct Services (Local Network)
- Organizr, Ombi, Nginx Proxy Manager, MariaDB
- Tautulli, Kometa, Muse, Scrutiny, Threadfin
- Can be exposed via Nginx Proxy Manager with SSL

## Common Tasks

### View Service Logs
```bash
docker-compose -f arr-stack/compose.yaml logs -f prowlarr
docker-compose -f website/compose.yaml logs -f nginx-proxy-manager
docker-compose -f misc/compose.yaml logs -f scrutiny
```

### Stop All Services
```bash
docker-compose -f arr-stack/compose.yaml -f website/compose.yaml -f misc/compose.yaml down
```

### Update Container Images
```bash
cd arr-stack && docker-compose pull && docker-compose up -d
cd ../website && docker-compose pull && docker-compose up -d
cd ../misc && docker-compose pull && docker-compose up -d
```

### Access Database
```bash
docker exec -it db mysql -u root -p
```

### Check Container Status
```bash
docker ps -a
docker stats
```

## Troubleshooting

### Gluetun Won't Connect
- Verify ExpressVPN credentials in `arr-stack.env`
- Check VPN config: `arr-stack/configs/gluetun/Toronto.ovpn`
- View logs: `docker logs gluetun`
- Ensure TUN device is available: `ls /dev/net/tun`

### Services Not Starting
- Check dependencies: `docker ps`
- View logs: `docker logs <container_name>`
- Verify `.env` file exists and variables are set
- Ensure configs directories exist with proper permissions

### Database Connection Errors
- Confirm MariaDB is running: `docker ps | grep db`
- Check credentials in `website.env`
- View logs: `docker logs db`
- Reset database: `docker-compose -f website/compose.yaml down -v && docker-compose -f website/compose.yaml up -d`

### High Disk Usage
- Check media libraries: `du -sh /mnt/Starlink/*`
- Monitor with Scrutiny: `http://<NAS-IP>:9090`
- Review download directories

## Security & Best Practices

### Secrets Management
- `.env` files are in `.gitignore` — never commit them
- Credentials include: API keys, VPN credentials, database passwords
- If accidentally exposed, immediately rotate all credentials
- Use strong passwords for database and services

### Network Isolation
- VPN services prevent ISP/local network logging of downloads
- Nginx Proxy Manager controls external access with SSL certificates
- Database is not exposed to external networks
- Organize non-VPN services on isolated Docker network if needed

### Container Security
- All services run as UID 568 (except Nginx Proxy Manager which requires root)
- Volumes mounted read-only where possible
- Capabilities minimized (only Gluetun needs NET_ADMIN for VPN)
- Regular security updates via `docker-compose pull`

### Backup Strategy
**What to backup**:
- `configs/` directories (service configurations)
- `.env` files (keep secure, encrypted)
- Custom indexer configs
- Radarr/Sonarr profiles

**What NOT to backup**:
- Docker images (re-downloadable)
- Downloads/ directories (temporary)
- Log files

Backup command:
```bash
for stack in arr-stack website misc; do
  tar -czf ${stack}-backup-$(date +%Y%m%d).tar.gz ${stack}/configs/ ${stack}/*.env
done
```

## Performance Tuning

### Disk I/O
- Media libraries on dedicated storage
- Database on fast SSD
- Downloads on separate volume
- Monitor with `docker stats`

### Memory Management
- Radarr/Sonarr: 200-400MB each
- MariaDB: 500MB-1GB
- Deluge: 500MB+
- Monitor: `docker stats`

### Network
- Gluetun adds latency (necessary for privacy)
- Use Nginx Proxy Manager for local service routing
- Direct connections for Plex library access

## Hardware

**Server**: TrueNAS Scale on Haswell i7-4790K
- **CPU**: Intel i7-4790K (8 cores)
- **RAM**: 32GB DDR3
- **Storage**: 12× HDDs (8×12TB + 4×20TB)
- **Network**: 10GbE interface + SAS HBA

## Automated Updates (Renovate)

Docker image and GitHub Actions versions in this repo are updated by [Renovate](https://www.renovatebot.com/) via the scheduled workflow in `.github/workflows/renovate.yml` (Sundays at 4 AM UTC, or on demand from the Actions tab).

- **LinuxServer images** (`lscr.io/linuxserver/*`) use custom versioning for `ls###` build tags and extended Docker Hub tag pagination — see [RENOVATE.md](./RENOVATE.md).
- **Compose files** are scanned by the built-in `docker-compose` manager only (no duplicate regex manager).
- **Floating tags** such as `organizr/organizr:latest` are not updated automatically; pin a version tag in `website/compose.yaml` if you want Renovate to track Organizr.

Open the **Dependency Dashboard** issue on GitHub to see pending updates or trigger a manual run.

[→ Full Renovate configuration](./RENOVATE.md)

## Stack Maintenance

### Monthly Tasks
- [ ] Update all images: `docker-compose pull && docker-compose up -d`
- [ ] Check disk health: Scrutiny dashboard
- [ ] Review error logs: `docker logs <service>`

### Quarterly Tasks
- [ ] Backup configurations
- [ ] Review Plex library with Tautulli
- [ ] Audit active downloads

### Security Tasks
- [ ] Rotate API keys if exposed
- [ ] Check for security updates
- [ ] Review VPN connection (Gluetun logs)

## Documentation

- [Renovate / dependency updates](./RENOVATE.md) — LinuxServer tagging, schedules, workflows

Each stack has detailed documentation:
- [ARR Stack README](./arr-stack/README.md) — Media automation, VPN routing, setup workflow
- [Website Stack README](./website/README.md) — Web services, SSL, database, dashboard
- [Misc Stack README](./misc/README.md) — Utilities, monitoring, music bot

## Contributing

When modifying stacks:
1. Test changes in `local.compose.yaml` first
2. Update relevant README if ports/volumes change
3. Update environment variable documentation
4. Only commit `compose.yaml`, `.gitignore`, documentation, and `.gitattributes`
5. Never commit `.env`, `configs/`, or local tooling paths such as `mcps/`

## Related Resources

- [TrueNAS Documentation](https://www.truenas.com/docs/)
- [Portainer Documentation](https://docs.portainer.io/)
- [Docker Compose Reference](https://docs.docker.com/compose/compose-file/)
- [Prowlarr Docs](https://wiki.servarr.com/prowlarr)
- [Radarr Docs](https://wiki.servarr.com/radarr)
- [Sonarr Docs](https://wiki.servarr.com/sonarr)
- [Questarr](https://github.com/doezer/questarr)
