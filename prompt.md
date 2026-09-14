# Claude Code Prompt — Scaffold a Weekly Video Production Project

> Fill in the variables block, paste the whole thing into Claude Code from an empty parent directory (e.g. `~/projects/`), and let it run.

---

## VARIABLES (edit these before pasting)

```
VIDEO_SLUG        = sep-3-<short-topic>          # repo name, kebab-case, week-prefixed
VIDEO_TITLE       = "<Working title of the video>"
VIDEO_HYPOTHESIS  = "<One-sentence claim the video makes>"
GITHUB_OWNER      = rifaterdemsahin
TEMPLATE_REPO     = https://github.com/rifaterdemsahin/sep-1-future-of-jobs
SUPABASE_URL      = https://<project-ref>.supabase.co
SUPABASE_ANON_KEY = <anon key>
SUPABASE_DB_URL   = postgresql://postgres:<password>@db.<project-ref>.supabase.co:5432/postgres
CF_WORKER_NAME    = ${VIDEO_SLUG}
LOCAL_PORT        = 30080
```

---

## CONTEXT

I produce one YouTube video per week for an AI-education channel. Every video gets its own pre-production repository built from the same reference project: `${TEMPLATE_REPO}`. That project is a **static HTML site with zero build step**, backed by **Supabase Postgres** for all authored content, deployed to **Cloudflare Workers** (canonical) with **GitHub Pages** acting only as a redirect. It models pre-production as a linked pipeline of pages, each feeding the next:

```
Research → Arguments → Script → Design → Previsualisation → (Assets, Todo alongside)
```

Reference architecture (copy this shape exactly):

```
${VIDEO_SLUG}/
├── index.html                  # root redirect stub → html/index.html
├── html/
│   ├── index.html              # Stage 1  Research: source links, counter-sources, pro/con video table, footage leads
│   ├── arguments.html          # Stage 2  Arguments: premise, arguments, conclusion (+ Kokoro VO bar)
│   ├── script.html             # Stage 3  Script: timed voiceover beats + audio player
│   ├── design.html             # Stage 4  Design: palette, typography, pacing/motion specs as a 3-act story
│   ├── previsualisation.html   # Stage 5  Previs: 9-panel shot board, each with a plain-English note
│   ├── assets.html             # Stage 6  Assets: B-roll catalog mapped to script sections + YouTube search terms
│   └── todo.html               # Stage 7  Kanban production board
├── css/shared.css              # theme variables (7 themes), body reset, top nav
├── js/
│   ├── supabase-client.js      # one shared window.sb client per page
│   ├── content-db.js           # fetches content_blocks, exposes .ready + getBlocks(page, section)
│   ├── nav.js                  # single-source top nav, hue-graded link colours
│   ├── pipeline.js             # stage stepper rendered on every stage page
│   ├── links.js                # cross-stage "Linked to…" multi-select pickers (reads previous stage's blocks)
│   ├── theme.js                # Dark / Light / Midnight / Sepia / Ocean / Grape / High Contrast
│   ├── ratings.js              # 1–5 star rating + re-sort
│   ├── notes.js                # bottom "Notes for Video Production Agent" bar + per-item notes + DB explainer modal
│   ├── assets.js               # add-to-assets engine, Azure upload
│   └── audio-clips.js          # manifest of Kokoro TTS clips already in Azure Blob (skip regeneration)
├── images/                     # storyboard panels
├── transcripts/                # raw source transcripts backing Research
├── supabase/
│   ├── schema.sql              # videos, content_blocks, assets, notes, item_notes, ratings, audio_clips
│   ├── apply-schema.js         # schema executor (Node + pg)
│   ├── seed.js                 # inserts the videos row
│   ├── seed-content.js         # SOURCE OF TRUTH for all stage content; upserts by id, safe to re-run
│   └── README.md
├── .agents/skills/link-validator/   # keep — validates every external link on the site
├── scripts/
├── .env                        # git-ignored
├── .gitignore  .assetsignore  .wranglerignore
├── wrangler.toml
├── package.json
├── CLAUDE.md                   # repo workflow rules (see Constraints)
├── PROJECT_TEMPLATE_SPECS.md
├── MENU_SPECS.md
├── REPORT.md
└── README.md
```

Supabase schema (unchanged from the template):

| table | key columns |
|---|---|
| `videos` | `id` (= `${VIDEO_SLUG}`), `title`, `description`, `slug`, `status` (`draft`/`production`/`review`/`published`), timestamps |
| `content_blocks` | `id`, `page` (e.g. `script.html`), `section` (e.g. `beats`), `position`, `type` (`link-card` / `argument-card` / `beat` / `shot-panel`), `data` jsonb |
| `assets` | `id`, `type`, `title`, `description`, `source`, `url`, `comment`, `added_at` |
| `notes`, `item_notes` | freeform + per-card annotations |
| `ratings` | 1–5 per item id |
| `audio_clips` | manifest of Kokoro clips in Azure Blob Storage |

Content is **never hardcoded in HTML**. Every page waits on `ContentDB.ready` and renders client-side from `content_blocks`, keyed by stable item ids so ratings/notes/links survive re-seeds.

---

## ROLE

You are a senior platform engineer acting as my video-production tooling lead. You know this codebase's conventions cold, you prefer copying a proven structure over inventing a new one, and you treat the pre-production site as production infrastructure: reproducible, seeded from code, deployed with one command, verified in a browser before you say "done".

---

## CONSTRAINTS

**Repo & git**
1. Single branch only: `main`. Never create, check out, or push any other branch.
2. Create the GitHub repo as `${GITHUB_OWNER}/${VIDEO_SLUG}` (public) with `gh repo create --source=. --push`. Do not push to the template repo.
3. Commit in small logical commits. After the final push, open `https://github.com/${GITHUB_OWNER}/${VIDEO_SLUG}/commit/<sha>` in Chrome (`open -a "Google Chrome" <url>`, never the default browser).
4. Rewrite `CLAUDE.md` to point at the new repo URL and keep all its workflow rules (local server check, single-branch rule, change → preview → commit → push → review-on-GitHub).

**Stack — do not deviate**
5. Static HTML + vanilla JS + one shared CSS. No framework, no bundler, no build step. Pages use relative `../js/`, `../css/`, `../images/` paths.
6. Supabase Postgres is the only content store. No localStorage for authored content. `seed-content.js` remains the single source of truth.
7. Cloudflare Workers is canonical (`wrangler.toml`, `npx wrangler deploy`). GitHub Pages must keep the `location.replace` redirect to the Workers URL, triggered only on a `github.io` hostname so local runs are unaffected. Bare `/` redirects to `/html/index.html`.
8. Keep Kokoro TTS + Azure Blob wiring in `audio-clips.js` and `assets.js`; do not swap TTS providers.
9. Keep the 7-theme switcher, ratings, notes, cross-stage links and the pipeline stepper working on every page.

**Secrets & environment**
10. Write `.env` from the VARIABLES block. Confirm `.env` is in `.gitignore` before the first commit; never commit it, and never echo secret values into logs, README, or commit messages.
11. Local preview server must run on port `${LOCAL_PORT}` (> 30000). Check whether it is already up before starting another.

**Content**
12. Strip all "Job Apocalypse / Elon Musk" content: `seed-content.js`, `transcripts/`, `images/`, `REPORT.md`, `README.md`, page `<title>`s, nav labels, meta descriptions. Replace with `${VIDEO_TITLE}` / `${VIDEO_HYPOTHESIS}` placeholders and **one example block per type** (`link-card`, `argument-card`, `beat`, `shot-panel`) so each page renders non-empty and I can see the shape to fill in.
13. Preserve every item-id convention from the template so `ratings.js`, `notes.js`, and `links.js` keep resolving.
14. Do not invent research, arguments, or script beats for the actual video — that is my job. Placeholders must be obviously placeholders (`TODO:` prefix).

**Verification**
15. Nothing is "done" until: schema applied, seed run without error, all 7 pages load on `http://localhost:${LOCAL_PORT}/html/…` with zero console errors, the pipeline stepper and nav render, the link-validator skill passes, and `wrangler deploy` succeeds (or is cleanly skipped with a stated reason if Cloudflare auth is missing).

---

## TASK

Execute in this order. Stop and ask me only if a credential is missing; otherwise run end-to-end.

1. **Clone as template.** `git clone --depth 1 ${TEMPLATE_REPO} ${VIDEO_SLUG}`, `cd` in, remove `.git`, `git init -b main`.
2. **Rename & rebrand.** Replace every occurrence of `sep-1-future-of-jobs` with `${VIDEO_SLUG}` (repo URLs, `wrangler.toml` name, `videos.id`, GitHub Pages redirect targets, `package.json` name, README, CLAUDE.md). Replace the video title/hypothesis everywhere per Constraint 12.
3. **Environment.** Write `.env`; verify `.gitignore` coverage; `npm install`.
4. **Database.** `node supabase/apply-schema.js` → `node supabase/seed.js` (videos row: `${VIDEO_SLUG}`, `${VIDEO_TITLE}`, `${VIDEO_HYPOTHESIS}`, status `draft`) → `node supabase/seed-content.js` with the placeholder blocks.
5. **Local verify.** Start `python3 -m http.server ${LOCAL_PORT}` if not running; open `http://localhost:${LOCAL_PORT}/html/index.html` in Chrome; click through all 7 pages; report console errors and fix them.
6. **Link validation.** Run `.agents/skills/link-validator` and fix or remove dead links.
7. **Docs.** Update `README.md` (new title, new repo/Workers URLs, unchanged folder/pipeline sections), `CLAUDE.md` (new repo URL), and add `WEEKLY_CHECKLIST.md`:
   - Mon Research → Tue Arguments → Wed Script + VO → Thu Design + Previs → Fri Assets + edit handoff → Sat/Sun publish
   - each day lists the page to fill, the `seed-content.js` section to edit, and the re-seed command.
8. **Deploy.** `npx wrangler deploy`; record the resulting `*.workers.dev` URL in README and in the GitHub Pages redirect.
9. **Publish.** Commit, `gh repo create ${GITHUB_OWNER}/${VIDEO_SLUG} --public --source=. --push`, then open the commit page in Chrome.
10. **Report.** Finish with exactly this block, nothing else after it:

```
## SCAFFOLD REPORT
repo:            https://github.com/${GITHUB_OWNER}/${VIDEO_SLUG}
workers_url:     <url or "skipped: <reason>">
pages_verified:  index | arguments | script | design | previsualisation | assets | todo  → <pass/fail each>
console_errors:  <count>
link_validator:  <pass/fail, n fixed>
seed_rows:       videos=<n> content_blocks=<n>
secrets_committed: <none | LIST>
next_action_for_erdem: <one line>
```
