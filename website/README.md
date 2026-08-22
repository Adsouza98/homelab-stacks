# Website Stack

A complete web service stack featuring a dashboard, request management system, and reverse proxy with SSL termination.

## Services

### Organizr
**Homelab Dashboard** — A customizable homepage to organize and access all homelab services.

- **Image**: `organizr/organizr:latest` (floating tag — not updated by Renovate; pin a version tag to enable automated bumps)
- **Container Name**: `organizr`
- **Restart Policy**: Unless stopped

#### Ports
| Port | Service |
|------|---------|
| 85 | Web UI |

#### Environment
- `PUID=568` — Process user ID
- `PGID=568` — Process group ID
- `TZ=America/New_York` — Timezone

#### Volumes
- `/mnt/Starlink/Stacks/homelab-stacks/website/configs/organizr:/config` — Configuration and dashboard data

#### Access
Visit `http://<host>:85` to access the Organizr dashboard.

---

### Ombi
**Request/Demand Management** — User-friendly interface for requesting movies, TV shows, and music additions.

- **Image**: `lscr.io/linuxserver/ombi:v4.53.10-ls255`
- **Container Name**: `ombi`
- **Restart Policy**: Unless stopped
- **Dependency**: Requires the `mariadb` service to be healthy

#### Ports
| Port | Service |
|------|---------|
| 3579 | Web UI |

#### Environment
- `PUID=568` — Process user ID
- `PGID=568` — Process group ID
- `TZ=America/New_York` — Timezone
- `BASE_URL=/` — Optional URL path prefix (commented out by default)

#### Volumes
- `/mnt/Starlink/Stacks/homelab-stacks/website/configs/ombi:/config` — Configuration and request data

#### Access
Visit `http://<host>:3579` to access Ombi's request interface.

---

### Nginx Proxy Manager
**Reverse Proxy & SSL Manager** — Manages reverse proxy rules, subdomains, and SSL certificates (Let's Encrypt).

- **Image**: `jc21/nginx-proxy-manager:2.15.1`
- **Container Name**: `nginx-proxy-manager`
- **Restart Policy**: No automatic restart
- **Dependency**: Requires the `mariadb` service to be healthy

#### Ports
| Port | Service |
|------|---------|
| 80 | HTTP |
| 82 | Admin UI |
| 443 | HTTPS |

#### Environment
- `PUID=0`, `PGID=0` — Root privileges for port binding
- `DB_MYSQL_HOST=mariadb` — Database hostname
- `DB_MYSQL_PORT=3306` — Database port
- `DB_MYSQL_USER=npm` — Database user
- `DB_MYSQL_PASSWORD` — Database password (from env vars)
- `DB_MYSQL_NAME=npm` — Database name

#### Volumes
- `/mnt/Starlink/Stacks/homelab-stacks/website/configs/nginx-proxy-manager/data:/data` — Proxy rules and configuration
- `/mnt/Starlink/Stacks/homelab-stacks/website/configs/nginx-proxy-manager/letsencrypt:/etc/letsencrypt` — SSL certificates
- `/mnt/Starlink/Stacks/homelab-stacks/website/configs/nginx-proxy-manager/theme-park/98-themepark-npm/98-themepark:/etc/cont-init.d/99-themepark` — Theme customization

#### Access
- Admin UI: `http://<host>:82`
- Default credentials: Check NPM documentation or logs on first run

---

### MariaDB (Ombi and Nginx Proxy Manager)
**Database Backend** — Dedicated MariaDB instance for Ombi and Nginx Proxy Manager, using InnoDB tables.

- **Image**: `mariadb:11.4`
- **Container Name**: `mariadb`
- **Restart Policy**: Unless stopped

#### Environment
- `MYSQL_ROOT_PASSWORD` — Root password (from env vars)

#### Volumes
- `/mnt/Starlink/Stacks/homelab-stacks/website/configs/mariadb:/var/lib/mysql` — Database files

#### Configuration
- UTF-8 (`utf8mb4`) character set and collation
- Dynamic InnoDB row format
- `max_allowed_packet=256M`
- `innodb-buffer-pool-size=512M`
- Healthcheck required by Ombi and Nginx Proxy Manager before startup
- Nginx Proxy Manager uses the `npm` database and user
- Nginx Proxy Manager tables have been converted from Aria to InnoDB

#### Port
- `3306` (internal, not exposed) — MariaDB port

---

## Quick Start

1. Configure environment variables in `website.env`:
   ```env
   DB_PASSWORD=<database_password>
   DB_ROOT_PASSWORD=<root_password>
   ```

2. Start all services:
   ```bash
   docker compose up -d
   ```

3. Access the services:
   - **Organizr Dashboard**: `http://<host>:85`
   - **Ombi Requests**: `http://<host>:3579`
   - **Nginx Proxy Manager Admin**: `http://<host>:82`

## Configuration

Service configurations are stored in `./configs/`:
```
configs/
├── organizr/                    # Organizr dashboard config
├── ombi/                        # Ombi request system config
├── nginx-proxy-manager/
│   ├── data/                    # Proxy rules and settings
│   ├── letsencrypt/             # SSL certificates
│   └── theme-park/98-themepark/ # Theme customization
└── mariadb/                      # Ombi and Nginx Proxy Manager database files
```

## Service Relationships

```
Ombi ──┐
       ├─→ MariaDB (mariadb)
NPM ───┘
```

Ombi and Nginx Proxy Manager use the shared `mariadb` service. Both wait for its healthcheck to pass before starting.

## Notes

- **Nginx Proxy Manager** runs as root (`PUID=0`) to bind to privileged ports (80, 443)
- **Database credentials** must be configured in `website.env` before starting services
- Nginx Proxy Manager uses InnoDB tables in the active `mariadb` database
- All data persists in mounted volumes for backup and recovery
- SSL certificates are automatically managed by Let's Encrypt via Nginx Proxy Manager
