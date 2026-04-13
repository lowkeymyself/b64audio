# b64audio

A fully portable, zero-dependency audio editor that runs entirely from a single HTML file. All audio is stored as lossless 16-bit PCM encoded in base64 — no server, no install, no build step. Open it in any modern browser and start editing.

**Think of it as a zip file for audio editing.** One file. Self-contained. Drag it anywhere — a USB stick, a cloud drive, an email attachment, a corporate network with no admin rights — double-click, and you have a full audio editor. No installer dialog, no "checking for updates," no terms of service. It just works.

### Get it

- **Download it officially** — [b64.audio](https://b64.audio/)
- **Or grab it here** — grab `index.html` from [GitHub](https://github.com/lowkeymyself/b64audio) and open it locally
- **Self-host it** — drop the file on any static host. That's the entire deployment.

## Why base64

Every audio format that achieves meaningful compression (MP3, OGG, AAC) uses perceptual coding — lossy transforms that make direct sample manipulation impossible. Multiply a raw MP3 byte by a volume factor and you destroy the frame headers. There is no way to perform sample-accurate editing on compressed audio without first decoding it.

b64audio solves this by converting all input to uncompressed 16-bit PCM WAV on import, then encoding the raw bytes as a base64 string. This string is the single source of truth. Every operation follows the same pipeline:

```
atob() → raw PCM bytes → transform samples → btoa()
```

The result is a fully lossless editing workflow. No generation loss, no re-encoding artifacts, no lossy round-trips. The audio you export is sample-identical to what you edited.

## Portability

The entire editor is one HTML file. Not "a project that builds to one file" — one file, period. No frameworks, no dependencies, no bundler, no node_modules. It runs offline, from a `file://` URL, from a USB drive handed to someone who has never heard of npm.

The exported `.b64.txt` files are plain ASCII strings. Store them in a JSON field, paste them into a text box, commit them to a repo, send them through a chat message. Any channel that can carry text can carry your audio — losslessly, with zero binary encoding headaches.

## Features

### Clip-based editing

Audio is divided into clips — contiguous frame ranges rendered as individual rounded-corner segments on the waveform with visible gaps and white outlines between them.

- **Split** (`S`) — divide a clip at the cursor into two independent clips
- **Select clips** (`Ctrl+Click`) — toggle selection on individual clips; selected clips highlight in purple
- **Delete** (`Del`) — remove selected clips or frame regions
- **Copy / Paste** (`Ctrl+C` / `Ctrl+V`) — copy clips or selections to an internal clipboard, paste at cursor
- **Trim by dragging** — drag clip edges directly on the waveform. Inner edges (split points) slide freely between adjacent clips; outer edges trim PCM data

### Non-destructive controls

- **Ripple toggle** (`R`) — when ripple is on, delete and trim operations remove audio and shift the timeline. When off, they silence the region instead, preserving duration and clip positions
- **Snap** (`M`) — magnetic snap to clip edges with an adaptive threshold that scales with zoom level

### Effects

All effects operate on the active scope: frame selection > selected clips > entire file.

| Effect | Description |
|--------|-------------|
| **Volume** | Scale all samples by a factor (0.5× – 10×), clamped to 16-bit range |
| **Fade In** | Linear ramp from silence over a configurable percentage of the range |
| **Fade Out** | Linear ramp to silence over a configurable percentage of the range |
| **Reverse** | In-place frame reversal via web worker |
| **Normalize** | Peak detection + gain scaling to 0 dBFS via web worker |
| **Speed (resample)** | Linear-interpolation resampling — changes both speed and pitch |
| **Speed (keep pitch)** | Overlap-Add (OLA) time-stretching — changes speed, preserves pitch, via web worker |
| **Insert Silence** | Write zero-valued frames at a specified position and duration |

Stereo is fully supported. All operations process each channel per frame.

### Zoom and navigation

- **Ctrl+Scroll** on the waveform to zoom in/out centered on cursor (up to 128×)
- **+/-** keys or toolbar buttons for zoom control
- **Scroll wheel** pans horizontally when zoomed
- **Scrollbar** appears below the waveform at zoom levels above 1×
- **Minimap** in the bottom panel shows the full waveform overview with a viewport indicator; click or drag to navigate
- **Time ruler** adapts tick density to the visible duration at the current zoom level

### Multi-file editing

Open multiple files simultaneously via drag-and-drop or the tab bar. Each file maintains independent undo/redo stacks and clip state.

- **Bulk operations** — apply volume, normalize, reverse, or fades across multiple files at once
- **Diff** — compare two files side by side with overlaid waveforms, a difference visualization, and full metadata comparison

### Playback

- Click the waveform to position the cursor; press **Space** to play from that position
- Playhead tracks the current position in real time during playback
- Edits made during playback are hot-reloaded — the audio source is swapped and seeked to the same position without interruption

### Import and export

| Format | Description |
|--------|-------------|
| **WAV** | Standard 16-bit PCM WAV file — lossless, universally compatible |
| **Base64 text** | Raw `.b64.txt` file containing the full WAV as an ASCII string |

Import from any browser-decodable format (MP3, WAV, OGG, FLAC, AAC). Import from base64 by pasting a string or loading a `.b64.txt` file. Data URL prefixes are stripped automatically.

## Keyboard shortcuts

| Key | Action |
|-----|--------|
| `Space` | Play / pause |
| `S` | Split at cursor |
| `R` | Toggle ripple edit |
| `M` | Toggle snap |
| `Del` / `Backspace` | Delete selection or clips |
| `Ctrl+Z` | Undo |
| `Ctrl+Y` / `Ctrl+Shift+Z` | Redo |
| `Ctrl+C` | Copy |
| `Ctrl+V` | Paste at cursor |
| `Ctrl+X` | Cut (copy + delete) |
| `Ctrl+A` | Select all |
| `Ctrl+S` | Download WAV |
| `Ctrl+Click` | Toggle clip selection |
| `Alt+Drag` | Range selection on waveform |
| `Ctrl+Scroll` | Zoom in/out |
| `+` / `-` | Zoom in/out |
| `Esc` | Deselect / close panels |

## Performance

- **Byte cache** — decoded PCM bytes are cached and only re-decoded when the base64 string changes
- **Subsampled rendering** — waveform drawing samples at most 8 frames per pixel column regardless of zoom level
- **Viewport culling** — only visible clips are rendered during zoom; off-screen clips are skipped entirely
- **Throttled redraws** — scroll and zoom operations batch through `requestAnimationFrame` to prevent redundant draws
- **Web workers** — reverse, normalize, speed change, and OLA time-stretching run in inline workers to keep the UI responsive on large files

## Mobile

Touch-optimized with full support for cursor positioning, scrubbing, and edge trimming via touch events. Responsive layout collapses the toolbar, hides the sidebar, and adapts the waveform height for smaller screens.

## How to run

```
Open index.html.
```

## Technical notes

- All audio is stored as 16-bit signed PCM WAV (44-byte RIFF header + interleaved samples)
- Base64 encoding uses chunked `btoa()` with 8192-byte segments to avoid call stack limits
- Undo/redo stores full `{b64, clips}` snapshots — simple and correct, but memory-intensive for large files
- The OLA time-stretching implementation uses a 2048-sample Hann window with 75% overlap
- Clip boundaries are metadata only — they reference frame indices into the single PCM buffer, not separate audio data

