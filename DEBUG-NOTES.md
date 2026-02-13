# Valentine Photo Book — Debug Notes

## Problem
`index.html` crashes Chrome (likely on mobile / iPhone via Safari or Chrome).

## Test Version
`test.html` — stripped down to isolate the crash cause:
- 5 slides only (cover + 3 photos + final message)
- NO WebGL fluid shader
- NO canvas particle system
- NO fancy animation classes (glow/heart/rise/reveal/warm/clip-path)
- NO hearts burst DOM spam
- NO `loadAll()` eager-loading 66 media files at once
- Images loaded directly via `src=` (only 3 images)
- Simple CSS `opacity` fade transition
- Simple `setInterval` auto-advance (3s per slide)
- Basic hold-to-pause

## Top 3 Suspected Crash Causes (in order of likelihood)

### 1. 🔴 WebGL Fluid Shader — MOST LIKELY
The `initFluid()` function runs a **full-screen fragment shader with 5-iteration FBM (fractal Brownian motion)** on every single frame via `requestAnimationFrame`. This is:
- A **continuous GPU-intensive loop** running the entire time the page is open
- Rendering to a full-viewport canvas underneath everything
- Combined with the particle canvas, that's **two full-screen canvases** being redrawn every frame
- On mobile GPUs (especially iPhone), this alone can cause thermal throttling and tab crashes
- The shader does `5 iterations × multiple texture lookups × full screen pixels` — extremely heavy for a mobile GPU

**Why it's #1**: WebGL shader crashes are the single most common cause of mobile Chrome/Safari tab kills. The browser's GPU process hits memory or thermal limits and the OS kills it.

### 2. 🟡 Loading 66 Media Files Simultaneously
`loadAll()` is called on tap-to-begin and sets `src` on ALL 66 slides at once:
- **58 images** (JPEGs, likely 100-500KB each in `_web` format)
- **8 videos** (MP4s, likely 2-10MB each — `06.mp4`, `28.mp4`, `29.mp4`, `30.mp4`, `48.mp4`, `49.mp4`, `50.mp4`, `66.mp4`)
- Total estimated simultaneous load: **50-150MB+**
- Browser must decode and hold all of these in memory
- Video elements with `autoplay` will try to buffer/decode even when not visible
- This creates a massive memory spike right at the moment the user taps "begin"

**Why it's #2**: The lazy loading (`loadAround`) was partially there but `loadAll()` overrides it. On a 3-4GB RAM iPhone, holding 8 decoded videos + 58 decoded images in memory simultaneously can easily trigger an OOM kill.

### 3. 🟠 Animation/DOM Complexity Compounding
Multiple simultaneous animation systems all running via `requestAnimationFrame`:
- WebGL fluid shader loop (rAF)
- Canvas particle system (rAF) — 45 particles redrawn every frame
- CSS animations with `will-change: opacity, transform` on ALL 67 slides
- `clip-path` animations (`t-reveal`) which are notoriously expensive on mobile
- `filter: blur()` and `filter: brightness()` animations (`t-glow`, `t-warm`) — these trigger full-layer repaints
- Hearts burst creating/destroying DOM elements every 3 slides
- Progress bar `requestAnimationFrame` loop
- All of this **stacks**: 3-4 concurrent rAF loops + CSS filter animations + DOM churn

**Why it's #3**: Any one of these is fine alone, but the combination creates GPU/CPU contention that pushes the device over the edge.

## Testing Strategy

| Test | What it isolates | Expected result |
|------|-----------------|-----------------|
| `test.html` works fine | Confirms crash is NOT in password logic, basic slide mechanics, or audio playback | Baseline — should work |
| Add WebGL shader back to test.html | Tests if GPU shader alone crashes it | If crashes → shader is the culprit |
| Keep test.html but load all 66 images | Tests memory from media loading | If crashes → memory from media |
| Keep test.html but add fancy animations | Tests animation complexity | If crashes → animation overhead |

## Quick Fixes to Try (after identifying cause)

- **If WebGL**: Remove fluid shader entirely, or add `{powerPreference: 'low-power'}` to WebGL context, or only run shader on desktop
- **If memory**: Keep lazy loading, never call `loadAll()`, only load ±2 slides, add `loading="lazy"` to img tags, don't set video `src` until needed
- **If animations**: Remove `filter: blur/brightness` animations, avoid `clip-path` on mobile, reduce `will-change` usage, limit rAF loops to 1
