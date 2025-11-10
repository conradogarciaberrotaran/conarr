# Homarr Dashboard Setup

Homarr is your unified dashboard for managing all Conarr services in one place.

## Access

http://raspberrypi.local:5000

## Initial Setup

1. Complete the initial setup wizard
2. Create your first dashboard
3. Start adding integrations and widgets

## Adding Integrations

### Sonarr Integration
1. Settings → Integrations → Add Integration
2. Type: **Sonarr**
3. Name: `Sonarr`
4. URL: `http://sonarr:8989`
5. API Key: (from Sonarr → Settings → General → API Key)
6. Test and Save

### Radarr Integration
1. Settings → Integrations → Add Integration
2. Type: **Radarr**
3. Name: `Radarr`
4. URL: `http://radarr:7878`
5. API Key: (from Radarr → Settings → General → API Key)
6. Test and Save

### Prowlarr Integration
1. Settings → Integrations → Add Integration
2. Type: **Prowlarr**
3. Name: `Prowlarr`
4. URL: `http://prowlarr:9696`
5. API Key: (from Prowlarr → Settings → General → API Key)
6. Test and Save

### Jellyfin Integration
1. Settings → Integrations → Add Integration
2. Type: **Jellyfin**
3. Name: `Jellyfin`
4. URL: `http://jellyfin:8096`
5. Username: (your Jellyfin username)
6. Password: (your Jellyfin password)
7. Test and Save

### RDTClient (Real-Debrid) Integration
1. Settings → Integrations → Add Integration
2. Type: **qBittorrent**
3. Name: `Real-Debrid`
4. URL: `http://rdtclient-rd:6500`
5. Username: `admin` (or your RDTClient username)
6. Password: (your RDTClient password)
7. Test and Save

### RDTClient (TorBox) Integration
1. Settings → Integrations → Add Integration
2. Type: **qBittorrent**
3. Name: `TorBox`
4. URL: `http://rdtclient-torbox:6501`
5. Username: `admin` (or your RDTClient username)
6. Password: (your RDTClient password)
7. Test and Save

## Adding Widgets

### Calendar Widget
Shows upcoming TV shows and movies.

1. Add Widget → **Calendar**
2. Select Integrations: Sonarr, Radarr
3. Configure time period (default: next 30 days)
4. Save

### Downloads Widget (Real-Debrid)
Monitor and control downloads.

1. Add Widget → **Download Client**
2. Select Integration: Real-Debrid
3. Features:
   - View active downloads
   - Download/upload speeds
   - Progress bars
   - Pause/Resume individual downloads
   - Delete downloads
4. Save

### Downloads Widget (TorBox)
1. Add Widget → **Download Client**
2. Select Integration: TorBox
3. Save

### Media Server Widget (Jellyfin)
Monitor active streaming sessions.

1. Add Widget → **Media Server**
2. Select Integration: Jellyfin
3. Toggle: "Show only currently playing" (optional)
4. Features:
   - View active sessions
   - See what's being watched
   - Click session for technical details (codec, resolution, transcoding)
5. Save

### Indexer Manager Widget
Monitor Prowlarr indexers status.

1. Add Widget → **Indexer Manager**
2. Select Integration: Prowlarr
3. Save

### Service Links
Quick access to all services.

1. Add Widget → **App**
2. Type: `Sonarr`
3. URL: `http://raspberrypi.local:8989`
4. Icon: Search for "Sonarr"
5. Save
6. Repeat for: Radarr, Prowlarr, Bazarr, Jellyfin

## Recommended Dashboard Layout

```
┌──────────────────────────────────────┐
│  Calendar Widget                     │
│  (Upcoming Shows & Movies)           │
├──────────────────┬───────────────────┤
│  Jellyfin        │  Real-Debrid      │
│  Active Sessions │  Downloads        │
├──────────────────┼───────────────────┤
│  TorBox          │  Prowlarr         │
│  Downloads       │  Indexers         │
├──────────────────┴───────────────────┤
│  Service Links                       │
│  [Apps Row]                          │
└──────────────────────────────────────┘
```

## Widget Features

### Download Widget Controls
- **Pause/Resume**: Click icons on individual downloads
- **Pause All**: Button at bottom of widget
- **Delete**: Trash icon (with confirmation)
- **View Details**: Click download for full info

### Media Server Widget
- **Session Details**: Click any active session
- **Stats**: Codec info, resolution, bitrate, transcoding status
- **Real-time**: Auto-updates as streams start/stop

### Calendar Widget
- **Hover**: See episode/movie details
- **Color Coding**: Different colors for shows vs movies
- **Navigation**: Click arrows to change time period

## Useful Additional Widgets

### Weather Widget
1. Add Widget → **Weather**
2. Enter your location
3. Shows current temp and forecast

### Clock Widget
1. Add Widget → **Clock**
2. Shows current time and date

### Notebook Widget
1. Add Widget → **Notebook**
2. Store notes, passwords, setup info
3. Rich text formatting

## Tips

- Drag and drop to rearrange widgets
- Resize widgets by dragging corners
- Use multiple dashboards for different purposes
- Enable WebSocket for real-time updates
- Test integrations regularly to ensure they're working

## Troubleshooting

### Integration won't connect
- Check service is running: `docker ps`
- Verify API key is correct
- Ensure Homarr can reach service (use internal Docker URLs)

### Widget shows no data
- Verify integration is connected (green indicator)
- Check service logs for errors
- Re-test integration in settings

### Downloads widget empty
- Ensure downloads are active
- Verify RDTClient credentials are correct
- Check that qBittorrent integration type is selected
