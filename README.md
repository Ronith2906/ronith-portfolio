# Portfolio — Sylvester Ronith Reagan

Single file. `index.html` holds all HTML, CSS and JS. No build step. Push to the
`master` branch of `ronith-portfolio` and GitHub Pages serves it as-is.

```
index.html
assets/
  hero.webm                 1.3MB   VP9 + Opus
  hero.mp4                  836KB   H.264 + AAC
  hero-poster.jpg            52KB
  Sylvester_Ronith_Reagan_Resume.pdf
```

## The hero video is done

Take 1 of the five (`IMG_2381`) was selected on measurement: sharpest of the set,
longest usable duration, and the only take with hands on the keyboard rather than
propped under the chin. Processing applied:

- **Mirrored horizontally.** You sat left of frame, but the video sits right of the
  hero behind a gradient that dissolves its left edge into red. Unmirrored, your face
  landed inside that fade. Now the laptop dissolves and your face stays clear.
- **Cropped** to 864x1440 from the rotated 1080x1920 source, trimming dead ceiling.
- **Desaturated with lifted contrast**, so `mix-blend-mode: multiply` over the red
  panel produces a clean duotone rather than washed-out pink.
- **Faded to white at both ends.** No natural loop point existed - posture drifts
  throughout, and every candidate cut visibly jumped. Under multiply, white resolves
  to pure red, so you dissolve into the panel and the seam disappears. Lip sync intact.
- **Audio normalised to -16 LUFS.** The source sat near -29, far too quiet to hear.

To swap in a new take later, keep the filenames and rerun:

```bash
SRC=raw.mov
VF="hflip,crop=1080:1800:0:110,scale=864:1440,setsar=1,hue=s=0,eq=contrast=1.22:brightness=-0.07:gamma=0.92,fade=t=in:st=0:d=0.4:color=white,fade=t=out:st=14.9:d=0.4:color=white"
AF="loudnorm=I=-16:TP=-1.5:LRA=11,afade=t=in:st=0:d=0.35,afade=t=out:st=14.9:d=0.4"

ffmpeg -ss 0.47 -t 15.3 -i $SRC -vf "$VF" -af "$AF" \
  -c:v libx264 -crf 27 -pix_fmt yuv420p -c:a aac -b:a 96k \
  -movflags +faststart assets/hero.mp4
ffmpeg -i assets/hero.mp4 -c:v libvpx-vp9 -crf 36 -b:v 0 -row-mt 1 \
  -c:a libopus -b:a 96k assets/hero.webm
ffmpeg -ss 3 -i assets/hero.mp4 -frames:v 1 -q:v 4 assets/hero-poster.jpg
```

## One thing left before launch

**The contact form.** GitHub Pages is static and cannot process a POST. Create a free
form at [formspree.io](https://formspree.io), then replace `YOUR_FORM_ID` in the
`<form action>` near the bottom of `index.html`. Until you do, the form displays
"Form endpoint not configured yet." rather than failing silently. A honeypot field is
already wired for spam.

## Design system

```
--ink    #0A0A0B   page background
--paper  #F5F3F0   text on dark
--signal #E01B24   the red - CTAs, accents, dividers only
--muted  #8A8A8F   secondary text
--card   #141416   raised surface
```

Type: **Archivo** variable (the width axis drives the oversized wordmarks),
**Space Grotesk** for body, **JetBrains Mono** for stats, tags and rail labels.

The signature element is the **execution trace rail** down the left edge - one node
per section, lighting red as you scroll, labels on hover. It reads as a LangGraph
agent trace, which is what you build, so the structural device encodes something true
rather than decorating. Below 900px it collapses into a top progress bar.

## Where to edit

| What | Find |
|---|---|
| Hero headline & blurb | `class="hero__name"` / `hero__blurb` |
| At-a-glance stats & links | `class="glance__panel"` |
| About copy | `id="about"` |
| Skill chips | `class="skillcol"` |
| Jobs | `class="tl__list"` |
| Projects | `id="projects"` |
| Credentials | `id="creds"` |
| Colours & type | `:root` at the top of `<style>` |

## Two implementation notes

**Do not add `z-index` to `.hero__media`.** It creates a stacking context that isolates
`mix-blend-mode: multiply` from the red behind it, and the duotone silently reverts to
flat grayscale. The container carries `background: var(--signal)` for the same reason.

**Video crop is anchored with `object-position: center 14%`**, not `bottom`. Bottom
anchoring crops the face out entirely on desktop.

## Quality floor in place

- Responsive to 375px
- `:focus-visible` rings on every interactive element
- `prefers-reduced-motion` disables reveals, smooth scroll and video autoplay
- Single `<h1>`, labelled buttons, `rel="noopener noreferrer"` on all external links
- `muted` + `playsinline` on the video, so autoplay is not blocked and iOS does not
  force fullscreen
- No localStorage, no framework, no icon package - inline SVG only

## Two hero looks — swap by editing three lines

`assets/` ships both treatments of the same take:

| Look | Files | Character |
|---|---|---|
| **Photographic** (default) | `hero.webm` / `hero.mp4` / `hero-poster.jpg` | Warm red duotone. Reads "real person, real desk." |
| **Screenprint** | `hero-avatar.webm` / `hero-avatar.mp4` / `hero-avatar-poster.jpg` | Four-tone posterize. Room erased to flat red. Reads "designed." |

To switch to the screenprint, change these three attributes in the `<video>` block:

```html
poster="assets/hero-avatar-poster.jpg"
<source src="assets/hero-avatar.webm" type="video/webm">
<source src="assets/hero-avatar.mp4"  type="video/mp4">
```

Nothing else changes — same dimensions, same duration, same crop anchor.

The screenprint works because you were dark against a bright wall: pushing highlights
to pure white erases the room, and under `multiply` white resolves to flat red. Tone
banding is quantised to four levels after temporal denoise, so it does not flicker
between frames the way naive posterisation does.
