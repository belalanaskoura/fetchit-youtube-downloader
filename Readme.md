# FetchIt — YouTube Downloader

A modern, fast, and feature-rich desktop application for downloading YouTube videos and audio. Built with **Tauri 2.0**, **React**, **Tailwind CSS**, and **yt-dlp**.

## Overview

FetchIt is a sleek, dark-themed YouTube downloader that lets you download media in multiple formats and quality levels with real-time progress tracking. All downloads are stored in a local SQLite database with download history, including thumbnails for quick visual reference.

### Key Highlights

- **Fast & Efficient**: Parallel fragment downloads (16 concurrent fragments) for 3-5x faster downloads
- **Multiple Formats**: Audio (MP3, M4A, WAV) and Video (MP4, WEBM, MKV) with extensive quality options
- **Real-time Progress**: Live percentage, speed, and ETA tracking with animated progress card
- **Download History**: SQLite-backed history with thumbnails, delete individual entries or clear all
- **Smart Defaults**: Auto-detects your system's Downloads folder, remembers settings between sessions
- **Lightweight**: ~50MB app size, minimal dependencies, runs on Windows, macOS, and Linux

---

## Features

### Core Functionality
- **Audio Downloads**
  - Formats: MP3, M4A, WAV
  - Quality: 128 kbps, 192 kbps, 256 kbps, 320 kbps, Lossless (WAV)
  - Auto-extracts best available audio stream

- **Video Downloads**
  - Formats: MP4, WEBM, MKV
  - Quality: 360p, 480p, 720p, 1080p, 1440p, 2160p (4K)
  - Combines best video + audio with FFmpeg

### User Interface
- **Clean Dark Theme**: OKLCH color system, smooth transitions
- **Cascading Selectors**: Type → Format → Quality dynamically updates
- **Sidebar History**: Thumbnail previews, timestamps, right-click delete
- **Progress Card**: Animated states (idle, downloading, processing, complete)
- **Paste Button**: Quick URL input from clipboard
- **Directory Picker**: Browse and select custom save locations

### Download Management
- **Cancellable Downloads**: Stop mid-transfer with a single click
- **Auto-open**: Opens parent folder when download completes
- **Error Handling**: Clear error messages, retry support
- **Concurrent Downloads**: Background queue management (future-ready)

### Data & History
- **SQLite Database**: Local storage (`downloads.db`)
- **Rich Entries**: Title, URL, format, quality, file size, thumbnail, timestamp
- **Persistent**: Downloads survive app restarts
- **Cleanup**: Per-item delete or bulk clear

---

## Architecture

### Technology Stack

| Component | Technology | Version |
|-----------|-----------|---------|
| Desktop Framework | Tauri | 2.0 |
| Frontend | React | 18+ |
| Styling | Tailwind CSS | v4 |
| UI Components | Shadcn/UI | Latest |
| Backend | Rust | stable |
| Database | SQLite | via tauri-plugin-sql |
| Downloader | yt-dlp | 2026.3.3+ |
| Media Processing | FFmpeg | 5.0+ |
| State Animation | Framer Motion | Latest |

### Project Structure

```
Link_to_Media/
├── README.md                      ← Detailed documentation
├── CLAUDE.md                      ← Development guide for AI assistants
├── run_app.bat                    ← Windows launcher (sets PATH, runs dev server)
├── config.json                    ← Legacy app config
├── requirements.txt               ← Legacy app dependencies
├── youtube_downloader.py          ← Legacy Python/CustomTkinter app
│
├── sidecar/
│   └── yt_dlp_worker.py          ← Python child process: yt-dlp IPC handler
│
└── tauri-app/                     ← Primary Tauri + React app
    ├── src/
    │   ├── main.tsx               ← React entry point
    │   ├── App.tsx                ← Root component: layout, SQLite, Tauri events
    │   ├── App.css                ← App-level styles
    │   ├── index.css              ← Global Tailwind @theme (oklch dark mode)
    │   ├── vite-env.d.ts          ← Vite type definitions
    │   │
    │   ├── assets/
    │   │   └── logo.png           ← FetchIt app logo (transparent)
    │   │
    │   ├── components/
    │   │   ├── ui/                ← Shadcn-style headless components
    │   │   │   ├── button.tsx
    │   │   │   ├── input.tsx
    │   │   │   ├── badge.tsx
    │   │   │   ├── select.tsx
    │   │   │   ├── scroll-area.tsx
    │   │   │   └── separator.tsx
    │   │   │
    │   │   ├── DownloadForm.tsx   ← URL input, Type/Format/Quality cascade
    │   │   ├── ProgressCard.tsx   ← Download progress animation
    │   │   └── Sidebar.tsx        ← History panel with thumbnails
    │   │
    │   └── lib/
    │       ├── types.ts           ← TypeScript type definitions
    │       └── utils.ts           ← Helper functions (format, merge classes, etc.)
    │
    ├── src-tauri/
    │   ├── src/lib.rs             ← Tauri commands & sidecar orchestration
    │   ├── Cargo.toml             ← Rust dependencies
    │   ├── tauri.conf.json        ← Tauri app configuration
    │   └── capabilities/
    │       └── default.json       ← Plugin permissions (sql, dialog, opener)
    │
    ├── vite.config.ts             ← Vite build configuration
    ├── tsconfig.json              ← TypeScript config
    ├── package.json               ← npm dependencies & scripts
    ├── tailwind.config.js         ← Tailwind CSS config
    ├── postcss.config.js          ← PostCSS config
    │
    └── index.html                 ← HTML entry point
```

### Component Hierarchy

```
App.tsx (Root)
│
├── Sidebar.tsx
│   ├── Logo + branding (h-20 w-20, object-contain)
│   ├── History header (Downloads count, Clear all button)
│   │
│   └── History items (ScrollArea)
│       └── HistoryItem × N
│           ├── Thumbnail image
│           ├── Title, format badge, quality
│           ├── Timestamp (relative)
│           ├── Delete button (always visible)
│           └── Action buttons (hover: Open, Show Folder)
│
└── Main content (flex-col)
    ├── DownloadForm.tsx
    │   ├── URL input + Paste button
    │   ├── Type selector (Audio/Video)
    │   ├── Format selector (cascaded)
    │   ├── Quality selector (cascaded)
    │   ├── Save location picker
    │   └── Download button
    │
    └── ProgressCard.tsx
        ├── Animated progress bar (Framer Motion)
        ├── Percentage + speed display
        ├── Cancel button (visible during download)
        └── Status message (error/success)
```

---

## Data Flow

### Download Lifecycle

```
1. User Interface (React)
   └─ DownloadForm: User enters URL, selects Type/Format/Quality
                    Clicks Download → onDownloadStart()

2. App.tsx (React)
   └─ Stores meta in pendingMeta ref
   └─ Calls Tauri command: start_download(url, format, quality, output_dir)

3. Tauri Backend (Rust)
   └─ Spawns sidecar: python sidecar/yt_dlp_worker.py
   └─ Sends JSON to stdin: {"action": "download", ...}
   └─ Reads stdout line-by-line (JSON events)
   └─ Emits Tauri events: download://progress, download://processing, etc.

4. React Frontend (Listener)
   └─ Receives Tauri events
   └─ Updates state: downloadState, progress, speed
   └─ ProgressCard animates UI
   └─ On download://complete:
      - Insert into SQLite
      - Update history sidebar
      - Auto-open download folder

5. Sidecar (Python)
   └─ Receives command via stdin
   └─ Builds yt-dlp options (format, quality, post-processors)
   └─ Calls ydl.extract_info() with progress hook
   └─ Emits progress events to stdout
   └─ Returns complete/error/cancelled status
```

### IPC Communication

**Rust ↔ Python (stdin/stdout JSON)**

```
← Rust sends:
  {"action": "download", "url": "...", "format": "mp3", "quality": "192", "output_dir": "..."}
  {"action": "cancel"}

→ Python responds:
  {"type": "progress", "percent": 45.2, "speed": "1.2 MB/s"}
  {"type": "finished"}
  {"type": "complete", "filepath": "...", "title": "...", "thumbnail": "..."}
  {"type": "error", "message": "..."}
  {"type": "cancelled"}
```

---

## Database

**SQLite database:** `downloads.db` (auto-created on first run)

**Table schema:**
```sql
CREATE TABLE downloads (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  title TEXT,
  url TEXT,
  filepath TEXT,
  format TEXT,
  quality TEXT,
  downloaded_at TEXT,
  file_size INTEGER DEFAULT 0,
  thumbnail TEXT DEFAULT ''
);
```

**Example entry:**
```
id: 1
title: "Never Gonna Give You Up"
url: "https://www.youtube.com/watch?v=dQw4w9WgXcQ"
filepath: "/Users/username/Downloads/Never Gonna Give You Up.mp3"
format: "mp3"
quality: "320"
downloaded_at: "2026-03-05T13:58:00.000Z"
file_size: 4500000
thumbnail: "https://i.ytimg.com/vi/dQw4w9WgXcQ/maxresdefault.jpg"
```

---

## Requirements

### System Requirements
- **OS**: Windows 10+, macOS 10.13+, Linux (Ubuntu 18.04+)
- **RAM**: 512 MB minimum (1 GB recommended)
- **Disk**: 200 MB for app + variable per download

### Dependencies

| Dependency | Version | Purpose |
|------------|---------|---------|
| Rust | stable (2024+) | Backend compilation |
| Node.js | 18+ | Frontend build |
| Python | 3.8+ | Sidecar process |
| yt-dlp | 2026.3.3+ | Core downloader |
| FFmpeg | 5.0+ | Audio/video encoding |

### Installation

#### 1. Install Rust
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

#### 2. Install Node.js
Download from [nodejs.org](https://nodejs.org/) — LTS recommended

#### 3. Install Python 3.8+
Download from [python.org](https://www.python.org/)

#### 4. Install yt-dlp
```bash
pip install yt-dlp
```

#### 5. Install FFmpeg

**Windows:**
- Download from [ffmpeg.org](https://ffmpeg.org/download.html)
- Extract and add to PATH

**macOS:**
```bash
brew install ffmpeg
```

**Linux (Ubuntu/Debian):**
```bash
sudo apt update && sudo apt install ffmpeg
```

#### 6. Clone & Install
```bash
git clone https://github.com/yourusername/Link_to_Media.git
cd Link_to_Media/tauri-app
npm install
```

---

## Running

### Quick Start (Windows)
```bash
# From repo root
double-click run_app.bat
```

### Manual Start
```bash
cd tauri-app
export PATH="$HOME/.cargo/bin:$PATH"  # Linux/macOS
npm run tauri dev
```

### First Run
- Rust backend compiles (~1-2 minutes)
- App window opens
- Subsequent runs are fast

### Production Build
```bash
npm run tauri build
```

Output: `src-tauri/target/release/bundle/`

---

## Usage

### Download Audio

1. Paste YouTube URL (or use Paste button)
2. Select **Type** → Audio
3. Choose **Format**: MP3, M4A, or WAV
4. Pick **Quality**: 128, 192, 256, 320 kbps, or Lossless
5. (Optional) Change save location
6. Click **Download**

### Download Video

1. Paste YouTube URL
2. Select **Type** → Video
3. Choose **Format**: MP4, WEBM, or MKV
4. Pick **Quality**: 360p to 4K
5. (Optional) Change save location
6. Click **Download**

### History Management

- **View**: Sidebar shows all downloads with thumbnails
- **Delete**: Click trash icon on entry
- **Clear All**: Click "Clear all" button in sidebar header
- **Open**: Click "Open" to play/view file
- **Show Folder**: Click "Folder" to browse parent directory

### Troubleshooting

| Issue | Solution |
|-------|----------|
| "Video unavailable" | Check region restrictions, try VPN or update yt-dlp |
| "Python not found" | Ensure Python is in PATH or use full path |
| "FFmpeg not found" | Install FFmpeg and add to PATH |
| Downloads freeze | Update yt-dlp: `pip install -U yt-dlp` |
| App won't launch | Check console output from `npm run tauri dev` |

---

## Keeping yt-dlp Updated

YouTube frequently breaks compatibility. Update regularly:

```bash
pip install -U yt-dlp
yt-dlp --version
```

---

## Development

### Tech Details

- **React**: Functional components with hooks
- **Tailwind v4**: @theme dark mode with OKLCH colors
- **TypeScript**: Full type coverage
- **Rust**: Async IPC with JSON serialization
- **SQLite**: Async queries via tauri-plugin-sql

### Key Files to Know

- `src/App.tsx` — State management, Tauri listeners, SQLite init
- `src/components/DownloadForm.tsx` — URL input and type/format/quality logic
- `src/components/ProgressCard.tsx` — Download state animation
- `src/components/Sidebar.tsx` — History display and deletion
- `src-tauri/src/lib.rs` — Tauri commands and sidecar spawning
- `sidecar/yt_dlp_worker.py` — yt-dlp wrapper and IPC handler

---

## Building for Release

```bash
cd tauri-app
npm run tauri build
```

Creates:
- **Windows**: `.msi` installer
- **macOS**: `.dmg` disk image
- **Linux**: `.AppImage` bundle

---

## Contributing

1. Fork repository
2. Create feature branch: `git checkout -b feature/amazing-feature`
3. Make changes and test thoroughly
4. Commit: `git commit -m "Clear description"`
5. Push: `git push origin feature/amazing-feature`
6. Create Pull Request

---

## License

MIT License — See LICENSE for details

---

## Support

- Issues: GitHub Issues
- Discussions: GitHub Discussions
- Documentation: See CLAUDE.md for developer guide

---

## Changelog

### v0.1.0 (March 5, 2026)
- Initial Tauri 2.0 release
- Audio & video downloads with multiple formats/qualities
- SQLite history with thumbnails
- Dark theme with Tailwind v4 + OKLCH
- Real-time progress with Framer Motion
- Auto-open downloads folder
- History management (delete individual/clear all)
- Updated logo branding
- Improved delete button visibility
- Logo size optimization (80×80px with aspect ratio preservation)
