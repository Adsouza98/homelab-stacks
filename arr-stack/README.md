# ARR Stack

A complete media automation and management ecosystem with VPN-routed services for private downloading and indexing. This stack manages movies, TV shows, and their subtitles through automated workflows.

## Architecture

All download and indexing services route through **Gluetun** (VPN) for privacy. Tautulli and Kometa run without VPN for direct Plex communication.

```
Gluetun (VPN) ─┬─ Prowlarr (Indexer)
               ├─ Radarr (Movies)
               ├─ Radarr-4K (4K Movies)
               ├─ Sonarr (TV Shows)
               ├─ Sonarr-4K (4K TV Shows)
               ├─ Bazarr (Subtitles)
               ├─ Flaresolverr (Anti-captcha)
               ├─ SABnzbd (Usenet)
               └─ Deluge (Torrents)

Tautulli (Direct) ─ Plex Monitoring
Kometa (Direct) ─── Library Metadata
```

---

## Services

### Gluetun
**VPN Provider & Network Router** — OpenVPN container that routes traffic through ExpressVPN. All *arr services and downloaders use this container's network stack.

- **Image**: `qmcgaw/gluetun:v3.41.1`
- **Container Name**: `gluetun`
- **Restart Policy**: Unless stopped
- **Capabilities**: `NET_ADMIN` (required for VPN tunnel)
- **Device**: `/dev/net/tun` (TUN device for VPN)

#### Environment
- `VPN_SERVICE_PROVIDER=custom` — Using custom OpenVPN config
- `VPN_TYPE=openvpn` — OpenVPN protocol
- `OPENVPN_USER` — ExpressVPN username (from env vars)
- `OPENVPN_PASSWORD` — ExpressVPN password (from env vars)
- `OPENVPN_CUSTOM_CONFIG=/gluetun/ovpn/Toronto.ovpn` — Toronto location
- `OPENVPN_FLAGS=--tls-timeout 120` — TLS timeout configuration

#### Ports (exposed by Gluetun for services)
| Port | Service |
|------|---------|
| 9696 | Prowlarr |
| 7878 | Radarr |
| 8787 | Radarr-4K |
| 8989 | Sonarr |
| 9898 | Sonarr-4K |
| 6767 | Bazarr |
| 8191 | Flaresolverr |
| 8080 | SABnzbd |
| 8112 | Deluge Web UI |
| 6881/udp, 6881/tcp | Deluge Torrenting |

#### Critical Note
All services that use `network_mode: service:gluetun` depend on this container being healthy.

---

### Tautulli
**Plex Library Monitoring** — Monitors Plex Media Server activity, usage statistics, and playback history.

- **Image**: `lscr.io/linuxserver/tautulli:v2.17.1-ls231`
- **Container Name**: `tautulli`
- **Port**: `8181`
- **Network**: Direct (not routed through VPN)
- **Restart Policy**: Unless stopped

#### Environment
- `PUID=568`, `PGID=568` — Process user/group
- `TZ=America/New_York` — Timezone
- Theme-Park customization enabled (Organizr theme)

#### Volumes
- `/config` — Tautulli configuration and database

#### Access
`http://<host>:8181` — Plex monitoring interface

---

### Kometa
**Library Metadata Manager** — Automatically manages and updates Plex library metadata, collections, and overlays.

- **Image**: `lscr.io/linuxserver/kometa:v2.3.1-ls97`
- **Container Name**: `kometa`
- **Network**: Direct (not routed through VPN)
- **Restart Policy**: Unless stopped

#### Environment
- `PUID=568`, `PGID=568` — Process user/group
- `TZ=America/New_York` — Timezone
- `KOMETA_CONFIG=/config/config.yml` — Configuration file path
- `KOMETA_TIME=03:00` — Scheduled run time (3 AM)
- `KOMETA_RUN=False` — Run on startup (disabled)
- `KOMETA_TEST=False` — Test mode (disabled)
- `KOMETA_NO_MISSING=False` — Include missing metadata (disabled)

#### Volumes
- `/config` — Kometa configuration and metadata files

#### Notes
Kometa runs on a scheduled basis (3 AM by default) to update Plex library metadata without requiring manual intervention.

#### Volumes
- `/gluetun/ovpn/` — OpenVPN config files

---

### Prowlarr
**Indexer Manager** — Centralizes and manages search indexers (torrent sites, newsgroups, etc.) for the *arr services.

- **Image**: `lscr.io/linuxserver/prowlarr:2.3.5.5327-ls147`
- **Container Name**: `prowlarr`
- **Port**: `9696`
- **Network**: Uses Gluetun VPN
- **Depends On**: Gluetun (healthy)

#### Environment
- `PUID=568`, `PGID=568` — Process user/group
- `TZ=America/New_York` — Timezone
- Theme-Park customization enabled (Organizr theme + text logo)

#### Volumes
- `/config` — Prowlarr configuration and indexer definitions
- `/mnt/Starlink/Movies/`, `/mnt/Starlink/Movies-4K/`, `/mnt/Starlink/TvShows/`, `/mnt/Starlink/TvShows-4K/`, `/Downloads` — Media paths

#### Access
`http://<host>:9696` — Indexer configuration interface

---

### Radarr
**Movie Automation** — Automatically searches for, downloads, and organizes movies.

- **Image**: `lscr.io/linuxserver/radarr:6.1.1.10360-ls303`
- **Container Name**: `radarr`
- **Port**: `7878`
- **Network**: Uses Gluetun VPN
- **Depends On**: Gluetun (healthy)

#### Environment
- `PUID=568`, `PGID=568` — Process user/group
- `TZ=America/New_York` — Timezone
- Theme-Park customization enabled (Organizr theme)

#### Volumes
- `/config` — Radarr configuration and database
- `/mnt/Starlink/Movies/` — Movie library
- `/Downloads` — Download staging area

#### Access
`http://<host>:7878` — Movie management interface

---

### Radarr-4K
**4K Movie Automation** — Manages 4K quality movie downloads separately from standard quality.

- **Image**: `lscr.io/linuxserver/radarr:6.1.1.10360-ls303`
- **Container Name**: `radarr-4k`
- **Port**: `8787`
- **Network**: Uses Gluetun VPN
- **Depends On**: Gluetun (healthy)

#### Environment
- `PUID=568`, `PGID=568` — Process user/group
- `TZ=America/New_York` — Timezone
- Theme-Park customization enabled (Organizr theme + 4K logo/favicon)

#### Volumes
- `/config` — Radarr-4K configuration and database
- `/mnt/Starlink/Movies-4K/` — 4K movie library
- `/Downloads` — Download staging area

#### Access
`http://<host>:8787` — 4K movie management interface

---

### Sonarr
**TV Show Automation** — Automatically searches for, downloads, and organizes TV episodes.

- **Image**: `lscr.io/linuxserver/sonarr:4.0.17.2952-ls311`
- **Container Name**: `sonarr`
- **Port**: `8989`
- **Network**: Uses Gluetun VPN
- **Depends On**: Gluetun (healthy)

#### Environment
- `PUID=568`, `PGID=568` — Process user/group
- `TZ=America/New_York` — Timezone
- Theme-Park customization enabled (Organizr theme + text logo)

#### Volumes
- `/config` — Sonarr configuration and database
- `/mnt/Starlink/TvShows/` — TV show library
- `/Downloads` — Download staging area

#### Access
`http://<host>:8989` — TV show management interface

---

### Sonarr-4K
**4K TV Show Automation** — Manages 4K quality TV episodes separately from standard quality.

- **Image**: `lscr.io/linuxserver/sonarr:4.0.17.2952-ls311`
- **Container Name**: `sonarr-4k`
- **Port**: `9898`
- **Network**: Uses Gluetun VPN
- **Depends On**: Gluetun (healthy)

#### Environment
- `PUID=568`, `PGID=568` — Process user/group
- `TZ=America/New_York` — Timezone
- Theme-Park customization enabled (Organizr theme + 4K logo/favicon)

#### Volumes
- `/config` — Sonarr-4K configuration and database
- `/mnt/Starlink/TvShows-4K/` — 4K TV show library
- `/Downloads` — Download staging area

#### Access
`http://<host>:9898` — 4K TV show management interface

---

### Bazarr
**Subtitle Management** — Automatically finds and downloads subtitles for movies and TV shows.

- **Image**: `lscr.io/linuxserver/bazarr:v1.5.6-ls349`
- **Container Name**: `bazarr`
- **Port**: `6767`
- **Network**: Uses Gluetun VPN
- **Depends On**: Gluetun (healthy)

#### Environment
- `PUID=568`, `PGID=568` — Process user/group
- `TZ=America/New_York` — Timezone
- Theme-Park customization enabled (Organizr theme)

#### Volumes
- `/config` — Bazarr configuration and database
- All media libraries (Movies, Movies-4K, TvShows, TvShows-4K) — For subtitle scanning and placement

#### Access
`http://<host>:6767` — Subtitle management interface

---

### Flaresolverr
**Cloudflare Bypass** — Solves Cloudflare challenges for web scraping by indexers.

- **Image**: `ghcr.io/flaresolverr/flaresolverr:v3.5.0`
- **Container Name**: `flaresolverr`
- **Port**: `8191`
- **Network**: Uses Gluetun VPN
- **Depends On**: Gluetun (healthy)

#### Environment
- `LOG_LEVEL=info` — Logging level
- `LOG_HTML=false` — Don't log HTML responses
- `CAPTCHA_SOLVER=none` — No captcha solving
- `TZ=America/New_York` — Timezone

#### Usage
Indexers use this internally to bypass Cloudflare protection.

---

### SABnzbd
**Usenet Downloader** — Downloads from Usenet newsgroups using NZB files from Prowlarr.

- **Image**: `lscr.io/linuxserver/sabnzbd:5.0.3-ls255`
- **Container Name**: `sabnzbd`
- **Port**: `8080`
- **Network**: Uses Gluetun VPN
- **Depends On**: Gluetun (healthy)

#### Environment
- `PUID=568`, `PGID=568` — Process user/group
- `TZ=America/New_York` — Timezone
- Theme-Park customization enabled (Organizr theme)

#### Volumes
- `/config` — SABnzbd configuration and queue
- `/Downloads/Sabnzbd/` — Downloaded content staging area

#### Access
`http://<host>:8080` — Download queue and settings interface

---

### Deluge
**Torrent Client** — Downloads torrents from Prowlarr with private IP through VPN.

- **Image**: `lscr.io/linuxserver/deluge:2.2.0-ls377`
- **Container Name**: `deluge`
- **Ports**: `8112` (Web UI), `6881/udp`, `6881/tcp` (Torrenting)
- **Network**: Uses Gluetun VPN
- **Depends On**: Gluetun (healthy)

#### Environment
- `PUID=568`, `PGID=568` — Process user/group
- `TZ=America/New_York` — Timezone
- Theme-Park customization enabled (Organizr theme)

#### Volumes
- `/config` — Deluge configuration
- `/Downloads/Deluge/` — Downloaded torrents staging area

#### Access
`http://<host>:8112` — Torrent management interface

---

### Tautulli
**Plex Monitoring & Analytics** — Monitors Plex Media Server usage and provides stats/notifications.

- **Image**: `lscr.io/linuxserver/tautulli:v2.17.1-ls231`
- **Container Name**: `tautulli`
- **Port**: `8181`
- **Network**: Direct (no VPN) — connects to Plex server
- **Restart Policy**: Unless stopped

#### Environment
- `PUID=568`, `PGID=568` — Process user/group
- `TZ=America/New_York` — Timezone
- Theme-Park customization enabled (Organizr theme)

#### Volumes
- `/config` — Tautulli configuration and database

#### Access
`http://<host>:8181` — Plex monitoring dashboard

---

### Kometa
**Library Metadata & Automation** — Manages Plex library metadata, collections, and automation rules.

- **Image**: `lscr.io/linuxserver/kometa:v2.3.1-ls97`
- **Container Name**: `kometa`
- **Network**: Direct (no VPN) — connects to Plex server
- **Restart Policy**: Unless stopped

#### Environment
- `PUID=568`, `PGID=568` — Process user/group
- `TZ=America/New_York` — Timezone
- `KOMETA_CONFIG=/config/config.yml` — Config file path
- `KOMETA_TIME=03:00` — Daily run time (3 AM)
- `KOMETA_RUN=False` — Manual execution only
- `KOMETA_TEST=False` — Production mode
- `KOMETA_NO_MISSING=False` — Include missing items

#### Volumes
- `/config` — Kometa configuration and templates

#### Usage
Runs automatically at 03:00 daily to update library metadata and create collections in Plex.

---

## Quick Start

1. Configure environment variables in `arr-stack.env`:
   ```env
   OPENVPN_USER=your_expressvpn_username
   OPENVPN_PASSWORD=your_expressvpn_password
   ```

2. Ensure ExpressVPN config exists at:
   ```
   configs/gluetun/Toronto.ovpn
   ```

3. Start all services:
   ```bash
   docker-compose up -d
   ```

4. Access the services:
   - Prowlarr: `http://<host>:9696` — Configure indexers first
   - Radarr: `http://<host>:7878` — Add movies
   - Sonarr: `http://<host>:8989` — Add TV shows
   - Bazarr: `http://<host>:6767` — Configure subtitles
   - Deluge: `http://<host>:8112` — Check downloads
   - SABnzbd: `http://<host>:8080` — Check downloads
   - Tautulli: `http://<host>:8181` — View stats
   - Radarr-4K: `http://<host>:8787` — 4K movies
   - Sonarr-4K: `http://<host>:9898` — 4K shows

## Configuration

Service configurations are stored in `./configs/`:
```
configs/
├── gluetun/            # VPN config and .ovpn files
├── prowlarr/           # Indexer definitions
├── radarr/             # Movie profiles and definitions
├── radarr-4k/          # 4K movie profiles
├── sonarr/             # TV profiles and definitions
├── sonarr-4k/          # 4K TV profiles
├── bazarr/             # Subtitle definitions and languages
├── sabnzbd/            # Usenet server configs
├── deluge/             # Torrent client settings
├── tautulli/           # Plex monitoring config
└── kometa/             # Plex metadata templates
```

## Setup Workflow

1. **Configure Prowlarr**: Add indexers (NZBGeek, 1337x, ThePirateBay, etc.)
2. **Configure Radarr/Sonarr**: Set quality profiles, media paths, connect to Prowlarr
3. **Configure Download Clients**: Radarr/Sonarr → SABnzbd/Deluge
4. **Configure Bazarr**: Add to Radarr/Sonarr, set language preferences
5. **Add Content**: Use Radarr/Sonarr UI to add movies/shows (triggers automated downloads)
6. **Monitor**: Check Tautulli for Plex usage stats

## Network Architecture

- **VPN Services**: Prowlarr, Radarr, Sonarr, Bazarr, Flaresolverr, SABnzbd, Deluge
  - All traffic routed through ExpressVPN (Toronto)
  - Private IP for torrent/usenet activities
- **Direct Services**: Tautulli, Kometa
  - Direct connection to Plex server
  - Local network access only

## Important Notes

- **Gluetun must be healthy** before any dependent services will start
- **All VPN services will fail** if Gluetun is down or misconfigured
- **Prowlarr acts as the indexer hub** — configure it first, then connect all *arr services to it
- **Separate 4K instances** allow quality-specific rules and profiles
- **Usenet requires subscription** to newsgroups (e.g., Newshosting, Frugal Usenet)
- **Indexers require configuration** — add custom indexers or presets in Prowlarr
- **Backup configs regularly** as they contain API keys and custom settings
