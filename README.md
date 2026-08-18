# Rubric

Verbatim scripture memorisation, built for thirty-year retention rather than
next month. Offline, no account, no server, no tracking. Everything stays on
your device.

Design rationale: `MEMORY-DESIGN.md`.

## Deploy to GitHub Pages

1. Create a repo, drop these five files in the root:
   `index.html` · `sw.js` · `manifest.webmanifest` · `icon-192.png` · `icon-512.png`
2. Settings → Pages → Source: *Deploy from a branch* → `main` / `root`
3. Open the URL on Android → menu → **Add to Home screen**

It installs as a standalone app with its own icon and works fully offline
after the first load. iOS: Share → Add to Home Screen.

## Getting your verses in

Paste from anywhere — the reference is detected automatically. Type them in
as you go rather than bulk-importing: producing the text yourself lays down a
stronger trace than importing ever would, and you'll only ever add a few
hundred.

## What it does that other apps don't

- **Nothing is ever finished.** No "mastered" button. Dropping a verse once
  you've got it right is how people lose what they memorised twenty years ago.
- **Cold random-start drills.** Dropped in mid-verse. If you can only start
  from word one, you have one way in.
- **Automatic word-level scoring.** Self-grading is where verbatim memory
  quietly dies — you accept a paraphrase as close enough and the wording
  erodes for years without registering as a failure.
- **Calibration.** It asks whether you'll get a verse, then shows you how far
  your confidence runs ahead of your memory.
- **A hard cap of two new verses a day**, which it will not lift.
- **Listener mode.** Hand someone your phone; they tap what you miss.

## Back up

Verses tab → Export. This is the only copy. Storage is IndexedDB, which
survives normal use but not "clear site data".

## Editing it

Everything is in `index.html` — no build step, no dependencies. The scheduler
is FSRS (v4.5 weights, `W[]` near the top). Target retention is 0.90 by
default; raising it to 0.95 roughly doubles your daily reviews and 0.99
multiplies them by about eleven.
