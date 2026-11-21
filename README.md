# Tune Tap Simulator

A fan-made, single-file rhythm game in the style of Twisted-Wonderland Rhythmic. The whole game is one `index.html`: no build step, no server, no dependencies. It ships **without songs**. You add your own `custom_song_*.zip` files and they are kept in your browser.

> Unofficial fan project. Not affiliated with Disney, Aniplex, Bushiroad or any other rights holder. All related names, characters, music and artwork belong to their respective owners.

**Contents**
[Run it](#run-it) · [How the sim works](#how-the-sim-works) · [Get songs](#get-songs-dlc) · [Custom song zip format](#custom-song-zip-format) · [Code tour](#code-tour) · [Hacking](#hacking-on-it)

---

## Run it

| Way | How |
| --- | --- |
| Open locally | Double-click `index.html`, or run `python3 -m http.server 8000` and open `http://localhost:8000` |
| GitHub Pages | Push the repo, then **Settings → Pages → Deploy from a branch → `main` / root** |
| Any static host | Upload `index.html` (about 13 MB) |

Works in current Chrome, Edge, Firefox and Safari, on desktop and phones. On a phone held upright the game rotates itself to landscape (switch off under Display settings).

## How the sim works

### Flow

1. **Title screen** → tap to enter.
2. **Song list** (the Rhythmic screen). Songs are grouped by chapter, which for DLC is the pack the song belongs to (for example "SIF Tournament Song Pack"), in the same order as the Downloader. The game finds the pack from the zip's file name, using the `PACKS` table in the code, so keep the original `custom_song_*.zip` names. Songs it does not recognise go into "Other songs".

**Bulk import:** you can also pick one pack zip that contains many `custom_song_*.zip` files (for example the zip the Downloader's "Download pack" button makes). The game opens it, imports every song inside and remembers them one by one. Pick a song and the right-hand panel shows the jacket, your high score and the difficulty buttons.
3. **Difficulty**: Easy, Normal, Hard, Expert, whichever the song's chart has.
4. **Note style**: Blue & Red, Extra, Green, or Purple (see below).
5. **START** for a real run, **DEMO** to watch the game play the chart itself.
6. After a 3-2-1 count-in the song plays and notes fall toward the hit line. The result screen shows your rank, accuracy, combo and a per-judgement count.

### Note styles

The chart itself is the same; the style changes how it maps onto the screen.

| Style | Lanes | Idea |
| --- | --- | --- |
| Blue & Red | 2 | Two rings, two sides. The classic one |
| Extra | 4 | Four lanes, each with its own tone |
| Green | 1 | A single lane. Easiest to read |
| Purple | free | Notes appear at their chart position; tap them where they land |

### Note types

Each note in a chart has a time, a position (1 to 9) and an effect code.

| Effect | Note |
| --- | --- |
| `1` | Tap |
| `3` / `13` | Hold: press on the note and keep holding for `effect_value` seconds |
| `11`, `12`, `14` | Flick / swipe |
| two notes within 10 ms | Treated as a simultaneous pair (shown linked) |

Positions 1 to 5 are the left side, 6 to 9 the right side.

### Judgement and score

You tap near the note's time. The game picks the closest unhit note in your lane that is within 0.18 s.

| Judgement | Window (±) | Base points | Accuracy weight |
| --- | --- | --- | --- |
| PERFECT | 0.07 s | 1000 | 1.0 |
| GREAT | 0.12 s | 700 | 0.7 |
| GOOD | 0.18 s | 400 | 0.4 |
| MISS | beyond | 0 | 0 |

Each PERFECT or GREAT adds a combo bonus of `min(combo, 50) × 10`. A GOOD or a MISS resets the combo. The maximum score of a chart is the sum of `1000 + min(i, 50) × 10` over all `i` notes, so a full-combo of PERFECTs is exactly 100%.

Rank is `score / max score`: **SS** at 95% or more, **S** 80%, **A** 60%, **B** 40%, otherwise **C**. Your best score per song, difficulty and note style is stored in `localStorage`.

### Settings

- **Note speed** 0.5× to 3× (how fast notes fall) and **Song speed** 0.5× to 1.5× (tempo of music and chart).
- **Auto play**, **offset** (to fix audio lag), separate BGM, tap-sound and effect volumes.
- **Landscape mode** and **fullscreen on start**.
- Per-song **Edit** (pencil on the jacket): rename, set a different MV, or play the original audio while the MV still plays.

## Get songs (DLC)

On first launch the list says **"Please add your DLC!"**.

1. Download `custom_song_*.zip` files. The companion repo **tunetap-dlc-downloader** has a website for this (coming soon). Until then, copy the zips from its `songs/` folder.
2. Tap **＋** in the song list (or the same button in the settings under *Add DLC songs*) and select one or many zips.
3. Each zip appears in the list under its artist. They are saved in your browser's IndexedDB, so they are still there next launch, and nothing is uploaded anywhere.
4. **Remove added** in the same dialog clears everything you added.

### Link the game to your DLC website

Open `index.html`, find this line near the top of the main script and set it:

```js
const DLC_SITE='';   // e.g. 'https://you.github.io/tunetap-dlc-downloader/'
```

Empty means the empty screen shows "DLC pack website: coming soon". Set means it shows a clickable link.

## Custom song zip format

A song is one zip with these files (names are fixed, no folders):

```
manifest.json     metadata and which difficulties exist
audio             the song (Ogg Vorbis / Opus, MP3, etc.; the browser decodes it)
jacket            cover art (PNG, JPEG or WebP)
chart_1.json      Easy
chart_2.json      Normal
chart_3.json      Hard
chart_4.json      Expert      (any chart may be missing)
```

`manifest.json` (only the fields the game reads are shown):

```json
{
  "format": 1,
  "name": "新時代",
  "name_en": "New Genesis",
  "artist": "Roselia",
  "artist_en": "Roselia",
  "bpm": 175,
  "levels": [
    { "level": 1, "level_number": 3 },
    { "level": 2, "level_number": 5 },
    { "level": 3, "level_number": 8 },
    { "level": 4, "level_number": 11 }
  ]
}
```

`level` is the difficulty slot (1 to 4) and `level_number` is the number shown as "Lv.". A chart file is a JSON array of notes:

```json
[
  { "timing_sec": 4.2857, "position": 3, "effect": 1, "effect_value": 0 },
  { "timing_sec": 6.0504, "position": 7, "effect": 3, "effect_value": 1.008 }
]
```

The second note is a hold: it starts at 6.05 s on position 7 and lasts about 1 s. Notes with a time before 0 or a position outside 1 to 9 are dropped, and a chart needs at least 5 notes.

### Make a zip yourself

```python
import json, zipfile

notes = [{"timing_sec": 1.0 + i * 0.5, "position": 3 + (i % 2) * 4, "effect": 1, "effect_value": 0} for i in range(40)]
manifest = {"format": 1, "name": "My Song", "name_en": "My Song", "artist": "Me", "artist_en": "Me",
            "bpm": 120, "levels": [{"level": 2, "level_number": 5}]}

with zipfile.ZipFile("custom_song_my_song.zip", "w") as z:
    z.writestr("manifest.json", json.dumps(manifest, ensure_ascii=False))
    z.write("my_song.ogg", "audio")
    z.write("cover.png", "jacket")
    z.writestr("chart_2.json", json.dumps(notes))
```

Import it with **＋**. If the zip has no `manifest.json` or no `audio`, the game says it could not read it.

## Code tour

Everything is in `index.html`, in this order:

1. **Markup and CSS**: screens (`#home`, `#menu`, `#end`, settings, pause) and the game canvas `#c`. Fonts and UI art are inlined as base64.
2. **Core loop**: constants `W_P`, `W_GR`, `W_GO` (judge windows), `APPR` (approach time), the `songs` array, then `loop()`, which draws every frame and calls `oaSync()` (see below).
3. **Chart handling**
   - `sifNotes(raw)` turns raw chart JSON into internal notes `{t, s, pos, d, f, ...}` (time, side, position, hold length, flick) and marks simultaneous pairs.
   - `lanify(notes, nl)` maps notes onto the lanes of the chosen note style and drops notes that would overlap.
4. **Judging**: `hit(side, pid, cx)` finds the nearest note in the lane; `award(side, d)` turns the time difference `d` into PERFECT/GREAT/GOOD, updates `score`, `combo` and `cnt`. `calcMax()` is the maximum score, `showEnd()` computes the rank and saves the high score.
5. **Clock**: `T()` is the single source of song time. For normal songs it is the video or audio element's `currentTime` minus `cur.vo`; in demo mode it is `performance.now()` based. Everything (notes, judging, effects) is drawn from `T()`.
6. **Songs**: `prep(s, d)` builds the playable note list for a song and difficulty; `launch()` starts a run; `renderList()` and `renderDetail()` draw the menu.
7. **DLC import**: `unzip()` is a small zip reader (it uses the browser's `DecompressionStream` for deflated entries), `addDlcZip(file)` reads `manifest.json`, `audio`, `jacket` and the charts and pushes a song object into `songs`. `dbo()` / `dbPut()` persist the original zips in IndexedDB (`tt_dlc`), and on startup every stored zip goes through `addDlcZip` again.
8. **Original audio with MV** (`oaUrl`, `oaP`, `oaSet`, `oaGo`, `oaSync`): a separate `Audio` element plays the original song while the MV video is muted, kept in sync with drift correction against `T()`.

Storage keys: `tt_set` (settings), `tt_hs_*` (high scores), `tt_oa` (original-audio choices), IndexedDB `tt_dlc` (your zips).

## Hacking on it

- **Add a judgement tier or change windows**: edit `W_P`, `W_GR`, `W_GO` and the `award()` points.
- **Change how fast notes fall**: `APPR` is the base approach time; the Note speed slider divides it.
- **New note style**: add a branch in `lanify()` and in the position helper used for drawing (`TX`).
- **Test headlessly** with Playwright:

```js
const { chromium } = require('playwright');
(async () => {
  const b = await chromium.launch();
  const p = await b.newPage({ viewport: { width: 1280, height: 640 } });
  p.on('pageerror', e => console.log('ERR', e.message));
  await p.goto('file://' + process.cwd() + '/index.html');
  await p.mouse.click(640, 320);                         // leave the title screen
  const zip = await p.$('input[type=file][accept*=zip]');
  await zip.setInputFiles('custom_song_example.zip');   // import a song
  await p.waitForTimeout(3000);
  await p.screenshot({ path: 'shot.png' });
  await b.close();
})();
```

## Push to GitHub

```bash
git init -b main
git add .
git commit -m "Tune Tap Simulator"
git remote add origin https://github.com/<you>/tunetap-simulator.git
git push -u origin main
```
