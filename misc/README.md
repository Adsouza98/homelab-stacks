# Misc Stack

A collection of utility and monitoring services for the homelab, including music streaming, disk health monitoring, and IPTV streaming.

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

### Tesla Take The Wheel

YouTube and Plex media relay with per-user accounts and a canvas/WebCodecs player.
Source: [rafiknassif/teslatakethewheel](https://github.com/rafiknassif/teslatakethewheel).

- **Image**: `ghcr.io/rafiknassif/teslatakethewheel:1.0.0` (Linux amd64)
- **Container Name**: `tesla-take-the-wheel`
- **Restart Policy**: Unless stopped
- **Port**: `8097:8080` by default
- **Data**: `/mnt/Starlink/Stacks/homelab-stacks/misc/configs/tesla-take-the-wheel:/app/data`
- **Memory limit**: 2 GiB

The image tag is pinned for Portainer Git polling and Renovate updates. The
existing Misc Stack Renovate rule includes this image.

#### Portainer deployment

1. Prepare the data directory on the Docker host. The image runs as Node's
   UID/GID **1000:1000**; it does not implement `PUID`/`PGID`. Grant that identity
   write access through the dataset ACL, or for a new directory without a custom ACL:

   ```bash
   mkdir -p /mnt/Starlink/Stacks/homelab-stacks/misc/configs/tesla-take-the-wheel
   chown 1000:1000 /mnt/Starlink/Stacks/homelab-stacks/misc/configs/tesla-take-the-wheel
   chmod 700 /mnt/Starlink/Stacks/homelab-stacks/misc/configs/tesla-take-the-wheel
   ```

2. Deploy or update the existing Git-backed **misc** stack in Portainer using
   `misc/compose.yaml`. Enter these variables in the stack's environment editor
   as needed (for CLI use, pass `--env-file misc.env` from the `misc` directory):

   | Variable | Default | Purpose |
   |----------|---------|---------|
   | `TTW_PORT` | `8097` | Published host port |
   | `TTW_BIND_IP` | `0.0.0.0` | Host interface to publish on; use the NAS LAN IP for a proxy on another Docker network |
   | `TTW_BEHIND_PROXY` | `0` | Set to `1` after restricting port access to the trusted proxy |
   | `TTW_YOUTUBE_CLIENT_ID` | Empty | Google OAuth Web application client ID |
   | `TTW_YOUTUBE_CLIENT_SECRET` | Empty | Matching Google client secret |

   If the GHCR package is private, add `ghcr.io` under Portainer Registries with
   a GitHub username and a token with `read:packages` access. No local build is needed.

3. Open `http://<NAS-IP>:8097` and create the first **admin** account before
   making the app public. Each user links their own Plex account in Settings.
   Accounts, sessions, service credentials and settings persist under `/app/data`;
   back up this directory and keep it out of Git.

#### HTTPS and reverse proxy

The Tesla browser needs a public HTTPS hostname. In Nginx Proxy Manager, create a
proxy host for your chosen domain, forwarding via **http** to the NAS LAN IP on
port **8097** (or `TTW_PORT`), and enable SSL. NPM runs in the separate website
stack, so `127.0.0.1` would point to its own container.

Restrict the relay's published port to your proxy using host/network firewall
rules, then set `TTW_BEHIND_PROXY=1` and redeploy. This allows the app to use the
forwarded HTTPS scheme and hostname when generating the Google callback URL.
Forward `Host`, `X-Forwarded-Host` and `X-Forwarded-Proto` from the trusted proxy.
In the proxy host's Advanced configuration, disable response buffering for streaming:

```nginx
proxy_buffering off;
proxy_request_buffering off;
proxy_read_timeout 3600s;
```

Use the app's own account login. It supports multiple users; no shared Plex token
or account password belongs in this Compose file.

#### YouTube sign-in

Search and playback work without Google OAuth. For account features, enable
**YouTube Data API v3**, create a Google OAuth **Web application** client, add
your account as a test user if the consent screen is in Testing, and register:

```text
https://<your-public-hostname>/api/youtube/oauth/callback
```

Set both `TTW_YOUTUBE_CLIENT_ID` and `TTW_YOUTUBE_CLIENT_SECRET` in Portainer and
redeploy. Open the app at that HTTPS hostname, then Settings → YouTube account →
**Sign in with Google**. The callback must match the value displayed in Settings
exactly. Keep credentials in Portainer or an untracked `misc.env`.

The stack overrides the image's authenticated health check with
`/api/auth/state`, so it reports healthy before an account is created.

---

## Quick Start

1. Configure environment variables in `misc.env` (Discord token, API keys)
2. Start all services:
   ```bash
   docker-compose up -d
   ```
3. Access the services:
   - Scrutiny: `http://<host>:9090`
   - Threadfin: `http://<host>:34400`
   - Muse: Runs as Discord bot (invite to server via Discord)

## Configuration

All service configurations are stored in `./configs/`:
```
configs/
├── muse/              # Muse bot data
├── threadfin/
│   ├── conf/          # Playlists and settings
│   └── temp/          # Temporary cache
```

Scrutiny stores its configuration locally in `./config` and time-series data in `./influxdb`.

## Notes

- **Scrutiny** requires privileged access (`SYS_RAWIO`) to read SMART data from all drives
- All services persist their data in mounted volumes for configuration and state preservation
- **Threadfin** and **Muse** use non-root user IDs (UID/GID 568) for security
