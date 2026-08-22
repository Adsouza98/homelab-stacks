# Misc Stack

A collection of utility and monitoring services for the homelab, including music streaming, disk health monitoring, IPTV streaming, and TeslaCam clip viewing.

## Services

### Muse
**Discord Music Bot** — Music streaming for Discord with support for YouTube and Spotify.

- **Image**: `ghcr.io/museofficial/muse:2.11.5`
- **Restart Policy**: Always
- **Config Path**: `/mnt/Starlink/Stacks/homelab-stacks/misc/configs/muse`

#### Required Environment Variables
- `DISCORD_TOKEN` — Discord bot token for authentication
- `YOUTUBE_API_KEY` — YouTube API key for music search/streaming
- `SPOTIFY_CLIENT_ID` — Spotify application client ID
- `SPOTIFY_CLIENT_SECRET` — Spotify application client secret

#### Setup
Ensure credentials are configured in `misc.env` or your environment before starting.

---

### Scrutiny
**Disk Health Monitoring** — Real-time monitoring and tracking of disk health using SMART data.

- **Image**: `ghcr.io/analogj/scrutiny:v0.9.2-omnibus`
- **Container Name**: `scrutiny`
- **Restart Policy**: No automatic restart (manual restart required)

#### Ports
| Port | Service |
|------|---------|
| 9090 | Web UI |
| 8086 | InfluxDB Admin Interface |

#### Monitored Devices
Monitors all 12 drives in the system:
- `/dev/sda` through `/dev/sdm` (excluding `/dev/sdl`)

#### Volumes
- `./config:/opt/scrutiny/config` — Configuration files
- `./influxdb:/opt/scrutiny/influxdb` — Time-series database for historical data
- `/run/udev:/run/udev:ro` — Udev rules for drive detection

#### Capabilities
- `SYS_RAWIO` — Required for SMART data access

#### Access
Visit `http://<host>:9090` to view disk health dashboards and alerts.

---

### Threadfin
**IPTV/TV Streaming Server** — M3U/XMLTV-compatible streaming server for IPTV playlists and EPG.

- **Image**: `fyb3roptik/threadfin:1.2.37`
- **Container Name**: `threadfin`
- **Restart Policy**: Unless stopped

#### Ports
| Port | Service |
|------|---------|
| 34400 | Web UI & API |

#### Environment
- `PUID=568` — Process user ID
- `PGID=568` — Process group ID
- `TZ=America/Toronto` — Timezone

#### Volumes
- `/mnt/Starlink/Stacks/homelab-stacks/misc/configs/threadfin/conf` — Configuration and playlist data
- `/mnt/Starlink/Stacks/homelab-stacks/misc/configs/threadfin/temp` — Temporary files and cache

#### Access
Visit `http://<host>:34400` to configure playlists and access the IPTV interface.

---

### TeslaCam Viewer
**TeslaUSB clip browser** — Dark-theme web UI for Saved / Sentry / Recent clips archived to TrueNAS. Does not need the Pi online.

- **Image**: `ghcr.io/adsouza98/teslacam-viewer:1.1.0`
- **Container Name**: `teslacam-viewer`
- **Restart Policy**: Unless stopped

Version is pinned so Portainer git polling and Renovate can pick up new tags. Bump happens when `VERSION` in [Adsouza98/teslacam-viewer](https://github.com/Adsouza98/teslacam-viewer) is published to GHCR.

#### Ports
| Port | Service |
|------|---------|
| 8000 | Web UI (override with `TESLACAM_PORT`) |

#### Environment (set in Portainer)
| Variable | Default | Description |
|----------|---------|-------------|
| `PUID` | `568` | Process user ID (TrueNAS apps) |
| `PGID` | `568` | Process group ID |
| `TZ` | `America/Toronto` | Timezone |
| `TESLACAM_MEDIA_PATH` | `/mnt/Starlink/Tesla/TeslaUSB` | Host path to TeslaUSB dataset |
| `TESLACAM_AUTH_USER` | (empty) | Optional basic-auth username |
| `TESLACAM_AUTH_PASS` | (empty) | Optional basic-auth password |
| `TESLACAM_PORT` | `8000` | Host port |

Leave auth empty to disable login.

#### Volumes
- `${TESLACAM_MEDIA_PATH}:/media:ro` — TeslaUSB archive (read-only)
- `/mnt/Starlink/Stacks/homelab-stacks/misc/configs/teslacam-viewer` — thumbnail cache

#### GHCR pull (private image)
Add GitHub Container Registry in Portainer:
- URL: `https://ghcr.io`
- Username: GitHub username
- Password: PAT with `read:packages`

#### Access
Visit `http://<host>:8000` (or `TESLACAM_PORT`).

---

## Quick Start

1. Configure environment variables in `misc.env` (Discord token, API keys, TeslaCam media path/auth)
2. Start all services:
   ```bash
   docker-compose up -d
   ```
3. Access the services:
   - Scrutiny: `http://<host>:9090`
   - Threadfin: `http://<host>:34400`
   - TeslaCam Viewer: `http://<host>:8000`
   - Muse: Runs as Discord bot (invite to server via Discord)

## Configuration

All service configurations are stored in `./configs/`:
```
configs/
├── muse/              # Muse bot data
├── teslacam-viewer/   # thumbnail cache
└── threadfin/
    ├── conf/          # Playlists and settings
    └── temp/          # Temporary cache
```

Scrutiny stores its configuration locally in `./config` and time-series data in `./influxdb`.

## Notes

- **Scrutiny** requires privileged access (`SYS_RAWIO`) to read SMART data from all drives
- All services persist their data in mounted volumes for configuration and state preservation
- **Threadfin**, **Muse**, and **TeslaCam Viewer** use non-root user IDs (UID/GID 568) for security
