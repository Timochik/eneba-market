# AI session log

> **This is a summary, not the full log.** The assignment asks for the *full* conversation (all prompts and responses). Export it from the Claude conversation where this project was built (share link, or copy it into a `.md` file), and add any later sessions.

| | |
|---|---|
| Assistant | Claude Opus 5.5 (Anthropic), as an agent with a shell, GitHub, Gmail, web search and artifact tools |
| Dates | 24–26 Nov 2026 |
| Who wrote the prompts | Aleksey and Tymofii together, in one Claude conversation run from Aleksey's Claude account |
| Where the code lives | first built in a scratch repository, then moved here (`Timochik/eneba-market`) on 26 Nov 2026 |

## What was asked, and what the agent did

| # | Prompt (paraphrased) | What the agent did |
|---|---|---|
| 1 | Find the assignment email and show what the site should look like; publish a preview on GitHub Pages | Found the email with the PDF *Eneba Internship Assignment*, read the requirements, and built a clickable static prototype in Eneba's style: store, filters, typo-tolerant search, product page with seller offers, cart, test checkout with keys, accounts. Tested it in a headless browser and published a preview |
| 2 | Push it to a new GitHub repository | Pushed the code and added a GitHub Pages workflow. The first deploy failed until Pages was enabled in the repo settings |
| 3 | Add a Go backend according to the requirements | Wrote `backend/` using the standard library (plus the `pgx` Postgres driver behind a build tag): catalog filtering with facet counts, fuzzy search, one service per homepage block, PBKDF2 + JWT auth, test-mode payments, transactional checkout that issues keys, orders. It has two stores behind one interface (JSON file and PostgreSQL) with a shared contract test. Connected the frontend to the API with a static fallback, and added CI (tests, real Postgres, Docker smoke test), a Dockerfile and a Render config. CI failed twice (missing `go.sum` entries for pgx, a wrong check in the smoke test); both were fixed |
| 4 | How hard is EN/DE? Explain frontend ↔ backend; compare Go with Java, Node.js and Python | README sections: a sequence diagram, measured Go numbers (≈6 MB binary, <0.3 s start, ≈12 MB RAM), a comparison table including PHP (the assignment allows PHP or Go), and i18n estimates |
| 5 | Prepare everything so Tymofii can take over | `HANDOFF.md`, `ROADMAP.md`, `INTERVIEW_NOTES.md` and this log |
| 6 | Is a database really needed? | Explained why: the assignment requires one, and checkout, accounts and orders need durable, transactional writes. Recommended PostgreSQL |
| 7 | What does Azure offer for Postgres? Suggest an innovation and write it up | Researched Azure (Flexible Server free tier, HorizonDB, retirement of Cosmos DB for PostgreSQL) and wrote `DEPLOY_AZURE.md` with a workflow that publishes the Docker image to GHCR. Proposed "smart search with pills" (query → filter pills, results grouped by game, a "works in your country" mark) and wrote it up in `SMART_SEARCH.md` / `SMART_SEARCH-ru.md`. Restructured `ROADMAP.md` and added `ROADMAP-ru.md` |
| 8 | Comment the tricky places in the Go code | Added comments explaining disjunctive facets, the JWT header pin and constant-time compare, atomic file writes, `FOR UPDATE` row locks and charge-before-write. The review found two real bugs, each fixed with a regression test: price filters dropped a cent (`max=19.99` acted as 19.98), and multi-item checkouts could deadlock in Postgres (lines are now locked in a fixed order). Added `PROMPT_PLAYBOOK.md` with structured prompts for the next steps |
| 9 | Demo steps; create a free Postgres on Azure | Wrote a 5-minute demo script and guided the Azure portal setup field by field. Azure refused the deployment with a subscription policy violation |
| 10 | Use Vercel instead | Vercel's own Postgres is now Neon from the Vercel Marketplace, so the agent guided creating a Neon database (Free plan, Frankfurt). The backend then ran locally against Neon on a laptop |
| 11 | Deploy the app for free | Added `backend/vercel.json` (Go preset, Postgres driver added at build time) and `DEPLOY_VERCEL.md`, and pointed the GitHub Pages site at the Vercel API. Fixed the Pages workflow (separate upload and deploy jobs) after repeated "artifact metadata" failures |
| 12 | Instructions for Windows | `DEPLOY_WINDOWS.md` and `HANDOFF_WINDOWS.md`: PowerShell equivalents (`$env:` instead of `export`, no `&&`, generating a secret without `openssl`) |
| 13 | Move everything into `Timochik/eneba-market` | Renamed the Go module to `github.com/Timochik/eneba-market/backend`, updated all URLs (Pages site, Vercel API, CORS origin), synced the latest docs, ran the tests and pushed. Also created a go-live checklist spreadsheet. The Vercel deployment succeeded |
| 14 | Update this log in the new repository | This file |

## Decisions worth mentioning in the submission

- **Static fallback.** The site works on GitHub Pages with no server, so reviewers always have something to click. When the API is reachable, it switches to live data automatically.
- **Prices are never trusted from the client.** Checkout reads prices and stock inside one database transaction, with row locks.
- **One `Store` interface, two databases, one contract test.** This keeps the demo setup-free without giving up a real database.
- **Deterministic smart search instead of an LLM.** It is cheap and testable, and every rule can be explained.
- **Known gap:** the frontend is not React yet (see `ROADMAP.md`).
