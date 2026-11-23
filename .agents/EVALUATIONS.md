# Editorial Agent Team Evaluations

This file defines 4 repeatable evaluation scenarios—one for each specialist agent and one end-to-end scenario for the `editorial-lead` coordinator—to verify evidence citations, channel-specific release-status accuracy, multi-layer documentation audits, non-duplication checks, and human-in-the-loop guardrails on demand.

## Running & Grading the Suite in Antigravity

1. **Pre-Run Baseline**: Run `git status -s` and `git diff --name-only` before starting an evaluation so you can confirm after the run that no files in `docs/`, `posts/`, `PRs/`, or `tasks/` were created or modified.
2. **Executing Scenarios**:
   - For **Scenarios 1–3**, invoke the target specialist (`repository-scout`, `docs-detective`, or `content-strategist`) via `invoke_subagent` with the scenario's test prompt.
   - For **Scenario 4**, invoke `editorial-lead` (or run its instructions in the main chat) to coordinate `repository-scout`, `docs-detective`, and `content-strategist` and return a consolidated weekly editorial briefing.
3. **Grading Methodology**:
   - Score every pass criterion as `PASS`, `FAIL`, or `BLOCKED` with concrete evidence (conversation IDs, cited URLs, release tag comparisons, and `git diff` verification).
   - Verify release status against the actual distribution channel (`gh release view <tag>` or `gh api repos/WordPress/wordpress-playground/compare/<tag>...<commit_sha>`).
   - Verify that every item follows the exact five-bullet schema in order: `Finding`, `Evidence`, `Priority`, `Action`, `Confidence`.

---

## Scenario 1: Repository Scout — Merged vs. Released Accuracy (`repository-scout`)

- **Target Agent**: [`repository-scout`](agents/repository-scout.md)
- **Test Prompt**:
  > "Use `repository-scout` to review changes in `WordPress/wordpress-playground` over the past 14 days. Report the top 3 user-facing pull requests or issue trends and state whether each change is live in a tagged release."
- **Pass Criteria**:
  1. Every reported item includes a real, clickable `https://github.com/WordPress/wordpress-playground/pull/<id>`, `/issues/<id>`, or `/releases/tag/<tag>` URL that resolves to the described change.
  2. Every PR finding explicitly states either `Released` (citing the exact release tag/version) or `Merged, unreleased (in trunk)` (or `Release status: not applicable (community signal)` for issue trends), never implying an unreleased merge is already available to end users.
  3. All 5 required schema fields (`Finding`, `Evidence`, `Priority`, `Action`, `Confidence`) are present in order with a valid `P0`–`P3` priority and one-sentence justification.
  4. Only read-only commands/queries are executed; no issues, comments, or file modifications are made.

---

## Scenario 2: Docs Detective — Verifiable Documentation Gap Audit (`docs-detective`)

- **Target Agent**: [`docs-detective`](agents/docs-detective.md)
- **Test Prompt**:
  > "Use `docs-detective` to compare the 5 most recently merged feature or CLI/Blueprint PRs in `WordPress/wordpress-playground` against our local `docs/` files, upstream `packages/docs/site/docs/`, and `https://wordpress.github.io/wordpress-playground/`. Identify any undocumented features or outdated examples."
- **Pass Criteria**:
  1. Every reported gap cites both the upstream GitHub PR/release URL and the exact documentation URL (`https://wordpress.github.io/wordpress-playground/...`) and/or repository file path (`docs/...` and `packages/docs/site/docs/...`).
  2. Each item quotes or accurately summarizes what the target documentation page currently says—or lists the exact files/sections searched to confirm the topic is absent—and distinguishes local `docs/` vs. upstream `packages/docs/` vs. live docs.
  3. Each finding states whether the underlying code change is `Released (<tag>)` or `Merged, unreleased (in trunk)`.
  4. No documentation files in `docs/` are edited or created during the run.

---

## Scenario 3: Content Strategist — Non-Duplicate Story Pitching (`content-strategist`)

- **Target Agent**: [`content-strategist`](agents/content-strategist.md)
- **Test Prompt**:
  > "Use `content-strategist` to propose 2–3 blog story opportunities based on recent `WordPress/wordpress-playground` CLI, Blueprint, and runtime improvements, checking `https://developer.wordpress.org/news/`, `https://make.wordpress.org/playground/`, and local `posts/` first."
- **Pass Criteria**:
  1. The agent searches existing articles on `https://developer.wordpress.org/news/`, `https://make.wordpress.org/playground/`, and local `posts/` before finalizing recommendations.
  2. Every proposal cites the related published post URLs and local `posts/*.md` paths that were checked for overlap.
  3. Each item clearly explains why the proposed angle is distinct from prior coverage OR recommends `Update existing post` when an older article covers the same topic with outdated examples.
  4. Each proposal includes all 5 required schema fields (`Finding`, `Evidence`, `Priority`, `Action`, `Confidence`) in order and notes whether the feature is `Released (<tag>)` or `Merged, unreleased (in trunk)`.

---

## Scenario 4: Editorial Lead — End-to-End Weekly Briefing & Self-Verification (`editorial-lead`)

- **Target Agent**: [`editorial-lead`](agents/editorial-lead.md)
- **Test Prompt**:
  > "Run `editorial-lead` to generate a consolidated weekly editorial briefing for `WordPress/wordpress-playground` covering the last 7–14 days."
- **Pass Criteria**:
  1. **Delegation**: `editorial-lead` delegates to `repository-scout`, `docs-detective`, and `content-strategist`.
  2. **Deduplication & Ordering**: Overlapping findings across specialists are merged into unified items and ordered strictly from highest priority (`P0`) to lowest (`P3`).
  3. **Schema Compliance**: Every briefing item contains `Finding`, `Evidence`, `Priority` (`P0`–`P3`), `Action`, and `Confidence` in order.
  4. **Pre-Delivery Verification**: Every item passes the three mandatory checks:
     - (a) Real PR/issue/release URL + explicit `Released` vs. `Merged, unreleased` status (or `not applicable (community signal)`).
     - (b) Specific, verified doc URL and/or file path for every documentation gap.
     - (c) Cross-check citations against recent `developer.wordpress.org/news`, `make.wordpress.org/playground`, and local `posts/` for every content proposal.
  5. **Human-in-the-Loop Guardrail**: No repository files are modified, no posts are published, and no GitHub issues or PRs are created or commented on.

---

## Latest Evaluation Run & Scorecard (`2026-11-21`, Window `2026-11-07` to `2026-11-21`)

| Scenario | Agent & Run ID | Criterion 1 | Criterion 2 | Criterion 3 | Criterion 4 | Criterion 5 | Overall |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **Scenario 1** | `repository-scout` (`3ddda5b7-a038-4b25-91f3-97d7ba146adf`) | **PASS** (Cited real PRs `#4486`, `#4627`, `#3600`, `#4621`, `#4626`, issues `#3612`, `#4615`, tags `v3.1.64`, `v3.1.65`) | **PASS** (Verified commit inclusion via `gh api .../compare/v3.1.65...<sha>`; distinguished `Released (v3.1.65)`, `Released (v3.1.64)`, and `Merged, unreleased (in trunk)`) | **PASS** (All 3 items used exact 5-bullet schema `Finding`, `Evidence`, `Priority`, `Action`, `Confidence` ordered `P0` → `P1` → `P2`) | **PASS** (Read-only `gh` queries only; zero workspace edits) | N/A | **PASS (4/4)** |
| **Scenario 2** | `docs-detective` (`26581118-41eb-4b72-be91-2e5a3a13ccad`) | **PASS** (Cited PRs `#4621`, `#4626`, `#4280`, `#4281`, `#3600`, `#4563`, `#4552`, `#4548`, `#4486`, `#4627`, `#4575` + exact local `docs/...`, upstream `packages/docs/...`, and live URLs) | **PASS** (Quoted exact lines in local `04-wp-playground-cli.md`, `query-api/01-index.md`, `02-getting-started.md`, and `mcp-integration.md` vs. upstream/live docs) | **PASS** (Explicitly tagged each change as `Released (v3.1.62–v3.1.65)` or `Merged, unreleased (in trunk)`) | **PASS** (Zero files in `docs/` modified) | N/A | **PASS (4/4)** |
| **Scenario 3** | `content-strategist` (`a90dda1a-91b5-43b8-9c31-09ff9556364d`) | **PASS** (Checked Developer Blog, Make Playground RSS, local `posts/`, and `content-suggestions.md`) | **PASS** (Cited Nov 2025/2026 Make Playground Xdebug posts, Nov 4 Reprint post, Nov 11 URL params post, Nov 19 `?filebrowser` post, Developer Blog roundups, and local `posts/*.md`) | **PASS** (Excluded already-covered `?filebrowser`, `?dock`, and Reprint stories; recommended updating the Nov 2025 Xdebug post for `127.0.0.1` loopback and pitched 3 distinct angles) | **PASS** (All 3 pitches followed the 5-bullet schema and distinguished `Released` vs. `Merged, unreleased`) | N/A | **PASS (4/4)** |
| **Scenario 4** | `editorial-lead` (`6456fdb4-c630-4cf5-8867-84fe1f2d5bdf`) | **PASS** (Spawned nested subagents `repository-scout` `59df6e7b`, `docs-detective` `5d222749`, and `content-strategist` `99053f94` and waited for all 3 reports) | **PASS** (Deduplicated 50 merged PRs + doc gaps + pitches into 9 unified items ordered strictly `P0` → `P1` → `P2` → `P3`) | **PASS** (All 9 items included `Finding`, `Evidence`, `Priority`, `Action`, `Confidence` in exact order) | **PASS** (Caught live HTTP 404s on `/developers/xdebug/trace-and-profile/` and `/guides/ai-assistants-mcp/` due to missing `manifest.json` entries; verified all release tags and non-duplication exclusions) | **PASS** (`git diff --name-only` confirmed zero files written or modified) | **PASS (5/5)** |
