# Claude Code Prompt — Scaffold a Weekly Video Production Project (v2: closed learning loop)

> Fill in the variables block, paste the whole thing into Claude Code from an empty parent directory (e.g. `~/projects/`), and let it run.

---

## VARIABLES (edit these before pasting)

```
VIDEO_SLUG            = sep-3-<short-topic>          # repo name, kebab-case, week-prefixed
VIDEO_TYPE            = weekly-video                 # weekly-video | course-module — see note below
VIDEO_TITLE           = "<Working title of the video>"
VIDEO_HYPOTHESIS      = "<One-sentence claim the video makes>"
VIEWER_TAKEAWAY       = "<After this video the viewer can ___>"                    # REQUIRED for both types
AUDIENCE_DELIVERABLE  = "<repo / prompt / checklist / template the viewer can run in <5 min>"   # REQUIRED for both types
SOURCE_CONTENT_URL    = <url of the popular/viral content this reverse-engineers>   # weekly-video only
LEARNING_OBJECTIVE    = "<what the learner can do after watching>"                 # course-module only
HANDS_ON_KEY_RESULTS  = "<concrete, verifiable artifact the learner produces>"      # course-module only
GITHUB_OWNER          = rifaterdemsahin
TEMPLATE_REPO         = https://github.com/rifaterdemsahin/sep-1-future-of-jobs
AZURE_KEY_VAULT_NAME  = <key-vault-name>             # Azure Key Vault holding the shared Supabase secrets (secrets: supabase-url, supabase-anon-key, supabase-db-url)
CF_WORKER_NAME        = ${VIDEO_SLUG}
LOCAL_PORT            = 30080
```

🔐 **No Supabase credentials are pasted here.** They are fetched at scaffold time straight from `${AZURE_KEY_VAULT_NAME}` — see Constraint 10.

🆔 **`VIDEO_ID` is not supplied by me — you generate it.** It's the row key for this video in the SHARED `videos` table, independent of `VIDEO_SLUG`. Generate a short unique id (e.g. `uuidgen | tr 'A-Z' 'a-z' | cut -c1-8`), then query the shared `videos` table (`select id from videos where id = '<candidate>'`) and regenerate on any collision before using it anywhere. See Task step 3.

🎯 **`VIDEO_TYPE` drives duration and content shape — exactly one, only its matching fields:**
- 🎥 **`weekly-video`** — regular YouTube channel upload. Locked to **6 minutes**. Reverse-engineers `${SOURCE_CONTENT_URL}`: a popular/viral piece of existing content whose hook, structure, pacing and title/thumbnail pattern this video deconstructs and rebuilds on our topic — leave `LEARNING_OBJECTIVE` / `HANDS_ON_KEY_RESULTS` blank.
- 🎓 **`course-module`** — one module inside a structured course. Locked to **3 minutes**, hard cap. Opens from `${LEARNING_OBJECTIVE}` and builds toward `${HANDS_ON_KEY_RESULTS}` — leave `SOURCE_CONTENT_URL` blank.

🧠 **`VIEWER_TAKEAWAY` and `AUDIENCE_DELIVERABLE` are required for BOTH types.** A video without a stated viewer outcome is not allowed to scaffold — stop and ask me if either is blank.

---

## CONTEXT

I produce short-form videos for an AI-education channel and course, of two kinds: a **weekly-video** (regular 6-minute YouTube upload that reverse-engineers a popular piece of existing content) or a **course-module** (a 3-minute-locked module inside a structured course, built around a learning objective and a hands-on key result). Every video — either kind — gets its own pre-production repository built from the same reference project: `${TEMPLATE_REPO}`. That project is a **static HTML site with zero build step**, backed by **one shared Supabase Postgres project reused across every video, of both kinds** (all rows scoped by `${VIDEO_ID}`, credentials pulled from Azure Key Vault — never a new project per video), deployed to **Cloudflare Workers** (canonical) with **GitHub Pages** acting only as a redirect.

The channel's identity is a **teach-to-learn flywheel**. The pre-production site is therefore not only a production tool — it is the public record of *how I learned the topic*, and it must close the loop on my learning method:

```
Real/Unknown → Imagined → Journal → Formula → Symbols → Semblance → Testing
```

The pipeline models this as linked pages, each feeding the next:

```
Unknowns → Research → Arguments → Script → Design → Previsualisation → (Assets, Todo, Journal alongside) → Retro
  Stage 0    Stage 1     Stage 2     Stage 3   Stage 4        Stage 5          Stage 6/7/8               Stage 9
```

- **Stage 0 Unknowns** captures what I believe *before* researching, what I'm unsure of, and how the hypothesis could be wrong. It is snapshotted and never edited after Research begins; instead each question is later marked `answered` / `changed-my-mind` / `still-open`.
- **Journal** is the decision log: every argument cut, beat trimmed or shot dropped keeps its *why*.
- **Retro** is the post-publish testing stage: hypothesis verdict, retention notes, CTR, top comment questions, what I'd change — and for weekly videos, whether the hook/pacing borrowed from `${SOURCE_CONTENT_URL}` actually held.
- Every video extracts **one Formula** (a reusable rule) into a shared `patterns` table, so future course modules can query the formulas of past weekly videos.
- Every video ships **one Audience Deliverable** the viewer can run in under five minutes, linked from the description.

Reference architecture (copy this shape exactly — new files marked `NEW`):

```
${VIDEO_SLUG}/
├── index.html                  # root redirect stub → html/index.html
├── html/
│   ├── unknowns.html           # Stage 0  Unknowns: NEW — prior beliefs, open questions, falsification conditions (snapshot + status)
│   ├── index.html              # Stage 1  Research: source links, counter-sources, pro/con video table, footage leads
│   ├── arguments.html          # Stage 2  Arguments: premise, arguments, conclusion (+ Kokoro VO bar)
│   ├── script.html             # Stage 3  Script: timed voiceover beats + audio player; viewer_takeaway pinned at top
│   ├── design.html             # Stage 4  Design: palette, typography, pacing/motion specs as a 3-act story
│   ├── previsualisation.html   # Stage 5  Previs: 9-panel shot board, each with a plain-English note
│   ├── assets.html             # Stage 6  Assets: B-roll catalog mapped to script sections + YouTube search terms + audience deliverable card
│   ├── todo.html               # Stage 7  Kanban production board
│   ├── journal.html            # Stage 8  Journal: NEW — decision log rendered from item_notes where decision=true, plus the Formula block
│   └── retro.html              # Stage 9  Retro: NEW — post-publish verdict form + reverse-engineer comparison
├── css/shared.css              # theme variables (7 themes), body reset, top nav
├── js/
│   ├── supabase-client.js      # one shared window.sb client per page
│   ├── content-db.js           # fetches content_blocks, exposes .ready + getBlocks(page, section)
│   ├── nav.js                  # single-source top nav, hue-graded link colours (now 10 pages)
│   ├── pipeline.js             # stage stepper rendered on every stage page (Stage 0 → Stage 9)
│   ├── links.js                # cross-stage "Linked to…" multi-select pickers (reads previous stage's blocks)
│   ├── theme.js                # Dark / Light / Midnight / Sepia / Ocean / Grape / High Contrast
│   ├── ratings.js              # 1–5 star rating + re-sort
│   ├── notes.js                # bottom "Notes for Video Production Agent" bar + per-item notes (+ decision flag) + DB explainer modal
│   ├── assets.js               # add-to-assets engine, Azure upload
│   ├── audio-clips.js          # manifest of Kokoro TTS clips already in Azure Blob (skip regeneration)
│   ├── unknowns.js             # NEW — question status toggles (answered / changed-my-mind / still-open), snapshot lock
│   ├── retro.js                # NEW — retro form writes to retros table; renders hypothesis verdict banner
│   └── patterns.js             # NEW — reads/writes the video's Formula row; lists formulas from other videos (read-only)
├── images/                     # storyboard panels
├── transcripts/                # raw source transcripts backing Research
├── supabase/
│   ├── schema.sql              # videos, content_blocks, assets, notes, item_notes, ratings, audio_clips, retros, patterns
│   ├── apply-schema.js         # schema executor (Node + pg) — additive only against the shared project
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
├── WEEKLY_CHECKLIST.md
└── README.md
```

Supabase schema (same shape as the template, shared across videos, scoped by `video_id`; additions marked `NEW`):

| table | key columns |
|---|---|
| `videos` | `id` (= `${VIDEO_ID}`), `slug` (= `${VIDEO_SLUG}`), `title`, `description`, `video_type` (`weekly-video`/`course-module`), `status` (`draft`/`production`/`review`/`published`), `viewer_takeaway` text NEW, `audience_deliverable` jsonb NEW (`{title, url, run_time_minutes}`), `published_at` NEW, timestamps |
| `content_blocks` | `id`, `video_id` (= `${VIDEO_ID}`), `page` (e.g. `script.html`), `section` (e.g. `beats`), `position`, `type` (`link-card` / `argument-card` / `beat` / `shot-panel` / `question-card` NEW / `audience-deliverable` NEW), `data` jsonb |
| `assets` | `id`, `video_id`, `type`, `title`, `description`, `source`, `url`, `comment`, `added_at` |
| `notes`, `item_notes` | `video_id` + freeform + per-card annotations; `item_notes.decision` boolean NEW (true = this note explains a cut/keep/change and appears in Journal) |
| `ratings` | `video_id` + 1–5 per item id |
| `audio_clips` | `video_id` + manifest of Kokoro clips in Azure Blob Storage |
| `retros` NEW | `video_id`, `hypothesis_verdict` (`held`/`partly`/`failed`), `retention_note`, `ctr`, `avg_view_duration_s`, `top_comment_questions` jsonb, `source_comparison` (weekly-video only: did the borrowed hook/pacing hold?), `what_id_change`, `created_at` |
| `patterns` NEW | `id`, `video_id`, `formula` (one-sentence reusable rule), `symbol` (short mnemonic / name), `evidence` (which retro field supports it), `created_at` |

`question-card.data` shape: `{question, prior_belief, falsifier, status: 'open'|'answered'|'changed-my-mind'|'still-open', answer, snapshot_locked: bool}`.

🗄️ Because the Supabase project is **shared**, every query and every write from `content-db.js`, `assets.js`, `notes.js`, `ratings.js`, `audio-clips.js`, `unknowns.js`, `retro.js` and `patterns.js` must filter/insert by `video_id = ${VIDEO_ID}` — never assume the table only holds this video's rows. The one exception is `patterns.js`, which may **read** other videos' formulas for the cross-video library, but only ever **writes** rows carrying `${VIDEO_ID}`.

Content is **never hardcoded in HTML**. Every page waits on `ContentDB.ready` and renders client-side from `content_blocks`, keyed by stable item ids so ratings/notes/links survive re-seeds.

---

## ROLE

You are a senior platform engineer acting as my video-production tooling lead. You know this codebase's conventions cold, you prefer copying a proven structure over inventing a new one, and you treat the pre-production site as production infrastructure: reproducible, seeded from code, deployed with one command, verified in a browser before you say "done". You also understand that this site is public and is part of the teaching: it must show the reasoning trail, not just the polished result.

---

## CONSTRAINTS

**Repo & git**
1. Single branch only: `main`. Never create, check out, or push any other branch.
2. Create the GitHub repo as `${GITHUB_OWNER}/${VIDEO_SLUG}` (public) with `gh repo create --source=. --push`. Do not push to the template repo.
3. Commit in small logical commits. After the final push, open `https://github.com/${GITHUB_OWNER}/${VIDEO_SLUG}/commit/<sha>` in Chrome (`open -a "Google Chrome" <url>`, never the default browser).
4. Rewrite `CLAUDE.md` to point at the new repo URL and keep all its workflow rules (local server check, single-branch rule, change → preview → commit → push → review-on-GitHub).

**Stack — do not deviate**
5. Static HTML + vanilla JS + one shared CSS. No framework, no bundler, no build step. Pages use relative `../js/`, `../css/`, `../images/` paths.
6. Supabase Postgres is the **one shared** content store — the same project used by every weekly video, never a new project. No localStorage for authored content. `seed-content.js` remains the single source of truth, and every row it upserts must carry `video_id = ${VIDEO_ID}`.
7. Cloudflare Workers is canonical (`wrangler.toml`, `npx wrangler deploy`). GitHub Pages must keep the `location.replace` redirect to the Workers URL, triggered only on a `github.io` hostname so local runs are unaffected. Bare `/` redirects to `/html/index.html`.
8. Keep Kokoro TTS + Azure Blob wiring in `audio-clips.js` and `assets.js`; do not swap TTS providers.
9. Keep the 7-theme switcher, ratings, notes, cross-stage links and the pipeline stepper working on every page — including the three new pages.

**Secrets & environment**
10. 🔑 **Fetch Supabase credentials from Azure Key Vault — never ask me for them, never hardcode them.**
    ```
    az keyvault secret show --vault-name ${AZURE_KEY_VAULT_NAME} --name supabase-url        --query value -o tsv
    az keyvault secret show --vault-name ${AZURE_KEY_VAULT_NAME} --name supabase-anon-key   --query value -o tsv
    az keyvault secret show --vault-name ${AZURE_KEY_VAULT_NAME} --name supabase-db-url     --query value -o tsv
    ```
    Write the fetched values into `.env`. Confirm `.env` is in `.gitignore` before the first commit; never commit it, and never echo secret values into logs, README, commit messages, or the SCAFFOLD REPORT. If `az` isn't logged in or a secret is missing, stop and ask me — do not fall back to placeholder credentials for a shared database.
11. Local preview server must run on port `${LOCAL_PORT}` (> 30000). Check whether it is already up before starting another.

**Content**
12. Strip all "Job Apocalypse / Elon Musk" content: `seed-content.js`, `transcripts/`, `images/`, `REPORT.md`, `README.md`, page `<title>`s, nav labels, meta descriptions. Replace with `${VIDEO_TITLE}` / `${VIDEO_HYPOTHESIS}` placeholders and **one example block per type** (`link-card`, `argument-card`, `beat`, `shot-panel`, `question-card`, `audience-deliverable`) so each page renders non-empty and I can see the shape to fill in.
13. Preserve every item-id convention from the template so `ratings.js`, `notes.js`, and `links.js` keep resolving. New block types follow the same `<page>-<section>-<n>` id convention.
14. Do not invent research, arguments, script beats, prior beliefs, or retro results for the actual video — that is my job. Placeholders must be obviously placeholders (`TODO:` prefix).
15. 🎯 **Honor `${VIDEO_TYPE}` exactly — never blend the two shapes:**
    - `weekly-video`: script (`script.html`) must fit a spoken **6-minute** runtime (~900 words @150wpm). Stage 1 Research must capture `${SOURCE_CONTENT_URL}` as a `link-card` tagged `reverse-engineer-source`, with a placeholder note on what made it work (hook, structure, pacing, title/thumbnail pattern) for me to fill in. `retro.html` must show the `source_comparison` field.
    - `course-module`: script must fit a spoken **3-minute** runtime (~450 words @150wpm) — a hard cap; trim scope rather than exceed it. `script.html` must open with a block stating `${LEARNING_OBJECTIVE}`, and `previsualisation.html` / `assets.html` must build toward producing `${HANDS_ON_KEY_RESULTS}` as a distinct placeholder block. `retro.html` hides `source_comparison`.

**Learning loop — applies to BOTH types**
16. 🧠 **Stage 0 Unknowns is mandatory and precedes Research in the stepper.** Seed **three** `question-card` placeholders on `unknowns.html` (`TODO: what I currently believe`, `TODO: what I'm unsure of`, `TODO: how the hypothesis could be wrong`). `unknowns.js` must implement a **Snapshot lock**: once I click "Lock snapshot", `prior_belief` and `falsifier` become read-only; only `status` and `answer` remain editable. Research links (`links.js`) on `index.html` must offer "Linked to…" pickers that resolve to Stage 0 questions.
17. 🧠 **`viewer_takeaway` is required.** Store `${VIEWER_TAKEAWAY}` on the `videos` row and pin it as a read-only banner at the top of `script.html` and `retro.html`. If the variable is blank, stop and ask — do not scaffold a video with no viewer outcome.
18. 🧠 **Audience deliverable is a first-class block.** Seed one `audience-deliverable` block on `assets.html` from `${AUDIENCE_DELIVERABLE}` (`{title, url, run_time_minutes}`) and mirror it into `videos.audience_deliverable`. `README.md` must include a "For viewers" section that links it and links the public Workers preprod URL as "How this video was made". If `${AUDIENCE_DELIVERABLE}` names a repo/prompt/checklist that doesn't exist yet, the deliverable may default to the scaffolded repo itself (the scaffold is the demo).
19. 🧠 **Journal = decision log.** Add a `decision` checkbox to the per-item note UI in `notes.js`. `journal.html` renders every `item_notes` row with `decision = true` for this `video_id`, grouped by stage, newest first, each linking back to its item. Below that it renders the video's **Formula** block (`patterns` row: `TODO: reusable rule`, `TODO: symbol`) editable via `patterns.js`, followed by a read-only list titled "Formulas from other videos" (all `patterns` rows where `video_id != ${VIDEO_ID}`, showing slug + formula).
20. 🧠 **Retro closes the loop.** `retro.html` is a single form bound to the `retros` row for this `video_id` (create on first save). Fields: hypothesis verdict (radio: held / partly / failed), retention note, CTR, avg view duration, top comment questions (repeatable list), what I'd change, and — weekly-video only — source comparison. Saving a retro with a verdict sets a coloured verdict banner on `unknowns.html` and `journal.html` so the answer to Stage 0 is visible from Stage 0. Saving must not change `videos.status`; only I set `published`.
21. **Additive schema only.** `retros`, `patterns`, the new `videos` columns, `item_notes.decision` and the new block types are added with `CREATE TABLE IF NOT EXISTS` / `ALTER TABLE … ADD COLUMN IF NOT EXISTS` / check-constraint updates. Never drop or rewrite existing shared tables.

**Verification**
22. Nothing is "done" until: schema applied additively, seed run without error, all **10** pages load on `http://localhost:${LOCAL_PORT}/html/…` with zero console errors, the pipeline stepper (Stage 0 → 9) and nav render on every page, the Snapshot lock and decision checkbox work, the retro form round-trips a save, the link-validator skill passes, `wrangler deploy` succeeds (or is cleanly skipped with a stated reason if Cloudflare auth is missing), and the duration/content-shape rule for `${VIDEO_TYPE}` (Constraint 15) is satisfied.

---

## TASK

Execute in this order. Stop and ask me only if a credential is missing or `${VIEWER_TAKEAWAY}` / `${AUDIENCE_DELIVERABLE}` is blank; otherwise run end-to-end.

1. **Clone as template.** `git clone --depth 1 ${TEMPLATE_REPO} ${VIDEO_SLUG}`, `cd` in, remove `.git`, `git init -b main`.
2. **Environment.** 🔐 Fetch `supabase-url`, `supabase-anon-key`, `supabase-db-url` from Azure Key Vault `${AZURE_KEY_VAULT_NAME}` (Constraint 10); write `.env`; verify `.gitignore` coverage; `npm install`.
3. **Generate VIDEO_ID.** 🆔 Generate a short unique id (e.g. `uuidgen | tr 'A-Z' 'a-z' | cut -c1-8`), then query the shared database (`select id from videos where id = '<candidate>'`) — regenerate on any collision. Use the confirmed-unique value as `${VIDEO_ID}` for every step below.
4. **Rename & rebrand.** Replace every occurrence of `sep-1-future-of-jobs` with `${VIDEO_SLUG}` (repo URLs, `wrangler.toml` name, GitHub Pages redirect targets, `package.json` name, README, CLAUDE.md). Set `videos.id` to `${VIDEO_ID}` (not the slug) everywhere it's referenced in JS/SQL. Replace the video title/hypothesis everywhere per Constraint 12.
5. **Add the learning-loop pages.** Create `html/unknowns.html`, `html/journal.html`, `html/retro.html` and `js/unknowns.js`, `js/retro.js`, `js/patterns.js` following the existing page skeleton (shared CSS, nav, pipeline stepper, theme, notes bar). Extend `nav.js` and `pipeline.js` to Stage 0 → 9. Extend `notes.js` with the `decision` checkbox. Extend `links.js` so Research cards can link to Stage 0 questions.
6. **Database (shared, reused).** Do **not** run `apply-schema.js` destructively against a live shared project — first check whether the schema already exists (`videos`, `content_blocks`, etc. with a `video_id` column, plus `video_type` on `videos`); apply only what's missing, then apply the v2 additions from Constraint 21. Then `node supabase/seed.js` (videos row: id=`${VIDEO_ID}`, slug=`${VIDEO_SLUG}`, video_type=`${VIDEO_TYPE}`, title=`${VIDEO_TITLE}`, description=`${VIDEO_HYPOTHESIS}`, `viewer_takeaway`=`${VIEWER_TAKEAWAY}`, `audience_deliverable`=`${AUDIENCE_DELIVERABLE}`, status `draft`) → `node supabase/seed-content.js`, upserting placeholder blocks tagged with `video_id = ${VIDEO_ID}` only — including the three Stage 0 questions, the type-specific block from Constraint 15, the audience-deliverable block, and one placeholder `patterns` row — never touch another video's rows.
7. **Local verify.** Start `python3 -m http.server ${LOCAL_PORT}` if not running; open `http://localhost:${LOCAL_PORT}/html/unknowns.html` in Chrome; click through all 10 pages in stepper order; exercise Snapshot lock, decision checkbox, and a retro save; report console errors and fix them.
8. **Link validation.** Run `.agents/skills/link-validator` and fix or remove dead links.
9. **Docs.** Update `README.md` (new title, new repo/Workers URLs, "For viewers" section per Constraint 18, updated pipeline section with Stages 0–9), `CLAUDE.md` (new repo URL), and add `WEEKLY_CHECKLIST.md`:
   - **Mon** Unknowns (write 3 questions, lock snapshot) + Research → **Tue** Arguments (tick `decision` on every cut) → **Wed** Script + VO → **Thu** Design + Previs → **Fri** Assets + deliverable link + edit handoff → **Sat/Sun** publish → **+7 days** Retro (fill verdict, update Stage 0 statuses, write Formula)
   - each day lists the page to fill, the `seed-content.js` section to edit, and the re-seed command.
10. **Deploy.** `npx wrangler deploy`; record the resulting `*.workers.dev` URL in README (both the header and the "For viewers" section) and in the GitHub Pages redirect.
11. **Publish.** Commit, `gh repo create ${GITHUB_OWNER}/${VIDEO_SLUG} --public --source=. --push`, then open the commit page in Chrome.
12. **Report.** Finish with exactly this block, nothing else after it:

```
## SCAFFOLD REPORT
repo:            https://github.com/${GITHUB_OWNER}/${VIDEO_SLUG}
video_id:        ${VIDEO_ID} (generated, verified unique)
video_type:      ${VIDEO_TYPE} (locked duration: <6min | 3min>)
viewer_takeaway: <stored on videos row → ok/missing>
deliverable:     <title → url | "TODO placeholder">
workers_url:     <url or "skipped: <reason>">
pages_verified:  unknowns | index | arguments | script | design | previsualisation | assets | todo | journal | retro  → <pass/fail each>
loop_checks:     snapshot_lock=<pass/fail> decision_flag=<pass/fail> retro_roundtrip=<pass/fail> patterns_library=<n other videos shown>
console_errors:  <count>
link_validator:  <pass/fail, n fixed>
key_vault_secrets_fetched: <supabase-url | supabase-anon-key | supabase-db-url → ok/missing>
schema_changes:  <list of additive DDL applied, or "none needed">
seed_rows:       videos=<n> content_blocks=<n> patterns=<n> retros=<n> (scoped to video_id=${VIDEO_ID})
secrets_committed: <none | LIST>
next_action_for_erdem: <one line — normally "write the 3 Stage 0 questions and lock the snapshot">
```
