# AGENTS.md — how to work in this repo

Read this first. It applies to every coding agent (Cursor, Claude Code, Codex) and every human.

## What this is
**GapZero**: an agent that explains why business numbers don't match across billing, CRM and the ledger, until every dollar is attributed to records. It's our solo entry to Open Agent Hackathon 2026, track *The Agent That Can Explain Why*.
- Design and roadmap: [PLAN.md](PLAN.md)
- What's next: [TODO.md](TODO.md) (always pick the top unchecked item of the current phase)
- What we've learned: [LEARNINGS.md](LEARNINGS.md) (read before touching Zetaris, Meterless or H-MEM)
- Hackathon rules: `hackathon-docs/` (local only, gitignored). If anything here conflicts with them, the rules win.

## Non-negotiables
1. **No secrets in git, ever.** Credentials only come from env vars (`.env`, gitignored). Never print them, log them, or paste them into prompts or PR text. Redact in traces.
2. **No hardcoded or faked outputs.** Agents must never read `eval/` ground truth (a test enforces this). `output_examples/` are saved results and are never read at runtime. Traces come only from real execution.
3. **Read-only against data.** All agent SQL goes through `src/adapters/sqlGuard.ts` (SELECT/WITH only, single statement, table allowlist, LIMIT cap).
4. **Bounded everything.** Every network or model call has a timeout. Every loop has a cap. Failures return structured errors, never a hang.
5. **Don't overclaim.** README and slides describe only what the code does. Every output carries its run `mode`.
6. **Build in Cursor.** The hackathon requires Cursor as the development environment. Claude Code and others run alongside, disclosed in the README.

## How to write code (ponytail)
Climb the ladder and stop at the first rung that works:
1. Does it need to exist? If it's speculative, skip it and say so.
2. Does something in this repo already do it? Reuse it.
3. Stdlib or a platform feature? Use it.
4. An installed dependency? Use it. Don't add a dependency for a few lines.
5. Then write the minimum code that works.

No interfaces with one implementation, no config for values that never change, no scaffolding "for later". Fix root causes in the shared function, not per caller. Mark known corners with `// ponytail: <ceiling>, <upgrade path>`. Non-trivial logic gets one focused test.

## The loop (every unit of work)
1. **Pick:** top unchecked TODO item in the current phase. Mark it `[~]` in TODO.md.
2. **Plan:** a short note in the PR body covering approach, files touched and how it will be verified. If it's bigger than about half a day, split it into TODO items first.
3. **Branch:** `git switch -c feat/<slug>` (or `fix/`, `chore/`, `docs/`).
4. **TDD:** write a failing test, then make it pass. Keep tests next to behaviour, not mocks of mocks.
5. **Check locally:** `pnpm check` (lint, typecheck, tests, H-MEM conformance, audit). Also run `pnpm eval` if you touched agents, prompts or the simulator (paste the before/after into the PR), and `pnpm perf` if you touched adapters, SQL or the orchestrator.
6. **PR:** `gh pr create` with title `<type>: <what>`. The body covers why, what, test evidence, eval/perf delta, and the TODO item.
7. **Review:** run an independent review (`/code-review`, or a fresh agent asked to find bugs, security issues and over-engineering). Fix real findings. CI must be green.
8. **Merge:** `gh pr merge --squash --delete-branch`. Tick the TODO item with the PR number.
9. **Learn:** if anything surprised you, add a line to LEARNINGS.md. If a rule in this file was wrong, fix this file in the same PR.

## Security checklist (every PR)
- Inputs validated with zod at trust boundaries: HTTP body, LLM JSON output, Zetaris rows.
- LLM output is treated as untrusted. It never becomes SQL without passing the guard, and never becomes shell or file paths.
- No new dependency without a reason in the PR; `pnpm audit --prod` stays clean.
- gitleaks and CodeQL are green.

## Performance budgets (checked by `pnpm perf`)
- A sample-mode run with the recorded-LLM fixture finishes in under 30 s at p95.
- A single source query takes under 2 s at p95 in DuckDB; Zetaris query timeout is 20 s with 2 retries.
- A full live run targets under 8 min, with a hard cap.

## Commands (available from D1)
| Command | What |
|---|---|
| `pnpm dev` | API and page on :8000 |
| `pnpm gapzero run --input input_examples/example_1.json` | CLI run |
| `pnpm check` | Lint, types, tests, conformance, audit |
| `pnpm eval --seeds 20` | Accuracy against ground truth |
| `pnpm perf` | Performance budgets |
| `pnpm sim --seed 42` | Regenerate source data |

## Design work
UI work uses the `impeccable` and `design-taste-frontend` skills. One page; the waterfall is the hero.

## Keeping the plan alive
- TODO.md is updated in every PR (mark `[~]` when starting, `[x]` with the PR number when merged).
- PLAN.md §8 Roadmap is updated at the end of each day (status plus any re-plan).
- If we're behind, cut using PLAN.md §9's order. Never cut a submission deliverable.
