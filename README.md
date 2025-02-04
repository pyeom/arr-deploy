# arr-deploy
Deploy Arr Stack, no VPN (Not recommended)

## Install cockpit (Users/Group UI manager for Linux)

```
apt install --no-install-recommends cockpit -y
```

```
apt install wsdd
```

- Install this add-ons for cockpit (download with wget):
  - [File-sharing](https://github.com/45Drives/cockpit-file-sharing/releases)
  - [Identities](https://github.com/45Drives/cockpit-identities/releases)
  - [Navigator](https://github.com/45Drives/cockpit-navigator/releases)

```
apt install ./*.deb
```
Do all the user and group config, also the smb shares.

## Compose file
```
---
services:
  qbittorrent:
    image: lscr.io/linuxserver/qbittorrent:latest
    container_name: qbittorrent
    restart: unless-stopped
    ports:
      - 8080:8080 # web interface
      - 6881:6881 # torrent port
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Santiago
      - WEBUI_PORT=8080
      - TORRENTING_PORT=6881
    volumes:
      - /docker/qbittorrent:/config
      - /data:/data
  prowlarr:
    image: lscr.io/linuxserver/prowlarr:latest
    container_name: prowlarr
    ports:
      - 9696:9696
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Santiago
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /docker/prowlarr:/config
    restart: unless-stopped
  sonarr:
    image: lscr.io/linuxserver/sonarr:latest
    container_name: sonarr
    restart: unless-stopped
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Santiago
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /docker/sonarr:/config
      - /data:/data
    ports:
      - 8989:8989

  radarr:
    image: lscr.io/linuxserver/radarr:latest
    container_name: radarr
    restart: unless-stopped
    environment:
      - PUID=1000
      - PGID=100
      - TZ=America/Santiago
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /docker/radarr:/config
      - /data:/data
    ports:
      - 7878:7878
  bazarr:
    image: lscr.io/linuxserver/bazarr:latest
    container_name: bazarr
    restart: unless-stopped
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Santiago
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /docker/bazarr:/config
      - /data:/data
    ports:
      - 6767:6767
  flaresolverr:
    # DockerHub mirror flaresolverr/flaresolverr:latest
    image: ghcr.io/flaresolverr/flaresolverr:latest
    container_name: flaresolverr
    environment:
      - LOG_LEVEL=${LOG_LEVEL:-info}
      - LOG_HTML=${LOG_HTML:-false}
      - CAPTCHA_SOLVER=${CAPTCHA_SOLVER:-none}
      - TZ=America/Santiago
    ports:
      - "${PORT:-8191}:8191"
    restart: unless-stopped
  overseerr:
      image: lscr.io/linuxserver/overseerr:latest
      container_name: overseerr
      environment:
        - PUID=1000
        - PGID=1000
        - TZ=America/Santiago
      volumes:
        - /docker/overseer/config:/config
        - /data:/data
      ports:
        - 5055:5055
      restart: unless-stopped
```

Run the script:
```
docker compose up -d
```
