# Commands to build media server

## Creat docker network

```bash
docker network create --subnet=172.20.0.0/16 medianet
```

## Nzbget

```bash
docker create --name nzbget --restart always --net medianet --ip 172.20.0.2 -p 6789:6789 -v /mnt/datastore/nzbget/config:/config -v /mnt/datastore/nzbget/downloads:/downloads -e PUID=0 -e PGID=0 -e TZ=US linuxserver/nzbget
```

## Plex

```bash
docker create --name plex --restart always --net medianet --ip 172.20.0.3 -p 32400:32400/tcp -p 3005:3005/tcp -p 8324:8324/tcp -p 32469:32469/tcp -p 1900:1900/udp -p 32410:32410/udp -p 32412:32412/udp -p 32413:32413/udp -p 32414:32414/udp -e TZ=US -e PLEX_CLAIM=claim-ytap233o9vyRbNNgVvRj -e ADVERTISE_IP="http://10.10.1.20:32400/" -e PLEX_UID=0 -e PLEX_GID=0 -h MediaServer -v /mnt/datastore/plex/config:/config -v /mnt/datastore/plex/transcode:/transcode -v /mnt/tv:/tv -v /mnt/movies:/movies plexinc/pms-docker:plexpass
```

## Sonarr

```bash
docker create --name=sonarr --restart always --net medianet --ip 172.20.0.4 -p 8989:8989 -e PUID=0 -e PGID=0 -e TZ=US -v /mnt/datastore/sonarr/config:/config -v /mnt/datastore/nzbget/downloads:/downloads -v /mnt/tv:/tv linuxserver/sonarr:latest

```

## Radarr

```bash
docker create --name=radarr --restart always --net medianet --ip 172.20.0.5 -v /mnt/datastore/radarr/config:/config -v /mnt/datastore/nzbget/downloads:/downloads -v /mnt/movies:/movies -e TZ=US -e PGID=0 -e PUID=0 -p 7878:7878 linuxserver/radarr
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
  -v /mnt/datastore/netboot:/config   `# optional` \
  -v /mnt/datastore/netboot:/assets   `# optional` \
  --restart always \
  ghcr.io/netbootxyz/netbootxyz
```

```bash
/tool fetch url="https://boot.netboot.xyz/ipxe/netboot.xyz.kpxe"
/ip tftp add ip-addresses=10.10.1.0/24 req-filename=netboot.xyz.kpxe real-filename=netboot.xyz.kpxe allow=yes read-only=yes
/ip dhcp-server network set 0 next-server=10.10.1.20 boot-file-name=netboot.xyz.kpxe
```
