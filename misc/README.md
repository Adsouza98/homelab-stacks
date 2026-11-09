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

YouTube and Plex media relay with per-user accounts and a canvas/WebCodecs player. The NAS remuxes both sources into one fragmented MP4 so the Tesla browser can keep drawing on a canvas after the car leaves Park. It does not play through a `<video>` element, which the car pauses in Drive.

Source: [Adsouza98/teslatakethewheel](https://github.com/Adsouza98/teslatakethewheel), a fork of [rafiknassif/teslatakethewheel](https://github.com/rafiknassif/teslatakethewheel). The image is published by this fork's GitHub Action, not the upstream repository.

- **Image**: `ghcr.io/adsouza98/teslatakethewheel:1.1.1` (linux/amd64). Do not use `latest` in this stack.
- **Container name**: `tesla-take-the-wheel`
- **User**: `568:568` (Compose `user`; the image has no `PUID`/`PGID`)
- **Restart**: unless stopped
- **Port**: `${TTW_BIND_IP:-0.0.0.0}:${TTW_PORT:-8096}:8080`
- **Data**: `/mnt/Starlink/Stacks/homelab-stacks/misc/configs/tesla-take-the-wheel` mounted at `/app/data`
- **Memory limit**: 2 GiB, so a last-resort libx264 session cannot exhaust the 32 GB host

The tag is pinned for Portainer Git polling. Renovate's Misc Stack rule includes `teslatakethewheel`. The fork bumps `package.json` on every merged pull request and publishes that version; this stack follows the tag, not `latest`.

#### Portainer

1. Prepare the data directory. For a new directory without a custom ACL:

   ```bash
   mkdir -p /mnt/Starlink/Stacks/homelab-stacks/misc/configs/tesla-take-the-wheel
   chown 568:568 /mnt/Starlink/Stacks/homelab-stacks/misc/configs/tesla-take-the-wheel
   chmod 700 /mnt/Starlink/Stacks/homelab-stacks/misc/configs/tesla-take-the-wheel
   ```

   On an existing directory, stop the container first and apply 568:568 to the files inside as well (`users.json` and `users/<id>.json`), or grant the same access through the dataset ACL.

2. Update the Git-backed **misc** stack from `misc/compose.yaml`. Optional environment:

   | Variable | Default | Purpose |
   |----------|---------|---------|
   | `TTW_PORT` | `8096` | Published host port |
   | `TTW_LOG_LEVEL` | `info` | `error`, `info`, or `debug`. Recreate the container after changing it. |
   | `TTW_BIND_IP` | `0.0.0.0` | Host interface. NPM is another stack, so do not bind only `127.0.0.1`. |
   | `TTW_BEHIND_PROXY` | `0` | Set to `1` only after port 8096 is reachable solely from the proxy. |
   | `TTW_YOUTUBE_CLIENT_ID` | Empty | Google OAuth Web client ID |
   | `TTW_YOUTUBE_CLIENT_SECRET` | Empty | Matching client secret |
   | `TTW_OPENSUBTITLES_API_KEY` | Empty | OpenSubtitles.com API key. CC still works for tracks already on the file without it. |
   | `TTW_OPENSUBTITLES_USERNAME` | Empty | OpenSubtitles account, needed to download a searched subtitle |
   | `TTW_OPENSUBTITLES_PASSWORD` | Empty | That account's password |

   The GHCR package is private. In Portainer, add a `ghcr.io` registry for `adsouza98` with a token that has `read:packages`.

3. Open `http://<NAS-IP>:8096` and create the admin account before exposing the hostname. Each person links their own Plex account and YouTube account. Back up `/app/data` and do not commit it. Keep the host path stable across image pulls; credentials are not in the image.

#### HTTPS

The Tesla browser will not open a private address. In Nginx Proxy Manager, proxy the public hostname with HTTP to the NAS LAN IP on port 8096 and enable SSL. Forward `Host`, `X-Forwarded-Host`, and `X-Forwarded-Proto`. After the firewall limits 8096 to the proxy, set `TTW_BEHIND_PROXY=1` and redeploy. Until then leave it at `0`, or a client that can reach the port can spoof those headers.

Disable response buffering or the car sits on a black screen:

```nginx
proxy_buffering off;
proxy_request_buffering off;
proxy_read_timeout 3600s;
```

#### YouTube

Search works without Google OAuth. For subscriptions and liked videos, enable YouTube Data API v3, create an OAuth **Web application** client, add test users if the consent screen is in Testing, and register:

```text
https://<your-public-hostname>/api/youtube/oauth/callback
```

Set both YouTube variables in Portainer and redeploy. Sign in from Settings on that same hostname. The callback must match the value the app shows.

The health check calls `/api/auth/state`, which does not require a session, so a new container is healthy before the admin account exists.

#### Logs

Use the container Logs panel in Portainer. `debug` adds timings and the detected OAuth callback, and it does not log query strings, bodies, or authorization headers. Set `TTW_LOG_LEVEL=debug`, recreate, then return to `info`. Docker rotates at 10 MB and keeps three files.

Playback cost on this host: stream copy and audio-only AAC re-encode are cheap on the i7-4790K. Full `libx264` is the path to avoid.

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
