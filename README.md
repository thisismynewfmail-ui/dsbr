# RETROCAST 98

A livestream overlay system built as a 1996 operating system.

Every part of your broadcast — cameras, screen capture, chat, alerts, music, timers,
goals — is a draggable window on a 1920×1080 stage. You arrange the desktop; OBS
captures it. One HTML file, no build step, no dependencies, no server.

---

## Run it

Open `index.html` in Chrome or Edge. That's the whole install.

The page boots through a BIOS POST screen (press any key to skip), then drops into a
Turbo Vision-style editor: tool palette on the left, the broadcast stage in the middle,
a property inspector on the right, and a function-key bar along the bottom.

**Press F5 to hide the editor.** What's left is the overlay.

## Wire it into OBS

**Browser source (recommended)**

1. Add a **Browser** source, tick **Local file**, point it at `index.html`.
2. Width `1920`, height `1080`.
3. Tick **Control audio via OBS** if you use the audio meter.

For a hosted copy, append URL parameters:

| Parameter | Effect |
|---|---|
| `?broadcast=1` | Open with the editor already hidden |
| `?scene=LIVE` | Open on a named scene |
| `?theme=amber` | Force a palette |
| `?transparent=1` | Start with a transparent background |
| `?noboot=1` | Skip the BIOS screen |

They combine: `index.html?broadcast=1&scene=LIVE&theme=phosphor`

**Transparent overlay** — turn on `VIEW ▸ TRANSPARENT BACKGROUND` and set the backdrop
to `NONE`. OBS keeps the alpha channel, so your game shows through and only the frame
composites on top.

**Window capture** — open the page in a browser, press `F5` then `F11`, and capture that
window. Simplest route if you want camera permissions to behave like a normal tab.

---

## Window types

**Sources** — Camera · Screen Capture · Image/GIF · Video Loop · Web Embed
**Text** — Text Panel · ANSI Banner · News Ticker · Sticky Note · Console Log
**Readouts** — Clock/Timer · Hit Counter · Goal Bar · Now Playing · Spectrum · System Monitor
**Live data** — Chat · Alert Box · Live Captions · Poll/Tally
**Atmosphere** — FX Panel
**Web 1.0** — Link List · Decoration · Assistant

A few worth calling out:

- **Camera** — nine period-correct filters including VGA 16-colour Bayer dithering, CGA
  4-colour, 1-bit threshold and live ASCII conversion, plus a chroma key. Two windows
  pointed at the same physical camera share one stream, so a facecam and a main shot
  cost one device.
- **Chat** — joins Twitch's anonymous IRC gateway. No login, no API key, read only. Set
  the channel and it connects. A demo generator invents traffic so you can build the
  layout before you go live.
- **Live Captions** — speech recognition from the microphone, rendered as broadcast
  subtitles. Chrome and Edge only.
- **ANSI Banner** — rasterises type and re-emits it as CP437 shade characters, drawn as
  real 2×2 dither cells.
- **FX Panel** — fifteen demoscene routines (plasma, DOOM fire, starfield, tunnel,
  copper bars, Bayer dither, digital rain, 3D pipes, bouncing logo…), each rendered into
  a chunky low-res buffer and blitted up with smoothing off, like a mode 13h framebuffer.
- **System Monitor** — FPS, frame time, heap and object count are measured. CPU, NET and
  DISK are decorative gauges; they look the part, they are not telemetry.

## Scenes

Tabs under the stage. Each holds its own layout and backdrop — `STARTING SOON`, `LIVE`,
`BRB` and `OUTRO` ship as examples. `F6` cycles, `CTRL+1…9` jumps. Switching plays a
transition: CRT power cycle, Bayer dissolve, venetian blinds, VHS glitch, fade or hard cut.

A window marked **GLOBAL** in the Layout tab appears in every scene — useful for a
watermark or a permanent ticker.

## Themes

Twelve palettes: Borland Blue, Norton Commander, Amber Mono, Green Phosphor, IBM CGA,
EGA Arcade, Windows 98, **Hot Dog Stand**, GeoCities 96, Vaporwave, Matrix, Blue Screen.
`F9` cycles them. Nine window frames, from DOS double-line box drawing to Windows 95
bevels to bare.

Every colour in the default palettes is one of the sixteen IBM VGA hardware colours.

## Saving

- **Autosave** to local storage on every change.
- **Eight named slots** (`F2` / `F3`).
- **Export / import as text** (`CTRL+E` / `CTRL+I`) — copy the block, paste it on another
  machine. Optionally embeds your uploaded media.

Images and clips go into an IndexedDB vault and are referenced by id, so a config stays a
few kilobytes. `FILE ▸ MEDIA VAULT` lists what's stored and flags orphans.

---

## Keyboard

| Key | Action |
|---|---|
| `F1` | Help |
| `F2` / `F3` | Save / load a slot |
| `F4` | Hide both side panels |
| `F5` | Broadcast mode |
| `F6` | Next scene (`SHIFT` for previous) |
| `F7` / `F8` | Grid / snap |
| `F9` | Cycle theme |
| `F11` / `F12` | Full screen / CRT tube |
| `CTRL+1…9` | Jump to a scene |
| `CTRL+Z` / `CTRL+Y` | Undo / redo |
| `CTRL+D` | Duplicate selection |
| `CTRL+ALT+A` | Fire a test alert |
| `DELETE` | Delete selection |
| `TAB` | Select next window |
| Arrows | Nudge by the grid (`CTRL` for one pixel) |
| `SHIFT`+Arrows | Resize by the grid |
| `[` / `]` | Send back / bring to front |
| `SHIFT` while resizing | Keep aspect ratio |
| `ALT` while dragging | Ignore all snapping |
| `ESC` | Deselect, or leave broadcast mode |

## Scripting hook

The page exposes `window.RETROCAST` for bots, OBS scripts and the console:

```js
RETROCAST.alert({ type: "sub", name: "pixelgoblin", msg: "6 months!" });
RETROCAST.scene("BRB");            // by name or index
RETROCAST.broadcast(true);
RETROCAST.theme("amber");
RETROCAST.set("goal", "cur", 812); // set a prop on every window of a type
RETROCAST.chat("sysop_dave", "hello", "#FF55FF");
RETROCAST.export();                // config as a JSON string
```

Alert types: `follow`, `sub`, `raid`, `tip`, `bits`, `host`, `custom`.

---

## Notes

- **Nothing leaves the browser.** Layouts sit in local storage, media in IndexedDB, and
  camera frames are processed on a canvas in the page. There is no backend.
- **Twitch chat and live captions need a direct connection**, so they work in the
  standalone file and in OBS but not inside an embedded preview, which blocks WebSockets
  and camera access. The page says so rather than failing silently.
- **Performance.** One camera at 720p with a dither filter costs about what one demoscene
  effect costs. If OBS drops frames, lower `PIXEL RES` on camera windows first, then
  reduce the number of animated panels. The FPS counter is in the stage title bar.
- **Fonts are embedded** as base64 latin subsets, so the file renders correctly with no
  network at all. VT323, Press Start 2P, Silkscreen and Comic Neue are all SIL Open Font
  License 1.1.

## Files

| File | |
|---|---|
| `index.html` | The entire application |
| `DESIGN.md` | Concept exploration, aesthetic sources, architecture, ideas cut |
