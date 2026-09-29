---
name: hero-scroll-reveal
description: Apply the mobile scroll-reveal hero to an Open House landing page — the campus photo is pinned behind the hero copy as a faded background, the copy scrolls away to reveal the building, and the pin stops with the event date just above the roof. Use when asked to "apply the scroll reveal / reveal hero / BINUS hero effect" to a page folder in 01. Openhouse (e.g. `/hero-scroll-reveal itb-data-driven`).
---

# Hero scroll reveal (mobile/tablet ≤ 800px)

Reference implementation: `binus-applied-ai/` (search `styles.css` for "Scroll reveal"). Desktop is never changed — everything lives in `@media (max-width: 800px)`.

## What it does
1. Above the fold: hero copy (badge, H1, lead, CTA, event card) sits on a white veil; the campus photo shows faintly through the lower part.
2. On scroll: the copy and its veil move up, the photo stays pinned under the nav, the building is uncovered at full colour.
3. The pin ends with the event date just above the roof; then the photo, the optional strip below it (tools strip), and the collaboration strip scroll on normally.

No JS — `position: sticky` inside a one-cell grid.

## Preconditions — check before editing
- Page uses the `ddm-` layout from `itb-data-driven` / `binus-applied-ai`: `.ddm-cover > .ddm-cover__inner > .ddm-cover__copy + .ddm-cover__visual > img.ddm-cover__bldg`.
- The photo has a **transparent sky** (RGBA PNG/WebP). Check: `python3 -c "from PIL import Image; im=Image.open(P); print(im.mode, im.getpixel((5,5)))"` — alpha 0 at the top corners. If the sky is opaque, the building won't blend into the gradient; ask the user for a cut-out or accept a hard top edge.
- `git status` on the target folder is clean, or the user has confirmed you may edit on top of uncommitted work — another session may be editing the page.

## Steps

### 1. HTML — anything that sits *on* the photo must be a direct child of `.ddm-cover__inner`
Absolute children of the pinned photo would ride along with the pin and show through the veil over the copy. Move such blocks (e.g. `.ddm-tools`) out of `.ddm-cover__visual` so they become siblings after it. Pages with nothing on the photo skip this.

### 2. Desktop grid — place items explicitly
Once an extra item is in the grid, auto-placement pushes the photo to row 2. Add at base level (outside media queries):
```css
.ddm-cover__copy { grid-area: 1 / 1; }
.ddm-cover__visual { grid-area: 1 / 2; }
/* only if an item sits on the photo on desktop: */
.ddm-tools { position: relative; z-index: 3; grid-area: 1 / 2; align-self: end; justify-self: start;
  margin: 0 0 clamp(1rem, 2.4vw, 2rem) clamp(1rem, 2.4vw, 2rem); }
```
Replace any `position: absolute; left/bottom` on that item with the grid placement + margins above.

### 3. Mobile CSS — append near the end of `styles.css`, before `prefers-reduced-motion`
```css
@media (max-width: 800px) {
  /* Scroll reveal: campus pinned behind the hero copy; the copy's white veil scrolls away
     and uncovers the building. --tools-h = height of any grid row below the photo (the pin
     runs to the end of the grid, so subtract it); use 0px if there is none. */
  .ddm-cover { --nav-h: 76px; --stage: calc(100svh - var(--nav-h)); --tools-h: 5.4rem; --bldg-h: 76%; }
  .ddm-cover__inner { display: grid; grid-template-columns: 1fr; }
  .ddm-cover__visual, .ddm-cover__copy { grid-area: 1 / 1; }
  .ddm-cover__visual {
    position: sticky; top: var(--nav-h); z-index: 0; align-self: start;
    height: var(--stage); aspect-ratio: auto; min-height: 0; overflow: hidden;
    background: linear-gradient(180deg, SKY_TOP 0%, SKY_BOTTOM 100%);
  }
  .ddm-cover__visual::before { display: none; }
  .ddm-cover__bldg { top: auto; bottom: 0; height: var(--bldg-h); object-position: CROP_X bottom; }
  .ddm-cover__copy {
    z-index: 1; padding: 2.75rem 0 4.5rem;
    margin-bottom: calc(var(--stage) * .76 - 3rem - var(--tools-h)); /* pin ends with the date just above the roof */
    background: linear-gradient(180deg, #fff 0%, rgba(255,255,255,.93) 45%, rgba(255,255,255,.84) 88%, rgba(255,255,255,0) 100%);
    margin-inline: calc(-1 * var(--side)); padding-inline: var(--side); /* full-bleed veil */
  }
  /* only if a block was moved out of the photo in step 1: */
  .ddm-tools {
    grid-area: 2 / 1; justify-self: stretch; margin: 0 calc(-1 * var(--side));
    padding: 1rem var(--side) 1.15rem; background: #fff;
    border-left: 0; border-top: 4px solid ACCENT; box-shadow: none;
  }
}
@media (max-width: 560px) {
  .ddm-cover { --nav-h: 66px; }
  .ddm-cover__copy { padding-block: 2.5rem 4.5rem; }
}
```
Place this **after** any existing `max-width: 800px` / `560px` rules that touch `.ddm-cover__*`, so it wins on source order. Remove older mobile rules that set the photo's `aspect-ratio` or make the tools card `position: static` — they fight the pin.

### 4. Fill the placeholders per page
| Placeholder | How to choose | BINUS value |
|---|---|---|
| `SKY_TOP` / `SKY_BOTTOM` | Page's palette; pale → sky tone so the transparent sky blends | `#e2f3fc` / `var(--sky)` (#afd7f4) |
| `ACCENT` | Page's accent colour for the strip's top rule | `var(--binus)` |
| `CROP_X` | See below | `58%` |
| `--tools-h` | Measured height of the row under the photo (`el.offsetHeight`), in rem | `5.4rem` (86px) |
| `--nav-h` | Actual nav height at each breakpoint | 76px / 66px |

Keep `--bldg-h` at 76% and the `.76` in the margin **in sync** — the margin formula assumes the building occupies the bottom 76% of the stage. If you change one, change both.

**Choosing `CROP_X`:** the photo is scaled to `stage × 0.76` tall, so it renders `imgW × (stage×0.76 / imgH)` wide on a ~375px screen. Pick the x-position that centres the building/signage; if the sign is wider than the viewport, prefer the crop the user likes over fitting every letter (the BINUS user preferred 58% with "UNIVERSIT…" slightly cut over 67% with the full sign).

## Verify (preview server `openhouse-static`, port 4173)
Run at 375×812 and one other phone size (e.g. 383×750). A pane narrower than the requested width silently clears the emulation — re-check `innerWidth` in each JS call.
1. **Stop point** — scroll to the end of the pin and measure:
   ```js
   const c=document.querySelector('.ddm-cover'), t=document.querySelector('.ddm-tools');
   scrollTo({top: c.offsetTop + c.offsetHeight - (t?.offsetHeight||0) - innerHeight, behavior:'instant'});
   ```
   Screenshot: the event card (date · Zoom) sits just above the roof, no large empty sky band, no text over the building. `getBoundingClientRect()` on the `<img>` includes the transparent sky, so judge the roof line from the screenshot, not the img box.
2. **ATF** — at scroll 0 the photo is visible only faintly behind the CTA/event card; copy is readable.
3. **After the pin** — photo, then the tools strip, then the collaboration strip scroll up in order.
4. `document.documentElement.scrollWidth === innerWidth` (no horizontal scroll).
5. **Desktop** unchanged at 1280×800: photo in column 2, tools card bottom-left of the photo.
6. Reset the viewport to `desktop` when done.

## Tuning the user may ask for
- "Too much empty sky above the building" → raise `--bldg-h` and the `.76` together (not past ~0.8, or the sign crops hard).
- "Scroll should stop earlier/later" → adjust the `- 3rem` term (bigger = stops earlier, date higher above the roof).
- "Background too visible/too faint on ATF" → the veil stops at `.93` / `.84`; raise for more readable text, lower for a stronger image.
