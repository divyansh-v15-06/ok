# Start Here — Agent Onboarding Prompt

Copy the prompt below into an AI coding agent before assigning it implementation work. It is a repository-wide orientation prompt; after it completes the orientation, give the agent one of the focused prompts in `AGENT_PROMPTS.md`.

```text
You are joining a team competing in Speed Demon 2026. Your first task is orientation: establish the real repository state, understand the competition contract, identify available official materials, and recommend the next concrete implementation task. Do not edit code, delete files, install dependencies, submit to the board, or push commits during this orientation task unless I explicitly ask after your report.

Read these files completely, in this order:
1. `START.md` (this document)
2. `README.md` — the event contract, hard gates, scoring model, environment, and track summaries
3. `WIN_WALKTHROUGH.md` — strategy, recommended track order, validation workflow, and board discipline
4. `AGENT_PROMPTS.md` — ownership boundaries and exact per-track delivery prompts

Then inspect the repository without modifying it:
- list tracked/untracked/modified files with `git status --short`
- identify branch, HEAD commit, remotes, and recent commits
- list the top-level tree and every `track-*/` directory if present
- inspect each track’s Dockerfile, Cargo.toml, `.speed-demon.toml`, `src/`, `PERF.md`, and any existing build outputs/logs
- search for TODO/FIXME, incomplete implementations, tests, harness scripts, and documented expected hashes

Next, audit official competition materials. The README describes their expected location outside this submission repository. Locate and inspect what actually exists; do not assume all files are available:
- `../speed-demon-2026/CONVENTIONS.md`
- each relevant `specs/<track>.md`
- each relevant `reference/<track>.py`
- generators, `check.py`, harness README, starter template README, and Docker/image allowlist
- public full datasets, expected output hashes, release notes, and official clarifications/transcript if available

For each item, report its exact path and whether you inspected it, it is absent, or access is blocked. Official specifications and clarifications take precedence over repository summaries. Never invent a missing format, hash, input filename, or event rule.

Understand these non-negotiable constraints before recommending work:
- Each track is isolated and must build/run offline.
- Runtime reads `/data`, writes exactly one required output file under `/out`, and must exit successfully.
- Output is judged byte-for-byte by SHA-256.
- Dependencies, source LOC, peak memory, and wall time all affect score; zero runtime dependencies is the intended default.
- We need valid scores on at least three tracks for overall-championship eligibility.
- Do not modify another agent’s track or discard uncommitted work.

Your orientation report must be concise but evidence-based and include:
1. Current repository/branch state and what has already been built.
2. Materials inventory: available versus missing official specs, references, harness, datasets, hashes, and clarifications.
3. Per-track state: unstarted, baseline implemented, hash-verified, cap-compliant, or unknown — with evidence.
4. The most important blocker or correctness risk per track.
5. The recommended next task, owner scope, and why it has the highest championship value now.
6. A short command plan for validating that next task.

After I approve a track assignment, use the matching Track A/B/C/D prompt from `AGENT_PROMPTS.md`. Stay within that track directory, first create a correct hash-verified baseline, then optimize from measured bottlenecks. Maintain that track’s `PERF.md` throughout.
```

## Maintainer notes

- Use this document for the first agent on the project, a returning agent after time away, or an agent asked to triage shared work.
- Use `AGENT_PROMPTS.md` only after the orientation report identifies the track and confirms which official materials are present.
- If official data or expected hashes are not yet released, agents may build parsers and local fixtures but must label all unverified assumptions in `PERF.md`.
