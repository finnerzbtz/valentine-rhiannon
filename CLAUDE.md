# CLAUDE.md — Valentine's Day Photo Book for Rhiannon

## Project Overview
A password-protected, interactive Valentine's Day photo book built as a single-page static site hosted on GitHub Pages. Created for Finn to give to his girlfriend Rhiannon on Valentine's Day 2026.

**Live URL:** https://finnerzbtz.github.io/valentine-rhiannon/
**Repo:** https://github.com/finnerzbtz/valentine-rhiannon
**Password:** `babeiloveyousomuchwhatdaheck`

## Architecture
- **Single HTML file** (`index.html`) — all CSS + JS inline, no build step
- **Static hosting** via GitHub Pages (auto-deploys on push to `main`)
- **Media** stored in `photos/` directory (58 JPGs + 10 MP4s, ~27MB total)
- **Music** — `music.mp3` ("You and Me Song" by The Wannadies, ~5.9MB)
- **Web App Manifest** — installable as "For Babe ❤️" on home screen

## Features

### Core
- **Password-protected entry** — Custom passphrase locks the content
- **68 slides** — Cover page → 66 photo/video slides → final message page
- **Swipe navigation** — Touch swipe left/right, tap edges, or keyboard arrows
- **Dot indicators** — Bottom navigation dots showing current position
- **Fullscreen mode** — Requests fullscreen API on tap-to-begin (Android/iPad/desktop)

### Music & Auto-Advance
- **Background music** — "You and Me Song" by The Wannadies plays after tap-to-begin
- **Beat-synced auto-advance** — Photos advance every 2.308s (1 bar at 104 BPM)
- **Videos get double time** — 4.615s (2 bars) for video slides
- **Hold-to-pause** — Touch and hold to pause auto-advance, release to resume
- **Progress bar** — Gold gradient bar at top shows song progress
- **Music toggle** — ♫ button in top-right corner
- **Song ends on final page** — No loop, lands on the closing message

### Visual Effects
- **Static gradient aura** — Pink/red/orange radial gradients (replaced WebGL for mobile stability)
- **Slow Ken Burns zoom** — Photos gently zoom to 106% over 3 seconds
- **Heart bursts** — 4 floating heart emojis rise every 5 slides (capped for performance)
- **Dark aesthetic** — Black background (#0a0a0a), gold (#d4af37) text and accents

### Dynamic Text
- **"LOVE YOU ____"** — Big bubbly pink text (Fredoka font) at bottom of screen
- **Cycling pet names** synced to slide changes: BABE, BUB, BUNCH, CRUNCH, SCRUNCH, BUNNY, BUN, BOOBS, BABES, CRUNCHY, BUNCHY, SCRUNCHY, LOVELY, BUM, HONEYBUNCH, HONEYSCRUNCH, SCRUNCHBUNCH
- **Glowing gradient** — Pink gradient text with pulsing glow animation
- Word changes are anchored to goTo() — syncs with auto-advance, manual swipe, and hold-to-pause

### Final Page
- ❤️ heart emoji
- "you and me / always and forever"
- "forever yours x"

### Home Screen App
- **App name:** "For Babe ❤️"
- **Icon:** Pink/red heart on black background (180px apple-touch-icon)
- **Standalone mode** — Opens without browser chrome when added to home screen
- **Web manifest** with 192px and 512px icons

## Design Decisions

### Fonts
- **Playfair Display** — Headings (serif, elegant)
- **Cormorant Garamond** — Body text (light, romantic)
- **Fredoka** — "LOVE YOU ___" text (rounded, bubbly, fun)

### Color Palette
- Background: `#0a0a0a` (near-black)
- Primary: `#d4af37` (gold)
- Love text: `#ff2060` → `#ff4080` → `#ff60a0` (pink gradient)
- Background aura: deep reds, pinks, warm gold (static radial gradients)

### Mobile-First Design
- Photos: 92vw max-width, 68vh max-height
- Slide padding: 20px top, 50px bottom (room for text)
- Love text: `clamp(44px, 12vw, 80px)` — responsive sizing
- Dots: 4px, positioned above love text
- All effects tested on iPhone viewport (390×844)

### iOS Compatibility
- `playsinline muted autoplay` on all videos for iOS inline playback
- Tap-to-begin overlay required for iOS audio autoplay policy
- `apple-mobile-web-app-capable` meta tag for home screen standalone mode
- No WebGL (crashes mobile), no heavy CSS animations

## Technical Notes

### Lazy Loading Strategy
- **Images:** Progressive loading — current slide ±2 ahead, +4 forward
- **Videos:** Only current ±1 slides have video src set
- **Video unloading:** Videos >2 slides away get their `src` removed to free memory
- **No bulk loading** — `loadAll()` was removed as it caused 100MB+ memory spikes

### Performance (Mobile-Safe)
- No WebGL shaders (caused GPU crashes on mobile)
- No canvas particle system (disabled for stability)
- No CSS filter animations (blur/brightness cause layout thrashing)
- Static gradient background instead of animated blobs
- Heart burst DOM elements capped, removed after animation
- Auto-advance pauses when tab is backgrounded (visibilitychange)
- Single rAF loop for progress bar only

### What Was Removed (Crash Prevention)
These features were built but disabled because they crashed real phones:
1. **WebGL fluid shader** — 5-layer FBM fragment shader, too heavy for mobile GPUs
2. **CSS animated aura blobs** — 5 large elements with will-change, compositing overload
3. **Canvas particle system** — Additional rAF loop + canvas compositing
4. **CSS filter transitions** — blur() and brightness() in keyframes caused thrashing
5. **Fancy slide transitions** — clip-path circle reveal, filter-based glow/warm effects

### Crash Debugging History
1. **loadAll() loading all videos** — 100MB+ memory spike → Fixed: progressive lazy loading
2. **WebGL shader at full res 60fps** — GPU overload → Removed entirely
3. **5 animated CSS aura blobs** — Compositing overload → Replaced with static gradient
4. **iOS autoplay blocking** — Music wouldn't play → Fixed: tap-to-begin overlay
5. **Animation fill-mode: both** — clip-path at 0% hid slides → Removed transitions
6. **Variable ordering** — `mbtn`/`bgm` referenced before declaration → Fixed
7. **Audio loop + ended event** — Slideshow restarting → Fixed: stop at final slide
8. **Multiple unlock calls** — Password input event on every keystroke → Fixed: guard flag

## File Structure
```
valentine-deploy/
├── index.html          # Main site (all-in-one HTML/CSS/JS)
├── test.html           # Stripped-down debug version (5 slides)
├── music.mp3           # "You and Me Song" by The Wannadies
├── manifest.json       # Web app manifest ("For Babe ❤️")
├── favicon.ico         # Heart favicon
├── favicon-16.png      # 16px favicon
├── favicon-32.png      # 32px favicon
├── apple-touch-icon.png # 180px iOS home screen icon
├── icon-192.png        # 192px manifest icon
├── icon-512.png        # 512px manifest icon
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

### Change pet name words
Search for `petWords` array in the script and edit the list.

### Change the music
Replace `music.mp3` with a new file. Update `BAR_MS` constant if BPM differs:
```
BAR_MS = (60 / BPM) * 4 * 1000
```

### Re-enable effects (at your own risk)
Search for `// initPtcl()` and uncomment to restore particles.
The WebGL fluid shader code has been removed — check git history to restore.

## Git Workflow
Push to `main` → GitHub Pages auto-deploys in ~60 seconds.
```bash
git add .
git commit -m "description"
git push origin main
```

### Branches
- `main` — production (deployed)
- `full-screen-mode` — merged into main
- `claude/fix-webgl-mobile-background-Q0yxY` — merged into main

## Testing
Visual Playwright setup at `/home/node/clawd/visual-playwright/` for screenshot-based testing.
Mobile emulation: iPhone (390×844, 3x scale, touch enabled).
```bash
cd /home/node/clawd/visual-playwright
node scripts/vp.mjs goto "https://finnerzbtz.github.io/valentine-rhiannon/" --screenshot shots/test.png
```
