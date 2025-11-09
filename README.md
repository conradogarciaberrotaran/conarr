# Media Server Setup Guide

## Prerequisites
- Docker and Docker Compose installed
- Real-Debrid account with API key
- TorBox account with API key
- External SSD/HDD for media storage (strongly recommended)

## Storage Setup

### Option 1: External Drive (Recommended)

**Why?** Running on SD card causes overheating, slow performance, and SD card wear.

1. **Connect your external drive** (SSD recommended, like Crucial X9)

2. **Find the drive's UUID:**
   ```bash
   sudo blkid
   ```
   Look for your drive (e.g., `LABEL="Crucial X9"`) and note the `UUID`

3. **Create mount point:**
   ```bash
   sudo mkdir -p /media/storage
   sudo chown $USER:$USER /media/storage
   ```

4. **Get your UID and GID:**
   ```bash
   id -u  # Should be 1000
   id -g  # Should be 1000
   ```

5. **Add to `/etc/fstab` for auto-mount on boot:**
   ```bash
   echo "UUID=YOUR_UUID /media/storage exfat defaults,uid=1000,gid=1000,umask=0022 0 0" | sudo tee -a /etc/fstab
   ```
   Replace `YOUR_UUID` with the UUID from step 2. Change `exfat` to `ext4` or `ntfs-3g` if needed.

6. **Mount the drive:**
   ```bash
   sudo mount -a
   df -h | grep storage  # Verify it's mounted
   ```

7. **Create media directories:**
   ```bash
   mkdir -p /media/storage/media/{downloads,shows,movies}
   ```

### Option 2: SD Card (Not Recommended)

Only use if you don't have an external drive. Performance will be poor and SD card will wear out quickly.

```bash
mkdir -p ~/media/{downloads,shows,movies}
sudo chown -R $USER:$USER ~/media
```

Then update all paths in `docker-compose.yaml` from `/media/storage/media/` to `~/media/`

## Important: File Structure for Hardlinks

This setup follows [TRaSH Guides](https://trash-guides.info/File-and-Folder-Structure/How-to-set-up/Docker/) recommendations for optimal performance:

**Key Principle:** All containers mount `/media/storage/media` as `/data` internally. This enables:
- ✅ **Instant imports** via hardlinks (no file copying)
- ✅ **Atomic moves** (instant file relocation)
- ✅ **Disk space savings** (same file in multiple locations uses space only once)

**Structure:**
```
/media/storage/media/
  ├── downloads/     (Download clients write here)
  ├── shows/         (Sonarr imports here)
  └── movies/        (Radarr imports here)
```

**Why this works:** When Sonarr/Radarr import from `/data/downloads` to `/data/shows` or `/data/movies`, Docker sees them on the same `/data` mount, allowing instant hardlinks instead of slow copying.

**Memory Limits:** Each container has memory limits to prevent any single container from overwhelming the Pi's 4GB RAM and causing system freezes.

## Initial Setup

### 1. Start Services
```bash
docker-compose up -d
```

### 2. Configure RDTClient (Real-Debrid) - http://raspberrypi.local:6500
- Login with default credentials: Username: `admin`, Password: `admin`
- **Change password immediately**: Settings → Authentication → Change password (note this down for Sonarr/Radarr)
- Settings → Provider: Select "RealDebrid"
- Enter your Real-Debrid API key
- Download Path: `/data/downloads`
- Mapped Path: Leave empty
- Save settings

### 3. Configure RDTClient (TorBox) - http://raspberrypi.local:6501
- Login with default credentials: Username: `admin`, Password: `admin`
- **Change password immediately**: Settings → Authentication → Change password (note this down for Sonarr/Radarr)
- Settings → Provider: Select "TorBox"
- Enter your TorBox API key
- Download Path: `/data/downloads`
- Mapped Path: Leave empty
- Save settings

### 4. Initial Setup Sonarr and Radarr (Get API Keys)

#### Sonarr - http://raspberrypi.local:8989
- Complete initial setup wizard
- Settings → General → Security → Copy the **API Key** (you'll need this for Prowlarr)

#### Radarr - http://raspberrypi.local:7878
- Complete initial setup wizard
- Settings → General → Security → Copy the **API Key** (you'll need this for Prowlarr)

### 5. Configure Prowlarr - http://raspberrypi.local:9696
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
  - **Add Sonarr**:
    - Prowlarr Server: `http://prowlarr:9696`
    - Radarr Server: `http://sonarr:8989`
    - API Key: (use API key from step 4)
    - Sync Level: **Full Sync**
    - Test and Save
  - **Add Radarr**:
    - Prowlarr Server: `http://prowlarr:9696`
    - Radarr Server: `http://radarr:7878`
    - API Key: (use API key from step 4)
    - Sync Level: **Full Sync**
    - Test and Save
  - Click **Sync App Indexers** button to sync all indexers to both apps
  - Prowlarr will auto-sync indexers (with RSS enabled) to both apps

### 6. Configure Sonarr Download Clients - http://raspberrypi.local:8989
- **Verify Indexers**: Settings → Indexers
  - Check that indexers from Prowlarr are listed
  - Verify **Enable RSS** and **Enable Automatic Search** are checked
  - If not synced, go back to Prowlarr and click **Sync App Indexers**
- Settings → Media Management
  - Root Folder: Add `/data/shows` (NOT `/tv` - important for hardlinks!)
  - Enable "Rename Episodes"
  - File Management: Enable "Unmonitor Deleted Episodes" (optional)
- Settings → Download Clients
  - **Completed Download Handling**: Enable (should be on by default at top of page)
  - Add Download Client
  - **RDTClient (TorBox)**:
    - Type: "qBittorrent"
    - Host: `localhost`
    - Port: `6501`
    - Username: `admin` (or the username you set)
    - Password: (the password you set in step 3)
    - Category: `sonarr`
    - Priority: `1` (1 = highest)
  - **RDTClient (Real-Debrid)**:
    - Type: "qBittorrent"
    - Host: `localhost`
    - Port: `6500`
    - Username: `admin` (or the username you set)
    - Password: (the password you set in step 2)
    - Category: `sonarr`
    - Priority: `2`

### 7. Configure Radarr Download Clients - http://raspberrypi.local:7878
- **Verify Indexers**: Settings → Indexers
  - Check that indexers from Prowlarr are listed
  - Verify **Enable RSS** and **Enable Automatic Search** are checked
  - If not synced, go back to Prowlarr and click **Sync App Indexers**
- Settings → Media Management
  - Root Folder: Add `/data/movies` (NOT `/movies` - important for hardlinks!)
  - Enable "Rename Movies"
  - File Management: Enable "Unmonitor Deleted Movies" (optional)
- Settings → Download Clients
  - **Completed Download Handling**: Enable (should be on by default at top of page)
  - Add Download Client
  - **RDTClient (TorBox)**:
    - Type: "qBittorrent"
    - Host: `localhost`
    - Port: `6501`
    - Username: `admin` (or the username you set)
    - Password: (the password you set in step 3)
    - Category: `radarr`
    - Priority: `1` (1 = highest)
  - **RDTClient (Real-Debrid)**:
    - Type: "qBittorrent"
    - Host: `localhost`
    - Port: `6500`
    - Username: `admin` (or the username you set)
    - Password: (the password you set in step 2)
    - Category: `radarr`
    - Priority: `2`

### 8. Configure Bazarr - http://raspberrypi.local:6767
- **Settings → Providers**:
  - Add subtitle providers (click the + button for each):
    - SubDivx (for Spanish)
    - Subtitulamos.tv (for Spanish)
    - OpenSubtitles.com (requires free account)
    - Subscene
    - Add any other providers you want
  - Configure each provider with credentials if needed
  - Save
- **Settings → Languages**:
  - Click + to add languages you want subtitles for (e.g., English, Spanish, etc.)
  - Save
- **Settings → Languages → Language Profiles**:
  - Click + to create a new profile
  - Name: `Default` (or your preferred name)
  - Add languages in priority order (drag to reorder)
  - Save
- **Settings → Sonarr**:
  - Enable: Yes
  - Address: `sonarr`, Port: `8989`
  - API Key: (from Sonarr)
  - Default Language Profile: Select the profile you created above
  - Save and Test
- **Settings → Radarr**:
  - Enable: Yes
  - Address: `radarr`, Port: `7878`
  - API Key: (from Radarr)
  - Default Language Profile: Select the profile you created above
  - Save and Test

### 9. Configure Jellyfin - http://raspberrypi.local:8096
- Complete initial setup wizard
- Add Media Libraries:
  - Shows: `/data/shows`
  - Movies: `/data/movies`

### 10. Configure Homarr - http://raspberrypi.local:5000
- Add widgets for all services using the URLs and API keys configured above

## Download Workflow

1. **Search**: Add content in Sonarr (TV) or Radarr (Movies)
2. **Find**: Prowlarr searches configured indexers and returns results
3. **Send**: Sonarr/Radarr sends torrent to RDTClient (TorBox first, then Real-Debrid based on priority)
4. **Cloud Download**: RDTClient adds torrent to debrid service (downloads on their servers, not yours)
5. **Local Download**: RDTClient downloads completed files from debrid service to `/data/downloads` (host: `/media/storage/media/downloads`)
6. **Import**: Sonarr/Radarr automatically:
   - Detects completed download in `/data/downloads`
   - **Creates hardlink** (instant, no copying!) to `/data/shows` or `/data/movies`
   - Renames file according to naming scheme
   - Removes original from `/data/downloads`
7. **Subtitles**: Bazarr automatically downloads subtitles for imported content
8. **Watch**: Content appears in Jellyfin

**Note**: Steps 5-7 happen automatically via Completed Download Handling. No manual file moving required.
**Note**: Hardlinks are instant and use no extra disk space - the same file appears in both locations!
**Note**: All containers mount `/media/storage/media` as `/data` for atomic moves and hardlinks (following [TRaSH Guides](https://trash-guides.info/)).

## Service URLs
- Sonarr: http://raspberrypi.local:8989
- Radarr: http://raspberrypi.local:7878
- Prowlarr: http://raspberrypi.local:9696
- Bazarr: http://raspberrypi.local:6767
- Jellyfin: http://raspberrypi.local:8096
- RDTClient (Real-Debrid): http://raspberrypi.local:6500
- RDTClient (TorBox): http://raspberrypi.local:6501
- qBittorrent: http://raspberrypi.local:8080
- Homarr: http://raspberrypi.local:5000

## Notes
- RDTClient mimics qBittorrent API, so configure it as qBittorrent in Sonarr/Radarr
- Update PUID/PGID in docker-compose.yaml if not 1000
- Update timezone from Europe/Madrid if needed
- All containers have memory limits to prevent system freezes on Pi 4 with 4GB RAM

## Memory Limits (Raspberry Pi 4GB)

Each container has memory limits to prevent overwhelming the system:
- **Sonarr/Radarr**: 512MB max (256MB reserved)
- **Jellyfin**: 768MB max (384MB reserved)
- **RDTClient (both)**: 384MB max (192MB reserved)
- **Prowlarr/Bazarr/Homarr**: 256MB max (128MB reserved)

**Total**: ~3.3GB max, leaving headroom for system operations.

## Performance Tips (Raspberry Pi 4GB)

### Normal Behavior:
- System may slow down during active downloads/imports (this is expected)
- Let operations complete before starting new ones
- One show/movie at a time recommended

### If System Freezes:
1. **Check memory usage**: `ssh cono@raspberrypi.local "free -h"`
2. **Check container stats**: `ssh cono@raspberrypi.local "docker stats --no-stream"`
3. **Restart problematic container**: `ssh cono@raspberrypi.local "docker restart <container>"`

### Optimization:
- **Limit RDTClient downloads**: Settings → Max Downloads (1-2 concurrent)
- **Monitor temperature**: `vcgencmd measure_temp` (add cooling if >70°C)
- **Consider upgrading**: Raspberry Pi 5 8GB (~$95) provides 2-3x better performance

### Hardware Recommendations:
- ✅ **Current setup works** but is at the limit with 4GB RAM
- ✅ **Raspberry Pi 5 8GB** - Best upgrade for this workload
- ✅ **Mini PC with N100 CPU** (~$150) - Even more powerful alternative
