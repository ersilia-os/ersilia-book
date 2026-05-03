# Notes for Claude

Quick reference for working on this repo. The user-facing overview is in `README.md`; this file is meant for LLM sessions and captures conventions that aren't otherwise obvious from the code.

## What this repo is

Source for the **Ersilia Book**, a GitBook published at https://ersilia.gitbook.io/ersilia-book. All content lives under `book/`. GitBook builds and deploys automatically from `main` — there is no local build step. To preview changes, push to a branch and let GitBook render it, or read the rendered Markdown directly.

`.gitbook.yaml` pins the content root to `./book/`. Anything outside `book/` (root `README.md`, `LICENSE`) is repo metadata, not part of the published book.

## How the book is structured

- **`book/SUMMARY.md`** is the source of truth for the sidebar / table of contents. A page only appears in the navigation if it's listed here. Pages not in `SUMMARY.md` are orphans — they exist on disk but are unreachable from the published nav.
- **`book/README.md`** is the landing page of the book itself (rendered at the root of the published site).
- Each top-level section lives in its own directory (`welcome-to-ersilia/`, `ersilia-model-hub/`, `chemistry-tools/`, `contributors/`, `styles/`, `about-us/`).
- **`book/.gitbook/assets/`** holds images and binary assets referenced from pages.

When adding a new page: create the `.md` file under the right section directory and register it in `SUMMARY.md` at the position it should appear.

## Markdown / GitBook conventions used here

- **Frontmatter** (YAML, optional). Common keys:
  - `description: <one-line summary>` — surfaces under the page title in GitBook.
  - `hidden: true` — page is excluded from the published book. **Pages with `hidden: true` should not be listed in `SUMMARY.md`** (they were the source of dangling sidebar entries before).
- **GitBook block syntax** is used freely alongside regular Markdown:
  - `{% hint style="info|warning|danger|success" %} ... {% endhint %}` for callouts.
  - `{% code title="..." %} ... {% endcode %}` for titled code blocks.
  - `{% stepper %} ... {% endstepper %}` for step-by-step procedures.
  - `{% content-ref url="..." %} ... {% endcontent-ref %}` for cross-page link cards.
- **Internal links** use relative paths between `.md` files, e.g. `../ersilia-model-hub/local-inference.md`. The published site rewrites these to clean URLs.
- **`/broken/pages/<id>` is a GitBook artifact**, not a valid link. GitBook leaves these placeholders behind when a link's target is deleted. They render as broken links on the published site. Treat any `/broken/pages/...` you encounter as a bug; replace with a real relative path (or an external URL) or remove the link entirely.
- **File naming**: use descriptive kebab-case slugs (`encryption-of-ai-ml-models.md`, not `page-2.md`). Generic GitBook-template names (`page-1.md`, `page-2.md`) have been seen leaking through; rename them when you spot them.
- **Email addresses** in prose: write `hello@ersilia.io` plainly. Don't use `\[at]` obfuscation — it has appeared inconsistently and reads worse than the plain form.

## Verification

There is no test suite. After edits, sanity-check with greps:

```bash
# 1. No GitBook broken-link placeholders
grep -rn "/broken/pages/" book/

# 2. SUMMARY.md doesn't reference deleted or renamed files
for f in $(grep -oE '\(([^)]+\.md)\)' book/SUMMARY.md | tr -d '()'); do
  test -f "book/$f" || echo "MISSING: $f"
done

# 3. Hidden pages aren't surfaced in the nav
grep -lE '^hidden: true' book/**/*.md \
  | sed 's|^book/||' \
  | xargs -I{} grep -l "{}" book/SUMMARY.md && echo "Hidden page leaked into SUMMARY"
```

If you renamed or deleted files, also do a full-tree link grep for the old path:

```bash
grep -rn "old-filename\.md" book/
```

For typo-prone domain terms seen in past edits, a regex sweep helps:

```bash
grep -rnE "Quantitiative|contiuous|dimentionality|inpection|Guassianization|distilledm" book/
```

## Tone and voice

A descriptive guide, not a prescriptive one — this is what the book actually reads like, and what to preserve when editing.

- **First-person plural.** Pages speak as "we" and "our" (e.g. "*our main platform*", "*we have tested our tools on…*"). Keep this when editing; don't switch to passive or third-person to "sound more documentary."
- **Mission-aware framing.** The book is openly tied to Ersilia's mission (open science, infectious / neglected diseases, the Global South). It's fine for that framing to surface in introductions — don't strip it for brevity.
- **Mixed audience.** Each section assumes a different reader: the Welcome and Chemistry sections lean toward chemists / biologists, the Model Hub developer docs assume command-line and Python comfort, About Us is written for the general public. Match the section you're editing rather than imposing one register across the book.
- **Bold for key terms in introductions.** Introductory paragraphs commonly bold the first occurrence of a domain term (e.g. "**AutoML**", "**ensemble modeling**", "**descriptors**"). Keep this on existing pages; if you're adding a new intro, follow the same pattern.
- **Hint blocks are reserved for callouts**, not decoration. `{% hint style="warning" %}` is used for "this isn't fully tested" or "be careful with IP-sensitive data"; `{% hint style="info" %}` for sidebars and contact pointers. Don't add them just to break up text.
- **Citations** are written academic-style: author list, journal/venue, year, with the linked title — e.g. *Turon, Hlozek et al, Nature Communications, 2023*. Prefer the published version over a preprint when both exist.
- **Default to American English.** The book has been historically mixed (`organisation` ≈ 40 / `organization` ≈ 15, `-ise`/`-ize` mixed, `behavior` already exclusively American), but the convention going forward is American. Use `-ize` / `-ization`, `color`, `behavior`, `analyze`, `center`, `customize`, `recognize`, `prioritize`, `summarize`, `optimize`, `standardize`, `utilize`, etc. When editing a page for any other reason, quietly normalize the spelling on that page as you go. Exceptions: don't change spelling inside quoted material, proper nouns, organisation names ("Ersilia Open Source **Initiative**" stays as-is — and any external party's name keeps their spelling), URLs, code, citation titles, or content licensed from elsewhere (e.g. the Contributor Covenant text in the Code of Conduct).
- **Don't rewrite for voice.** The author voice varies between sections (and sometimes between paragraphs) and that variation is fine. In an editorial pass, fix typos, broken links, factual errors, structural duplication, and British-vs-American inconsistency — but leave the underlying prose voice alone unless the user explicitly asks for a rewrite.

## Things to know about the content

- **Outreachy internship pages** (`book/contributors/internships/outreachy-*.md`) are archival cohort records. They tend to accumulate broken links and stale instructions over time. Don't refactor them as a side-effect of unrelated work — touch only when the user asks.
- **`book/quick-start/`** contains orphan files (`installation.md`, `molecular-weight.md`, `antibiotic-activity-prediction.md`) not registered in `SUMMARY.md`. The canonical install page is `book/ersilia-model-hub/local-inference.md`. Leave the quick-start orphans alone unless asked — the user may revive them as a future section.
- **`book/about-us/strategic-plan-2021-2023.md`** is intentionally retained on disk with `hidden: true` for archival purposes; don't re-add it to `SUMMARY.md`.
- **ZairaChem** has one canonical page (`chemistry-tools/automated-activity-prediction-models/zairachem.md`). A near-duplicate `accurate-automl-with-zairachem.md` previously existed and was merged in; don't re-introduce it.
