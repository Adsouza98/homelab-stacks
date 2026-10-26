# Misc Stack

A collection of utility and monitoring services for the homelab, including music streaming, disk health monitoring, IPTV streaming, and the Vela cabin player.

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

### Tesla Media Player (Vela)

**Cabin player for the Tesla browser** — YouTube and Plex on the passenger screen. The picture is decoded with WebCodecs and drawn on a canvas, so Tesla's Drive-mode block on `<video>` does not stop playback.

- **Image**: `ghcr.io/adsouza98/tesla-media-player:0.2.1`
- **Source**: [Adsouza98/tesla-media-player](https://github.com/Adsouza98/tesla-media-player)
- **Container Name**: `tesla-media-player`
- **Restart Policy**: Unless stopped
- **Not VPN-routed.** The car must be able to reach this port, and the container must be able to reach Plex on the LAN.

**Passenger use only.** Watching video on the driver's display while the car is moving is illegal and dangerous.

#### Ports
| Port | Service |
|------|---------|
| 8095 | Web UI and proxied streams |

#### Environment
- `PUID=568` / `PGID=568`
- `TZ=America/Toronto`
- `VELA_PASSWORD` — owner cabin password (`APP_PASSWORD` inside the container). Leave the username blank on the sign-in screen to use it. If empty, the first start writes one to `generated-password` in the config volume.
- `VELA_OWNER_EMAIL` — email on your Plex account. Also the Google account if you turn Google on. Fills the owner seat when that field is still blank.
- `VELA_OWNER_PLEX` — your Plex username. Same rule: only fills a blank field.
- `VELA_TEST_PASSWORD` — password for the Grok test seat (username `grok`). If empty, the first start writes it to `test-user` in the config volume and shows it once in Settings.
- `VELA_PUBLIC_URL` — public https origin, for example `https://vela.example.com`. Needed only if Google sign-in is on.
- `VELA_GOOGLE_CLIENT_ID` / `VELA_GOOGLE_CLIENT_SECRET` — optional. Leave empty. The Tesla browser usually blocks Google's own login page, so Plex is the car path.
- `PLEX_URL` — defaults to `http://192.168.1.42:32400`. Must be reachable from the container.
- `PLEX_TOKEN` — Plex token for the library. The Tesla never sees this. This is not the sign-in token.
- `VELA_QUALITY` — `480`, `720`, `1080` (default), or `max`

Two seats only: the owner, and the Grok test user. A Plex or Google account is refused until that seat lists the username or email. Sign in once with the cabin password, then use Settings → Link Plex, or set `VELA_OWNER_EMAIL` and `VELA_OWNER_PLEX` before the first start. The test seat can play. It cannot change the Plex token, YouTube cookies, or who is allowed in.

Nginx Proxy Manager and Cloudflare only carry the connection. Do not add a second login there.

#### Volumes
- `/mnt/Starlink/Stacks/homelab-stacks/misc/configs/tesla-media-player` — settings, seats, generated passwords, and YouTube cookies

Put a Netscape `cookies.txt` exported from a YouTube Premium browser session at:

```text
/mnt/Starlink/Stacks/homelab-stacks/misc/configs/tesla-media-player/youtube-cookies.txt
```

Or upload it from Vela's Settings page after signing in.

#### HTTPS (required for full quality)

WebCodecs only runs in a secure context. Point Nginx Proxy Manager at `http://<nas>:8095` with websockets enabled, force SSL, and add this under Advanced:

```nginx
proxy_buffering off;
proxy_request_buffering off;
```

Without that, the proxy buffers the stream and the picture never starts. Plain HTTP on port 8095 still loads the UI and can use compatibility mode (MPEG-1, heavier on the 4790K, lower quality).

#### Access
`http://<host>:8095` on the LAN, or the HTTPS hostname you create in Nginx Proxy Manager. Open the HTTPS name in the Tesla browser. In the car, use **Sign in with Plex** and approve the code on your phone at `plex.tv/link`.

---

## Quick Start

1. Configure environment variables in `misc.env` (Discord token, API keys, Vela password, Plex token)
2. Create the Vela config directory if it does not exist:
   ```bash
   mkdir -p /mnt/Starlink/Stacks/homelab-stacks/misc/configs/tesla-media-player
   ```
3. Start all services:
   ```bash
   docker-compose up -d
   ```
4. Access the services:
   - Scrutiny: `http://<host>:9090`
   - Threadfin: `http://<host>:34400`
   - Vela: `http://<host>:8095` (put this behind HTTPS before using it in the car)
   - Muse: Runs as Discord bot (invite to server via Discord)

## Configuration

All service configurations are stored in `./configs/`:
```
configs/
├── muse/              # Muse bot data
├── threadfin/
│   ├── conf/          # Playlists and settings
│   └── temp/          # Temporary cache
└── tesla-media-player/  # Vela settings, seats, cookies, generated passwords
```

Scrutiny stores its configuration locally in `./config` and time-series data in `./influxdb`.

## Notes

- **Scrutiny** requires privileged access (`SYS_RAWIO`) to read SMART data from all drives
- All services persist their data in mounted volumes for configuration and state preservation
- **Threadfin**, **Muse**, and **Vela** use non-root user IDs (UID/GID 568) for security
- **Vela** is for passenger use only. Full H.264 quality needs HTTPS in front of port 8095.
