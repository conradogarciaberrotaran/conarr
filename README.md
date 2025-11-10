# Conarr Media Server

Automated media server with VPN on Raspberry Pi using Sonarr, Radarr, Prowlarr, Bazarr, Jellyfin, and RDTClient.

## Prerequisites
- Docker and Docker Compose installed
- Real-Debrid and/or TorBox accounts with API keys
- Mullvad VPN account (get WireGuard config from https://mullvad.net/en/account/wireguard-config)
- External SSD/HDD for media storage (strongly recommended over SD card)

## Setup

### 1. Storage
```bash
# Mount external drive
sudo mkdir -p /media/storage
sudo chown $USER:$USER /media/storage

# Add to /etc/fstab for auto-mount:
# UUID=YOUR_UUID /media/storage exfat defaults,uid=1000,gid=1000,umask=0022 0 0
sudo mount -a

# Create media directories
mkdir -p /media/storage/media/{downloads,shows,movies}
```

### 2. VPN Configuration
Create `.env` file with your Mullvad credentials:
```bash
MULLVAD_PRIVATE_KEY=your_private_key_from_wireguard_config
MULLVAD_ADDRESSES=10.x.x.x/32
MULLVAD_SERVER_CITY=Madrid
```

### 3. Start Services
```bash
docker compose up -d
```

## Configuration

### RDTClient - [Real-Debrid](http://raspberrypi.local:6500) | [TorBox](http://raspberrypi.local:6501)
1. Login with `admin`/`admin` (change password immediately)
2. Settings → Provider: Select "RealDebrid" or "TorBox"
3. Enter your API key
4. Download Path: `/data/downloads`
5. Mapped Path: `/data/downloads`

**Important**: The categories in Sonarr/Radarr must be `sonarr` and `radarr` exactly (not `tv-sonarr` or `movies-radarr`).

### Prowlarr - http://raspberrypi.local:9696
1. **Add FlareSolverr proxy** (for Cloudflare-protected indexers):
   - Settings → Indexers → Add Proxy
   - Host: `http://localhost:8191`
   - Tag: `flaresolverr`
2. **Add indexers**: 1337x, EZTV, TorrentGalaxy, YTS, etc.
   - For Cloudflare-protected sites, add the `flaresolverr` tag
3. **Add apps**:
   - Sonarr: `http://localhost:8989` + API key
   - Radarr: `http://localhost:7878` + API key
4. Click **"Sync App Indexers"**

### Sonarr - http://raspberrypi.local:8989
1. Settings → Media Management → Root Folder: `/data/shows`
2. Settings → Download Clients → Add both (Type: qBittorrent):
   - **TorBox**: Host `localhost`, Port `6501`, Category `sonarr`, Priority `1`
   - **Real-Debrid**: Host `localhost`, Port `6500`, Category `sonarr`, Priority `2`

### Radarr - http://raspberrypi.local:7878
1. Settings → Media Management → Root Folder: `/data/movies`
2. Settings → Download Clients → Add both (Type: qBittorrent):
   - **TorBox**: Host `localhost`, Port `6501`, Category `radarr`, Priority `1`
   - **Real-Debrid**: Host `localhost`, Port `6500`, Category `radarr`, Priority `2`

### Bazarr - http://raspberrypi.local:6767
1. Settings → Providers: Add subtitle providers (SubDivx, OpenSubtitles, Subscene, etc.)
2. Settings → Languages: Add your preferred languages
3. Settings → Languages: Create a language profile
4. Settings → Languages → **Default profile for newly added shows**: Select "Default"
5. Settings → Sonarr: Host `localhost`, Port `8989`, API key
6. Settings → Radarr: Host `localhost`, Port `7878`, API key

### Jellyfin - http://raspberrypi.local:8096
1. Complete initial setup wizard
2. Add libraries:
   - TV Shows: `/data/shows`
   - Movies: `/data/movies`
3. **Samsung TV app**: Use https://github.com/PatrickSt1991/Samsung-Jellyfin-Installer for one-click installation

## How It Works

1. **Search**: Add a show in Sonarr or movie in Radarr
2. **Find**: Prowlarr searches all configured indexers
3. **Send**: Sonarr/Radarr sends the torrent to RDTClient (TorBox first, then Real-Debrid)
4. **Cloud Download**: RDTClient adds torrent to your debrid service (they download it on their servers)
5. **Local Download**: RDTClient downloads the completed file to `/data/downloads/sonarr` or `/data/downloads/radarr`
6. **Import**: Sonarr/Radarr automatically detects the completed download and creates a **hardlink** to `/data/shows` or `/data/movies`
7. **Subtitles**: Bazarr automatically downloads subtitles for the imported content
8. **Watch**: Content appears in Jellyfin instantly

## Service URLs
- Sonarr: http://raspberrypi.local:8989
- Radarr: http://raspberrypi.local:7878
- Prowlarr: http://raspberrypi.local:9696
- Bazarr: http://raspberrypi.local:6767
- Jellyfin: http://raspberrypi.local:8096
- RDTClient (Real-Debrid): http://raspberrypi.local:6500
- RDTClient (TorBox): http://raspberrypi.local:6501
- Homarr: http://raspberrypi.local:5000
