# Misc Stack

A collection of utility and monitoring services for the homelab, including Discord bots, disk health monitoring, and IPTV streaming.

## Services

### Muse
**Discord Music Bot** — Music streaming for Discord with support for YouTube and Spotify.

- **Image**: `ghcr.io/museofficial/muse:2.11.7`
- **Restart Policy**: Always
- **Config Path**: `/mnt/Starlink/Stacks/homelab-stacks/misc/configs/muse`

#### Required Environment Variables
- `DISCORD_TOKEN` — Discord bot token for authentication
- `YOUTUBE_API_KEY` — YouTube API key for music search/streaming
- `SPOTIFY_CLIENT_ID` — Spotify application client ID
- `SPOTIFY_CLIENT_SECRET` — Spotify application client secret

#### Setup
Ensure credentials are configured in `misc.env` or Portainer stack env before starting.

---

### Groksito
**Grok Discord bot** — Conversational Grok in Discord using SuperGrok OAuth (no xAI API credits).

- **Images**:
  - `ghcr.io/lupintic/groksito-discord-bot:0.2.0-pre.1` (bot)
  - `ghcr.io/lupintic/groksito-discord-bot-web:0.2.0-pre.1` (LAN dashboard)
- **Restart Policy**: Unless stopped
- **Auth**: `GROK_AUTH_MODE=oauth` (SuperGrok / X Premium+ quota)
- **Video generation**: off (`ENABLE_VIDEO_GENERATION=false`)
- **Network**: LAN only. Do **not** attach to Gluetun / ExpressVPN.

#### Ports
| Port | Service |
|------|---------|
| 8010 | Groksito web dashboard (status / quotas / safe config) |

#### Required Portainer environment variables
- `GROKSITO_DISCORD_BOT_TOKEN` — Discord bot token for Groksito (**not** Muse's `DISCORD_TOKEN`)
- `GROKSITO_ALLOWED_GUILD_IDS` — Discord server ID (snowflake). Restricts the bot to that guild.

See `groksito.env.example`.

#### Volumes
- `/mnt/Starlink/Stacks/homelab-stacks/misc/configs/groksito/data` — conversation state
- `/mnt/Starlink/Stacks/homelab-stacks/misc/configs/groksito/oauth` — SuperGrok OAuth tokens (`xai_oauth_tokens.json`)

Create both folders before the first deploy (apps user `568` if you chown them).

#### Discord application (one-time)
1. [Discord Developer Portal](https://discord.com/developers/applications) → New Application.
2. Bot → Reset Token → paste into `GROKSITO_DISCORD_BOT_TOKEN`.
3. Privileged Gateway Intents → enable **Message Content Intent** only.
4. Invite with Send Messages, Embed Links, Attach Files, Read History, Add Reactions, Slash Commands.
5. Developer Mode → right-click server name → Copy Server ID → `GROKSITO_ALLOWED_GUILD_IDS`.
6. Restrict the Groksito role: deny View Channel on every channel except `#commands`.

#### SuperGrok OAuth (one-time, after the volume exists)
The GHCR image has `CMD ["groksito"]` and **no ENTRYPOINT**. Flags after the image name replace the binary unless you pass `groksito` first.

Phone / Terminus (recommended — no SSH tunnel):

```bash
docker run --rm -it \
  -e GROK_AUTH_MODE=oauth \
  -e GROK_OAUTH_TOKEN_FILE=/app/oauth/xai_oauth_tokens.json \
  -e GROK_OAUTH_PORT=56121 \
  -v /mnt/Starlink/Stacks/homelab-stacks/misc/configs/groksito/oauth:/app/oauth \
  ghcr.io/lupintic/groksito-discord-bot:0.2.0-pre.1 \
  groksito --login-oauth --print-url-only --manual-paste
```

Open the printed URL in the phone browser, sign in with SuperGrok, then copy the address bar (`http://127.0.0.1:56121/callback?code=...` — the page will fail to load; that is expected) and paste it into the SSH session.

Confirm tokens, then restart the bot:

```bash
ls -l /mnt/Starlink/Stacks/homelab-stacks/misc/configs/groksito/oauth/xai_oauth_tokens.json
docker restart groksito
```

OAuth is experimental. A 403 means that SuperGrok surface is blocked for the account.

#### What Groksito can do

Chat is the main interface. There is no `/ask` command. Mention `@Groksito` or reply to it in `#commands`.

**Natural language (mention / reply)**
- Chat with Grok (`grok-4.3`) using SuperGrok quota
- Native vision: attach or link images, or reply to a message that has one
- Generate an image: "generate an image of …" (Grok Imagine)
- Edit an image: attach a picture and ask for changes
- Read text aloud: "read this in voice" / "lee esto en voz alta"
- Video generation is **disabled** in this stack (`ENABLE_VIDEO_GENERATION=false`)
- Native web / X search when the model needs it
- Can reply, react, or open a thread

**Slash commands**
| Command | What it does |
|---------|----------------|
| `/audio` | Generate TTS audio (voices: eve, ara, rex, sal, leo; language control) |
| `/stmchr` | Fixed popular Steam character chart |
| `/steamchart` | Custom Steam games chart |
| `/topgames` | Live top games from Steam Charts |
| `/topkorea` | Live top 10 Korean PC bang games (TheLog) |
| `/korea50` | Weekly top 50 Korean games (Gamemeca) |

**Context menu**
- Right-click a message → Apps → **Leer en voz alta** — TTS of that message.

**Rate limit:** 6 requests / 60 seconds per user (before LLM calls). Image/TTS limits come from the SuperGrok plan, not the bot.

#### Access
- Dashboard: `http://<host>:8010`
- Bot: mention in `#commands` after it shows online

---

### Scrutiny
**Disk Health Monitoring** — Real-time monitoring and tracking of disk health using SMART data.

- **Image**: `ghcr.io/analogj/scrutiny:v0.9.3-omnibus`
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

## Quick Start

1. Configure environment variables in Portainer (or `misc.env`): Discord tokens, API keys, Groksito guild ID
2. Create Groksito data/oauth folders, complete SuperGrok OAuth once
3. Start all services:
   ```bash
   docker-compose up -d
   ```
4. Access the services:
   - Scrutiny: `http://<host>:9090`
   - Threadfin: `http://<host>:34400`
   - Groksito dashboard: `http://<host>:8010`
   - Muse / Groksito: Discord bots (invite each application separately)

## Configuration

All service configurations are stored in `./configs/`:
```
configs/
├── muse/              # Muse bot data
├── groksito/
│   ├── data/         # Groksito conversation state
│   └── oauth/        # SuperGrok OAuth tokens (do not commit)
└── threadfin/
    ├── conf/          # Playlists and settings
    └── temp/          # Temporary cache
```

Scrutiny stores its configuration locally in `./config` and time-series data in `./influxdb`.

## Notes

- **Scrutiny** requires privileged access (`SYS_RAWIO`) to read SMART data from all drives
- All services persist their data in mounted volumes for configuration and state preservation
- **Threadfin** and **Muse** use non-root user IDs (UID/GID 568) for security
- **Groksito** and **Muse** are separate Discord applications with separate tokens
- **Groksito** stays off the VPN stack so the Discord gateway remains stable
