# Media Server with VPN

## Setup

### 1. Storage
```bash
sudo mkdir -p /media/storage
sudo chown $USER:$USER /media/storage
# Add to /etc/fstab: UUID=YOUR_UUID /media/storage exfat defaults,uid=1000,gid=1000,umask=0022 0 0
sudo mount -a
mkdir -p /media/storage/media/{downloads,shows,movies}
```

### 2. VPN Config
Create `.env`:
```bash
MULLVAD_PRIVATE_KEY=your_key
MULLVAD_ADDRESSES=10.x.x.x/32
MULLVAD_SERVER_CITY=Madrid
```

### 3. Start
```bash
docker compose up -d
```

## Configuration

### RDTClient - http://raspberrypi.local:6500 & :6501
- Login: `admin`/`admin` (change password)
- Provider: RealDebrid or TorBox + API key
- Download Path: `/data/downloads`
- Mapped Path: `/data/downloads`

### Prowlarr - http://raspberrypi.local:9696
- Add FlareSolverr proxy: `http://localhost:8191`
- Add indexers
- Add apps: Sonarr (`http://localhost:8989`) + Radarr (`http://localhost:7878`)
- Sync App Indexers

### Sonarr - http://raspberrypi.local:8989
- Root Folder: `/data/shows`
- Download Clients (qBittorrent type):
  - TorBox: `localhost:6501`, category `sonarr`, priority 1
  - Real-Debrid: `localhost:6500`, category `sonarr`, priority 2

### Radarr - http://raspberrypi.local:7878
- Root Folder: `/data/movies`
- Download Clients (qBittorrent type):
  - TorBox: `localhost:6501`, category `radarr`, priority 1
  - Real-Debrid: `localhost:6500`, category `radarr`, priority 2

### Bazarr - http://raspberrypi.local:6767
- Add subtitle providers
- Add languages + create profile
- Languages → Default profile for newly added shows: "Default"
- Sonarr: `localhost:8989` + API key
- Radarr: `localhost:7878` + API key

### Jellyfin - http://raspberrypi.local:8096
- Add libraries: `/data/shows` and `/data/movies`
- **Samsung TV**: https://github.com/PatrickSt1991/Samsung-Jellyfin-Installer

## URLs
- Sonarr: http://raspberrypi.local:8989
- Radarr: http://raspberrypi.local:7878
- Prowlarr: http://raspberrypi.local:9696
- Bazarr: http://raspberrypi.local:6767
- Jellyfin: http://raspberrypi.local:8096
- RDTClient (Real-Debrid): http://raspberrypi.local:6500
- RDTClient (TorBox): http://raspberrypi.local:6501
- Homarr: http://raspberrypi.local:5000

## Notes
- All services route through Mullvad VPN and use `localhost` to communicate
- Hardlinks work because everything mounts `/media/storage/media` as `/data`
- Update PUID/PGID (1000) and timezone (Europe/Madrid) in docker-compose.yaml if needed
