---
name: council
description: "Run the council review workflow in Codex: fan out parallel reviewer agents for craft, failure modes, and architecture/roadmap fit, optionally invoke Claude Code CLI as the outside-model lane, then synthesize a verdict. Use when the user asks for council, /council, multi-agent plan review, or a Jiro/adversary/foundation review."
---

# Council

Run the council inside Codex using the shared council prompts. The reviewer
prompts live in `references/` — load only the ones needed for the current
run.

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
   agent review, spawn three Codex agents in parallel. Give each the **mode** as
   an explicit first line (see `references/council-command.md` Step 2) — in code
   mode, include the `git diff` since the reviewers review the change, not a
   description of it:
   - `jiro`: use `references/jiro.md`
   - `adversary`: use `references/adversary.md`
   - `foundation`: use `references/foundation.md`
4. Run the outside-model lane with Claude Code CLI using
   `references/claude-outside.md`.
   - First verify `claude --version`.
   - If unavailable, return `VERDICT: skip` for that lane.
   - Use `claude -p` from the repo root and pass the plan/proposal plus file
     pointers. Let Claude read code itself.
5. Collect verdicts: `ship it`, `revise`, `rethink`, or `skip`.
6. Synthesize one council verdict using the format in
   `references/council-command.md`.
   - Overall verdict is the most severe non-skip reviewer verdict.
   - Include cross-reviewer agreement first.
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
- Do not paste large code into the outside-model prompt. Give file paths and
  let the outside model inspect the repo.
