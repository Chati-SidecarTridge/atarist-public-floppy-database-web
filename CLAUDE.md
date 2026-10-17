# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Browsable website of the floppy images downloadable with the SidecarTridge
Multi-device (see `README.md`). It is a public, static reproduction of the
device's on-board "Public Floppy Database" page (`browser_home.shtml` in the
sibling `md-browser` firmware repo), adapted to run for any visitor with no
device attached.

**Stack — plain static, no build.** The only local files are `index.html`,
`styles.css`, and `favicon.png`. `styles.css` started as a verbatim copy of
`md-browser`'s (so it carries some unused file-manager rules); site-only rules
(e.g. the Search/Browse mode toggle) are appended at the end. The page is driven
by Alpine.js and styled with Pure.css + Font Awesome, all loaded from CDNs. Run
locally with `python3 -m http.server` and open the root. There is no lint or test
tooling; verify changes in the browser.

**Code layout.** All logic is one Alpine component, `appManager()`, in an inline
`<script>` in `index.html`'s `<head>`, bound via `x-data` on `<body>`. Everything
is derived from `parsedEntries` through methods (no cached state), and the markup
has two views switched by `mode`: `'search'` (Recent + Archive sections filtered by
the search box and taxonomy chips) and `'browse'` (letter chips with pages of
`BROWSE_PAGE_SIZE`). The `<head>` also has the SEO metadata (title, description,
OG/Twitter tags, JSON-LD). If you change the copy, keep those consistent with it.

**Cookie banner and analytics.** The block before `</head>` is copied verbatim
from `../rp-webusb-picoflash/index.html` (same as sidecartridge.com):
orestbida/cookieconsent v2.9.2 (MIT) inlined, plus the sidecartridge.com GA4
property `G-6VT9RXTJ2N` as a `type="text/plain" data-cookiecategory="analytics"`
script. GA runs only after consent and only loads gtag on `*.sidecartridge.com`,
so local tests send no hits. On sidecartridge.com hosts `cookie_domain` is
`sidecartridge.com`, so the `cc_cookie` choice is shared with the main site. The
banner uses the default light theme (no `c_darkmode` body class). The footer's
"Cookies" link (`data-cc="c-settings"`) reopens the settings.

**Data.** The catalog is fetched client-side from
`https://ataristdb.sidecartridge.com/db/*.csv` (`a`–`z`, `0`–`9`, `_`), and each
entry links to its image at `https://ataristdb.sidecartridge.com/<path>`. Because
GitHub Pages is HTTPS-only and browsers block http subresource fetches as mixed
content, that host must serve the CSVs over HTTPS.

CSV rows are `;`-separated with exactly 6 fields. Other rows are skipped. Only
fields 1, 3, 5 and 6 are used: `name`, `time` (Unix seconds, often empty),
`label` (the taxonomy chip), and `path`. **A non-empty `time` is what makes an
entry "new":** the Recent section and the Browse "New" chip show timestamped
entries newest-first, while the Archive section and the letter chips show only
untimestamped entries, sorted by name. Rows from `_.csv` are deduplicated against
what's already loaded. Failed fetches are silently ignored.

**Deploy.** Pushing to `main` deploys to GitHub Pages via
`.github/workflows/deploy.yml` (no build; uploads the repo root, so every tracked
file is published). The custom domain is in `CNAME`
(`ataristpublicfloppydb.sidecartridge.com`). `.nojekyll` disables Jekyll. Pages source must be set to "GitHub
Actions" in repo settings.

**Faithfulness note.** The page is a faithful port of `browser_home.shtml`'s browse
experience. Device-only features (microSD folder picker, remote-download progress,
"Return to Booster", SD-card gating, and the `.cgi`/SSI backend) were removed
because they only exist on the physical firmware; the per-entry download control
became a direct link to the image.

**Backlog.** `docs/epics/` mirrors `md-browser`'s epics/stories/tasks tracker and is
**git-ignored** (local-only). Regenerate its dashboard with `./docs/epics/cockpit.sh`.

---

## Working style

These behavioral guidelines bias toward caution over speed. For trivial tasks, use judgment.

### 1. Think before coding

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

### 2. Simplicity first

Minimum code that solves the problem. Nothing speculative.
- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

### 3. Surgical changes

Touch only what you must. Clean up only your own mess.
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it — don't delete it.
- When your changes orphan an import/variable/function, remove it. Don't remove pre-existing dead code unless asked.

The test: every changed line should trace directly to the user's request.

### 4. Goal-driven execution

Define success criteria. Loop until verified.
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan with a verification check per step.

### 5. No AI attribution

Never add AI-tool attribution to commits, PR descriptions, code comments,
docs, or any other artifact. This means **no**:
- "Generated with Claude Code", "Co-authored by Claude", "Made with ChatGPT",
  or any similar phrasing.
- `Co-Authored-By: Claude …`, `Co-Authored-By: ChatGPT …`, or any other
  AI co-author trailer.
- "AI-assisted", "written with the help of an LLM", etc., as comments or
  changelog entries.

Write the message as the human author. Do not mention AI tools used to
produce the work.
