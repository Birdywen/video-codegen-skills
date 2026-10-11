---
name: font-strategy
contains: deterministic-font-loading-and-cjk-self-hosting-for-render-handoff
description: Use when a code-generated video contains text, especially CJK / non-Latin / branded fonts, and the render happens on another machine. Fixes blank-at-frame-0 text and tofu boxes by loading a font BEFORE frame 0 via a render gate, and by self-hosting the font file so the render does not depend on network access. Triggers on "text is blank / missing", "Chinese shows boxes / tofu", "CJK font", "font not loading", "render on a machine without internet".
---

# Font Strategy — deterministic, offline-safe text

Two failures ruin code-generated video with text:

1. **Blank at frame 0** — the font resolves *after* the browser starts capturing frames.
2. **Tofu boxes (boxes)** — the chosen font lacks CJK/non-Latin glyphs.

Both are preventable. Never rely on a system font for a render handed to another machine.

## Rule 0 — gate the render on font load

Every engine can wait for fonts before frame 0:

- **Remotion**: `delayRender()` + `document.fonts.ready` (or a font loader that does it for you).
- **Canvas / Puppeteer**: `await document.fonts.ready` before the first capture.
- **Manim**: pass `font="Noto Sans CJK SC"` to `Text(...)` (and install the font).

If you don't gate, the first N frames render with a fallback and flash.

## Rule 1 — self-host, don't hotlink

The render machine may be offline. Ship the font file inside the project
(`public/fonts/`) and reference it locally. Never depend on a CDN at render time.

### Where to get CJK fonts (verified sources)

- **Noto Sans SC** (Simplified Chinese), regular/bold/black OTF:
  `https://github.com/notofonts/noto-cjk/raw/main/Sans/SubsetOTF/SC/NotoSansSC-Regular.otf`
  (also `-Bold.otf`, `-Black.otf`; subset `SC` = Simplified Chinese, ~8 MB each)
- Static-hinted TTF alternatives and other scripts live in the same `noto-cjk` repo
  and on Google Fonts. Pick the subset for your script: `SC`, `TC`, `JP`, `KR`.

> Verify the download with `file X.otf` -> "OpenType font data". A 404 saves an HTML page;
> that will silently fail to render.

## Rule 2 — Remotion local-font recipe (copy this)

Put files in `public/fonts/`, then:

```ts
// src/fonts.ts
import { loadFont } from '@remotion/fonts';
import { staticFile } from 'remotion';

export const FAMILY = 'NotoSansSC';

// loadFont wraps delayRender/continueRender internally — do this at module scope.
const reg   = loadFont({ family: FAMILY, url: staticFile('fonts/NotoSansSC-Regular.otf'), weight: '400' });
const bold  = loadFont({ family: FAMILY, url: staticFile('fonts/NotoSansSC-Bold.otf'),    weight: '700' });
const black = loadFont({ family: FAMILY, url: staticFile('fonts/NotoSansSC-Black.otf'),   weight: '900' });

export const ensureFonts = () => Promise.all([reg, bold, black]);
```

Pin `@remotion/fonts` to the same version as `remotion`.

```json
{ "dependencies": { "@remotion/fonts": "4.0.290", "remotion": "4.0.290" } }
```

Reference the family in styles: `fontFamily: 'NotoSansSC, sans-serif'`.

## Rule 3 — fallback chain

Always end the stack with a generic: `fontFamily: 'NotoSansSC, "PingFang SC", "Microsoft YaHei", sans-serif'`.
So even if the self-hosted file is missing, Latin text still renders (CJK may not — that's
exactly what you must test).

## Rule 4 — ligatures / numeral style

For a techy look, prefer tabular numerals so live HUD counters don't jitter width:
`fontVariantNumeric: 'tabular-nums'`. Disable problematic ligatures if a font has them.

## Troubleshooting
| Symptom | Cause | Fix |
|---|---|---|
| Text blank for first ~0.2s | no render gate | `delayRender` / `document.fonts.ready` |
| Chinese = boxes | Latin-only font | load a CJK subset (Noto Sans SC) |
| Bold looks like regular | only one weight loaded | load each weight separately with its `weight` |
| Works locally, blank remotely | CDN blocked offline | self-host the .otf in `public/fonts/` |
| Whole render hangs | a loaded URL 404s and never resolves | `file` the font; fix the path |

## Pre-delivery check
- [ ] Font file is inside the project (not a URL).
- [ ] Render is gated on font load.
- [ ] Every weight used is loaded.
- [ ] Fallback stack ends in a generic family.
- [ ] `file <font>` returned "OpenType font data".
