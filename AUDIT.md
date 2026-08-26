# Pre-launch audit — davinpeart.github.io

Read-only audit. No site file was modified. Findings verified by parsing every page,
resolving every link and image path, and rendering all six pages in a real browser at
1280px and 375px.

Note: `cv.html` was edited on disk partway through this audit (the named-students
supervision table was removed). All `cv.html` line numbers below refer to the current
122-line file.

---

## Must fix before launch

### 1. `portfolio_text.rtf` is tracked by git and will be published

`git ls-files` lists `portfolio_text.rtf`. On launch it is fetchable at
`https://davinpeart.github.io/portfolio_text.rtf`. It is the build-instruction document
written for whoever assembled the site, and it contains internal working notes including:

> "don't know the original citation, please find it"

> "This does not come close to conforming to BARG - don't know what else to cite, Stan docs?"

> "Go ahead and add whatever CV configuration you want to the CV hyperlink"

> "Must crop the caption out of the model audit figure"

A hiring manager who finds this reads the site's scaffolding and a candid note that one
page's central citation does not support the claim made from it.

**Fix:** `git rm --cached portfolio_text.rtf`, add it to `.gitignore`, and move the file
out of the repository directory.

### 2. `model-documentation.html` — the seven model diagrams render broken on every device

Lines 41–49 wrap each diagram in a bare `<div class="mount">` inside `.plate-grid`, with
no `<figure>` and no `.plate` ancestor. Every rule that would size and frame them is a
descendant selector requiring one of those ancestors:

- `figure img, figure svg { width:100%; height:auto }` (style.css:156)
- `.plate img { width:100%; height:auto }` (style.css:169)
- `figure .mount { background; border; padding:16px }` (style.css:157)
- `.plate .mount { … }` (style.css:165)

None of them match. Measured in the browser:

| | grid `.mount` (lines 42–48) | `figure .mount` (line 58, same page) |
|---|---|---|
| computed background | `rgba(0,0,0,0)` | `rgb(255,255,255)` |
| computed border-top | `0px` | `1px` |
| computed padding | `0px` | `16px` |
| image rendered width | **1371px** in a 342px cell | 678px in a 712px column |

Resulting page `scrollWidth`: **2025px at a 1280px viewport, 1387px at 375px**. The page
scrolls sideways everywhere, each diagram appears as a hugely magnified fragment, and the
diagrams lose their white mount, border and padding. No other page overflows at either
width. This is the flagship page of the site.

**Fix (either):** wrap each diagram in `<figure class="plate">` the way `figures.html`
does; or add to style.css:

```css
.plate-grid .mount{background:var(--plate);border:1px solid var(--line);border-radius:8px;padding:16px}
.plate-grid .mount img{display:block;width:100%;height:auto}
```

### 3. `index.html:50–51` — stray CV link inside the Figures entry title

```html
<span class="t"><a href="figures.html">Figures</a>
    <a href="cv.html">CV</a></span>
```

Both links sit inside the same `.t` title span, so the third item on the homepage renders
as a single 19px serif heading reading **"Figures CV"**, above a description about
illustration. Confirmed in the DOM. This is on the first page anyone sees.

**Fix:** delete line 51. (The RTF instruction "add whatever CV configuration you want to
the CV hyperlink" appears to have been half-applied here.)

### 4. `model-documentation.html:58` — alt text states the wrong subject count

Alt text says "twenty-seven per-subject model predictions". Panel g of
`figures/audit-acquisition.png` contains **22** panels: 1920, 1923, 1925, 1926, 1927,
1928, 1929, 1930, 1931, 1932, 1933, 1934, 1935, 1938, 2324, 2325, 2326, 2327, 2330, 2332,
2333, 2334.

**Fix:** "twenty-two". (The rest of that alt text is accurate — panel a is the
specification, b the prior predictive check, c–d posterior predictive checks, e–f
convergence diagnostics, h three Savage-Dickey tests.)

---

## Should fix

### 5. No `og:image` on any page, though the card exists and the reference block requires it

`grep og:image *.html` returns zero hits on all six pages. `assets/card.png` exists and is
exactly 1200×630 — the correct size. `assets/head.txt:15` specifies the tag as part of the
shared `<head>` pattern:

```html
<meta property="og:image" content="https://davinpeart.github.io/assets/card.png">
```

Four of five content pages declare `twitter:card` as `summary_large_image`, which without
an image renders as a bare text card. This is what a recruiter sees when the link is
pasted into Slack, LinkedIn or email.

**Fix:** add the tag after `og:url` on `index.html`, `count-differences.html`,
`model-documentation.html`, `figures.html`, `cv.html`.

### 6. `cv.html:15` — `twitter:card content="summary"` where every other page uses `summary_large_image`

`head.txt:16` specifies `summary_large_image`. **Fix:** match the other four pages.

### 7. `cv.html:79` — "Nine assignments" but the table below lists eight

The table at lines 80–87: PSYC\*3290 (F2023, F2024, F2025) = 3, PSYC\*3410 = 1,
PSYC\*2330DE = 1, PSYC\*1010 = 1, PSYC\*4750 = 1, NEUR\*2000 = 1. **Total 8.**

("eight consecutive semesters" is exactly right: F2022, W2023, F2023, W2024, F2024, W2025,
F2025, W2026.)

**Fix:** change to "Eight assignments", or add the missing row.

### 8. `cv.html:74` — "Nine conference presentations … including" then names eight

2 Canadian Neuroscience Meetings + 1 EBPS + 2 CCNP + 1 IBNS + 2 SONA = 8. "Including"
technically hedges it, but a CV reader counts.

**Fix:** name the ninth, or drop the number and let the list stand.

### 9. `cv.html:74` — ordinal ordering breaks mid-sentence

"the 19th and 18th" (descending), "the 45th and 44th" (descending), then "the 42nd and
43rd" (ascending). **Fix:** "the 43rd and 42nd".

### 10. `cv.html:74` — "International Behavioural Neuroscience Society" is misspelled

The society is the International **Behavioral** Neuroscience Society (IBNS); it is
US-based and its name uses the American spelling. The European **Behavioural**
Pharmacology Society in the same sentence is correctly British. Applying house style to a
proper noun is the kind of slip a reviewer in this field notices.

**Fix:** "Behavioral".

### 11. Two `<figure>` elements carry no `<figcaption>`

- `count-differences.html:104–106`
- `model-documentation.html:57–59`

All six figures on `figures.html` have one. The PPC comparison at
`count-differences.html:104` is the payoff figure of that page, and its only description
currently lives in alt text — invisible to sighted readers.

**Fix:** add figcaptions; or drop the `<figure>` wrapper where no caption is intended.

### 12. `count-differences.html:75–77` — the Stan snippet is not valid Stan as shown

```stan
real skellam_lpmf(int y, real mu, real phi, real log_bessel) {
```

This sits at top level, between `transformed data` and `model`. User-defined functions
must live inside a `functions { }` block, and that block must come first in the program.
The audience for this page is precisely the set of readers who will notice.

**Fix:** wrap it in `functions { … }`, or add one line noting the blocks are excerpts
shown out of program order.

### 13. `model-documentation.html:33` — the lede reads as scaffolding, not his voice

> "A model is not finished when it converges. Specification, priors, diagnostics,
> per-subject fit and the test belong in one artifact, and I build that artifact every time."

Three signals it is not his:

- An aphoristic opener followed by a claim about himself ("I build that artifact every
  time"). The stated voice has no self-promotion; elsewhere the site concedes before
  overturning — "There is a reason ANOVA is used for this task … But we can do better"
  (count-differences.html:40) — and uses first-person plural in technical passages.
- It is the only `.lede` on a content page. No other page has one.
- It corresponds to nothing in the source text this page was written from; that source
  says only "Should apply the one subtitle only rule here and on every page in the site."

Also "artifact" is the only American `-ifact` on an otherwise British-leaning site
(it also appears in the meta description at line 7 and og:description at line 13).

**Fix:** cut it, or replace with a concrete sentence in the register of the body text.

### 14. `model-documentation.html:39` — a claim the figures do not support

> "All seven below are drawn at the same scale, so differences in height track how many
> hierarchical levels each model carries."

All seven PNGs are identical 1371×2324 canvases. Once finding #2 is fixed and each renders
at `width:100%`, they will all be exactly the same height and there will be no height
differences to track. This sentence is also not in the source text for the page.

**Fix:** drop the sentence, or re-export the diagrams at their true relative heights if
the claim is meant literally.

### 15. `count-differences.html:114` — subject–verb disagreement

> "the abundance of zeros in the response variable **are** structural"

**Fix:** "is structural".

### 16. Savage-Dickey vs Greenhouse–Geisser — two dash conventions in one sentence

`cv.html:99` writes "Savage-Dickey Bayes factors" (hyphen) and "Greenhouse–Geisser
correction" (en dash). Both are two-person eponyms. The site's convention elsewhere is the
en dash: "Sprague–Dawley" (index.html:74, cv.html:64), "mean&ndash;variance"
(count-differences.html:46).

**Fix:** "Savage–Dickey" at `cv.html:99`, `model-documentation.html:55`, and
`model-documentation.html:58` (alt text).

### 17. Apostrophes mix straight and typographic

- Straight `'`: `cv.html:49` "Master's", `cv.html:55` "Dean's", `cv.html:63` and
  `index.html:73` "What's next"
- Typographic `&rsquo;`: `index.html:84` "site's", `model-documentation.html:70` "model's",
  `cv.html:94` "master's"

In Iowan Old Style the straight apostrophe is visibly wrong next to a curly one, and
`cv.html` now contains both.

**Fix:** use `&rsquo;` (or the literal ’) throughout.

### 18. `404.html` — nav and footer are not identical to the rest of the site

- Nav (line 13) has Work and Figures but **no CV link**. It is the only page where the nav
  differs.
- Footer (lines 23–27) has OSF, ORCiD, GitHub but **no Email**. Every other page has four.
- No skip link, and `<main class="doc">` (line 15) has no `id="main"`.

**Fix:** add `<a href="/cv.html">CV</a>` to the nav, the mailto to the footer, and the skip
link + `id="main"` for parity. Keep the root-absolute `/assets/…` paths — those are
correct, since a 404 can be served from any path depth.

### 19. `index.html:61` — "Position" section holds education history; `cv.html` calls it "Education"

Same three rows, two labels. The degree is also rendered two ways:

- `index.html:65` — "BSc (Hons, with Distinction)"
- `cv.html:40` — "BSc, Honours (with Distinction)"

**Fix:** rename the index section "Education" and pick one degree format.

### 20. `index.html:53` promises graphical abstracts; `figures.html` shows none

> "Signalling cascades, circuit anatomy, task schematics, graphical abstracts."

`figures.html` shows signalling cascades (Fig 1), circuit anatomy (Figs 2, 5), receptor
distribution (Fig 3), neurobiology (Fig 4) and a task timeline (Fig 6) — no graphical
abstract. `figures/graphical-abstract.png` is committed but unused.

**Fix:** add it to `figures.html`, or drop the phrase from the index blurb.

### 21. `count-differences.html:105` — alt text says "Normal" where the prose says "Gaussian"

The alt reads "Left, Normal: …" while lines 40, 93 and 108 all say "Gaussian". A reader on
alt text alone has to make the mapping unaided.

**Fix:** "Left, Normal (the Gaussian model): …", or relabel the figure panel.

### 22. `count-differences.html:64` — verify "18 evaluations per gradient instead of 415"

These numbers cannot be checked against anything in the repository, and they do not appear
in the source text the page was written from (which says only "once per gradient
evaluation using a cache-and-lookup approach"). 18 implies max|y| = 17; 415 implies
N = 415. Worth confirming against the actual fit before this goes into job applications.

### 23. `count-differences.html:105` — missing `loading="lazy"`

The only image on the page, sitting five sections and three code blocks below the fold.
The other two eager images on the site (`model-documentation.html:42`,
`figures.html:43`) are each the first image near the top of their page, so eager loading
there is a sensible LCP choice. This one is not a LCP candidate.

**Fix:** add `loading="lazy"`.

---

## Consider

### 24. Drafting scaffolding still ships in the CSS

`style.css:236–249` defines `.todo` and `.todo .lbl` under the comment *"delete the block
once written"*. No page uses either. Also unused: `pre .s` (:141), `.mono` (:130), `th`
(:180 — no page has a `<th>`), and `h2` (:106) / `.plate h2` (:171), dead by the
deliberate no-`<h2>` decision. Harmless, but the source is public.

### 25. "polarisation" is the site's only `-isation`

`figures.html:43` (alt text). Everything else is `-ization`: parameterization ×6,
generalization ×5, visualization ×2, catheterization ×2, plus ovariectomized, discretized,
optimize, generalized. Under Oxford spelling — which the site otherwise follows, British
`-our`/`-re` with `-ize` — this should be "polarization".

### 26. Do **not** "correct" "non-centred" at `count-differences.html:116`

The baked-in label inside `figures/audit-acquisition.png` reads "Non-Centred
Parameterization", so the prose and the figure already agree. Oxford spelling makes
`-centred` + `-ization` internally consistent. Leave it.

### 27. Decimal style

`count-differences.html:116` writes ".8 to .9"; `model-documentation.html:70` writes
"0.1%". Use leading zeros in both.

### 28. Entity vs literal punctuation is split by file

`count-differences.html` and `model-documentation.html` use `&mdash;`/`&ndash;`;
`index.html`, `cv.html` and `404.html` use literal — and –. `cv.html` is now mixed within
itself: lines 93–94 use `&ndash;`/`&rsquo;` after the recent edit, lines 38–51 use
literals. Renders identically — source tidiness only.

### 29. `table{display:block}` costs the table semantics

`style.css:178` sets `display:block; overflow-x:auto` on every `<table>`, which makes it
scrollable but also drops the implicit table role in several screen readers, and forces
the `thead,tbody,tr{width:100%}` patch at :179. Wrapping tables in a
`<div style="overflow-x:auto">` and leaving `display:table` keeps both. Minor here, since
the tables are layout-ish and header-less.

### 30. `figures.html` repeats `style="border:0"` five times

Lines 31, 35, 39, 59, 74, all defeating `section{border-bottom}`. One `.plates section{border:0}`
rule would replace them. Separately, line 35 is the only section on the site with no
`.eyebrow`.

### 31. The "seven models, each … tested" claim is only demonstrated once

`index.html:42` says "Seven models, each specified, prior-checked, diagnosed, predicted and
tested on a single page." The page shows seven specification diagrams but only one audit
composite. `figures/audit-reinstatement.png` is committed and unused — adding it would
make the claim visibly true for two of the seven.

### 32. 24 unused images are committed and publicly fetchable

`audit-reinstatement`, `behaviour`, `cfos-pca`, `correlation-matrices`,
`graphical-abstract`, `morphine-pc1/pc2/pca`, `network-full`, `network-schematic`,
`ovarian-cycles`, `priors`, `ribbon-cs/ds/precs`, `stockwars`, `subject-posteriors`, plus
all seven files in `figures/originals/`. They cost nothing, but each is reachable by URL.
Worth deciding deliberately about `stockwars.jpg` in particular.

### 33. No lateral link between the two work pages

`model-documentation.html` and `count-differences.html` are reachable only via the index.
The three-item nav is clearly deliberate and the pages *are* transitively reachable, but a
reader who finishes one has to go back to Work to find the other. One "next" line at the
foot of each would close the loop without touching the nav.

---

## Checked and clean

**Structure**

- All six pages are well-formed. Parsed with Python's `HTMLParser`: no unclosed tags, no
  mismatched nesting, no stray closing tags, no closings on void elements.
- Exactly one `<h1>` per page. Zero `<h2>` and zero `<h3>` site-wide. Sections carry
  `<p class="eyebrow">` only — matching the stated design decision.
- Figure numbering on `figures.html` runs 1–6, no gaps or repeats, and the
  `<figure class="plate">` / `<div class="mount">` / `<figcaption><b>Figure N.</b>` pattern
  is identical across all six.

**Navigation**

- Every internal `href` resolves to a file that exists. No dead internal links anywhere.
- `aria-current="page"` is present and on the correct link: `index.html` (Work),
  `figures.html` (Figures), `cv.html` (CV). Correctly absent from
  `count-differences.html` and `model-documentation.html`, which have no nav entry.
- The footer is byte-identical on index, count-differences, model-documentation, cv and
  figures (figures adds only `class="wide"`, as designed). Only `404.html` differs — see #18.
- The masthead nav block is byte-identical across all five content pages apart from the
  `aria-current` attribute and figures' `wide` class.

**Errors**

- Every `<img>` has non-empty, genuinely descriptive alt text. No exceptions.
- Every `<img src>` resolves to a file on disk. No broken paths.
- No unescaped `&`, `<` or `>` anywhere in prose or markup. All three `<pre>` blocks in
  `count-differences.html` are free of characters requiring escaping. Entities used
  (`&mdash; &ndash; &nbsp; &amp; &rsquo; &lambda; &sigma; &sup2; &#770;`) are all valid.
- Every CSS custom property referenced in CSS or HTML is defined. The eight unused ones
  (`--slate-1..6`, `--teal-5`, `--brick-5`) are documented in the file itself as
  "reference only; figures ship with these baked in".
- Every class used in HTML has a matching CSS rule.
- `<html lang="en">` on all six pages. Skip link present and pointing at a real `#main` on
  all five content pages.
- `.DS_Store` is gitignored and untracked.

**Rendering** (browser-verified at 1280px and 375px)

- Zero horizontal overflow on index, count-differences, figures, cv and 404 at both widths.
- `.figure-wide` behaves exactly as its comment claims: at 375px the page `scrollWidth`
  stays at 375 while the image overflows *inside* its own `overflow-x:auto` mount. Same for
  the `<pre>` blocks.
- `assets/card.png` is exactly 1200×630 — correct for an OG card once referenced (#5).
- The baked-in caption was successfully cropped out of `figures/audit-acquisition.png`.

**Cross-page facts**

- The publication list is **byte-for-byte identical** between `index.html:73–77` and
  `cv.html:63–67`: same author strings and order, years, article titles, journal names,
  volume/issue/page numbers, DOIs, and the same `^` shared-first-authorship footnote.
- Journal names are spelled out in full on both pages and never abbreviated. The American
  spellings ("Neuroscience and Biobehavioral Reviews", "Hormones and Behavior") are
  correct as the journals' actual titles, as is "Nature Human Behaviour" at
  `model-documentation.html:63`.
- Author-year citation format is consistent: `Peart & Murray (2026)` and `Peart et al.
  (2024)` on `figures.html:60,75`, `Peart et al., 2024` inline at
  `count-differences.html:38`, `(Skellam, 1946)`, `(Koopman, Lit & Lucas, 2017)`,
  `(Bulmer, 1974)`, `(Kruschke, 2015)`, `(Kruschke, 2021)` — all author-year, all matching
  a full reference in a `.callout`.
- Education dates agree exactly between `index.html:63–65` and `cv.html:38–40`.
- The awards table (`cv.html:46–56`) is correctly sorted descending by end year throughout.
- `index.html:42` "Seven models" matches the seven diagrams on `model-documentation.html`
  and the seven files `model-diagram-1..7.png`.

**Terminology**

- "Skellam-normal" — identical in all 8 occurrences, same hyphen, same lowercase `n`.
- "Bayesian" — capitalised in all 7 occurrences. "Stan" — capitalised in all 10.
- "parameterization / parameterized / parameterizations" — `-z-` in all 6 occurrences.
- "non-centred" appears once and matches the figure label (see #26).
- British house style is consistent in his own prose: "behavioural", "modelling",
  "signalling", "anaesthesia", "centre/centred", "behaviours". American forms appear only
  inside journal titles and published paper titles, where they are correct.
- "estrous / estrogen" is American throughout and consistently so, matching the published
  paper titles.

**Technical content**

- The Stan mathematics on `count-differences.html` is correct. With `phi` = λ₁+λ₂ and
  `mu` = λ₁−λ₂, `sqrt(square(mu) + square(delta))` does recover the variance for
  `delta` = 2√(λ₁λ₂); `log(phi + mu) - log(phi - mu)` is log(λ₁/λ₂); and the Bessel
  argument really is `delta`, which genuinely does not vary with the observation — so the
  cache-and-lookup optimisation is sound. The `target += …_lpmf(y | …)` syntax is correct,
  as is the claim at line 58 that the variance must exceed the absolute mean. The only
  problem with the code is the missing `functions` block (#12).
- `figures/audit-acquisition.png` alt text is accurate apart from the subject count (#4):
  panel a is the specification, b the prior predictive check, c–d posterior predictive
  checks, e–f convergence diagnostics, h three Savage-Dickey tests. All verified against
  the image.
