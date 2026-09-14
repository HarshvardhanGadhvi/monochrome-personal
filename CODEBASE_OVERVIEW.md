# Monochrome Codebase Overview

## What is Monochrome?

**Monochrome** is an open-source, privacy-respecting, ad-free web-based music player built as a Progressive Web App (PWA). It streams high-quality music from Tidal via a custom API wrapper and provides a beautiful, minimalist interface for music playback and discovery.

Key characteristics:
- **High-quality audio**: Supports lossless, Hi-Res, and Dolby Atmos streams
- **Privacy-first**: No ads, analytics, or tracking (except optional Plausible analytics)
- **Progressive Web App**: Works offline, installable on devices, uses service workers for caching
- **Multi-platform**: Runs in browsers, can be installed on mobile/desktop
- **Fully featured**: Playlists, podcasts, lyrics, visualizers, keyboard shortcuts, scrobbling, accounts

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    Browser / PWA Frontend                   │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  User Interface (Preact Components)                  │  │
│  │  - AppHeader, AppNavigation, NowPlayingBar          │  │
│  │  - Pages: Home, Search, Album, Playlist, etc        │  │
│  └──────────────────────────────────────────────────────┘  │
│           ▲                                 ▲                │
│           │                                 │                │
│  ┌────────┴──────────────┐      ┌──────────┴──────────┐   │
│  │   Player Engine       │      │   Music API         │   │
│  │  - Playback control   │      │  - Search           │   │
│  │  - Queue management   │      │  - Fetch metadata   │   │
│  │  - Audio effects      │      │  - Get stream URLs  │   │
│  │  - Crossfade          │      │  - Lyrics support   │   │
│  │  - Media Session      │      │  - Apple Music      │   │
│  │  - Visualizers        │      │  - Podcasts API     │   │
│  └───────────────────────┘      └─────────────────────┘   │
│           ▲                                 ▲                │
│           └─────────────────────────────────┘                │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │   Local Storage Layer                                │  │
│  │  - IndexedDB: Favorites, history, queue, settings  │  │
│  │  - LocalStorage: Cache, preferences, auth tokens   │  │
│  │  - Service Workers: Asset & media caching          │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
           ▲                                 ▲
           │                                 │
           └─────────────────────────────────┘
                   Network Layer
                        ▲
        ┌───────────────┼───────────────┐
        │               │               │
   ┌─────────┐    ┌──────────┐   ┌──────────────┐
   │   Tidal │    │ Apple    │   │   Podcasts   │
   │   API   │    │ Music    │   │   API        │
   │ (via    │    │ Search   │   │              │
   │ proxy)  │    │ API      │   │              │
   └─────────┘    └──────────┘   └──────────────┘
```

---

## Directory Structure

```
monochrome-personal/
├── src/                          # Preact UI components (TypeScript/TSX)
│   ├── app/                      # Main app chrome components
│   │   ├── AppHeader.tsx         # Top navigation bar
│   │   ├── AppNavigation.tsx     # Sidebar navigation
│   │   ├── NowPlayingBar.tsx     # Music player bar at bottom
│   │   └── UtilitySurfaces.tsx   # Modals, context menus, panels
│   ├── routes/                   # Page-level components
│   │   ├── HomePage.tsx          # Home/discover page
│   │   ├── SearchPage.tsx        # Search results
│   │   ├── CatalogDetailPages.tsx # Album/Playlist/Mix/Folder pages
│   │   ├── PodcastsPage.tsx      # Podcast browsing
│   │   └── RecentPage.tsx        # Recently played & unreleased
│   ├── ui/                       # UI utilities
│   │   ├── Icon.tsx              # Icon component wrapper
│   │   └── icons.ts              # SVG icon imports
│   ├── styles/                   # CSS stylesheets
│   │   └── modern.css            # Main styling
│   └── main.tsx                  # Preact app entry point
│
├── js/                           # Core JavaScript logic
│   ├── api.js                    # Main music API wrapper (Tidal integration)
│   ├── music-api.js              # High-level music service (1 of 644 lines)
│   ├── player.js                 # Audio playback engine
│   ├── app.js                    # App initialization & setup
│   ├── storage.js                # Settings & preferences management
│   ├── db.js                     # IndexedDB database layer
│   ├── cache.js                  # API response caching
│   ├── ui.js                     # HTML rendering utilities
│   ├── router.js                 # Page routing & URL management
│   ├── events.js                 # Event handlers & interactions
│   ├── ui-interactions.js        # UI event listeners
│   ├── audio-context.js          # Web Audio API setup
│   ├── lyrics.js                 # Lyrics display & syncing
│   ├── visualizers/              # Butterchurn visualizer integration
│   ├── accounts/                 # User account management
│   │   ├── auth.js               # Authentication logic
│   │   ├── authApi.js            # Appwrite auth API
│   │   ├── pocketbase.js         # PocketBase sync & data storage
│   │   └── config.js             # Auth configuration
│   ├── download-service.js       # Track downloading
│   ├── ffmpeg.js                 # FFmpeg for audio processing
│   ├── hls-downloader.js         # HLS stream downloading
│   ├── dash-downloader.ts        # DASH stream downloading
│   ├── utils.js                  # Common utilities
│   ├── proxy-utils.js            # Proxy/URL wrapping
│   ├── multi-scrobbler.js        # Last.fm & ListenBrainz scrobbling
│   ├── HiFi.ts                   # Tidal HiFi API wrapper
│   ├── podcasts-api.js           # Podcasts API integration
│   ├── apple-music-api.js        # Apple Music search integration
│   ├── community-playlists.js    # Community playlist support
│   └── ... (many more utility modules)
│
├── functions/                    # Cloudflare Workers routes
│   ├── api/                      # API route handlers
│   ├── album/[id].js             # Album detail pages
│   ├── artist/[id].js            # Artist detail pages
│   ├── playlist/[id].js          # Playlist detail pages
│   ├── track/[id].js             # Track pages
│   ├── unreleased/               # Unreleased music routes
│   └── ... (other route handlers)
│
├── public/                       # Static assets
│   ├── index.html                # Main HTML (very large, contains inline scripts)
│   ├── manifest.json             # PWA manifest
│   ├── assets/                   # Images, icons, splash screens
│   ├── lib/                      # External JavaScript libraries
│   └── fonts/                    # Custom fonts
│
├── images/                       # SVG assets (inline)
├── database/                     # Database schemas
│   └── pb_schema.json            # PocketBase collection schema
├── docker/                       # Docker setup
├── .github/workflows/            # CI/CD pipelines
└── vite.config.ts               # Build configuration (Vite)
```

---

## Key Components Explained

### 1. **Music API Layer** (`js/music-api.js`, `js/api.js`)

The heart of the application - handles all communication with music streaming services:

```javascript
// MusicAPI is a singleton that provides unified interface to:
const api = MusicAPI.instance;

// Search across all providers
api.search('query')                    // Uses Apple Music, falls back to Tidal
api.searchTracks('query')
api.searchAlbums('query')
api.searchArtists('query')

// Get track details
api.getTrack('track-id')              // Fetch full track metadata

// Stream URLs - the critical part
api.getStreamUrl('track-id', 'HIGH')  // Get audio stream URL from Tidal
api.getPlaybackInfo('track-id')       // Get DRM info & stream manifest

// Podcasts
api.searchPodcasts('query')
api.getPodcastEpisodes('podcast-id')

// Player interactions
api.canPlayLegacyStream(track)        // Check if track can be played
```

**How Tidal Integration Works:**
1. Monochrome uses a custom proxy wrapper around Tidal's API
2. The app authenticates with Tidal servers (credentials handled securely)
3. When a track is played, it requests the stream URL from Tidal
4. Returns either an HLS/DASH manifest (for protected streams) or direct MP3 URL
5. Stream is proxied through a custom proxy service (to handle CORS, authentication)

### 2. **Player Engine** (`js/player.js`)

Manages all audio playback:

```javascript
const player = Player.instance;

// Playback control
player.play()              // Start/resume playback
player.pause()             // Pause playback
player.seekTo(seconds)     // Jump to specific time

// Queue management
player.queue               // Array of tracks to play
player.setQueue(tracks)    // Set new queue
player.next()              // Skip to next track
player.previous()          // Go back to previous

// Audio effects (Web Audio API)
player.applyAudioEffects() // Apply equalizer, replay gain, etc.
player.setVolume(0.8)      // Set volume (0-1)

// Quality control
player.setQuality('LOSSLESS')  // Switch audio quality

// Media session (lock screen controls)
player.setupMediaSession() // Enable system media controls
```

**Key Features:**
- Supports multiple audio qualities (Dolby Atmos, Hi-Res Lossless, Lossless, High, Low)
- Crossfade between tracks
- Sleep timer
- Replay gain normalization
- Radio mode (autoplay similar tracks)
- Queue shuffle and repeat modes

### 3. **User Interface Layer** (`src/app/`, `src/routes/`)

Built with **Preact** (lightweight React alternative) + TypeScript:

```
AppHeader                    ← Top bar (search, user menu)
  └── AppNavigation          ← Sidebar (nav menu)
HomePage                     ← Home/discovery page
SearchPage                   ← Search results
CatalogDetailPages           ← Album/Playlist/Folder/Mix detail
PodcastDetailPage            ← Podcast browser
NowPlayingBar                ← Player bar with now-playing controls
UtilitySurfaces              ← Modals & context menus
```

Entry point in `src/main.tsx` renders these components into `index.html`.

### 4. **Local Storage** (`js/db.js`, `js/storage.js`)

**IndexedDB** (structured client-side database):
- `favorites_tracks`, `favorites_albums`, `favorites_artists` - User's favorites
- `history_tracks` - Recently played
- `playlists` - Saved playlists (with track list)
- `queue_state` - Current queue position
- `community_playlists` - Downloaded playlists

**LocalStorage** (key-value):
- `monochrome-api-instances-v9` - List of API proxy servers
- `monochrome-settings-*` - User preferences (quality, theme, etc.)
- `monochrome-cache-*` - HTTP cache of API responses
- `volume` - Current volume level
- Auth tokens for accounts

**Service Workers** (Workbox):
Caches assets & media with strategies:
- Scripts: NetworkFirst (or CacheFirst in prod)
- Styles/Fonts: CacheFirst
- Images: CacheFirst (60-day expiry)
- Audio/Video: CacheFirst with range request support (60-day expiry)

### 5. **Settings & Configuration** (`js/storage.js`)

The app has extensive preferences:

```javascript
// Audio quality preference
downloadQualitySettings.setQuality('LOSSLESS')

// Theme management
themeManager.setTheme('dark')

// Playback settings
autoplaySettings.enable()          // Auto-play similar tracks
radioSettings.enable()             // Enable radio mode
crossfadeSettings.setDuration(5)   // 5 second crossfade

// Display preferences
nowPlayingSettings.showCover()     // Show album art
coverArtSizeSettings.setSize('medium')

// API instances (for streaming)
apiSettings.loadInstancesFromGitHub()  // Load list of proxy servers
```

### 6. **Account System** (`js/accounts/`)

Optional cloud sync using **Appwrite** and **PocketBase**:

```javascript
const { authManager, syncManager } = await import('./accounts/auth.js');

// Sign in with Google, Discord, GitHub
authManager.signIn('google')

// Cloud sync
syncManager.syncLibrary()          // Upload favorites/history
syncManager.downloadLibrary()      // Download from other devices
```

Features:
- Cross-device sync of favorites, playlists, history
- Public profiles
- Listening parties (sync playback with friends)
- Last.fm scrobbling

### 7. **Lyrics** (`js/lyrics.js`)

Fetches lyrics from Genius API:
- Synced/karaoke mode with scrolling
- Embedded in now-playing panel
- Fallback to search if track not found

### 8. **Download Service** (`js/download-service.js`, `js/ffmpeg.js`)

Download tracks for offline listening:
- Uses FFmpeg (WebAssembly) for audio conversion
- Converts HLS/DASH streams to MP3/FLAC/etc
- Embeds metadata (ID3 tags)
- Progress tracking

### 9. **Visualizers** (`js/visualizers/`)

Uses **Butterchurn** (Milkdrop visualizer for JavaScript):
- Animated visualizations synced to audio
- Preset cycling
- Auto-cycle on play

### 10. **Router** (`js/router.js`)

Handles URL-based navigation (SPA routing):
```javascript
// Routes like:
/ 

/search?q=foo
/album/123
/artist/456
/playlist/abc
/podcasts
/recent
/user/@username
```

---

## Data Flow Example: Playing a Song

```
1. User clicks play on track
   └─> handleTrackAction('play', track)

2. Player receives track
   └─> player.loadTrack(track)

3. Get playback info from API
   └─> api.getPlaybackInfo(trackId)
   └─> Tidal returns: stream URL + rights info

4. Audio element loads stream URL
   └─> audio.src = streamUrl
   └─> audio.play()

5. Service worker intercepts fetch
   └─> Checks cache, proxies if needed
   └─> Streams encrypted data from Tidal

6. Audio decodes & plays
   └─> Browser's audio codec handles decryption
   └─> Web Audio API applies effects (EQ, replay gain)
   └─> Speaker output

7. Scrobbling (if enabled)
   └─> multi-scrobbler.scrobble(track)
   └─> Sends to Last.fm/ListenBrainz after 50% played

8. History tracking
   └─> db.addToHistory(track)
   └─> Saves to IndexedDB
```

---

## Build & Deployment

**Build Tool**: Vite (modern bundler)

```bash
npm run dev          # Development server at localhost:5173
npm run build        # Production build to /dist
npm run test         # Run tests with Vitest
npm run lint         # Check code quality
```

**Vite Plugins**:
- `vite-plugin-pwa` - PWA support (Service Workers, manifest)
- `authGatePlugin` - Auth wall (redirect to login if needed)
- `uploadPlugin` - Handle file uploads
- `blobAssetPlugin` - Inline small assets as data URLs
- `svgUse` - SVG sprite support

**Deployment**:
- Cloudflare Pages (serverless hosting)
- Contains 648 tracks of JavaScript across multiple pages

---

## Tech Stack Summary

**Frontend:**
- **Framework**: Preact (lightweight React)
- **Language**: TypeScript + JavaScript
- **Styling**: CSS3 with dark theme
- **Build**: Vite
- **Testing**: Vitest + Playwright

**APIs & Services:**
- **Music**: Tidal API (via proxy), Apple Music Search
- **Podcasts**: Custom podcasts API
- **Lyrics**: Genius API
- **Auth**: Appwrite + PocketBase
- **Scrobbling**: Last.fm API, ListenBrainz API
- **Hosting**: Cloudflare Pages/Workers

**Key Libraries:**
- `hls.js` - HLS stream playback
- `shaka-player` - DASH stream playback
- `butterchurn` - Visualizers
- `fuse.js` - Fuzzy search
- `ffmpeg.js` - Audio processing (WebAssembly)
- `jose` - JWT handling
- `pocketbase` - Database client

---

## Key Concepts

### Quality Levels (Tidal)
1. **LOW** - 96 kbps MP3
2. **HIGH** - 320 kbps MP3
3. **LOSSLESS** - 16-bit/44.1kHz (FLAC) - ~1411 kbps
4. **HI_RES_LOSSLESS** - Up to 24-bit/192kHz
5. **DOLBY_ATMOS** - Spatial audio (various formats: EAC3, AC4)

### DRM & Streaming
- Most Tidal streams are encrypted with Tidal's DRM
- Browser's built-in audio codec (EME - Encrypted Media Extension) handles decryption
- HLS.js handles adaptive bitrate selection & manifest parsing
- Proxy service ensures CORS headers and authentication

### PWA Features
- Installable on desktop/mobile
- Offline support (cached assets + media)
- Background sync of favorites
- Notification support
- Media controls on lock screen

---

## Common Workflows

### Adding a New Feature

1. **API Integration**: Add method to `js/api.js` or `js/music-api.js`
2. **State Management**: Add setting to `js/storage.js` if needed
3. **UI**: Create Preact component in `src/` and render it
4. **Data Persistence**: Update `js/db.js` IndexedDB schema if needed
5. **Styling**: Add CSS to `src/styles/modern.css`

### Handling Authentication Issues

1. Check `js/accounts/auth.js` for login flow
2. Verify tokens in `js/accounts/pocketbase.js` for sync
3. API errors often due to expired Tidal credentials - refresh in settings

### Performance Optimization

- Use `UIRenderer` for DOM updates (batching)
- Leverage IndexedDB caching to avoid API calls
- Service Workers cache media for offline playback
- Preload next track via `player.preloadCache`

---

## Debugging Tips

**Check Console for Errors:**
```javascript
// In browser DevTools console
MusicAPI.instance                   // Check if initialized
Player.instance                     // Check player state
localStorage.getItem('volume')      // Check settings
```

**Network Tab:**
- Look for requests to `https://lol.samidy.workers.dev` (Tidal proxy)
- Audio streams should show in Network → XHR/Fetch

**Application Tab (DevTools):**
- Service Workers: Check if registered & running
- Cache: View cached assets & media
- IndexedDB: Browse favorites, history, playlists

**Performance:**
- Lighthouse audit shows PWA compliance
- Bundle analyzer: `npm run build` creates `bundle-stats.html`

---

## Summary

Monochrome is a sophisticated web music player that:
1. Abstracts Tidal's API behind a clean interface
2. Manages playback with Web Audio API & native `<audio>` element
3. Uses IndexedDB for rich client-side data
4. Caches aggressively with Service Workers for offline support
5. Provides beautiful UI with Preact for fast rendering
6. Enables optional cloud sync for multi-device access

The architecture prioritizes **privacy** (no tracking), **quality** (hi-res audio support), and **user control** (customizable UI, local-first data).
