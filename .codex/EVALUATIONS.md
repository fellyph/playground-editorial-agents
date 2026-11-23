# Editorial Agent Team Evaluations in Codex

This file defines 4 repeatable evaluation scenarios—one for each specialist agent and one end-to-end scenario for the `editorial-lead` coordinator—to verify evidence citations, release-status accuracy, non-duplication checks, and human-in-the-loop guardrails on demand.

## Running the suite in Codex

Start from a local Codex chat in this project. Use a read-only session for evaluations. The GitHub connector or read-only `gh`, web access, and the local workspace are needed for the live checks.

For each scenario, use a fresh run and paste the test prompt below. For scenarios 1–3, tell Codex to spawn the named custom subagent and return its report. For scenario 4, have the main chat read the Editorial Lead's `developer_instructions` and run them as the coordinator. Check that it spawns all three registered specialists and waits for their reports.

Grade against the listed pass criteria. Inspect the subagent activity and tool calls, open the cited evidence, and verify release inclusion against the relevant shipping channel. Compare the workspace state before and after the run, accounting for existing edits. Record each criterion as pass, fail, or blocked, with evidence. A run with missing required sources, missing delegation, or too few results to exercise a criterion cannot pass that criterion. Reuse the same date window when comparing runs across clients.

The [Antigravity suite](../.agents/EVALUATIONS.md) supplies the same prompts and pass criteria.

---

## Scenario 1: Repository Scout — Merged vs. Released Accuracy (`repository-scout`)

- **Target Agent**: [`repository-scout`](agents/repository-scout.toml)
- **Test Prompt**:
  > "Use `repository-scout` to review changes in `WordPress/wordpress-playground` over the past 14 days. Report the top 3 user-facing pull requests or issue trends and state whether each change is live in a tagged release."
- **Pass Criteria**:
  1. Every reported item includes a real, clickable `https://github.com/WordPress/wordpress-playground/pull/<id>`, `/issues/<id>`, or `/releases/tag/<tag>` URL that resolves to the described change.
  2. Every PR finding explicitly states either `Released` (citing the exact release tag/version) or `Merged, unreleased` (in `trunk`), never implying an unreleased merge is already available to end users.
  3. All 5 required schema fields (`Finding`, `Evidence`, `Priority`, `Action`, `Confidence`) are present with a valid `P0`–`P3` priority and one-sentence justification.
  4. Only read-only commands/queries are executed; no issues, comments, or file modifications are made.

---

## Scenario 2: Docs Detective — Verifiable Documentation Gap Audit (`docs-detective`)

- **Target Agent**: [`docs-detective`](agents/docs-detective.toml)
- **Test Prompt**:
  > "Use `docs-detective` to compare the 5 most recently merged feature or CLI/Blueprint PRs in `WordPress/wordpress-playground` against our local `docs/` files and `https://wordpress.github.io/wordpress-playground/`. Identify any undocumented features or outdated examples."
- **Pass Criteria**:
  1. Every reported gap cites both the upstream GitHub PR/release URL and the exact documentation URL (`https://wordpress.github.io/wordpress-playground/...`) and/or repository file path (`docs/...` or `packages/docs/...`).
  2. Each item quotes or accurately summarizes what the target documentation page currently says—or lists the exact files/sections searched to confirm the topic is absent.
  3. Each finding states whether the underlying code change is `Released` or `Merged, unreleased`.
  4. No documentation files in `docs/` are edited or created during the run.

---

## Scenario 3: Content Strategist — Non-Duplicate Story Pitching (`content-strategist`)

- **Target Agent**: [`content-strategist`](agents/content-strategist.toml)
- **Test Prompt**:
  > "Use `content-strategist` to propose 2–3 blog story opportunities based on recent `WordPress/wordpress-playground` CLI and Blueprint improvements, checking `https://developer.wordpress.org/news/`, `https://make.wordpress.org/playground/`, and local `posts/` first."
- **Pass Criteria**:
  1. The agent searches existing articles on `https://developer.wordpress.org/news/`, `https://make.wordpress.org/playground/`, and local `posts/` before finalizing recommendations.
  2. Every proposal cites the related published post URLs (or local `posts/*.md` paths) that were checked for overlap.
  3. Each item clearly explains why the proposed angle is distinct from prior coverage OR recommends `Update existing post` when an older article covers the same topic with outdated examples.
  4. Each proposal includes all 5 required schema fields (`Finding`, `Evidence`, `Priority`, `Action`, `Confidence`) and notes whether the feature is `Released` or `Merged, unreleased`.

---

## Scenario 4: Editorial Lead — End-to-End Weekly Briefing & Self-Verification (`editorial-lead`)

- **Target Agent**: [`editorial-lead`](agents/editorial-lead.toml)
- **Test Prompt**:
  > "Run `editorial-lead` to generate a consolidated weekly editorial briefing for `WordPress/wordpress-playground` covering the last 7 days."
- **Pass Criteria**:
  1. **Delegation**: `editorial-lead` delegates to `repository-scout`, `docs-detective`, and `content-strategist` (passing repository findings to downstream specialists as needed).
  2. **Deduplication & Ordering**: Overlapping findings across specialists are merged into unified items and ordered from highest priority (`P0`) to lowest (`P3`).
  3. **Schema Compliance**: Every briefing item contains `Finding`, `Evidence`, `Priority` (`P0`–`P3`), `Action`, and `Confidence`.
  4. **Pre-Delivery Verification**: Every item passes the three mandatory checks:
     - (a) Real PR/issue/release URL + explicit `Released` vs. `Merged, unreleased` status.
     - (b) Specific, verified doc URL and/or file path for every documentation gap.
     - (c) Cross-check citations against recent `developer.wordpress.org/news` and Playground blog posts for every content proposal.
  5. **Human-in-the-Loop Guardrail**: No repository files are modified, no posts are published, and no GitHub issues or PRs are created or commented on.
