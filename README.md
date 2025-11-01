# Media Server Setup Guide

## Prerequisites
- Docker and Docker Compose installed
- Real-Debrid account with API key
- TorBox account with API key
- ~/media/downloads, ~/media/shows, ~/media/movies directories created

## Initial Setup

### 1. Start Services
```bash
docker-compose up -d
```

### 2. Configure RDTClient (Real-Debrid) - http://raspi.local:6500
- Login with default credentials (check first run logs if needed)
- Settings → Provider: Select "RealDebrid"
- Enter your Real-Debrid API key
- Download Path: `/data/downloads`
- Mapped Path: Leave empty
- Save settings

### 3. Configure RDTClient (TorBox) - http://raspi.local:6501
- Login with default credentials
- Settings → Provider: Select "TorBox"
- Enter your TorBox API key
- Download Path: `/data/downloads`
- Mapped Path: Leave empty
- Save settings

### 4. Initial Setup Sonarr and Radarr (Get API Keys)

#### Sonarr - http://raspi.local:8989
- Complete initial setup wizard
- Settings → General → Security → Copy the **API Key** (you'll need this for Prowlarr)

#### Radarr - http://raspi.local:7878
- Complete initial setup wizard
- Settings → General → Security → Copy the **API Key** (you'll need this for Prowlarr)

### 5. Configure Prowlarr - http://raspi.local:9696
- Complete initial setup wizard
- **Configure FlareSolverr proxy first** (for Cloudflare-protected indexers):
  - Settings → Indexers → Add Proxy
  - Name: `flaresolverr`
  - Tags: Type `flaresolverr` and press Enter (creates tag automatically)
  - Host: `http://flaresolverr:8191`
  - Test and Save
- Settings → Indexers → Add Indexer
  - **Free indexers**: 1337x, EZTV, Kickass Torrents, LimeTorrents, Nyaa, TorrentGalaxy, YTS, ThePirateBay
  - For Cloudflare-protected indexers (like 1337x): Add the `flaresolverr` tag when configuring
  - Test each indexer after adding
- Settings → Apps → Add Application
  - Add Sonarr: `http://sonarr:8989` (use API key from step 4)
  - Add Radarr: `http://radarr:7878` (use API key from step 4)
  - Prowlarr will auto-sync indexers to both apps

### 6. Configure Sonarr Download Clients - http://raspi.local:8989
- Settings → Media Management
  - Root Folder: Add `/tv`
  - Enable "Rename Episodes"
  - **Completed Download Handling**: Enable (should be on by default)
  - File Management: Enable "Unmonitor Deleted Episodes" (optional)
- Settings → Download Clients → Add Download Client
  - **RDTClient (Real-Debrid)**:
    - Type: "qBittorrent"
    - Host: `rdtclient-rd`
    - Port: `6500`
    - Category: `sonarr` (optional but recommended)
    - Priority: `1` (1 = highest)
  - **RDTClient (TorBox)**:
    - Type: "qBittorrent"
    - Host: `rdtclient-torbox`
    - Port: `6500`
    - Category: `sonarr`
    - Priority: `2`
- Settings → Download Clients → Remote Path Mappings
  - Not needed (all containers use `/downloads`)

### 7. Configure Radarr Download Clients - http://raspi.local:7878
- Settings → Media Management
  - Root Folder: Add `/movies`
  - Enable "Rename Movies"
  - **Completed Download Handling**: Enable (should be on by default)
  - File Management: Enable "Unmonitor Deleted Movies" (optional)
- Settings → Download Clients → Add Download Client
  - **RDTClient (Real-Debrid)**:
    - Type: "qBittorrent"
    - Host: `rdtclient-rd`
    - Port: `6500`
    - Category: `radarr` (optional but recommended)
    - Priority: `1` (1 = highest)
  - **RDTClient (TorBox)**:
    - Type: "qBittorrent"
    - Host: `rdtclient-torbox`
    - Port: `6500`
    - Category: `radarr`
    - Priority: `2`
- Settings → Download Clients → Remote Path Mappings
  - Not needed (all containers use `/downloads`)

### 8. Configure Bazarr - http://raspi.local:6767
- Settings → Sonarr
  - Enable: Yes
  - Address: `sonarr`, Port: `8989`
  - API Key: (from Sonarr)
- Settings → Radarr
  - Enable: Yes
  - Address: `radarr`, Port: `7878`
  - API Key: (from Radarr)
- Languages → Add your preferred subtitle languages

### 9. Configure Jellyfin - http://raspi.local:8096
- Complete initial setup wizard
- Add Media Libraries:
  - Shows: `/data/tvshows`
  - Movies: `/data/movies`

### 10. Configure Homarr - http://raspi.local:5000
- Add widgets for all services using the URLs and API keys configured above

## Download Workflow

1. **Search**: Add content in Sonarr (TV) or Radarr (Movies)
2. **Find**: Prowlarr searches configured indexers and returns results
3. **Send**: Sonarr/Radarr sends torrent to RDTClient (Real-Debrid or TorBox based on priority)
4. **Cloud Download**: RDTClient adds torrent to debrid service (downloads on their servers, not yours)
5. **Local Download**: RDTClient downloads completed files from debrid service to `~/media/downloads`
6. **Import**: Sonarr/Radarr automatically:
   - Detects completed download
   - Renames file according to naming scheme
   - Moves/hardlinks file to `~/media/shows` or `~/media/movies`
   - Removes from `~/media/downloads`
7. **Subtitles**: Bazarr automatically downloads subtitles for imported content
8. **Watch**: Content appears in Jellyfin

**Note**: Steps 5-7 happen automatically via Completed Download Handling. No manual file moving required.

## Service URLs
- Sonarr: http://raspi.local:8989
- Radarr: http://raspi.local:7878
- Prowlarr: http://raspi.local:9696
- Bazarr: http://raspi.local:6767
- Jellyfin: http://raspi.local:8096
- RDTClient (Real-Debrid): http://raspi.local:6500
- RDTClient (TorBox): http://raspi.local:6501
- qBittorrent: http://raspi.local:8080
- Homarr: http://raspi.local:5000

## Notes
- RDTClient mimics qBittorrent API, so configure it as qBittorrent in Sonarr/Radarr
- Warp SOCKS proxy available at raspi.local:1080 if needed for indexers
- Update PUID/PGID in docker-compose.yaml if not 1000
- Update timezone from Europe/Madrid if needed
