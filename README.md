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

## Initial Setup

### 1. Start Services
```bash
docker-compose up -d
```

### 2. Configure RDTClient (Real-Debrid) - http://raspi.local:6500
- Login with default credentials: Username: `admin`, Password: `admin`
- **Change password immediately**: Settings → Authentication → Change password (note this down for Sonarr/Radarr)
- Settings → Provider: Select "RealDebrid"
- Enter your Real-Debrid API key
- Download Path: `/data/downloads`
- Mapped Path: Leave empty
- Save settings

### 3. Configure RDTClient (TorBox) - http://raspi.local:6501
- Login with default credentials: Username: `admin`, Password: `admin`
- **Change password immediately**: Settings → Authentication → Change password (note this down for Sonarr/Radarr)
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

### 6. Configure Sonarr Download Clients - http://raspi.local:8989
- **Verify Indexers**: Settings → Indexers
  - Check that indexers from Prowlarr are listed
  - Verify **Enable RSS** and **Enable Automatic Search** are checked
  - If not synced, go back to Prowlarr and click **Sync App Indexers**
- Settings → Media Management
  - Root Folder: Add `/tv`
  - Enable "Rename Episodes"
  - File Management: Enable "Unmonitor Deleted Episodes" (optional)
- Settings → Download Clients
  - **Completed Download Handling**: Enable (should be on by default at top of page)
  - Add Download Client
  - **RDTClient (TorBox)**:
    - Type: "qBittorrent"
    - Host: `rdtclient-torbox`
    - Port: `6500`
    - Username: `admin` (or the username you set)
    - Password: (the password you set in step 3)
    - Category: `sonarr`
    - Priority: `1` (1 = highest)
  - **RDTClient (Real-Debrid)**:
    - Type: "qBittorrent"
    - Host: `rdtclient-rd`
    - Port: `6500`
    - Username: `admin` (or the username you set)
    - Password: (the password you set in step 2)
    - Category: `sonarr`
    - Priority: `2`

### 7. Configure Radarr Download Clients - http://raspi.local:7878
- **Verify Indexers**: Settings → Indexers
  - Check that indexers from Prowlarr are listed
  - Verify **Enable RSS** and **Enable Automatic Search** are checked
  - If not synced, go back to Prowlarr and click **Sync App Indexers**
- Settings → Media Management
  - Root Folder: Add `/movies`
  - Enable "Rename Movies"
  - File Management: Enable "Unmonitor Deleted Movies" (optional)
- Settings → Download Clients
  - **Completed Download Handling**: Enable (should be on by default at top of page)
  - Add Download Client
  - **RDTClient (TorBox)**:
    - Type: "qBittorrent"
    - Host: `rdtclient-torbox`
    - Port: `6500`
    - Username: `admin` (or the username you set)
    - Password: (the password you set in step 3)
    - Category: `radarr`
    - Priority: `1` (1 = highest)
  - **RDTClient (Real-Debrid)**:
    - Type: "qBittorrent"
    - Host: `rdtclient-rd`
    - Port: `6500`
    - Username: `admin` (or the username you set)
    - Password: (the password you set in step 2)
    - Category: `radarr`
    - Priority: `2`

### 8. Configure Bazarr - http://raspi.local:6767
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
3. **Send**: Sonarr/Radarr sends torrent to RDTClient (TorBox first, then Real-Debrid based on priority)
4. **Cloud Download**: RDTClient adds torrent to debrid service (downloads on their servers, not yours)
5. **Local Download**: RDTClient downloads completed files from debrid service to `/data/downloads` (host: `/media/storage/media/downloads`)
6. **Import**: Sonarr/Radarr automatically:
   - Detects completed download in `/data/downloads`
   - Renames file according to naming scheme
   - Moves/hardlinks file to `/tv` or `/movies` (host: `/media/storage/media/shows` or `/media/storage/media/movies`)
   - Removes from `/data/downloads`
7. **Subtitles**: Bazarr automatically downloads subtitles for imported content
8. **Watch**: Content appears in Jellyfin

**Note**: Steps 5-7 happen automatically via Completed Download Handling. No manual file moving required.
**Note**: All containers use `/data/downloads` internally for consistency. External drive mounted at `/media/storage/` for fast, reliable storage.

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
