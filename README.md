# arr-deploy
Deploy Arr Stack, no VPN (Not recommended)

## Create and mount volumes
You are probably using proxmox and an LXC container or a vm for this, attach two volumes to it:

- docker configs volume (/docker)
- vault storage volume (/data)

Create a user and give it permissions to modify those mounts:

`adduser <name>`
`usermod -aG sudo <name>`
`chown <name> /docker`
`chown <name> /data`

You obviously need docker and the compose plugin to do everything so install it from (the official source)[https://docs.docker.com/engine/]

### Config the mounts

For the /docker there's nothing much to do, just attach every service config in there.
As for the /data we'll do the following:

- Create folders:
`cd /data`
`mkdir -p downloads/{completed, incomplete, torrents}`
`mkdir Movies Shows`

Now everything is set up, just create the compose file in the home directory of your user (just use `cd` in the terminal).

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
  jellyseer:
    image: ghcr.io/fallenbagel/jellyseerr:latest
    init: true
    container_name: jellyseerr
    environment:
      - LOG_LEVEL=debug
      - TZ=America/Santiago
      - PORT=5055 #optional
    ports:
      - 5055:5055
    volumes:
      - /docker/jellyseer:/config
      - /data:/data
    restart: unless-stopped
```

Spin up the containers:
```
docker compose up -d
```

Spin up your Jellyfin instance too:
```
services:
  jellyfin:
    image: lscr.io/linuxserver/jellyfin:latest
    container_name: jellyfin
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Santiago
    volumes:
      - ./config:/config
      - /data:/data
    ports:
      - 8096:8096
      - 7359:7359/udp #Service Discovery
      - 1900:1900/udp #Client Discovery
    restart: unless-stopped
```
You can add the service to the compose file above or make them separate, no difference.

### Configure qbittorrent:
- Retrieve the randomly generated password
```
docker logs qbittorrent
```
- Enter: <my_ip>:8080 and login with `admin/password`, navigate to settings -> Web UI and change the password

### Configure indexers and other Arr apps:
- Create a tag for flaresolverr inside prowlarr and then add indexers
- Configure the rest of the arr apps from prowlarr
- Configure qBittorrent as download agent inside Sonarr and Radarr
- Configure volumes inside Sonarr and Radarr
- Configre Jellyseer with your Jellyfin instance, Radarr and Sonarr
- Pick you favorite Movie/Show, wait until the download ends and enjoy it on Jellyfinn
