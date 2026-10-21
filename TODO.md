# GapZero — TODO

> Source of truth for what's next. Agents: take the **top unchecked** item in the current phase, mark it `[~]`, and tick it `[x]` when merged (add the PR number).
> Legend: `[ ]` todo · `[~]` in progress · `[x]` done · `[-]` dropped (say why) · 👤 = needs Shameer (account, keys, GUI)
> **Hard deadline: 27 Oct 2026 23:45 UTC. Our target: submitted by 27 Oct 18:00 UTC.**

## Phase 0 — Environment (before 22 Oct 00:00 UTC, no product code)

- [x] Read all hackathon docs and Meterless + Zetaris repos (see LEARNINGS.md)
- [x] Choose idea (GapZero) and track (The Agent That Can Explain Why)
- [x] PLAN.md, TODO.md, LEARNINGS.md, AGENTS.md, .env.example, .gitignore
- [x] Create public repo `devinwarp/gapzero` and push the planning docs (declared as pre-existing) — c4095c3
- [ ] 👤 Docker Desktop: memory 10 GB, CPUs 4+, **Use Rosetta** on; stop `projectops-dev-*` containers during the hackathon
- [x] 👤 Register at Zetaris Cloud, download `zetaris-platform.zip` (unzipped to ~/zetaris/zetaris-platform; admin password set in its .env, chmod 600)
- [~] Zetaris: `docker login` (👤 needs the Run-locally command), `./preflight.sh`, `docker compose pull && up -d`
- [ ] Zetaris: `./verify.sh` 9/9, `SELECT 1`, Pikachu query returns 2 rows
- [ ] 👤 Zetaris: create an API key in the web UI and put it in our `.env` (never paste it into chat)
- [~] 👤 Nightshift: installed to /Applications ("Nightshift M1-M4.app"); 👤 launch it, verify email; runtime not yet answering on :8088
- [ ] 👤 NVIDIA: build.nvidia.com key, then Nightshift Providers tab, then confirm Nemotron models are unlocked
- [ ] Nightshift: Lane 1 fast `nemotron-3.5-lightning-30b-a3b`, Lane 2 deep `nemotron-3.5-super-vl-120b-a12b` (or the closest available)
- [ ] 👤 Copy the Nightshift Console URL and runtime token into our `.env`
- [ ] Setup smoke test (curl): Zetaris UI :3000, `SELECT 1` over REST :8888, one Nemotron reply via Nightshift `/v1/chat/completions`
- [ ] 👤 Cursor: signed in, credit active, open this repo in Cursor (required dev environment)
- [ ] 👤 Attend participant onboarding, 21 Oct 12:00–13:30 UTC
- [ ] 👤 Watch the Data Day recording; check HackOS for Build/Power Day dates and NVIDIA credit instructions
- [ ] 👤 HackOS: profile complete, solo selected, track chosen

## Phase 1 — D1 · 22 Oct: scaffold, CI, spikes

- [ ] Scaffold: `package.json` (pnpm, ESM), `tsconfig`, Biome, Vitest, `src/config.ts` (zod-validated env)
- [ ] CI `ci.yml`: install, lint, typecheck, test, gitleaks, `pnpm audit --prod`; `codeql.yml`
- [ ] Branch protection on `main`: CI required, squash merge only, delete branch on merge
- [ ] **Spike A:** Postgres + MySQL in compose; Zetaris `CREATE DATASOURCE` (JDBC) and `REGISTER DATASOURCE TABLE`; query both
- [ ] **Spike B:** ledger REST service registered as a Zetaris REST table (confirmed JSON shape: top-level object + array of structs)
- [ ] **Spike C:** vendor the H-MEM reference; `HMEM_IMPL=... npx tsx conformance/runner.ts` passes in CI
- [ ] Record spike outcomes in LEARNINGS.md; switch to the fallback in PLAN.md §9 if any spike fails
- [ ] Simulator v1: seeded customers, orders, invoices, payments, refunds, deals, journal entries; CSV out
- [ ] Simulator faults (first 5): tz cutoff, refund not posted, duplicate deal, FX rounding, renamed customer
- [ ] Ground truth to `eval/`; test that `src/` never imports or reads `eval/`

## Phase 2 — D2 · 23 Oct: adapters and retrieval

- [ ] `adapters/zetaris.ts`: REST client (Bearer, X-Org-ID, fresh X-Request-ID), timeout, retry, one statement per call
- [ ] `adapters/duckdb.ts`: same interface over CSVs for `SAMPLE_MODE`
- [ ] `adapters/sqlGuard.ts`: SELECT/WITH only, single statement, table allowlist, LIMIT cap, plus tests (injection cases)
- [ ] `adapters/meterless.ts`: OpenAI client to the Runtime; `meterless-app` and `meterless-scope` headers; `/v1/models` discovery; 180 s timeout; structured errors
- [ ] `trace.ts`: JSONL per run plus readable stdout; secrets redacted; test that redaction works
- [ ] `src/metrics.ts`: metric cards (CRM closed-won bookings, billing net invoiced, ledger revenue postings) per period/region/segment
- [ ] Generic reconciliation primitives: key match, ±N-day window, amount match (exact/rounded/FX tolerance), normalised-name entity resolution, duplicate detection (used by offline mode and offered as tools in LLM mode)
- [ ] Retriever: schema cards, SQL generation, error-feedback self-correction (max 3), traced
- [ ] Simulator faults (remaining 3–5): currency mismatch, booked-not-invoiced, partial credit notes, test accounts, tax-inclusive

## Phase 3 — D3 · 24 Oct: the agent loop

- [ ] Shared run state (zod schema): gap, hypotheses, claims, attributed record IDs, residual
- [ ] Planner: gap measurement, hypothesis ranking, re-plan on Verifier feedback
- [ ] Verifier: sum check, disjointness check, spot-check sampling; blocks unsupported claims
- [ ] Writer: bridge data, evidence table, caveats, recommended actions
- [ ] Orchestrator: bounded state machine (tolerance, iteration cap, time budget), escalation with residual
- [ ] `POST /run` (zod input, 400 on bad input, structured errors) and `cli.ts`
- [ ] Eval harness `pnpm eval --seeds N`: cause recall/precision, $ accuracy, iterations; baseline recorded in LEARNINGS.md

## Phase 4 — D4 · 25 Oct: memory, packaging, judge path

- [ ] H-MEM integration: Planner recalls rules first; Verifier sends feedback; rule supersession; persistence in `.gapzero/`
- [ ] Transfer demo: a rule learned on period/region A cuts iterations on B (measured in eval)
- [ ] `Dockerfile` (node:24-slim, deps at build time, binds 0.0.0.0, `logs/` created)
- [ ] `SAMPLE_MODE=true` full run with no credentials and no network: DuckDB data + **no-model fallback** (template SQL per hypothesis family, heuristic planner; the Verifier is maths either way). Labelled in output and trace as `mode: offline`
- [ ] If a Meterless URL and token are present, sample mode uses the real model over DuckDB data (`mode: sample+llm`)
- [ ] **Fresh-clone validation** (disqualification-checklist §14 steps 1–9, including step 9 with an empty `.env`)
- [ ] Stretch: World Model engine for cross-system customer identity

## Phase 5 — D5 · 26 Oct: product, docs, hardening

- [ ] Web page: run form, waterfall filling live, evidence table, trace panel (design with `impeccable` + `design-taste-frontend`)
- [ ] `input_examples/` ×3+ and `output_examples/` ×3+ (real saved outputs, never read at runtime)
- [ ] `pnpm perf`: p95 run time and per-query latency budgets; results in README
- [ ] Security pass: gitleaks on full history, `pnpm audit`, CodeQL clean, SQL guard fuzz, no secrets in logs or screenshots
- [ ] README complete (see deliverables below)
- [ ] Vercel decision: replay viewer at `gapzero.infinitebarkah.com`, or skip

## Phase 6 — D6 · 27 Oct: submit (target 18:00 UTC)

- [ ] Demo video recorded, ≤4 min, secrets check frame by frame
- [ ] Slide deck finished
- [ ] Final fresh-clone validation on `main`
- [ ] 👤 Submission form completed on HackOS; links tested in a private window
- [ ] Backup copies of video (second host, local, README link)

---

## Submission deliverables (must all be ✅ before submitting)

### Required components
- [ ] **Demo video, 1–4 min** (aim 3:00–3:30), MP4 1080p, audio, named `OAH2026-GapZero-ExplainWhy-Demo.mp4`, link opens without permission. Shows: problem/user/track → solution → agents + collaboration → live run → Zetaris + Meterless doing real work → output + impact
- [ ] **Slide deck**, all 8 points: (1) problem + target users (2) solution (3) agent workflow/architecture (4) tech stack (5) how Zetaris + Meterless were used (6) what was unique about the sponsor-tech use (7) product visuals (8) future enhancements
- [ ] **Public GitHub repo** with README covering setup, usage and dependencies
- [ ] **Sponsor technology explanation:** Zetaris, Meterless (Nightshift + H-MEM), Cursor (+ Claude Code alongside), NVIDIA Nemotron, plus what was unique
- [ ] **Pre-existing components declared:** planning docs (PLAN/TODO/LEARNINGS/AGENTS, env templates) written 20–21 Oct; vendored H-MEM reference (Apache-2.0, open source)
- [ ] **Project details:** name, one-line + short description, solo member, track

### README must include
- [ ] Track, problem, user, one-line value
- [ ] Architecture diagram + agent table (role, can block/retry)
- [ ] Setup (Docker + local), usage (`curl POST /run`, CLI), dependencies
- [ ] `SAMPLE_MODE` instructions (no credentials)
- [ ] Sponsor technology section; AI and model usage disclosure (models, frameworks, tools, data, adapted code, limits, fallback)
- [ ] Data section: synthetic, seeded, labelled; licences
- [ ] Failure handling table; limitations; third-party licences
- [ ] Demo video link; pre-existing components

### Quality bar (from strong-submission and disqualification checklists)
- [ ] Runs from a clean clone with documented commands; no manual steps
- [ ] Programmatic entry point; structured in/out; useful errors, never a hang
- [ ] JSONL traces in `logs/` + stdout show handoffs, critique, retries, Zetaris + Meterless calls
- [ ] Outputs differ across inputs and seeds; nothing read from `output_examples/`
- [ ] No secrets anywhere (code, history, logs, video, slides)
- [ ] Every sponsor call has a timeout + fallback
- [ ] README and slides claim nothing the code doesn't do

### NVIDIA Nemotron Award (DGX Spark)
- [ ] A Nemotron lane pinned (not `meterless/auto`) for the deep agent work
- [ ] Demo shows the NVIDIA provider in Nightshift, a Nemotron lane, and one request with its model visible in run details
