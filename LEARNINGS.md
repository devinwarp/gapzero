# GapZero — Learnings

> Things we found out the hard way (or by reading carefully), so no agent has to rediscover them.
> **Add an entry whenever something surprises you:** a failed assumption, a quirk, a decision reversed. Newest first within each section. Format: `- **YYYY-MM-DD** — finding. → what we do about it.`

## Hackathon rules and judging

- **2026-10-20** — The Setup Guide (PDF) is stricter than the Official Rules: it requires **at least one Meterless Engine in a working feature**, with tests passing, *and* Nightshift as the primary AI provider. Just using the provider prompt doesn't count. → H-MEM is a core feature, with conformance in CI.
- **2026-10-20** — `official-rules.md` is marked WORKING DRAFT; the binding rules are on hackathon.genai.works. → Re-check the live event page before submitting.
- **2026-10-20** — Work committed before 22 Oct 00:00 UTC must be declared. → Only planning docs exist before then, and they're declared. No product code before the window.
- **2026-10-20** — Judges likely won't run Zetaris (registry credentials, 8 GB Docker, 30–45 min install). Sample mode is effectively the product for most judges. → Three run modes (PLAN.md §4); test the empty-`.env` path from a fresh clone.
- **2026-10-20** — The docs' own examples (ChurnWhy, SupplyLens, TriageDesk) will be copied by many teams. → We avoided churn and supplier reconciliation.
- **2026-10-20** — Meterless's starter template claims engine usage is "25% of the rubric". That's from Meterless's own agentathons; here Sponsor Tech is 10/100. → Don't over-invest in engine breadth at the cost of Impact (30).

## Zetaris

- **2026-10-20** — File tables (`CREATE LIGHTNING FILESTORE TABLE`) only read from S3, and **Zetaris always signs S3 requests, so a real AWS key is needed even for public buckets**. It can't read a laptop file. → Sources are Postgres and MySQL (JDBC `CREATE DATASOURCE`) plus a REST table.
- **2026-10-20** — `CREATE DATASOURCE` JDBC syntax is documented, but the getting-started repo marks it "verify against a live instance". → Spike A on D1.
- **2026-10-20** — `CREATE LIGHTNING DATABASE <x> DESCRIBE BY "..."` must exist before any `... FROM <x>`.
- **2026-10-20** — `CREATE SCHEMASTORE CONTAINER` can't be re-run (no `IF NOT EXISTS`; throws `LightningDdlParseException`). → The registration script treats "already exists" as success.
- **2026-10-20** — One SQL statement per request; the editor splits on `;`. Mixed-case or reserved JSON field names need backticks. → The SQL guard enforces a single statement.
- **2026-10-20** — REST JSON shapes: only "top-level object + array of structs" is confirmed working. A top-level array is a real risk. → The ledger API returns `{ "entries": [ ... ] }`.
- **2026-10-20** — REST API: `POST {ZETARIS_REST_URL}/sql-editor/sqls/run`, headers `Authorization: Bearer <key>`, `X-Org-ID: 1` (local), `X-Request-ID: <fresh uuid>`. Body `{queryId, sql, source:"SqlEditor", limit}`. Rows are in `data`. Port 8889 is internal; don't use it. API reference at `localhost:8888/redoc/index.html`.
- **2026-10-20** — Bundled samples are `NY_TAXI` and `TPCDS_DB` (plus `ZETARIS_SEED_TPCH=true` in setup). → `TPCDS_DB` is a fallback source if JDBC fails.
- **2026-10-20** — Images are x86; on Apple Silicon turn on Rosetta in Docker Desktop and expect a slow first start. Docker needs 8 GB (preflight fails below 6 GB).
- **2026-10-21**: The academy "Start Here" lesson is an earlier draft with organiser placeholders, so it differs from the Participant Setup Guide PDF. It uses the `zetaris/Freemium` GitHub repo with Docker Hub login, four containers (`zetaris-db/server/api/gui`), a GUI on **:9001** and default login `dev@account.com`. We follow the PDF route instead: `zetaris-platform.zip` from Zetaris Cloud, the `zetarishackathon.azurecr.io` registry, a GUI on **:3000**, and `admin@zetaris.com`. API :8888 and JDBC :10000 are the same in both. If the zip route breaks, Freemium is the fallback.
- **2026-10-21**: The `zetaris-platform` compose file publishes the UI as `${UI_PORT}:3000` on all interfaces. Setting `UI_PORT=127.0.0.1:3000` binds it to loopback only. The API, JDBC and metastore are already loopback-only, and Caddy runs only under the `tls` profile.

## Meterless

- **2026-10-20** — Engines are **specs plus a TypeScript reference**, not libraries. Conformance: `HMEM_IMPL=/abs/path/index.ts npx tsx conformance/runner.ts`. The module must export an `HMEM` class: `constructor({clock})`, `add()`, `query()`, `feedback()`, plus `store`, `ledger`, `sleep`, `dreaming`, `conflicts`. → TypeScript stack; vendor `engines/hmem/reference` (Apache-2.0) and adapt.
- **2026-10-20** — H-MEM records **must** have `source` and `confidence` (rejected by construction otherwise), plus `provenance`. They support `supersedes`/`supersededBy`, `feedback` deltas and a trust ledger. → Maps directly onto learned reconciliation rules.
- **2026-10-20** — World Model gives content-derived stable IDs (`ent_${sha256(type:normalizedName)}`) and has a `world-to-hmem-sync` example. → Stretch: cross-system customer identity.
- **2026-10-20** — Runtime contract (provider v4): base `http://127.0.0.1:8088/v1`; `Authorization: Bearer <token>` (placeholder `rt-local` is rejected); `meterless-app: <id>` on every call; `meterless-scope: user` on chat; model ids are routing modes (`meterless/auto|fast|deep|cheap|local|fusion`, `meterless/<class>.<disposition>`) read from `GET /v1/models`. `/healthz` only proves the process is up; `/v1/models` 401/403 means unauthorized. 180 s timeout. Auto-detect ports 8088, 8090, 8787, 11434.
- **2026-10-20** — `meterless/auto` may route to a non-NVIDIA model. → Pin a Nemotron-backed lane for award evidence.
- **2026-10-20** — The NVIDIA API key goes **only** in Nightshift's Providers tab, never in our `.env`. Our app only needs the Nightshift URL and runtime token from the Console. Keep the Meterless token out of Cursor's model settings (it would bill Meterless while coding).
- **2026-10-20** — Nightshift Mac editions: "M1+" is free for M1–M4; M5+ is a $199 preorder. → Our M4 Pro uses M1+.
- **2026-10-20** — Useful upstream: `templates/agentathon-starter` (H-MEM notebook agent), `examples/memory-compounding-research` (H-MEM + Markovian), and the CI prompt "Gate every change in CI" (run conformance per engine on PRs).
- **2026-10-21**: The academy lesson describes a hosted Meterless setup with `METERLESS_API_KEY`, `NEMOTRON_MODEL` and credit. The Runtime contract we verified is the local Nightshift setup: a Bearer token from the Console (also written to `<stateDir>/proxy-token`), with routing ids from `/v1/models`. We keep our env names. The point of their smoke test still applies (one reply from a Nemotron lane), and we run it as curl rather than adding Python to the repo.
- **2026-10-21**: Nightshift M1-M4 keeps its runtime token at `~/Library/Application Support/ai.meterless.nightshift.m1m4/runtime-token`. Out of the box (no providers) `/v1/models` lists `meterless/*` modes plus in-browser `webllm/*` models (Llama 3.2 1B/3B, Qwen2.5, Gemma 2, Phi-3.5), and `meterless/fast` routes to `webllm/Llama-3.2-1B`. That is a usable last-resort fallback, but no Nemotron appears until an NVIDIA key is added in Providers.

## Machine (Shameer's Mac)

- **2026-10-20** — M4 Pro, 24 GB RAM, macOS 27, ~109 GB free disk. Docker Desktop has only **7.75 GiB** and other `projectops-dev-*` containers are running. → Raise to 10 GB; stop those containers during the event.
- **2026-10-20** — Installed: Node 26, pnpm 11, bun, deno, Python 3.15, uv, gh (logged in, member of `devinwarp`), Docker 29, Cursor. **No Java** (only needed for JDBC clients; we use REST, so fine).

## Engineering

_(empty: add as we build)_
