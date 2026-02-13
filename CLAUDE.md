# CLAUDE.md — Valentine's Day Photo Book for Rhiannon

## Project Overview
A password-protected, interactive Valentine's Day photo book built as a single-page static site hosted on GitHub Pages. Created for Finn to give to his girlfriend Rhiannon on Valentine's Day 2026.

**Live URL:** https://finnerzbtz.github.io/valentine-rhiannon/
**Repo:** https://github.com/finnerzbtz/valentine-rhiannon
**Password:** `babeiloveyousomuchwhatdaheck`

## Architecture
- **Single HTML file** (`index.html`) — all CSS + JS inline, no build step
- **Static hosting** via GitHub Pages (auto-deploys on push to `main`)
- **Media** stored in `photos/` directory (58 JPGs + 10 MP4s, ~8.5MB total images)
- **Music** — `music.mp3` ("You and Me Song" by The Wannadies, ~5.9MB)

## Features

### Core
- **Password-protected entry** — Custom passphrase locks the content
- **68 slides** — Cover page → 66 photo/video slides → final message page
- **Swipe navigation** — Touch swipe left/right, tap edges, or keyboard arrows
- **Dot indicators** — Bottom navigation dots showing current position

### Music & Auto-Advance
- **Background music** — "You and Me Song" plays after tap-to-begin
- **Beat-synced auto-advance** — Photos advance every 2.308s (1 bar at 104 BPM)
- **Videos get double time** — 4.615s (2 bars) for video slides
- **Hold-to-pause** — Touch and hold to pause auto-advance, release to resume
- **Progress bar** — Gold gradient bar at top shows song progress
- **Music toggle** — ♫ button in top-right corner

### Visual Effects
- **Gold particle system** — Canvas-based floating gold particles (8-15 particles, throttled to ~20fps)
- **WebGL fluid shader** — Pink/red/orange/magenta flowing background (currently disabled for stability)
- **Valentine transitions** — 5 entrance animations: glow, heartbeat zoom, soft drift, circle reveal, warm dissolve (currently disabled for stability)
- **Heart bursts** — Floating heart emojis every 3 slides (currently disabled for stability)
- **Dark aesthetic** — Black background, gold (#d4af37) text and accents

### Final Page
- ❤️ heart emoji
- "you and me / always and forever"
- "forever yours x"

## Design Decisions

### Fonts
- **Playfair Display** — Headings (serif, elegant)
- **Cormorant Garamond** — Body text (light, romantic)

### Color Palette
- Background: `#0a0a0a` (near-black)
- Primary: `#d4af37` (gold)
- Accents: `rgba(212,175,55,*)` at various opacities
- Fluid shader: hot pink, red, orange, magenta (when enabled)

### iOS Compatibility
- `playsinline muted autoplay` on all videos for iOS inline playback
- Tap-to-begin overlay required for iOS audio autoplay policy
- Password unlock triggers overlay, real tap triggers audio

## Technical Notes

### Lazy Loading Strategy
- **Images:** All loaded eagerly on tap-to-begin via `loadAll()` (small enough to preload)
- **Videos:** Lazy-loaded via `loadAround()` — only current ±1 slides have video src set
- **Video unloading:** Videos >2 slides away get their `src` removed to free memory

### Performance Optimizations
- Particle canvas throttled to ~20fps (50ms frame budget)
- Fluid shader renders at 40% resolution on mobile, throttled to ~24fps
- All rAF loops skip frames when `document.hidden`
- Auto-advance pauses when tab is backgrounded
- Heart burst DOM elements capped at 30, removed after animation

### Known Issues & Current State
1. **Fluid shader disabled** — Was causing Chrome crashes on mobile. Needs GPU profiling before re-enabling. The shader (5-layer FBM) may be too heavy for some mobile GPUs.
2. **Fancy transitions disabled** — clip-path and filter animations caused flickering on some devices. Need testing per-transition.
3. **Heart bursts disabled** — Disabled alongside transitions.
4. To re-enable, search for `// disabled for stability` in index.html.

### Crash Debugging History
The site went through several crash-fix cycles:
1. **loadAll() loading all videos** — 100MB+ memory spike killing Chrome → Fixed: only eager-load images
2. **WebGL shader at full res 60fps** — GPU overload → Fixed: reduced resolution + throttled, then disabled
3. **iOS autoplay blocking** — Music wouldn't play → Fixed: tap-to-begin overlay
4. **Animation fill-mode: both** — clip-path starting at 0% hid slides → Fixed: changed to forwards
5. **Variable ordering** — `mbtn`/`bgm` referenced before declaration → Fixed: moved declarations up
6. **Audio loop + ended event** — Slideshow restarting instead of ending → Fixed: removed loop, stop at final slide
7. **Multiple unlock calls** — Password input event firing on every keystroke → Fixed: `unlocked` guard flag

## File Structure
```
valentine-deploy/
├── index.html          # Main site (all-in-one)
├── test.html           # Stripped-down debug version (5 slides, no effects)
├── music.mp3           # "You and Me Song" by The Wannadies
├── CLAUDE.md           # This file
├── DEBUG-NOTES.md      # Crash analysis notes
└── photos/
    ├── 01.jpg - 66.jpg/mp4  # 58 images + 10 videos
    └── (auto-numbered, mix of .jpg and .mp4)
```

## How to Update

### Add/remove photos
1. Add files to `photos/` directory
2. Add corresponding `<div class="slide">` block in index.html
3. Update slide numbering if needed
4. Push to main

### Change the password
Search for `const PW=` in index.html and update the string.

### Change the message
Search for `<div class="msg">` and `<div class="sign">` in the `#final` slide.

### Re-enable effects
Search for `// disabled for stability` and uncomment:
- `initFluid()` — fluid shader background
- `TRANSITIONS[...]` — fancy slide transitions
- `burstHearts()` — floating heart emojis

### Change the music
Replace `music.mp3` with a new file. Update `BAR_MS` constant if BPM differs:
```
BAR_MS = (60 / BPM) * 4 * 1000
```

## Git Workflow
Push to `main` → GitHub Pages auto-deploys in ~60 seconds.
```bash
git add .
git commit -m "description"
git push origin main
```
