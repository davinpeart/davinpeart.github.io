# davinpeart.github.io

Source for <https://davinpeart.github.io>. Hand-written static HTML and one
stylesheet. No build step, no dependencies, no generator — edit a file, commit,
push, and GitHub Pages serves it.

To preview, open the `.html` file in a browser directly. There is no dev server
and none is needed.

## Layout

| | |
| --- | --- |
| `index.html` | Selected work, position, publications, contact |
| `count-differences.html` | Skellam-normal case study. Code lives at [davinpeart/skellam-normal](https://github.com/davinpeart/skellam-normal) |
| `model-documentation.html` | Seven generative model diagrams and a supplement audit page |
| `figures.html` | Scientific illustration |
| `cv.html` | Full CV |
| `404.html` | Custom not-found page |
| `assets/style.css` | Every design token and rule, one file |
| `figures/` | Images served by the pages |
| `figures/originals/` | Pre-recolour source images, kept for reference |

## The `.gitignore` is a whitelist — read this before adding a file

`.gitignore` ignores **everything** (`*`) and then re-admits named files. This
keeps the repo to exactly what the site serves, but it has a sharp edge:

> **A new file will be silently ignored.** `git add` will appear to succeed and
> the file will never ship. The page will 404 in production while looking
> correct locally.

To add a page or an image, add a matching `!` line to `.gitignore` first, then
`git add`. Confirm with:

```
git check-ignore -v figures/new-thing.jpg   # prints a rule if it is being ignored
git status --ignored                        # lists everything git cannot see
```

Roughly twenty files currently sit in the working tree unseen by git, most of
them rendered figures held back for a later release. That is intended — but it
means "the file is right there" is never evidence that it is tracked.

Note that `assets/card.png` needed an explicit entry because `og:image`
references it by absolute URL, so no relative path in the HTML points at it.

## Figures

Settled treatment, applied to everything currently on the site:

- Cropped to the ink, not to the original canvas.
- Illustrations saved as JPEG at quality 90 with **4:4:4** chroma subsampling.
  Line art and thin strokes smear badly under the default 4:2:0.
- Sized at roughly **2×** their largest display width, for retina screens.
- `loading="lazy"` on every image below the first one on a page.
- Wide figures use `class="figure-wide"`, which breaks out of the 760px reading
  column to 1240px and scrolls horizontally on narrow screens.

Recolouring rule, learned the hard way: it works when a colour is **categorical**
(a data series, a highlighted structure, a density scale) and fails when it is
**naturalistic**. Anatomical figures keep their original flesh tones; only the
blues were shifted to slate across the set.

## Design

All tokens are CSS custom properties at the top of `assets/style.css`. The
governing rule is **cool is data, warm is interface** — the single warm mark
(`--mark`) is used for chrome and links only, never inside a figure.

Two registers share one token set:

- `.doc` (760px, 66ch measure) for reading pages — the vignette register.
- `.plates` (1080px) for `figures.html`, where illustrations would be wasted
  inside a reading column.

Theming is three-state: `:root` defines the full light palette, a
`prefers-color-scheme: dark` block guarded by `:not([data-theme="light"])`
redefines tokens for system dark, and `[data-theme="dark"]` lets an explicit
choice win. Never give a colour its only definition inside a media block.
