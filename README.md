# WordPress Playground Editorial Agent Team

A multi-agent editorial and documentation monitoring team for the [WordPress Playground](https://github.com/WordPress/wordpress-playground) open-source project (`WordPress/wordpress-playground`), its documentation (`packages/docs`, `https://wordpress.github.io/wordpress-playground/`, and `https://developer.wordpress.org/playground/`), and its blog content ([WordPress Developer Blog](https://developer.wordpress.org/news/) and [Make WordPress Playground](https://make.wordpress.org/playground/)).

This repository ships ready-to-use agent configurations and an evaluation suite for two agent environments:
- **Google Antigravity** (`.agents/agents/*.md` and `.agents/EVALUATIONS.md`)
- **OpenAI Codex** (`.codex/agents/*.toml` and `.codex/EVALUATIONS.md`)

---

## Agent Roster

| Agent | Role | Antigravity Config | Codex Config |
| :--- | :--- | :--- | :--- |
| **`editorial-lead`** | **Coordinator**: Scopes the time window, delegates investigation to the three specialists, deduplicates overlapping findings, enforces the `P0`–`P3` priority scale and 5-field briefing schema, and runs a 3-part pre-delivery verification check. | [`.agents/agents/editorial-lead.md`](.agents/agents/editorial-lead.md) | [`.codex/agents/editorial-lead.toml`](.codex/agents/editorial-lead.toml) |
| **`repository-scout`** | **"What changed?"**: Reviews recently merged PRs, opened/closed issues, recurring community questions, and official release tags in `WordPress/wordpress-playground`, strictly distinguishing **Released** vs. **Merged, unreleased (in trunk)**. | [`.agents/agents/repository-scout.md`](.agents/agents/repository-scout.md) | [`.codex/agents/repository-scout.toml`](.codex/agents/repository-scout.toml) |
| **`docs-detective`** | **"What's missing?"**: Audits recent project changes against local `docs/`, upstream `packages/docs/site/docs/`, and the live published documentation site to pinpoint undocumented features, outdated examples, or broken handbook pages. | [`.agents/agents/docs-detective.md`](.agents/agents/docs-detective.md) | [`.codex/agents/docs-detective.toml`](.codex/agents/docs-detective.toml) |
| **`content-strategist`** | **"What should we share?"**: Cross-checks recent project changes and community signals against the WordPress Developer Blog, Make WordPress Playground, and local `posts/` to propose 2–3 non-duplicate story opportunities or post updates. | [`.agents/agents/content-strategist.md`](.agents/agents/content-strategist.md) | [`.codex/agents/content-strategist.toml`](.codex/agents/content-strategist.toml) |

---

## Core Guardrails & Shared Output Schema

### 1. Strict Human-in-the-Loop Guardrail
All four agents operate in **read-only investigation and recommendation mode**:
- They **never** publish blog posts, modify repository source/documentation files, commit code, or open/comment on GitHub issues or pull requests automatically.
- All final editorial and publishing decisions remain with the human editor.

### 2. Channel-Specific `Merged` vs. `Released` Verification
Every repository change is verified against the actual distribution channel (e.g., comparing merge commit SHAs against published GitHub release tags via `gh api repos/WordPress/wordpress-playground/compare/<tag>...<sha>`):
- `Released (<tag/version>)` — Verified to be included in a tagged release or live deployment.
- `Merged, unreleased (in trunk)` — Merged into `trunk` after the latest release tag and not yet shipped in a tagged release.
- `Release status: not applicable (community signal)` — Recurring community issue or discussion thread without an underlying code change.

### 3. Shared Priority Scale (`P0`–`P3`)
- **`P0` (Critical)**: Released breaking change, security/regression notice, or actively broken/misleading documentation affecting users now.
- **`P1` (High)**: Newly released user-facing feature missing documentation, or high-frequency community confusion needing immediate clarification.
- **`P2` (Medium)**: Recently merged feature or notable improvement that creates a strong tutorial opportunity, documentation enhancement, or update to an existing post.
- **`P3` (Low)**: Minor example cleanup, polish, or evergreen content backlog idea.

### 4. Required 5-Field Briefing Item Format
Every reported item uses the exact five-bullet schema:
- **Finding**: Verified change, community signal, or documentation gap (explicitly stating `Released` vs. `Merged, unreleased`).
- **Evidence**: Clickable URLs to the exact GitHub PRs, issues, releases, documentation pages, local file paths, and/or existing blog posts checked.
- **Priority**: `P0`–`P3` with a one-sentence justification.
- **Action**: Recommended next step (`New post`, `Update existing post`, `Documentation update`, or `Follow-up issue`).
- **Confidence**: `High`, `Medium`, or `Low` (with a brief note on how it was verified).

---

## Installation & Usage

### Prerequisites
- Authenticated GitHub CLI (`gh auth status`) for read-only repository, PR, issue, and release inspection.
- Copy `.agents/` (for Antigravity) and/or `.codex/` + `AGENTS.md` (for Codex) into your WordPress Playground documentation or content workspace (or run directly from this repository).

### Running in Google Antigravity
Antigravity automatically discovers `.agents/agents/*.md` at the root of your workspace.
- **Full Weekly Editorial Briefing**:
  > "Run `editorial-lead` to generate a consolidated weekly editorial briefing for `WordPress/wordpress-playground` covering the last 7 days."
- **Individual Specialist**:
  > "Use `repository-scout` to review changes in `WordPress/wordpress-playground` over the past 14 days."

See [`.agents/EVALUATIONS.md`](.agents/EVALUATIONS.md) for the complete 4-scenario evaluation suite and latest scorecard.

### Running in OpenAI Codex
Codex discovers `.codex/agents/*.toml` and project guidance in [`AGENTS.md`](AGENTS.md).
- **Full Weekly Editorial Briefing**: Ask the main Codex project chat to read `.codex/agents/editorial-lead.toml` and run the coordinator workflow across `repository-scout`, `docs-detective`, and `content-strategist`.
- **Individual Specialist**: Ask Codex to spawn `repository-scout`, `docs-detective`, or `content-strategist` and return its 5-field report.

See [`.codex/EVALUATIONS.md`](.codex/EVALUATIONS.md) for Codex-specific evaluation instructions.
