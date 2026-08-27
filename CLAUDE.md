# Mahdi Academy website — context

Marketing/legal site for Mahdi Academy (see the app repo's root `CLAUDE.md` for full product context). **This is a separate git repository** (own `.git`, remote `https://github.com/tidjani-shaba/mahdi-academy-website.git`), nested inside the app repo's working directory but not a submodule — commit/push from inside `mahdi web/`, not from the parent repo.

## Hosting (settled — don't re-ask)
**GitHub Pages, custom domain `mahdi-academy.org`** (see `CNAME`). Static files only — no server-side rendering, no build step. `index.html` is the entire site (single page); `legal/privacy/index.html` and `legal/terms/index.html` are the only other real pages.

## SEO approach (built 2026-08-27)
**Goal, in the user's own words**: when someone searches things like "Bac D Mathématiques Cameroun", search engines should recognize Mahdi Academy covers that content and rank it — but the actual visible page must stay exactly the landing page it already is (CTA to download the app), never showing raw question/answer content the way competitor sites (that expose scraped PDFs directly) do.

**Why JSON-LD, not hidden text**: the tempting literal reading of "search engines should see it, visitors shouldn't" is visually-hidden keyword text (`display:none`, off-screen positioning, etc.) — but that's classified by Google as "Hidden text and links," a spam violation that risks a manual action. The correct, sanctioned mechanism for exactly this need is **structured data (JSON-LD)** — a `<script type="application/ld+json">` block is never rendered to any visitor by design, so it's the real "backend channel to search engines" the user was describing, without the cloaking risk. That's what's implemented.

**What's in the JSON-LD graph** (single `@graph` array before `</head>`):
- `EducationalOrganization` + `MobileApplication` (name, description, plan pricing as `Offer`s)
- One `Course` entity per subject that has real `ready` content (11 subjects — the 5 zero-content subjects (Allemand/Arabe/Chinois/EPS/ECM) are deliberately omitted, not claimed with a 0 count), each with a real question/chapter count pulled from a live Supabase query
- One `EducationalOccupationalProgram` per série (A/ABI/C/D), each `hasCourse`-linking (via `@id`) to the subjects real for that série
- `FAQPage` — mirrors the *visible* on-page FAQ text exactly (not enhanced/expanded), since FAQ structured data must match what a visitor actually sees to stay compliant

**Other SEO additions in the same round**: `<meta name="description">`, Open Graph + Twitter Card tags, `<link rel="canonical">`, `robots.txt`, `sitemap.xml` (covers `/`, `/legal/privacy/`, `/legal/terms/`) — none of this previously existed on the site at all (verified via grep before starting: zero meta description, zero structured data, zero sitemap).

**Content counts snapshot (live-queried from Supabase `sdtyxhihvzftriymedeq` on 2026-08-27)** — baked into the JSON-LD as static numbers since this is a static site with no build pipeline. **These will drift as content entry continues** (see the app repo's `content-log.md`) — re-run this query and manually update the `Course` descriptions' numbers periodically, there's no automation for it:
```sql
select s.name, s.series, count(distinct c.id) chapters, count(q.id) filter (where q.status='ready') ready_questions
from subjects s left join chapters c on c.subject_id=s.id left join questions q on q.chapter_id=c.id
group by s.name, s.series order by ready_questions desc;
```
Subjects with 0 ready questions as of this snapshot (excluded from structured data until they have real content): Allemand, Arabe, Chinois, EPS, ECM.

**Honest limitation**: this is on-page/structured-data SEO only — no backlinks, no content marketing, no per-keyword landing pages. If ranking against PDF-scraper competitor sites (mentioned by name during the request: sites that expose raw past-paper PDFs directly) proves hard with just this, the next lever is dedicated per-série static pages (e.g. `/bac-d/`) with real keyword-rich body copy — explicitly discussed and deferred in favor of the single-page approach for this round.

## Dark mode is the default theme, light mode's background is true white
- `localStorage['mahdi-theme']` persists the user's choice; `dark` is the fallback when nothing is stored.
- **Anti-flash-of-light fix (2026-08-27)**: the theme used to only get applied by a `<script>` at the very bottom of `<body>` — late enough that the page could paint in light mode for a moment before flipping to dark on every load. Fixed by adding a tiny blocking inline `<script>` immediately after `<meta charset>` in `<head>` that sets `data-theme` before any CSS renders. The bottom-of-body script is unchanged (still owns the toggle button + sun/moon icon swap) — it just no longer does the *first* paint.
- **Light mode background (2026-08-27)**: `--bg` was `#F8F9FA` (a slightly off-white cream, matching the app's own brand background) — changed to `#FFFFFF` per explicit request that light mode read as genuinely white. `--surface` was already `#FFFFFF`; cards/sections still separate visually via `--border` (`#E5E7EB`), not via a bg/surface contrast anymore. Dark mode's own values were left untouched.

## If asked to touch this site again
- Don't add a build step/framework — it's intentionally a single static `index.html`, keep edits inline.
- Any new visible marketing copy goes in the actual HTML body (visitor-facing); any new "tell search engines, don't show visitors" content goes in the JSON-LD graph, never as visually-hidden CSS-tricked text.
- Real Play Store / App Store links don't exist yet (the store badges in the hero are plain text, not `<a>` links) — add `installUrl`/`sameAs` to the `MobileApplication` JSON-LD once the app is actually published, not before.
