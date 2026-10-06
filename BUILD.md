# Furqan al-Hadith: Build Plan

A nested plan from high-level phases down to concrete tasks. Work top to bottom. Do not start a phase until the "Done when" line of the previous one is true.

---

## 0. Guiding principles

1. **Data model first.** Every later feature (grading, isnad, comparison) depends on it. Mistakes here are expensive to fix.
2. **Read-heavy app.** Hadith content rarely changes. Cache and pre-render aggressively.
3. **Every record is traceable.** Source, edition, numbering system, and grader are stored with the data, not assumed.
4. **Ship vertical slices.** One collection working end to end (ingest, API, UI, search) before adding the next.
5. **Human review gates for comparisons.** No automated match between Sunni and external sources goes live unreviewed.
6. **Respect source terms.** Check licenses and terms of service before ingesting or scraping anything.

---

## 1. Tech stack

### 1.1 Your choices (confirmed good fits)

| Layer | Choice | Why it fits |
|---|---|---|
| API | **FastAPI** (Python 3.12) | Async, auto OpenAPI docs, strong typing, and Python is ideal for scraping and text processing |
| Frontend | **Next.js** (App Router, TypeScript) | SSG/ISR for fast hadith pages, SEO, good RTL and i18n support |
| Database | **PostgreSQL** | Relational model fits books, chapters, narrators, and chains; has full-text search |
| Cache | **Redis** | Cache-aside for API responses, rate limiting, job queue |

### 1.2 Recommended additions

**Backend**
- **SQLAlchemy 2.0 (async) + asyncpg**: ORM and fast Postgres driver
- **Alembic**: database migrations
- **Pydantic v2**: validation and settings (already included with FastAPI)
- **orjson** (`ORJSONResponse`): faster JSON serialization
- **ARQ** (Redis-based async job queue): background ingestion and scraping jobs. Use Celery instead if you want a bigger ecosystem.
- **httpx**: async HTTP client for APIs
- **selectolax or BeautifulSoup**: HTML parsing. Add **Playwright** only for JavaScript-rendered pages.
- **structlog**: structured logging
- **slowapi or a Redis-based limiter**: rate limiting for the public API

**Database extensions**
- **pg_trgm**: fuzzy and partial matching
- **unaccent**: accent-insensitive search for transliterations
- **pgvector**: semantic search and topic matching using multilingual embeddings (very useful for the comparison layer)
- **PgBouncer**: connection pooling once you have real traffic

**Search**
- Start with **Postgres full-text search** on normalized Arabic (see Phase 5). Check whether your Postgres version ships an Arabic text-search configuration; if not, normalize the text yourself and use the `simple` config.
- If search quality or speed is not enough, add **Meilisearch** or **Typesense** (both handle Arabic reasonably well and are easy to run).

**Frontend**
- **Tailwind CSS** with logical properties (`ms-`, `me-`, `ps-`, `pe-`) for RTL
- **next-intl**: i18n and RTL routing
- **TanStack Query**: client-side data fetching and caching for interactive parts
- **shadcn/ui or Radix UI**: accessible components
- **Arabic fonts**: Amiri, Scheherazade New, or Noto Naskh Arabic (self-hosted via `next/font`)
- **Zod**: runtime validation
- **openapi-typescript or Orval**: generate TypeScript types and API client from FastAPI's OpenAPI schema, so the two sides never drift

**Infrastructure and tooling**
- **Docker + docker-compose**: identical local, CI, and production environments
- **GitHub Actions**: CI/CD
- **Cloudflare** (CDN, DNS, WAF): edge caching and DDoS protection
- **Sentry**: error tracking
- **OpenTelemetry + Prometheus/Grafana** (or a managed alternative): metrics and tracing
- **pytest, pytest-asyncio, Vitest, Playwright**: testing
- **Ruff + mypy** (Python), **ESLint + Prettier** (TypeScript), **pre-commit**: code quality

**Hosting options (pick one pattern)**
- Next.js on **Vercel**; FastAPI, Redis on **Fly.io / Railway / Hetzner VPS**; Postgres on **Neon / Supabase / managed Postgres**
- Or everything self-hosted on one VPS with Docker for the lowest cost

### 1.3 Performance boosters (the short list)

1. **Static generation with ISR** for hadith, chapter, and collection pages, with on-demand revalidation when data changes
2. **CDN edge caching** (Cloudflare) in front of both Next.js and the API for GET requests
3. **Redis cache-aside** for hot API responses, with TTLs and key versioning
4. **HTTP caching headers**: `ETag`, `Cache-Control`, `stale-while-revalidate`
5. **Keyset (cursor) pagination** instead of `OFFSET`, which is slow on large tables
6. **Targeted indexes**: composite, partial, and GIN indexes for search
7. **Materialized views** for heavy aggregations (counts per collection, topic stats)
8. **Avoid N+1 queries**: use `selectinload`/`joinedload`; log slow queries with `pg_stat_statements`
9. **Brotli/gzip compression** at the edge
10. **PgBouncer** to cap database connections
11. **Lazy-load** heavy content (tafsir, long isnads) and **code-split** the frontend
12. **PWA with service worker** for offline reading and repeat-visit speed
13. **Denormalized read model** (a pre-joined `hadith_view` or JSONB column) for the main hadith response

---

## 2. Repository structure (monorepo)

```
furqan-al-hadith/
├── apps/
│   ├── api/                  # FastAPI
│   │   ├── app/
│   │   │   ├── main.py
│   │   │   ├── core/         # config, security, logging, cache
│   │   │   ├── db/           # models, session, migrations (alembic)
│   │   │   ├── schemas/      # Pydantic models
│   │   │   ├── routers/      # hadith, quran, search, topics, compare, admin
│   │   │   ├── services/     # business logic
│   │   │   ├── repositories/ # DB queries
│   │   │   └── workers/      # ARQ jobs
│   │   └── tests/
│   └── web/                  # Next.js
│       ├── app/[locale]/...
│       ├── components/
│       ├── lib/              # generated API client
│       └── tests/
├── pipelines/                # data ingestion (separate from the API runtime)
│   ├── sunni/                # loaders for each collection
│   ├── connectors/           # external-source connectors (API/scrape)
│   ├── normalize/            # Arabic normalization, cleaning
│   └── validate/             # data quality checks
├── data/
│   ├── raw/                  # untouched source files (gitignored if large)
│   ├── processed/
│   └── SOURCES.md
├── docs/                     # METHODOLOGY.md, API.md, ADRs
├── infra/                    # docker, compose, CI, deploy
├── CONTRIBUTING.md
├── LICENSE
└── README.md
```

---

## 3. Phases

### Phase 0: Project setup and infrastructure

**Goal:** a running skeleton where every part talks to every other part.

- **0.1 Repo and tooling**
  - [ ] Create the repo, branch protection on `main`, PR template, issue templates
  - [ ] Add `LICENSE`, `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`
  - [ ] Set up pre-commit (Ruff, mypy, ESLint, Prettier)
- **0.2 Local environment**
  - [ ] `docker-compose.yml` with Postgres (with extensions), Redis, API, web
  - [ ] `.env.example` and typed settings in the API (Pydantic Settings)
  - [ ] `Makefile` or task runner: `make up`, `make migrate`, `make test`, `make seed`
- **0.3 API skeleton**
  - [ ] FastAPI app with `/health` and `/version`
  - [ ] Async SQLAlchemy session, Alembic configured
  - [ ] Redis connection and a cache helper
  - [ ] Structured logging, request IDs, global error handler
- **0.4 Web skeleton**
  - [ ] Next.js (App Router, TypeScript), Tailwind, next-intl with `en` and `ar` (RTL)
  - [ ] Layout with header, footer, theme (light/dark), Arabic fonts
  - [ ] Generated API client from OpenAPI, wired into the build
- **0.5 CI**
  - [ ] GitHub Actions: lint, type-check, unit tests, build, Docker image
  - [ ] Preview deployments for PRs (optional)
- **0.6 Legal and data audit (do this early)**
  - [ ] Read the licenses of `hapiam/hadith-json` and `AhmedBaset/hadith-json`; record attribution requirements in `data/SOURCES.md`
  - [ ] Decide which Arabic text and translations you are licensed to use

**Done when:** `make up` starts everything, CI is green, and the web page shows a value fetched from the API.

---

### Phase 1: Data model and ingestion foundation

**Goal:** a schema that can hold every Sunni collection plus gradings, and a repeatable ingestion pipeline. Prove it with the Forty Hadith Qudsi.

- **1.1 Schema design** (see section 4 for the table list)
  - [ ] Write the ERD and review it before coding
  - [ ] Implement core tables: `collections`, `books`, `chapters`, `hadith`, `hadith_texts`, `hadith_references`
  - [ ] Implement `narrators`, `hadith_chain_links` (isnad), `graders`, `gradings`
  - [ ] Implement `translations` and `translators` (with license field)
  - [ ] Add indexes, foreign keys, and constraints (unique per collection + numbering system + number)
  - [ ] First Alembic migration
- **1.2 Numbering systems**
  - [ ] `numbering_systems` table (e.g., Fath al-Bari, Dar-us-Salam, USC-MSA, Abdul-Baqi)
  - [ ] Every hadith can have several reference numbers, one per system
  - [ ] Mark the default system per collection
- **1.3 Arabic text handling**
  - [ ] Store **two** Arabic forms: `text_original` (with tashkeel) and `text_normalized` (for search)
  - [ ] Normalization function: strip diacritics, unify alef/hamza forms, ya/alef maqsura, ta marbuta, remove tatweel
  - [ ] Unit tests with real examples
- **1.4 Ingestion pipeline**
  - [ ] Loader interface: `extract → transform → validate → load`
  - [ ] Idempotent loads (re-running does not duplicate)
  - [ ] Raw files kept in `data/raw/` with a checksum and source URL
  - [ ] Validation rules: no empty Arabic text, numbering gaps reported, encoding checks, duplicate detection
  - [ ] Load report after every run (counts, errors, warnings)
- **1.5 First dataset: Forty Hadith Qudsi**
  - [ ] Write a loader for `qudsi40.json`
  - [ ] Verify all 40 against a second source by hand (sample at least 10)
  - [ ] Record source and license in `data/SOURCES.md`

**Done when:** the Qudsi 40 are in Postgres, re-running the loader changes nothing, and the validation report is clean.

---

### Phase 2: Core API

**Goal:** a fast, documented, cached read API.

- **2.1 Endpoints (v1, all under `/api/v1`)**
  - [ ] `GET /collections` and `GET /collections/{slug}`
  - [ ] `GET /collections/{slug}/books` and `/books/{id}/chapters`
  - [ ] `GET /hadith/{id}` and `GET /collections/{slug}/hadith/{number}?system=`
  - [ ] `GET /hadith` (list with filters: collection, book, chapter, grade, narrator) using cursor pagination
  - [ ] `GET /hadith/random` and `GET /hadith/daily`
  - [ ] `GET /narrators/{id}`
- **2.2 Response design**
  - [ ] Pydantic response models, consistent error format, versioned
  - [ ] Include grading(s), references in all numbering systems, and source info in every hadith response
  - [ ] `lang` parameter for translation selection
- **2.3 Caching**
  - [ ] Cache-aside helper with key pattern `v1:hadith:{id}:{lang}`
  - [ ] TTLs: long for hadith content, shorter for lists
  - [ ] Cache invalidation on data load (bump a version key)
  - [ ] `ETag` and `Cache-Control` headers
- **2.4 Protection**
  - [ ] Rate limiting by IP via Redis
  - [ ] CORS configured for the web origin
  - [ ] Input validation and max page sizes
- **2.5 Quality**
  - [ ] Unit tests for services, integration tests with a test database
  - [ ] Load test one endpoint with `k6` or `locust` to set a baseline

**Done when:** all endpoints return Qudsi data, OpenAPI docs are accurate, and the p95 for a cached hadith request is well under 100 ms.

---

### Phase 3: Frontend core

**Goal:** a good reading experience in Arabic and English.

- **3.1 Pages**
  - [ ] Home (daily hadith, collections list, search box)
  - [ ] Collection page, book/chapter pages, hadith page (`/[locale]/hadith/[collection]/[number]`)
  - [ ] About, Methodology, Sources, Contact
- **3.2 Rendering strategy**
  - [ ] SSG/ISR for hadith and collection pages with on-demand revalidation
  - [ ] Dynamic rendering only for search and user features
  - [ ] `generateMetadata` for SEO (title, description, canonical, Open Graph)
  - [ ] Structured data (JSON-LD) where useful
- **3.3 Hadith display component**
  - [ ] Arabic (large, readable font, optional tashkeel toggle) with English translation
  - [ ] Narrator, collection, book, chapter, numbering system selector
  - [ ] Grading badge with grader name and tooltip explaining the grade
  - [ ] Copy, share, and "cite this" (generates a proper citation string)
- **3.4 UX details**
  - [ ] Full RTL support and bidirectional text handling
  - [ ] Font size controls, light/dark mode
  - [ ] Keyboard navigation and accessibility (WCAG 2.1 AA)
  - [ ] Mobile-first layout
- **3.5 Quality**
  - [ ] Component tests, Playwright smoke tests (home → collection → hadith)
  - [ ] Lighthouse CI budget: Performance, Accessibility, SEO above 90

**Done when:** a user can browse the Qudsi 40 in Arabic and English on mobile and desktop, pages are statically served, and Lighthouse targets pass.

---

### Phase 4: Grading and verification system

**Goal:** make authenticity visible, sourced, and consistent. This is the core of the app.

- **4.1 Grading data**
  - [ ] Grade vocabulary: `sahih`, `hasan`, `da'if`, `da'if jiddan`, `mawdu'`, plus `unknown`
  - [ ] Each grading row stores: hadith, grader, grade, source (book/page/URL), date added
  - [ ] Multiple gradings per hadith supported
  - [ ] Seed `graders` (al-Albani, Ibn Hajar, al-Dhahabi, Shu'ayb al-Arna'ut, at-Tirmidhi, others you decide)
- **4.2 Grading sources**
  - [ ] Decide the primary grading source per collection (for Bukhari and Muslim: the book's own status, with note)
  - [ ] Write the policy in `docs/METHODOLOGY.md`: whose grading you show first and why
  - [ ] Import gradings from data sources that provide them; flag anything unsourced
- **4.3 Display**
  - [ ] Grading badge, plus a "see all gradings" panel
  - [ ] Clear message when a hadith has no grading ("not yet graded in this app")
  - [ ] Filters: show only sahih, hide da'if, etc.
- **4.4 Isnad and narrators**
  - [ ] Store chain order in `hadith_chain_links`
  - [ ] Narrator pages (name, death date if known, biographical sources, reliability verdicts when sourced)
  - [ ] Isnad visualization (simple chain view first, graph later)
- **4.5 Review tooling**
  - [ ] Admin script or panel to list ungraded or disputed hadith
  - [ ] Report-an-error button writing to an `issues` table

**Done when:** every hadith shows a grade with a named source, or an explicit "not yet graded" state.

---

### Phase 5: Search

**Goal:** fast search that works for Arabic and English, with transliteration tolerance.

- **5.1 Postgres search (first version)**
  - [ ] `tsvector` column for normalized Arabic and one for English translation
  - [ ] GIN indexes
  - [ ] Query normalization using the same Arabic normalizer as ingestion
  - [ ] Ranking with `ts_rank` plus exact-match boosts
- **5.2 Fuzzy and partial**
  - [ ] `pg_trgm` index for typo tolerance and transliterated names
  - [ ] Search by hadith number ("Bukhari 1", "Muslim 8")
  - [ ] Search by narrator name
- **5.3 API and UI**
  - [ ] `GET /search?q=&collection=&grade=&lang=` with cursor pagination
  - [ ] Highlighted snippets
  - [ ] Autocomplete for collections and narrators (cached in Redis)
  - [ ] Filters: collection, grade, narrator, chapter
- **5.4 Evaluate**
  - [ ] Build a test set of 30 to 50 queries with expected results
  - [ ] Measure quality and latency; if not enough, add Meilisearch or Typesense behind the same API
- **5.5 Semantic search (optional, later)**
  - [ ] Generate multilingual embeddings for each hadith, store in `pgvector`
  - [ ] Hybrid ranking: keyword score + vector similarity

**Done when:** your test set passes and typical search latency is under 200 ms.

---

### Phase 6: Scale Sunni hadith content

**Goal:** add collections one at a time using the proven pipeline.

Order: **Forty Hadith an-Nawawi → Sahih al-Bukhari → Sahih Muslim → Sunan Abi Dawud → Jami' at-Tirmidhi → Sunan an-Nasa'i → Sunan Ibn Majah → Muwatta Malik → Musnad Ahmad → Sunan ad-Darimi → Riyad as-Salihin**

For **each** collection, repeat this checklist:

- **6.x.1 Source and license**
  - [ ] Confirm a data source and its license; add to `data/SOURCES.md`
- **6.x.2 Ingest**
  - [ ] Write or adapt the loader
  - [ ] Map the numbering systems
  - [ ] Load into staging, then run validation (counts match the known total, no gaps beyond known ones)
- **6.x.3 Verify**
  - [ ] Manually spot-check at least 20 random hadith against a trusted print or site
  - [ ] Check chapter titles and book structure
- **6.x.4 Grading**
  - [ ] Import or add gradings (mandatory for the Sunan, Muwatta, Musnad Ahmad, and Darimi before release)
- **6.x.5 Publish**
  - [ ] Promote from staging, bump cache version, regenerate ISR pages
  - [ ] Re-run search quality tests
  - [ ] Announce in the changelog

**Special notes**
- Bukhari and Muslim have multiple numbering systems; store all you can map.
- Ibn Majah, Abu Dawud, Tirmidhi, and an-Nasa'i must not go live without gradings.
- Musnad Ahmad is very large; load it in batches and watch performance.

**Done when:** each collection passes validation and spot-checks, and the site stays fast with the full dataset.

---

### Phase 7: Quran and tafsir

**Goal:** the Quran as the primary source, linked with hadith.

- **7.1 Quran data**
  - [ ] Choose the Uthmani text source and record the license (e.g., Tanzil)
  - [ ] Tables: `surahs`, `ayat`, `ayah_translations`
  - [ ] Pages: surah list, surah reading view, single ayah view
- **7.2 Tafsir**
  - [ ] Add tafsir sources one at a time (at-Tabari, Ibn Kathir, al-Qurtubi, as-Sa'di) with license checks
  - [ ] Lazy-load tafsir on demand in the UI
- **7.3 Linking**
  - [ ] `hadith_ayah_links` table (hadith mentions or explains an ayah)
  - [ ] Show related hadith on ayah pages and related ayat on hadith pages
  - [ ] Links must be manually reviewed or sourced from a tafsir, not guessed

**Done when:** a reader can open any ayah, see translation and tafsir, and follow reviewed links to hadith.

---

### Phase 8: Topics and taxonomy

**Goal:** a topic system that the comparison layer can use.

- **8.1 Taxonomy**
  - [ ] Define a hierarchical topic tree (e.g., Tawhid → Names and Attributes; Prophethood; Companions; Prayer; etc.)
  - [ ] `topics` table with parent_id, Arabic and English names, descriptions
  - [ ] Review the topic list with someone knowledgeable before building on it
- **8.2 Tagging**
  - [ ] `hadith_topics` and `ayah_topics` tables with `source` (manual, imported, suggested) and `reviewed` flag
  - [ ] Import existing chapter titles as a starting point
  - [ ] Suggest tags with embeddings, but mark as unreviewed
- **8.3 Review workflow**
  - [ ] Admin queue of suggested tags to approve or reject
  - [ ] Only reviewed tags are visible publicly
- **8.4 UI**
  - [ ] Topic browse page, topic detail page listing Quran verses and hadith for the topic

**Done when:** at least 15 to 20 core topics have reviewed, well-sourced Quran and hadith entries.

---

### Phase 9: Comparison layer (external sources)

**Goal:** show other communities' own published sources next to the Sunni evidence for the same topic, accurately and in context. No self-built Shia or Ahmadiyya database.

- **9.1 Source registry**
  - [ ] `external_sources` table: name, community, base URL, access method (`api`, `scrape`, `manual-link`), license/terms notes, last reviewed date
  - [ ] For each candidate source, record: official or not, has API, terms of service summary, robots.txt status
  - [ ] Only enable sources whose terms allow your use
- **9.2 Legal and ethics checkpoint**
  - [ ] Read each site's terms and `robots.txt`
  - [ ] Decide per source: API, permitted scraping, or link-out only
  - [ ] Policy: short cited passages plus a link, never full-book copying
  - [ ] Optionally contact site owners for permission
- **9.3 Connectors** (`pipelines/connectors/`)
  - [ ] Base connector interface: `search(topic)`, `fetch(reference)`, `normalize(raw)`
  - [ ] API connectors first (more stable), then scrapers
  - [ ] Scraper rules: identify your user agent, respect `robots.txt`, slow rate limits, retry with backoff, cache raw pages
  - [ ] Run connectors as ARQ background jobs, never on user requests
  - [ ] Contract tests with saved sample responses (so layout changes are detected)
  - [ ] Monitoring: alert when a connector returns zero results or changes shape
- **9.4 Storage of external passages**
  - [ ] `external_passages` table: source, work title, volume, page/number, edition, original text, translation, URL, fetched date, content hash
  - [ ] Store the **minimum** needed (the cited passage and its immediate context), not whole works
  - [ ] Re-fetch periodically and flag changed content
- **9.5 Topic matching**
  - [ ] Map each topic to search terms in the external source's own vocabulary (manual, per source)
  - [ ] Optional: embedding similarity to propose candidate passages
  - [ ] Every proposed match goes into a **review queue**
- **9.6 Review workflow**
  - [ ] Reviewer sees: the passage, surrounding context, the Sunni entry it is matched to, and the source link
  - [ ] Checklist: accurate translation, not out of context, correct citation, relevant to the topic
  - [ ] Statuses: `proposed`, `approved`, `rejected`, `needs-context`
  - [ ] Only `approved` links are public; keep an audit log
- **9.7 Comparison UI**
  - [ ] Topic page with columns or tabs: Sunni sources vs. external community's sources
  - [ ] Each external quote shows book, volume, number or page, edition, context, and a "view original" link
  - [ ] Clear labeling of whose source each entry comes from
  - [ ] Report button: "this match is wrong or out of context"
- **9.8 Maintenance**
  - [ ] Scheduled link checker and content-change detector
  - [ ] Dashboard of source health

**Done when:** one pilot topic is fully compared using at least one external source, with every entry reviewed, cited, and linked back.

---

### Phase 10: User features and offline

**Goal:** features that bring readers back.

- **10.1 Accounts (only if needed)**
  - [ ] Choose auth (Auth.js, Clerk, or your own JWT)
  - [ ] Privacy: collect minimal data, document it
- **10.2 Personal features**
  - [ ] Bookmarks, collections of favorites, personal notes
  - [ ] Reading history and "continue reading"
  - [ ] Daily hadith (web push or email, opt-in)
- **10.3 Sharing**
  - [ ] Share cards (image generation with correct Arabic shaping)
  - [ ] Permalink and citation copy
- **10.4 PWA and offline**
  - [ ] Web app manifest and service worker
  - [ ] Offline cache for recently read hadith and chosen collections
- **10.5 Mobile apps (optional, later)**
  - [ ] Evaluate PWA vs. React Native/Flutter using the same API

**Done when:** a signed-in user can save and sync bookmarks, and the PWA works offline for downloaded collections.

---

### Phase 11: Hardening and launch

**Goal:** safe, fast, observable, and ready for the public.

- **11.1 Performance**
  - [ ] Load test with realistic traffic; fix top slow queries (`pg_stat_statements`, `EXPLAIN ANALYZE`)
  - [ ] Add PgBouncer and tune connection pools
  - [ ] Confirm CDN cache hit rate and cache headers
  - [ ] Review Redis memory use and eviction policy
- **11.2 Security**
  - [ ] Dependency scanning (Dependabot, `pip-audit`, `npm audit`)
  - [ ] Secrets in a secret manager, never in the repo
  - [ ] Security headers (CSP, HSTS), HTTPS only
  - [ ] Rate limits and abuse protection on all public endpoints
  - [ ] Admin routes protected and audited
- **11.3 Reliability**
  - [ ] Automated Postgres backups and a **tested** restore
  - [ ] Health checks and uptime monitoring
  - [ ] Zero-downtime migrations plan
  - [ ] Error tracking (Sentry), logs, metrics dashboards, alerts
- **11.4 Accessibility and content QA**
  - [ ] Screen reader testing, color contrast, keyboard-only use
  - [ ] Native Arabic speaker review of UI strings
  - [ ] Scholar review of methodology page and topic wording
- **11.5 Launch**
  - [ ] Soft launch to a small group for feedback
  - [ ] Public changelog, roadmap, and contact/report channel
  - [ ] Plan for ongoing data corrections

---

## 4. Core data model (reference)

| Table | Key columns / notes |
|---|---|
| `collections` | id, slug, name_ar, name_en, author, type (`sahih`, `sunan`, `musnad`, `compilation`), description |
| `books` | id, collection_id, number, name_ar, name_en |
| `chapters` | id, book_id, number, title_ar, title_en |
| `hadith` | id, collection_id, book_id, chapter_id, position, narrator_summary, is_qudsi |
| `hadith_texts` | hadith_id, text_original, text_normalized, tsvector |
| `translations` | id, hadith_id, lang, text, translator_id, tsvector |
| `translators` | id, name, license, source_url |
| `numbering_systems` | id, collection_id, name, is_default |
| `hadith_references` | hadith_id, numbering_system_id, number (unique per system) |
| `narrators` | id, name_ar, name_en, birth, death, kunya, notes |
| `hadith_chain_links` | hadith_id, narrator_id, position |
| `graders` | id, name, era, bio_url |
| `gradings` | id, hadith_id, grader_id, grade, source_ref, source_url, added_at |
| `surahs`, `ayat`, `ayah_translations` | Quran text and translations |
| `tafsir_works`, `tafsir_entries` | tafsir per ayah |
| `hadith_ayah_links` | hadith_id, ayah_id, relation, reviewed |
| `topics` | id, parent_id, slug, name_ar, name_en, description |
| `hadith_topics`, `ayah_topics` | item, topic_id, source, reviewed |
| `external_sources` | id, name, community, base_url, access_method, terms_notes, enabled |
| `external_passages` | id, source_id, work, volume, page_or_number, edition, text_original, text_translation, url, content_hash, fetched_at |
| `topic_external_links` | topic_id, passage_id, status, reviewer_id, reviewed_at, notes |
| `issues` | id, target_type, target_id, message, status |
| `users`, `bookmarks`, `notes` | user features (Phase 10) |
| `audit_log` | who changed what and when (reviews, gradings, imports) |

---

## 5. Caching strategy

| Content | Where cached | TTL / invalidation |
|---|---|---|
| Hadith, collection, chapter pages | Next.js ISR + CDN | Revalidate on data load |
| API hadith responses | Redis + CDN | Long TTL; version key bumped on load |
| API list endpoints | Redis | Short TTL (minutes) |
| Search results | Redis | Short TTL, keyed by normalized query |
| Autocomplete | Redis | Medium TTL |
| Daily hadith | Redis | Until midnight (UTC or chosen timezone) |
| External passages | Postgres (stored) + Redis | Re-fetch on schedule; flag changes |
| Rate limiting counters | Redis | Sliding window |

Use a global cache **version prefix** (e.g., `v7:`) so a data release invalidates everything with one change.

---

## 6. Testing strategy

- **Unit:** normalization, parsers, services, citation formatting
- **Integration:** API against a real Postgres (use Testcontainers or compose)
- **Data tests:** counts per collection, no duplicate numbers, every hadith has Arabic text and a reference
- **Connector contract tests:** saved responses from each external source
- **End to end:** Playwright flows (browse, search, view grading, compare topic)
- **Performance:** k6/locust baselines kept in CI as non-blocking checks
- **Accessibility:** axe checks in CI

---

## 7. Risks and mitigations

| Risk | Mitigation |
|---|---|
| Wrong or inconsistent hadith data from third-party JSON | Validate, spot-check against print editions, keep source and checksum |
| Numbering mismatches vs. printed books | Store multiple numbering systems; show which one is displayed |
| Missing or inconsistent gradings | Explicit "not yet graded" state; publish grading policy |
| Scraper breaks or violates terms | Prefer APIs, contract tests, link-out fallback, legal check per source |
| Out-of-context or mistaken comparisons | Mandatory human review, show context, report button, audit log |
| Copyright issues with translations or books | Record licenses per dataset; show excerpts and link out |
| Arabic search quality | Normalize consistently, test set, upgrade to Meilisearch/Typesense if needed |
| Slow pages with large datasets | ISR, CDN, indexes, keyset pagination, load tests |
| Scope creep | Finish each phase's "Done when" before starting the next |

---

## 8. Suggested timeline (adjust to your pace)

| Phase | Rough effort (solo) |
|---|---|
| 0. Setup | 1 week |
| 1. Data model and ingestion | 2 weeks |
| 2. Core API | 1 to 2 weeks |
| 3. Frontend core | 2 to 3 weeks |
| 4. Grading and verification | 2 weeks |
| 5. Search | 1 to 2 weeks |
| 6. Sunni collections | 1 to 2 weeks per major collection |
| 7. Quran and tafsir | 2 to 3 weeks |
| 8. Topics | 2 weeks plus ongoing review |
| 9. Comparison layer | 4 to 6 weeks |
| 10. User features | 2 to 4 weeks |
| 11. Hardening and launch | 2 to 3 weeks |

Treat these as planning guesses, not promises. Data verification and review usually take longer than coding.
