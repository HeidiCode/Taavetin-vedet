# Mökin vedet — project guide

A small, self-contained Finnish web app: a step-by-step guide for opening and
closing a cottage's water system across the seasons. Single page, no framework,
no build step.

- **Live site:** https://heidicode.github.io/Mokin-vedet/
- **Repo:** https://github.com/HeidiCode/Mokin-vedet (public)

## Layout

| Path | What it is |
|---|---|
| `index.html` | The whole app: static markup + inline JS (data, renderer, theme switcher) |
| `themes/original.css` | "Minimalismi" theme — soft, Manrope, teal |
| `themes/neo-brutalist.css` | "Neobrutalismi" theme — yellow ground, black borders, hard shadows, JetBrains Mono |
| `images/*.jpg` | Optimized task photos (committed) |
| `images/*.png` | Original full-size photos — kept locally, **gitignored** |

## How the app is structured (all inside `index.html`)

- `const processes = { ... }` — the content. Each process (e.g. `opening`) has
  `groups`, each group has `tasks`. A task has `title`, `detail`, `tools`,
  `warning`, and `photos: [...]`.
- `const PHOTOS = { p01: [...paths], ... }` — maps a photo **key** to one or
  more image paths. A key mapping to several paths renders as a **gallery**.
  Tasks reference photos by key, so one photo can appear in many tasks and you
  only edit the path once.
- The renderer resolves each ref with `PHOTOS[ref] || ref` (so a raw path or URL
  also works) and flattens arrays into the thumbnail grid + fullscreen lightbox.

## Step flow / progress

A process is a **linear sequence** (per the Neo-Brutalist design system). State is
a single "steps completed" count per process, held **in memory only**
(`progressState`, `{ [processId]: <doneCount> }`) — intentionally not persisted, so
it clears whenever the app is closed/reloaded; there is no manual reset. Each step's
status is derived from its global position `num` (1-based, across groups) vs `done`:

- `num-1 < done` → **done** (filled badge + check + `[Valmis]`, body shown; tap the
  head to rewind to that step)
- `num-1 === done` → **current** (accent badge + `[Kesken]`, body shown, carries the
  full-width **Merkitse valmiiksi** button)
- `num-1 > done` → **locked** (dimmed badge + `[Lukittu]`, **title only, no body**)

You advance one step at a time with *Merkitse valmiiksi* (`stepBy(+1)`); to go back,
tap any done step to rewind to it (`setDoneAndRender`). Marking the **last** step done
opens the completion modal (`#complete-modal`, "Kaikki vaiheet valmiit" + a check
button); closing it (`closeCompleteModal`) **resets the process to 0**. Home cards
show each process's `done/total`. The progress bar is **sticky** (`position: sticky;
top: 0`) with a full-bleed page-colored background, so it stays pinned as the step
list scrolls. Structure is shared HTML/JS in `index.html`; each theme styles it
(`.progress*`, `.task-badge`, `.card-task__tag`, `.card-task__head/__body`,
`.step-actions`, `.btn-complete`, `.modal*`, `.is-done`/`.is-current`/`.is-locked`).
There is **no manual accordion** — body visibility follows status.

## v2 — `v2/` (branch `v2-metsa`)

A second version of the app living in its own folder, so the v1 link keeps
working unchanged. GitHub Pages serves `main` at the repo root, so once merged
v2 is reachable at `https://heidicode.github.io/Mokin-vedet/v2/` while
`https://heidicode.github.io/Mokin-vedet/` stays exactly as it is.

| Path | What it is |
|---|---|
| `v2/index.html` | Same app, single theme, no switcher |
| `v2/themes/metsa.css` | The whole v2 look. All tunables are tokens in `:root` |

- **Images are shared, not copied** — `v2/index.html` references `../images/`.
  Add a photo once, at the repo root, and both versions see it.
- **One theme, no switcher.** The switcher markup, its styles and its script are
  gone from `v2/index.html`; the reduced-motion guard stays.

### What v2 changes, and why

Grew out of user testing on v1. Five findings, and the answer to each:

1. *Photos did not look openable* — each thumbnail carries an **AVAA** bar,
   drawn with `.attachment-thumb::after`, so the markup is unchanged and the
   button keeps its `aria-label="Avaa kuva N"`.
2. *Minimalismi's hierarchy was too flat* — phase heading 25px (was 18px), card
   titles 18px/600 (was 16px/normal).
3. *Neobrutalismi's red read as errors* — the alarm red is gone. One muted
   accent, and Huomio is a plain note whose label rides the top border as a tab.
4. *Type was too small* — body 17px with unitless line-heights, so it scales.
5. *The modal's icon button was not understood* — it now reads
   "Selvä, sulje ohje", with a line above saying the ohje returns to the start.
   That was always the behaviour; it just was not stated.

Two knock-on markup changes in `v2/index.html`: the **Valmis / Kesken** words are
`.sr-only` (the badge and the open body already carry the state, but screen
readers still need it), and a **locked** step shows a padlock (`icons.lock`)
instead of the word "Lukittu".

### Accessibility

Same WCAG 2.1 AA target as v1, and the same trap: never dim with `opacity`.
Colour pairs were checked against the tokens — every text pair passes, the
weakest being the small uppercase labels on the Valmis card at 4.6:1. The locked
card's dashed border is the one boundary that has to stay above 3:1, since it is
the only thing outlining that card. **Re-run axe-core against `v2/` after any
retint** — the token-level check does not cover focus order, names or the
lightbox.

## Themes (CSS Zen Garden model)

Same HTML, swappable stylesheet. The fixed switcher at the bottom rewrites the
`href` of `#theme-stylesheet`; the choice is saved in `localStorage` under
`tv-theme`. To add a theme: add a `themes/<name>.css`, register it in the
`THEMES` map and add a switcher button. The switcher's **own** styling lives in a
`<style>` block in `index.html`'s `<head>` (a soft light "segmented pill",
deliberately theme-independent so it looks the same under either stylesheet) — not
in the theme CSS files.

## Changing photos

1. Put the new image in `images/`. If it's a big PNG, make an optimized JPEG:
   `sips -s format jpeg -s formatOptions 70 -Z 1600 in.png --out in.jpg`
2. Point the relevant key in `PHOTOS` (around line ~150 of `index.html`) at the
   file. Keep the `images/*.png` originals — they're gitignored on purpose.
3. Add a Finnish description for the new file to the `ALT` map (keyed by image
   path, just below `PHOTOS`) — it's the image's alt text (accessibility).

## Accessibility

Targets **WCAG 2.1 AA**; both themes pass **axe-core** (contrast, names,
landmarks, headings) across the home, project, and completion-modal views. Keep it
that way — the easy regressions:

- **Contrast:** never dim things with `opacity` (it drops text below 4.5:1). Locked
  steps use explicit muted colors instead — `#31769b` on the Minimalismi card,
  `#666666` on the Neobrutalismi `#f9ecb8` locked card. The neo tokens are tuned
  for contrast: ground `#f7e186`, accent red `#c21925` (keeps red-on-yellow labels
  ≥4.5:1). Re-check if you retint.
- **Photos:** each thumbnail button has a numbered `aria-label` ("Avaa kuva N") and
  its image an alt from the `ALT` map — add an `ALT` entry with every new image.
- **Overlays:** `openCompleteModal` moves focus to the close button and traps Tab
  there; `closeCompleteModal` returns focus to the step list. The lightbox does the
  same — `openLightbox` remembers what opened it (`lightboxOpener`), focuses the
  close button, and `closeLightbox` hands focus back to that thumbnail. Both carry
  `role="dialog"` + `aria-modal`. Preserve if you touch them.
- **Motion:** transitions and the JS smooth-scroll honour `prefers-reduced-motion`
  (media query in the `<head>` + `matchMedia` guard in `scrollToCurrent`).
- **Structure:** `<html lang="fi">`, page title is the `<h1>`, group labels are
  `<h2>`, the theme switcher is a labeled landmark (`role="region"`). The viewport
  meta must **not** set `maximum-scale` (that blocks pinch-zoom).

Re-audit (no build): serve `axe.min.js` locally (or paste it into the console),
then `axe.run(document).then(r => console.log(r.violations))`. Check **both**
themes (swap the `#theme-stylesheet` href) and the project + modal views, not just
home.

## Preview locally

```bash
python3 -m http.server 8731   # then open http://localhost:8731/
```

## Deploy

GitHub Pages serves `main` at the repo root. **Any push to `main` auto-deploys**
(usually live within ~1 min). There is no separate build.

## Working notes / gotchas

- `index.html` embeds base64-free but is still one file; when it was photo-heavy
  it was too large to read line-by-line. Use `awk`/ranged reads for big files.
- The repo lives in **iCloud Drive**. Renaming the folder can make macOS briefly
  revoke the terminal's access to the whole iCloud Documents area ("Operation
  not permitted"). Fix: reopen the folder in Finder or restart the terminal app.
- When find/replacing text with non-ASCII (e.g. `ö`), use Python, not a `perl`
  one-liner — the latter mangled `ö` into `Ã¶` here. Verify with `grep -c "Ã"`.
