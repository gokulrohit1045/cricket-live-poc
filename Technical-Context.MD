# Technical Context

Engineering-practice contract for code-writing agents in this project. Every implementation must respect what's written here. If a constraint conflicts with the user's request, flag the conflict — don't silently bypass it.

## Overriding Principles
1. **Score data comes only from a licensed feed — never scraped.** All match and score data is fetched from the contracted provider API (CricketData.org / CricAPI); no code scrapes Cricbuzz, ESPNcricinfo, ICC or any other site, because scraping breaches their terms and puts the product at legal risk.

## Frameworks
- **Next.js (App Router) with TypeScript** — the whole application: React UI plus server-side route handlers that call the score provider. Exact versions are pinned in `package.json` at scaffold time (see *Toolchain pinning*).
- **Node.js (Active LTS at scaffold time)** — runtime for Next.js locally, in CI and on Azure App Service; exact version pinned in `.nvmrc`.

## 3rd-party tech
- **CricketData.org (CricAPI)** — the licensed source of fixtures and live scores for international matches between full-member national teams. Called only from Next.js server code; the API key is held server-side as an App Service application setting and never reaches the browser.
- **Azure App Service** — hosts the Next.js app.
- **Azure DevOps** — work tracking (`EnateInternal` / `Cricket Live POC`) and the CI pipeline that reports the `ci` status check on pull requests.
- **GitHub** — source hosting (`gokulrohit1045/cricket-live-poc`).

## Packages in use
**Testing:** `vitest` (unit and integration), `@playwright/test` (browser end-to-end), `msw` (recorded fakes at the CricAPI seam)

No other libraries are approved. Use what Next.js, React and the Node standard library provide first; a PR that adds a dependency states why the built-in option is insufficient.

## Toolchain pinning
- Policy: **exact-pinned**.
- Pinned tools: Node.js via `.nvmrc` and the `engines` field in `package.json`; Next.js, React, TypeScript and every other dependency at exact versions in `package.json` (no `^` or `~`); `package-lock.json` committed; CI installs with `npm ci`.
- Bumping: a version bump is its own PR that changes only the pinned versions and lockfile, and lands only with `ci` green.

## Instrumentation
- Structured JSON logs, one object per line, written to stdout with no logging library; read through the Azure App Service log stream.
- Levels: `error` — a request failed or the feed is unusable; `warn` — degraded (feed slow, stale, or quota nearly spent); `info` — normal lifecycle events; `debug` — off in the `poc` environment.
- Always logged: every CricAPI call (endpoint, HTTP status, latency, remaining quota if the response reports it) and every feed failure or stale-data condition.
- Never logged: the CricAPI key or any request header carrying it.
- No APM or tracing service is in use; adding one is an ADR.

## User Feedback Approach
- Channel: in-app only. No user accounts, no email, no push notifications.
- Tone: terse and factual — scoreboard style, no exclamation marks or chatter.
- Every score shows when it was last updated.
- Degraded states use a plain banner: feed delayed, feed unavailable, no live matches right now.

## Branching and Environments
- Model: trunk-based; full rules in `CONTRIBUTING.md`.
- Branches: `main`; `feature/<feature-id>-<slug>` per Feature; `story/<id>-<slug>` or `bug/<id>-<slug>` per work item, squash-merged.
- Environments: local development; `poc` — a single Azure App Service, deployed from `main` after the `ci` check passes.
- Protected: `main` (PR-only, linear history, squash-merge only, no force-push, no deletion).

## Git Guardrails
- Never force-push or rewrite history (`push --force`, `rebase`, `reset --hard`) on a shared branch without explicit confirmation.
- Never commit a secret: the CricAPI key and any `.env*` file stay out of git (`.env*` is gitignored).
- Hooks in place: `.githooks/pre-commit` blocks scratch and diff dump files (enable per clone with `git config core.hooksPath .githooks`).
- **HITL delivery opens and closes its own board state — nothing else will.** No skill in the HITL pipeline (`/factory-to-stories`, `/factory-check-security-design`, `/factory-triage`) sets an assignee, an iteration, or an in-progress state at creation — they deliberately leave a HITL-tagged Story, Bug, or wholly-HITL Feature untouched for a human to clear. So: **before** making any change while hand-delivering one, set `System.AssignedTo` to yourself, `System.IterationPath` to the current sprint, and `System.State` to `Active`. For a Feature, delivery starts with its first Story; while it is planned, the planning skills set its state. **When** the delivering PR merges and the work item is done, set `System.State` to `Resolved`/`Closed` yourself before ending the session. Skipping either half leaves the item unassigned, out of sprint, and looking untouched for the entire time it's actually being worked.

## Testing & the ratchet
> Seeded from the Factory standard — **opt-out, not opt-in**. Trim a line only with a deliberate, recorded decision; don't start from nothing. Full standard: <https://github.com/enateltd/factory-skills/blob/main/docs/testing-and-the-ratchet-standard.md> · the *why*: <https://github.com/enateltd/factory-skills/blob/main/docs/agentic-e2e-testing-principles.md>.

- **Test the contract, not the LLM.** Assert on the deterministic envelope (output shape, state transition, side-effect ordering), never on generated text or chain-of-thought.
- **Every seam gets a real-IO test on at least one side.** Mock-on-both-sides only proves the mocks agree; defects live at integration seams, not inside modules.
- **Three tiers, each catching a class the others can't.** Tier 1 — recorded-replay (real local I/O, fakes at the seams, replayed LLM fixtures; $0, every commit); Tier 2 — live-LLM, disposable (real subprocess against a per-run throwaway substrate; bounded cost, scheduled); Tier 3 — live (real production substrate and subprocess; continuous).
- **Platform matrix is non-negotiable for OS-touching code.** Filesystem/subprocess code runs Tier 1 + Tier 2 on every OS it supports, not just the dev box's.
- **Plan test coverage at the Feature level, not the Story level.** Each Feature declares at kickoff which tiers it needs, which fixtures/substrates must exist, and a Tier-2 cost ceiling.
- **The ratchet — every Tier-3 defect pins a cheap-tier regression.** A live defect's fix lands with a Tier-1 (or Tier-2) regression that would have caught it, encoding the **class** of defect **bounded by the seam** where it surfaced — not the exact reproducer, and not a project-wide sweep. A Story that surfaced from a live defect carries that ratcheting fixture as an acceptance criterion.
- **Disposable substrate, not shared.** Tier-2 substrate is created per run and torn down; reproducibility comes from a seed (tarball/manifest/script), never persistent state.
- **Assert on observable behaviour, not internal state.** For Tier 2/3 the target is what an operator could verify from external artefacts — tracker state, commit shape, ordered log events, posted comments.

**In this project:** Tier 1 replays recorded CricAPI responses through `msw`; Tier 2 calls the live CricAPI on a schedule, kept within the plan's request quota; Tier 3 is the `poc` environment.
