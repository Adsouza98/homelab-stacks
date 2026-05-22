# Website Stack

A complete web service stack featuring a dashboard, request management system, and reverse proxy with SSL termination.

## Services

### Organizr
**Homelab Dashboard** — A customizable homepage to organize and access all homelab services.

- **Image**: `organizr/organizr:latest`
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

- **Image**: `lscr.io/linuxserver/ombi:latest`
- **Container Name**: `ombi`
- **Restart Policy**: Unless stopped
- **Dependency**: Requires database service to be running

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

- **Image**: `jc21/nginx-proxy-manager:latest`
- **Container Name**: `nginx-proxy-manager`
- **Restart Policy**: No automatic restart

#### Ports
| Port | Service |
|------|---------|
| 80 | HTTP |
| 82 | Admin UI |
| 443 | HTTPS |

#### Environment
- `PUID=0`, `PGID=0` — Root privileges for port binding
- `DB_MYSQL_HOST=db` — Database hostname
- `DB_MYSQL_PORT=3306` — Database port
- `DB_MYSQL_USER` — Database user (from env vars)
- `DB_MYSQL_PASSWORD` — Database password (from env vars)
- `DB_MYSQL_NAME` — Database name (from env vars)

#### Volumes
- `/mnt/Starlink/Stacks/homelab-stacks/website/configs/nginx-proxy-manager/data:/data` — Proxy rules and configuration
- `/mnt/Starlink/Stacks/homelab-stacks/website/configs/nginx-proxy-manager/letsencrypt:/etc/letsencrypt` — SSL certificates
- `/mnt/Starlink/Stacks/homelab-stacks/website/configs/nginx-proxy-manager/theme-park/98-themepark-npm/98-themepark:/etc/cont-init.d/99-themepark` — Theme customization

#### Access
- Admin UI: `http://<host>:82`
- Default credentials: Check NPM documentation or logs on first run

---

### Database (MariaDB)
**Database Backend** — Stores configuration for Nginx Proxy Manager and Ombi.

- **Image**: `jc21/mariadb-aria:latest`
- **Container Name**: `db`
- **Restart Policy**: No automatic restart

#### Environment
- `PUID=568`, `PGID=568` — Process user ID and group ID
- `MYSQL_DATABASE` — Database name (from env vars)
- `MYSQL_USER` — Database user (from env vars)
- `MYSQL_PASSWORD` — Database password (from env vars)
- `MYSQL_ROOT_PASSWORD` — Root password (from env vars)

#### Volumes
- `/mnt/Starlink/Stacks/homelab-stacks/website/configs/mysql:/var/lib/mysql` — Database files

#### Port
- `3306` (internal, not exposed) — MySQL/MariaDB port

---

### MariaDB Client
**Database Utility** — Helper container for executing database commands and maintenance.

- **Image**: `mariadb:latest`
- **Container Name**: `mariadb-client`
- **Restart Policy**: Unless stopped
- **Command**: `sleep infinity` — Keeps container running for interactive access

#### Usage
Exec into the container to run database queries:
```bash
docker exec -it mariadb-client mysql -h db -u <user> -p <password>
```

---

## Quick Start

1. Configure environment variables in `website.env`:
   ```env
   DB_NAME=<database_name>
   DB_USER=<database_user>
   DB_PASSWORD=<database_password>
   DB_ROOT_PASSWORD=<root_password>
   ```

2. Start all services:
   ```bash
   docker-compose up -d
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
└── mysql/                       # MariaDB database files
```

## Service Dependencies

```
Ombi ──┐
       ├─→ MariaDB (db)
NPM ───┘
```

Both Ombi and Nginx Proxy Manager depend on the MariaDB database for configuration storage.

## Notes

- **Nginx Proxy Manager** runs as root (`PUID=0`) to bind to privileged ports (80, 443)
- **Database credentials** must be configured in `website.env` before starting services
- All data persists in mounted volumes for backup and recovery
- SSL certificates are automatically managed by Let's Encrypt via Nginx Proxy Manager
- The **MariaDB Client** container is optional but useful for database administration
