# Commands to build media server

## Creat docker network

```bash
docker network create --subnet=172.20.0.0/16 medianet
```

## Nzbget

```bash
docker create \
  --name nzbget \
  --restart always \
  --net medianet \
  --ip 172.20.0.2 \
  -p 6789:6789 \
  -v /mnt/datastore/nzbget/config:/config \
  -v /mnt/datastore/nzbget/downloads:/downloads \
  -e PUID=0 \
  -e PGID=0 \
  -e TZ=US \
  linuxserver/nzbget
```

## Plex

```bash
docker create \
  --name plex \
  --restart always \
  --net medianet \
  --ip 172.20.0.3 \
  -p 32400:32400/tcp \
  -p 3005:3005/tcp \
  -p 8324:8324/tcp \
  -p 32469:32469/tcp \
  -p 1900:1900/udp \
  -p 32410:32410/udp \
  -p 32412:32412/udp \
  -p 32413:32413/udp \
  -p 32414:32414/udp \
  -e TZ=US \
  -e ADVERTISE_IP="http://10.10.1.251:32400/" \
  -e PLEX_UID=0 \
  -e PLEX_GID=0 \
  -e PLEX_CLAIM=claim-Z54o53E_zw7yqEj12MFE \
  -h media-server \
  -v /mnt/datastore/plex/config:/config \
  -v /mnt/datastore/plex/transcode:/transcode \
  -v /mnt/tv:/tv \
  -v /mnt/movies:/movies \
  -v /mnt/filler:/filler \
  --device=/dev/dri/renderD128:/dev/dri/renderD128 \
  plexinc/pms-docker:plexpass
```

## Sonarr

```bash
docker create \
  --name=sonarr \
  --restart always \
  --net medianet \
  --ip 172.20.0.4 \
  -p 8989:8989 \
  -e PUID=0 \
  -e PGID=0 \
  -e TZ=US \
  -v /mnt/datastore/sonarr/config:/config \
  -v /mnt/datastore/nzbget/downloads:/downloads \
  -v /mnt/tv:/tv \
  linuxserver/sonarr:latest
```

## Radarr

```bash
docker create \
  --name=radarr \
  --restart always \
  --net medianet \
  --ip 172.20.0.5 \
  -v /mnt/datastore/radarr/config:/config \
  -v /mnt/datastore/nzbget/downloads:/downloads \
  -v /mnt/movies:/movies \
  -e TZ=US \
  -e PGID=0 \
  -e PUID=0 \
  -p 7878:7878 \
  linuxserver/radarr
```

## Overseerr

```bash
docker run -d \
  --name overseerr \
  -e TZ=US \
  -e PGID=0 \
  -e PUID=0 \
  --net medianet \
  --ip 172.20.0.6 \
  -p 5055:5055 \
  -v /mnt/datastore/overseerr:/app/config \
  --restart always \
  sctx/overseerr
```

## netbootxyz

```bash
docker run -d \
  --name=netbootxyz \
  -e TZ=US \
  -e PGID=0 \
  -e PUID=0 \
  --net medianet \
  --ip 172.20.0.7 \
  -e MENU_VERSION=2.0.59             `# optional` \
  -p 3000:3000                       `# sets webapp port` \
  -p 69:69/udp                       `# sets tftp port` \
  -p 8080:80                         `# optional` \
  -v /mnt/datastore/netbootxyz/config:/config   `# optional` \
  -v /mnt/datastore/netbootxyz/assets:/assets   `# optional` \
  --restart always \
  ghcr.io/netbootxyz/netbootxyz
```

```bash
/tool fetch url="https://boot.netboot.xyz/ipxe/netboot.xyz.efi"
/ip tftp add ip-addresses=10.10.1.0/24 req-filename=netboot.xyz.efi real-filename=netboot.xyz.efi allow=yes read-only=yes
/ip dhcp-server network set 0 next-server=10.10.1.20 boot-file-name=netboot.xyz.efi
```

## Calibre-Web
```bash
docker run -d \
  --name=calibre-web \
  -e PUID=0 \
  -e PGID=0 \
  -e TZ=Etc/UTC \
  -e DOCKER_MODS=linuxserver/mods:universal-calibre \
  -p 8083:8083 \
  --net medianet \
  --ip 172.20.0.8 \
  -v /mnt/datastore/calibre-web/config:/config \
  -v /mnt/datastore/calibre-web/books:/books \
  --restart always \
  lscr.io/linuxserver/calibre-web:latest
```

## Portainer
```bash
docker run -d \
  -p 8000:8000 \
  -p 9443:9443 \
  --name portainer \
  --net medianet \
  --ip 172.20.0.9 \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /mnt/datastore/portainer_data:/data \
  portainer/portainer-ee:lts
```

## NGINX
```yml
services:
  nginx-proxy-manager:
    image: jc21/nginx-proxy-manager:latest
    container_name: nginx
    restart: always
    ports:
      - "80:80"
      - "443:443"
      - "81:81"
    volumes:
      - ./mnt/datastore/nginx/data:/data
      - ./letsencrypt:/etc/letsencrypt
    networks:
      medianet:
        ipv4_address: 172.20.0.10

networks:
  medianet:
    external: true
```

```bash
docker compose up -d
```

## IT Tools
```bash
docker run -d \
  --name it-tools \
  --restart always \
  -p 8080:80 \
  --net medianet \
  --ip 172.20.0.11 \
  corentinth/it-tools:latest
```

## Audiobookshelf
```bash
docker run -d \
 --name audiobookshelf \
 -e PUID=0 \
 -e PGID=0 \
 -e TZ=Etc/UTC \
 -p 13378:80 \
 --net medianet \
 --ip 172.20.0.12 \
 --restart=always \
 -v /mnt/datastore/audiobookshelf:/config \
 -v /mnt/datastore/audiobookshelf:/metadata \
 -v /mnt/audiobooks:/audiobooks \
 -v /mnt/podcasts:/podcasts \
 ghcr.io/advplyr/audiobookshelf
```

## LazyLibrarian
```bash
docker run -d \
  --name=lazylibrarian \
  -e PUID=0 \
  -e PGID=0 \
  -e TZ=Etc/UTC \
  -p 5299:5299 \
  --net medianet \
  --ip 172.20.0.13 \
  -v /mnt/datastore/lazylibrarian/data:/config \
  -v /mnt/datastore/nzbget/downloads:/downloads \
  -v /mnt/audiobooks:/books \
  --restart always \
  lscr.io/linuxserver/lazylibrarian:latest
```

## ErsatzTV
```bash
docker run -d \
  --name ersatztv \
  -e PUID=0 \
  -e PGID=0 \
  -e TZ=Etc/UTC \
  -p 8409:8409 \
  --net medianet \
  --ip 172.20.0.14 \
  -v /mnt/datastore/ersatztv/data:/config \
  -v /mnt/filler:/filler \
  --restart always \
  ghcr.io/ersatztv/ersatztv
```

## Tunarr
```bash
docker run \
    --name tunarr \
    -v /mnt/datastore/tunarr/data:/config \
    -e PUID=0 \
    -e PGID=0 \
    -e "TZ=America/Chicago" \
    --net medianet \
    --ip 172.20.0.4 \
    -p 8000:8000 \
    --device=/dev/dri/renderD128:/dev/dri/renderD128 \
    --restart always \
    chrisbenincasa/tunarr
```


