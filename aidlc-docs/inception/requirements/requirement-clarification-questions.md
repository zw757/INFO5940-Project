# Requirements Clarification Questions: ClosetCast

Thank you for answering `requirement-verification-questions.md`. Most answers are clear and consistent. These follow-up questions cover three things:

- **Part A**: gaps created by switching from Gemini to OpenAI (Q2, Q18) and two smaller ambiguities (Q13, Q19)
- **Part B**: decisions the **Security Baseline** extension needs, now that you opted in
- **Part C**: decisions the **Resiliency Baseline** extension needs. Its rules *require* the team to answer these during Requirements; the AI may not decide them for you.

**How to answer:** same as before. Put a letter after `[Answer]:`, optionally followed by a short reason.

---

## Part A: Follow-ups on Your Answers

### Issue A1: Unlimited batch upload (Q2)
You chose "no upload limit, because OpenAI has no free-tier limit" and asked to be told about any issues. There are three:
1. **OpenAI still has rate limits.** Every paid account has per-minute request and token limits, set by your usage tier. A new account starts at the lowest tier, so a large batch can still hit `429 Too Many Requests`. A queue with retry and backoff is still needed.
2. **Cost scales with the batch.** Each photo is a paid vision request. Downscaling to 1024 px (already in the technical environment) keeps that cost small but not zero.
3. **The upload request itself.** Photos can be up to 10 MB each, so 100 photos in one HTTP upload could be 1 GB. That is slow, fragile on a laptop, and a denial-of-service risk under the Security Baseline (SECURITY-05 requires request body size limits).

### Clarification Question 1
How should batch photo upload be limited?

A) Up to 50 photos per batch (the request size cap still applies); tagging runs in a background queue with limited concurrency plus retry and backoff on `429` errors; the user can upload several batches in a row

B) No limit on the number of photos, but the browser uploads them one at a time to the server (so no single request is huge); tagging runs in the same rate-limited queue

C) Up to 20 photos per batch, with the same queue

X) Other (please describe after [Answer]: tag below)

[Answer]: C

### Issue A2: Which OpenAI model (Q18)
The technical environment named a Gemini model. With OpenAI, the model must support **image input, function calling and JSON-schema structured output**. Model names change often, so confirm the exact name on OpenAI's models page before Construction. Either way, the name is read from the `OPENAI_MODEL` setting, not written into code.

### Clarification Question 2
Which OpenAI model tier should ClosetCast use?

A) One small, low-cost model for everything (vision tagging, parsing, agent), set by `OPENAI_MODEL`

B) Two settings: a small model for vision tagging and itinerary parsing (`OPENAI_MODEL_FAST`), and a stronger model for the agent's reasoning (`OPENAI_MODEL_AGENT`)

C) One flagship (most capable) model for everything

X) Other (please describe after [Answer]: tag below)

[Answer]: B

### Clarification Question 3
OpenAI usage is billed per request. Should the app protect against unexpected cost?

A) Rely on the monthly spending limit set in the OpenAI dashboard; the app only logs approximate token usage per request

B) Also add an app-level guard: a configurable daily request cap (e.g., `MAX_LLM_REQUESTS_PER_DAY`), after which AI features return a clear error

C) No cost controls

X) Other (please describe after [Answer]: tag below)

[Answer]: A

### Clarification Question 4
Q18 says to update the technical environment for OpenAI. `docs/technical-environment.md` currently *prohibits* the `openai` SDK and describes Gemini throughout, and `README.md` and `docs/vision.md` also mention Gemini. How should these documents be updated?

A) Claude updates `docs/technical-environment.md`, `README.md` and `.env.example` now (Gemini → OpenAI, `google-genai` → `openai` SDK, Service Allow List, rate-limit notes), and the change is recorded in the audit log as a team decision; `docs/vision.md` keeps its original wording, with the change noted in `requirements.md`

B) The team updates the documents by hand; `requirements.md` records the override in the meantime

C) Leave the input documents unchanged; `requirements.md` is the source of truth and states that it overrides the technical environment on the LLM provider

X) Other (please describe after [Answer]: tag below)

[Answer]: A

### Ambiguity A3: Missing items in the packing list (Q13 = C)
You chose to show the best owned option in the outfit, with a "to buy" suggestion next to it. It isn't clear what the **packing list** should show.

### Clarification Question 5
How should the packing checklist show items the user doesn't own yet?

A) The packing list has only owned items, plus a separate "Buy before you go" section that lists the gap items and links to the gap finder

B) Gap items appear inside the packing list, marked "to buy"

C) Gap items never appear in the packing list; they are shown only in the gap finder

X) Other (please describe after [Answer]: tag below)

[Answer]: A

### Ambiguity A4: How WCAG 2.1 AA is checked (Q19 = B)
The technical environment has no browser test tooling, and it prohibits Playwright and Selenium (for scraping).

### Clarification Question 6
How should WCAG 2.1 AA compliance be checked?

A) Manually: run the axe DevTools browser extension on every page before each milestone and record the results in the build-and-test docs

B) Automatically: add Playwright with axe-core as a dev-only test dependency (an exception to the technical environment, for accessibility testing only)

C) Both: manual axe checks, plus automated HTML checks in pytest (e.g., every `<img>` has `alt`, every input has a `<label>`)

X) Other (please describe after [Answer]: tag below)

[Answer]: C

---

## Part B: Security Baseline Decisions

The Security Baseline was written mainly for cloud systems. For a single-user app bound to `127.0.0.1`, several rules will be marked **N/A** with a reason (for example, SECURITY-12 authentication, SECURITY-06 IAM policies and SECURITY-07 network ACLs). The two questions below are rules that *do* apply but need a choice.

### Clarification Question 7
SECURITY-01 requires encryption at rest for every data store. ClosetCast stores a SQLite file and clothing photos under `data/` on the laptop. How should this be met?

A) Rely on full-disk encryption (FileVault / BitLocker / LUKS); the README and setup instructions require it to be enabled, and the requirements record that as the control

B) Encrypt the SQLite database at the application level (e.g., SQLCipher); photos rely on disk encryption

C) Encrypt both the database and the photo files at the application level, with a key in `.env`

X) Other (please describe after [Answer]: tag below)

[Answer]: A

### Clarification Question 8
SECURITY-03 and SECURITY-14 require structured logs with a request ID, sent to a central place, and kept for a defined period. Where should logs go for a local app?

A) Structured JSON logs written to a rotating local file (`data/logs/`) and the console, with a request ID per request, no secrets or photo bytes, kept for 90 days; "central" means this single log location (no alerting service)

B) Console only (no log files)

X) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Part C: Resiliency Baseline Decisions

**Read this first.** The Resiliency Baseline targets production cloud workloads: multi-zone deployment, auto-scaling, disaster recovery and on-call. ClosetCast runs on one laptop and calls OpenAI, Open-Meteo and Nominatim. Most rules will be **N/A** for this app. The rules that still apply to a local app are timeouts, retries, graceful degradation (RESILIENCY-10), a health endpoint (RESILIENCY-06) and backups of local data (RESILIENCY-12). Many of these overlap with your answer to Q17. You may keep the extension or change your earlier answer.

### Clarification Question 9
Do you want to keep the Resiliency Baseline enabled?

A) Keep it enabled; answer Questions 10–16 below (most rules will be N/A with documented reasons)

B) Turn it off; the local-app concerns above (timeouts, retries, graceful degradation, health endpoint, data backup) are recorded as ordinary non-functional requirements instead (skip Questions 10–16)

X) Other (please describe after [Answer]: tag below)

[Answer]: B

### Clarification Question 10 (RESILIENCY-02): RTO/RPO goals and disaster recovery strategy
What are your Recovery Time Objective (RTO) and Recovery Point Objective (RPO) goals?

A) RPO/RTO: Hours — Backup & Restore strategy. Lowest cost. Data backed up; on failure, reinstall the app and restore `data/` from a backup. Suitable for non-critical workloads.

B) RPO/RTO: 10s of minutes — Pilot Light strategy (cloud standby infrastructure)

C) RPO/RTO: Minutes — Warm Standby strategy (cloud standby at reduced capacity)

D) RPO/RTO: Near real-time — Multi-site Active/Active

E) N/A — no disaster recovery target; losing local data is acceptable for the MVP

X) Other (please describe after [Answer]: tag below)

[Answer]: N/A – skipped because we chose CQ9 = B (Resiliency Baseline turned off)

### Clarification Question 11 (RESILIENCY-08): Regional topology
Does this workload need multi-zone or multi-region deployment?

A) Not applicable — the app runs locally on a single laptop with no cloud deployment (documented as an exception)

B) Single-region, multi-zone cloud deployment

C) Multi-region deployment

X) Other (please describe after [Answer]: tag below)

[Answer]: N/A – skipped because we chose CQ9 = B (Resiliency Baseline turned off)

### Clarification Question 12 (RESILIENCY-03): Change management process
How should changes to `main` be governed?

A) Use the team's existing process from the README: feature branches, pull requests reviewed by a teammate, and the quality gate (`ruff`, `mypy`, `pytest`) passing before merge

B) No process exists; AI-DLC should propose a lightweight one

C) N/A — exempt (student project); document the reason

X) Other (please describe after [Answer]: tag below)

[Answer]: N/A – skipped because we chose CQ9 = B (Resiliency Baseline turned off)

### Clarification Question 13 (RESILIENCY-04): CI/CD tooling
What CI/CD tooling should run the quality gate?

A) GitHub Actions: run `ruff`, `mypy`, `pytest` and `pip-audit` on every pull request (no deployment step, since the app runs locally)

B) No CI; each team member runs the quality gate locally before merging

X) Other (please describe after [Answer]: tag below)

[Answer]: N/A – skipped because we chose CQ9 = B (Resiliency Baseline turned off)

### Clarification Question 14 (RESILIENCY-04): Rollback mechanism
How should a bad change be rolled back?

A) Version-pinned rollback: check out the previous git tag/commit and run `uv sync`; the database schema is created at startup and only ever gains new tables or columns, so older versions still work

B) Version-pinned rollback plus a backup of `data/closetcast.db` taken automatically before any schema change

X) Other (please describe after [Answer]: tag below)

[Answer]: N/A – skipped because we chose CQ9 = B (Resiliency Baseline turned off)

### Clarification Question 15 (RESILIENCY-04): Deployment style
What deployment style applies?

A) Direct / in-place: each user runs `git pull`, then `uv sync`, then restarts the local server

B) Tagged releases: users only update to git tags marking tested milestones (e.g., `v0.1-demo`), not to the latest `main`

X) Other (please describe after [Answer]: tag below)

[Answer]: N/A – skipped because we chose CQ9 = B (Resiliency Baseline turned off)

### Clarification Question 16 (RESILIENCY-15): Incident response process
How are problems with the app handled?

A) GitHub Issues: bugs are filed as issues with steps to reproduce and the request ID from the logs; fixed through the normal PR process; a short write-up for any serious bug found during testing week

B) No formal process; AI-DLC should propose one

X) Other (please describe after [Answer]: tag below)

[Answer]: N/A – skipped because we chose CQ9 = B (Resiliency Baseline turned off)
