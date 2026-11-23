# Agent Guidelines (`playground-editorial-agents`)

## Project Structure

- **`.agents/agents/`**: Google Antigravity Markdown agent definitions (`editorial-lead.md`, `repository-scout.md`, `docs-detective.md`, `content-strategist.md`).
- **`.agents/EVALUATIONS.md`**: Antigravity 4-scenario evaluation suite and verification scorecard.
- **`.codex/agents/`**: OpenAI Codex TOML agent definitions (`editorial-lead.toml`, `repository-scout.toml`, `docs-detective.toml`, `content-strategist.toml`).
- **`.codex/EVALUATIONS.md`**: Codex 4-scenario evaluation suite and grading guide.

## Shared Editorial Rules

1. **Strict Human-in-the-Loop Guardrail**: Agents may ONLY read, investigate, and recommend. Never publish content, modify repository source/docs, commit code, or open/comment on GitHub issues or pull requests automatically.
2. **Merged vs. Released Accuracy**: Every code finding must verify release inclusion against the affected distribution channel (e.g., comparing merge commits against published tags in `WordPress/wordpress-playground`) and state `Released (<tag/version>)`, `Merged, unreleased (in trunk)`, or `Release status: not applicable (community signal)`.
3. **Verifiable Citations**: Every item must cite real, verified GitHub PR/issue/release URLs, exact documentation URLs (`https://wordpress.github.io/wordpress-playground/...` / `https://developer.wordpress.org/playground/...`) and file paths (`packages/docs/site/docs/...` or local `docs/...`), and existing blog posts checked on `https://developer.wordpress.org/news/` and `https://make.wordpress.org/playground/`.

## Returning Codex Editorial Reports

When the user asks Codex to run `repository-scout`, `docs-detective`, or `content-strategist` directly, wait for the named custom agent and relay its report with the five-field layout intact. Every item in the final response must have separate **Finding**, **Evidence**, **Priority**, **Action**, and **Confidence** bullets in that order. Preserve the specialist's citations, release status, priority justification, and confidence rationale. Ask the specialist to supply any missing field before returning the report. Put any requested run audit after the findings.
