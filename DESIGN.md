# RETROCAST 98 — Design Document

A stream overlay system disguised as a 1996 operating system.

---

## 1. The read

The brief: an HTML stream overlay tool — a "frame" layer a streamer runs on top of
their broadcast, the way StreamYard or a Twitch overlay pack does. It needs webcam
routing, arbitrary content windows, drag/resize, and savable layouts. The visual
brief is VGA / DOS / Web 1.0.

The trap in a brief like this is building a modern panel app and painting it green.
The interesting move is to make the *interface itself* the artifact: the overlay is
not a retro-skinned tool, it is a **virtual retro operating system** whose desktop
happens to be the broadcast canvas. Every element of the stream — camera, capture,
chat, alerts, music — is a window in that OS. The streamer runs the OS; the viewer
watches its desktop.

That single decision resolves nearly every downstream question. Draggable/resizable
windows aren't a feature bolted on, they're what an OS *is*. Savable layouts are
save-states. Scenes are virtual desktops. The property panel is an IDE inspector.
"Add an image window" is opening a file.

---

## 2. Aesthetic archaeology

Three source periods, deliberately kept distinct rather than blended into generic
"retro".

**DOS text mode (1981–95)** — CP437 box drawing (`╔═╗ ║ ╚╝ ░▒▓█`), the Turbo Vision
IDE, Norton Commander. 80×25 cells, 9×16 glyphs. Borland blue `#0000AA` grounds with
cyan structure and yellow highlights. Hard offset shadows in dark gray. A function-key
hint bar welded to the bottom of the screen. No anti-aliasing, no curves, no gradients
except dithered ones.

**VGA / demoscene (1987–95)** — mode 13h, 320×200×256. Plasma, fire, starfields,
copper bars, tunnels, palette cycling, Bayer dithering. Chunky pixels as a virtue.
The 16-color hardware palette is the source of truth.

**Web 1.0 (1994–2000)** — GeoCities tiled starfields, `<marquee>`, `<blink>`, hit
counters on odometer wheels, "Under Construction" barber poles, beveled 3D table
borders, rainbow rules, webrings, Comic Sans, Windows 95 chrome, the Hot Dog Stand
color scheme.

**The constraint that keeps it honest:** every color in the default themes is one of
the sixteen canonical IBM VGA colors. Not "inspired by" — the actual values
(`#000000 #0000AA #00AA00 #00AAAA #AA0000 #AA00AA #AA5500 #AAAAAA #555555 #5555FF
#55FF55 #55FFFF #FF5555 #FF55FF #FFFF55 #FFFFFF`). A palette with hardware behind it
reads differently than one picked by eye.

---

## 3. Design plan

**Color** (default theme, `BORLAND`)

| token | value | role |
|---|---|---|
| ground | `#0000AA` | desktop, window fill — IBM blue, not navy |
| structure | `#00AAAA` / `#55FFFF` | borders, rules, labels |
| accent | `#FFFF55` | the single loud color: titles, focus, hotkeys |
| chrome | `#AAAAAA` / `#555555` | menu bar, bevels, shadow |
| hot | `#FF5555` | alerts, record state, destructive actions |
| ok | `#55FF55` | status bar, live indicator, meters |

Eleven further themes ship as complete palette swaps: Norton, Amber Mono, Green
Phosphor, CGA, Windows 98, **Hot Dog Stand**, GeoCities, Vaporwave, Matrix, Blue
Screen, and a fully user-editable Custom.

**Type**

- `Press Start 2P` — display. Arcade bitmap, used with restraint: the logo, big
  banners, alert headlines. It is unreadable in quantity and that's the point.
- `VT323` — body and terminal. The DOS console face. Carries almost all running text.
- `Silkscreen` — utility. Tiny uppercase labels, meter legends, F-key bar.
- `Comic Neue` — reserved exclusively for the GeoCities theme, where it is correct.

**Layout**

Turbo Vision IDE. Menu bar across the top with red hotkey letters; a tool palette of
CP437 glyph buttons down the left; the 1920×1080 stage centered inside a double-line
viewport window; a property inspector on the right; a function-key status bar pinned
to the bottom. The desktop behind it all is a `░` dither texture. Panels collapse;
`F5` drops every piece of editor chrome and the stage fills the screen for broadcast.

**The risk**

The app boots. Not a spinner — a full BIOS POST: memory count-up, IDE device
detection, a device table, then a CRT power-on flash into the IDE. It costs the user
four seconds once and sets the entire frame of reference for everything after it.
Any key skips it.

---

## 4. Architecture

```
1920×1080 stage  ──scaled──>  viewport
   ├── backdrop layer   canvas — starfield / plasma / grid / fire / rain
   ├── window layer     the composite: N draggable windows, z-ordered
   ├── CRT layer        scanlines, aperture grille, noise, roll bar, vignette
   └── transition layer scene wipes
```

State is one JSON tree — canvas size, theme, CRT settings, backdrop, and a list of
scenes each holding a list of windows. Autosaved to `localStorage`, written to eight
named save slots, exported as text. Uploaded media (images, GIFs, video loops) goes to
IndexedDB and is referenced by id, so a config stays a few kilobytes and the browser
quota stays intact.

Window types are a registry. Each entry declares its defaults, a schema of inspector
fields, and a `mount()` returning `{ frame, update, destroy }`. The inspector builds
itself from the schema, so adding a type is one object — which is why there are
twenty-three of them rather than five.

---

## 5. Ideas considered and cut

- **Multi-select + group transform.** Real value, but the inspector becomes a
  multi-value merge problem. Per-window lock plus keyboard nudge covers the actual use.
- **Node-graph compositor.** Wrong altitude entirely. Streamers arrange boxes.
- **Screensaver on idle.** Delightful, catastrophic on a live broadcast. Shipped as an
  opt-in scene instead (Flying Windows / Pipes / DVD logo).
- **Server-side config sync.** No backend. Text export round-trips through any
  clipboard.
- **Real YouTube/Kick chat.** Needs API keys. Twitch's anonymous IRC gateway needs
  none, so that one ships for real.

## 6. Ideas kept that were not asked for

Live captions from the Web Speech API rendered as DOS subtitles. A PC-speaker
synthesizer for UI beeps (square waves, no assets). Twitch chat over anonymous IRC.
An alert queue with a public `RETROCAST.alert()` hook so OBS scripts can fire it.
`?broadcast=1&scene=LIVE` URL parameters so a browser source opens straight into the
right state. Undo/redo. Edge-snapping with alignment guides. Nine camera filters
including 4-color CGA dithering and live ASCII conversion. And the Konami code.
