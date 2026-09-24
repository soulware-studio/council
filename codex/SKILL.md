---
name: council
description: "Run the council review workflow in Codex: fan out parallel reviewer agents for craft, failure modes, and architecture/roadmap fit, optionally invoke Claude Code CLI as the outside-model lane, then synthesize a verdict. Use when the user asks for council, /council, multi-agent plan review, or a Jiro/adversary/foundation review."
---

# Council

Run the council inside Codex using the shared council prompts. The reviewer
prompts live in `references/` — load only the ones needed for the current
run.

## Models for councils run from Codex

By default, `jiro`, `adversary`, and `foundation` **inherit the parent Codex
session's model and reasoning effort**. Omit model and effort overrides when
spawning them. Honor explicit user or project `AGENTS.md` roster overrides.

When using `collaboration.spawn_agent` with an explicit override, set `model`,
`reasoning_effort`, and `fork_turns: "none"`. Full-history forks do not accept
model or effort overrides. Include the review mode, reviewer prompt, artifact,
and relevant context explicitly in each agent's message. On other Codex
runtimes, use the equivalent supported subagent controls; if a requested pin
cannot be applied, report that instead of silently inheriting.

The shared reviewer files' `model:` and `effort:` frontmatter configures
Claude-hosted reviewers; it does not select the Codex subagents' models. For
Codex runs, use the inherited session settings or the explicit user/project
override instead of that metadata.

Invoke the Claude outside reviewer with the exact model and effort specified
in `references/claude-outside.md`. If a pinned model or effort is unavailable,
report the failed lane; do not silently substitute a model or lower effort.
Honor an explicit user override for a particular run without changing these
persistent settings.

The shared command reference also describes a `grok` lane and per-run Grok
seat swaps. Those are Claude-host only: a Codex-hosted council runs its three
reviewers plus the Claude outside lane, and its `Ran:` line has no grok entry.

## Workflow

1. Determine the **mode** (`plan` vs `code`) and identify the artifact using
   `references/council-command.md` (Steps 0-1). Plan mode: prefer the plan or
   proposal already in conversation; if a path is named, use it exactly; do not
   pick the newest file from `~/.claude/plans/` unless context is empty and a
   plan was clearly implied. Code mode: the artifact is the branch diff
   (`git diff main...HEAD` plus uncommitted changes), with the originating plan
   pulled in as context if one exists.
2. If the artifact is informal conversation context, synthesize a compact
   3-10 bullet proposal and state that this is what the council is reviewing.
   Continue unless the user corrects it.
3. If the user explicitly asked for council, subagents, delegation, or parallel
   agent review, spawn three Codex agents in parallel with the model, effort,
   and context settings above. Give each the **mode** as an explicit first line
   (see `references/council-command.md` Step 2) — in code mode, include the
   `git diff` since the reviewers review the change, not a description of it:
   - `jiro`: use `references/jiro.md`
   - `adversary`: use `references/adversary.md`
   - `foundation`: use `references/foundation.md`
4. Run the outside-model lane with Claude Code CLI using
   `references/claude-outside.md`.
   - First verify `claude --version`.
   - If unavailable, return `VERDICT: skip` for that lane.
   - Use the pinned `claude -p` invocation in that reference from the repo root
     (it is confined to the repository and has no shell).
     Plan mode: pass the plan plus file pointers. Code mode: paste the raw
     `git diff` as that reference describes — Claude cannot run git — and let
     it read the changed files itself.
5. Collect verdicts: `ship it`, `revise`, `rethink`, or `skip`.
6. Synthesize one council verdict using the format in
   `references/council-command.md`.
   - Overall verdict is the most severe non-skip reviewer verdict.
   - Include cross-reviewer agreement first.
   - In `Ran:`, report the Codex agents' accepted spawn model/effort and the
     Claude lane's `MODEL:`/`EFFORT:` metadata. Label the outside lane `claude`.
     Do not use Claude reviewer frontmatter to report Codex models, and mark
     any failed or unverified lane explicitly.
   - Preserve the Claude outside-model response verbatim.
7. Act on the verdict per `references/council-command.md` Step 5. Plan mode: on
   `revise`, edit the plan file with concrete amendments; on `rethink`, do not
   rewrite without user direction; informal artifact → report in chat only.
   Code mode: produce a ranked fix list (`file:line` + fix), lead with findings
   2+ reviewers converged on, and do not auto-edit unless the user asked.

## Guardrails

- Parallel fan-out is the point; do not run council reviewers sequentially.
- Keep reviewer prompts unmodified unless the user explicitly asks for a
  changed lens.
- Do not paste curated code into the outside-model prompt. Give file paths and
  let the outside model inspect the repo; the one exception is the raw diff in
  code mode, because the Claude lane has no shell to produce it.
