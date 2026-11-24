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

YouTube and Plex media relay with per-user accounts and a canvas/WebCodecs player. The NAS remuxes both sources into one fragmented MP4 so the Tesla browser can keep drawing on a canvas after the car leaves Park. It does not play through a `<video>` element, which the car pauses in Drive. 2.0.0 adds Cast: the car shows a six-character code, and a phone signed in as the same account can send a video file, a link, or the camera. Nothing extra is installed on the phone. 2.1.0 adds a Cabin player layout beside the original Classic bar. Classic stays the default. Cabin is chosen under Settings → Player layout, uses larger touch targets, and moves the picture settings behind More when the browser window is shorter than 640px. 2.2.0 browses YouTube as rows and opens a row's name — Plex or YouTube — as a vertical category that keeps loading as you scroll. 2.3.1 keeps the right-hand clock on the full duration while a scrub moves the left-hand clock, and playback starts there on release. 3.0.0 adds Twitch and Kick. Twitch sign-in shows followed channels that are live. Kick has no follow list, so sign-in unlocks the live directory and channel lookup. 3.1.0 adds Home, Popular, and Categories on both. Twitch Home lists followed channels, with latest broadcasts and clips when they are offline. Kick still has no follows, clips, or past streams, so its Home is the most-watched live channels. 3.2.0 can shuffle every episode of a Plex show, or one season, and keeps playing the shuffled queue. Each movie or TV library lists Collections directly under Recently added. 3.2.1 fits the five home tiles in the windowed and fullscreen Tesla browser, ignores a tap that is not a drag on the progress bar, and keeps YouTube audio with the picture after a seek. 3.3.0 adds a Shorts tab and leaves Shorts out of the other YouTube pages unless Settings → YouTube account → Show Shorts on YouTube pages is on. Search still mixes Shorts with longer videos. 3.3.1 logs the account, the page, play, pause, seek, quality, and player errors. The stack sets `TTW_LOG_LEVEL=debug`, so API timings are included too. Thumbnail and static-file requests stay quiet. 3.3.3 resumes an H.264 file by seeking it directly. A saved position no longer waits on Plex's transcoder, which is what timed out on VisionQuest while The Body played from the start. 3.3.4 reads a subtitle track out of the file instead of waiting for Plex to convert it. 3.3.5 clears the subtitle line when captions are turned off, including after that cue ends and after a seek. 3.3.6 uses the YouTube video's own captions and hides CC when a video has none. Twitch and Kick do not show CC. 3.3.7 keeps the player bar inside the screen, so the duration and More are not cut off on the right. A short window folds picture controls behind More instead of clipping them. 3.3.8 keeps the pitch the same when playback speed changes, instead of shifting it and warning that it will. 3.3.10 adds YouTube Premium's Skip button: on a Premium account it appears on the picture for a few seconds over a stretch most viewers skip, then goes away. Turn it on under Settings → YouTube account if the account menu does not say Premium. 3.3.11 uses YouTube's own Jump ahead marker, so the button shows at the same moment as on the phone and lands on the same second. 3.3.12 drops the YouTube Premium checkbox. The marker is public, so Jump ahead shows on any YouTube video. 3.3.13 puts that button in the bottom-right corner and leaves it up until the jump point, or for 30 seconds, whichever comes first. 3.3.14 makes one tap on Jump ahead skip immediately, without bringing the controls up. A tap anywhere else still does. 3.4.0 adds the same button to Plex, labeled the way Plex prompts (Skip Intro, Skip Credits, Skip Commercial) and only for a Plex Pass account. 3.4.1 actually shows that button during Plex playback. 3.5.0 adds Prev and Next on a Plex episode, including one started from the home screen. The first and last episode say so. 3.5.1 writes the Plex playhead back to the server so another Plex app can resume. YouTube does not allow an app to do that. 3.5.2 keeps the picture centered when the Tesla browser is expanded to full screen. 3.6.0 suggests titles while a Plex or YouTube search is typed, and keeps that list above the on-screen keyboard. 3.7.0 remembers the Plex or YouTube video that was playing, and a refresh opens it again a few seconds back. Back clears it. Live streams are not remembered. 3.8.0 adds Release notes on the Settings version block. The list opens over the page, covers this major version and the one before it, and loads older versions as you scroll. 3.8.1 is test-only. 3.8.2 adds Plex request timeouts and friendly error messages. 3.8.3 fixes Plex search results overflowing the page and enlarges Settings touch targets. 3.8.4 fixes the YouTube idle page in the windowed browser. 3.8.5 is test-only. 3.8.6 is a CI-only optional release webhook, using GitHub Actions secrets in the Tesla Take The Wheel repository only, with no app, compose, or env changes. Nothing in 3.8.1 through 3.8.6 adds env vars, ports, volumes, or runtime config since 3.8.0. 3.9.0 adds chapter selection on YouTube and Plex: the seek bar splits at chapters, the chapter name shows while scrubbing, a Chapters button and a horizontal chapter strip appear, and a tap on a chapter jumps there. The -30s and +30s buttons are removed on every source. Double-tap left or right still seeks 10 seconds. Plex progress write-back and resume still work. 3.9.1 is CI-only: a hardened flaky headless Chrome test, and GitHub Actions moved to Node 24, with no app change. Nothing in 3.9.0 or 3.9.1 adds env vars, ports, volumes, or runtime config. Register `https://ttw.juniorchange.com/api/twitch/oauth/callback` and `https://ttw.juniorchange.com/api/kick/oauth/callback` on those apps, then set the four `TTW_TWITCH_*` and `TTW_KICK_*` variables and redeploy.

Source: [Adsouza98/teslatakethewheel](https://github.com/Adsouza98/teslatakethewheel), a fork of [rafiknassif/teslatakethewheel](https://github.com/rafiknassif/teslatakethewheel). The image is published by this fork's GitHub Action, not the upstream repository.

- **Image**: `ghcr.io/adsouza98/teslatakethewheel:3.9.1` (linux/amd64). Do not use `latest` in this stack.
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
   | `TTW_LOG_LEVEL` | `debug` | `error`, `info`, or `debug`. `info` is sign-in, pages, playback, and errors. `debug` adds API timings. Thumbnail requests stay quiet. Recreate the container after changing it. |
   | `TTW_BIND_IP` | `0.0.0.0` | Host interface. NPM is another stack, so do not bind only `127.0.0.1`. |
   | `TTW_BEHIND_PROXY` | `0` | Set to `1` only after port 8096 is reachable solely from the proxy. |
   | `TTW_YOUTUBE_CLIENT_ID` | Empty | Google OAuth Web client ID |
   | `TTW_YOUTUBE_CLIENT_SECRET` | Empty | Matching client secret |
   | `TTW_TWITCH_CLIENT_ID` | Empty | Twitch application client id from dev.twitch.tv |
   | `TTW_TWITCH_CLIENT_SECRET` | Empty | Matching Twitch secret |
   | `TTW_KICK_CLIENT_ID` | Empty | Kick application client id from the Developer tab |
   | `TTW_KICK_CLIENT_SECRET` | Empty | Matching Kick secret |
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

Use the container Logs panel in Portainer. This stack leaves `TTW_LOG_LEVEL` at `debug`. That records who signed in, which page they opened, what they played, where they seeked, quality changes, player errors, and API timings. Thumbnail and static-file requests are not logged. Query strings, bodies, and authorization headers are not logged. If Portainer already sets `TTW_LOG_LEVEL`, change that value to `debug` or the default in the compose file is ignored. Set it back to `info` when a request trace is no longer needed. Docker rotates at 10 MB and keeps three files.

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
