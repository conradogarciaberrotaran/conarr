# Media Server Setup with VPN

## Prerequisites
- Docker and Docker Compose installed
- Real-Debrid and/or TorBox accounts with API keys
- Mullvad VPN account
- External SSD/HDD for media storage (recommended)

## Quick Setup

### 1. Storage Setup
```bash
# Mount external drive
sudo mkdir -p /media/storage
sudo chown $USER:$USER /media/storage
# Add to /etc/fstab: UUID=YOUR_UUID /media/storage exfat defaults,uid=1000,gid=1000,umask=0022 0 0
sudo mount -a

# Create directories
mkdir -p /media/storage/media/{downloads,shows,movies}
```

### 2. VPN Configuration
1. Download Mullvad WireGuard config from https://mullvad.net/en/account/wireguard-config
2. Create `.env` file:
```bash
MULLVAD_PRIVATE_KEY=your_private_key_from_config
MULLVAD_ADDRESSES=10.x.x.x/32
MULLVAD_SERVER_CITY=Madrid
```

### 3. Start Services
```bash
docker compose up -d
```

## Configuration

### RDTClient (Real-Debrid) - http://raspberrypi.local:6500
- Login: `admin` / `admin` (change password immediately)
- Settings → Provider: RealDebrid
- API Key: Your Real-Debrid API key
- Download Path: `/data/downloads`
- Mapped Path: `/data/downloads`

### RDTClient (TorBox) - http://raspberrypi.local:6501
- Login: `admin` / `admin` (change password immediately)
- Settings → Provider: TorBox
- API Key: Your TorBox API key
- Download Path: `/data/downloads`
- Mapped Path: `/data/downloads`

### Prowlarr - http://raspberrypi.local:9696
1. Add FlareSolverr proxy:
   - Settings → Indexers → Add Proxy
   - Host: `http://localhost:8191`
2. Add indexers (use FlareSolverr tag for Cloudflare-protected sites)
3. Add apps:
   - Sonarr: `http://localhost:8989` (get API key from Sonarr)
   - Radarr: `http://localhost:7878` (get API key from Radarr)
4. Click "Sync App Indexers"

### Sonarr - http://raspberrypi.local:8989
- Settings → Media Management → Root Folder: `/data/shows`
- Settings → Download Clients → Add both:
  - Type: qBittorrent
  - TorBox: `localhost:6501`, Category: `sonarr`, Priority: 1
  - Real-Debrid: `localhost:6500`, Category: `sonarr`, Priority: 2

### Radarr - http://raspberrypi.local:7878
- Settings → Media Management → Root Folder: `/data/movies`
- Settings → Download Clients → Add both:
  - Type: qBittorrent
  - TorBox: `localhost:6501`, Category: `radarr`, Priority: 1
  - Real-Debrid: `localhost:6500`, Category: `radarr`, Priority: 2

### Bazarr - http://raspberrypi.local:6767
- Settings → Providers: Add subtitle providers
- Settings → Languages: Add languages and create profile
- Settings → Sonarr: `localhost:8989` + API key
- Settings → Radarr: `localhost:7878` + API key

### Jellyfin - http://raspberrypi.local:8096
- Add libraries: `/data/shows` and `/data/movies`
- **Samsung TV app**: Use https://github.com/PatrickSt1991/Samsung-Jellyfin-Installer for one-click installation

## How It Works

1. **VPN**: All services route through Mullvad via Gluetun
2. **Download**: Sonarr/Radarr → Prowlarr → RDTClient → Debrid service
3. **Import**: Files download to `/data/downloads/sonarr` or `/data/downloads/radarr`
4. **Hardlink**: Sonarr/Radarr instantly hardlink to `/data/shows` or `/data/movies` (no copying)
5. **Watch**: Content appears in Jellyfin

## Service URLs
- Sonarr: http://raspberrypi.local:8989
- Radarr: http://raspberrypi.local:7878
- Prowlarr: http://raspberrypi.local:9696
- Bazarr: http://raspberrypi.local:6767
- Jellyfin: http://raspberrypi.local:8096
- RDTClient (Real-Debrid): http://raspberrypi.local:6500
- RDTClient (TorBox): http://raspberrypi.local:6501
- Homarr: http://raspberrypi.local:5000

## Notes
- All services share Gluetun's VPN network and communicate via `localhost`
- Hardlinks work because all services mount `/media/storage/media` as `/data`
- RDTClient configured as qBittorrent in Sonarr/Radarr
- Update PUID/PGID (1000) and timezone (Europe/Madrid) in docker-compose.yaml if needed
