# Installing the council

This file is meant to be **handed to an agent**, not read by a human line by
line. Copy this entire file's contents and paste it into a Claude Code or
Codex CLI session, in the project (or with the global scope) where you want
`/council` available. The agent will do the rest.

---

## Agent instructions

You are installing "the council" — a four-reviewer plan/code review workflow —
from https://github.com/soulware-studio/council. If the user gave you a
different URL, a local clone, or a downloaded/extracted copy instead, use
that.

**Step 1 — get the files.** Retrieve (via fetch, or by reading a local clone)
these paths from the repo: `commands/council.md`, `agents/jiro.md`,
`agents/adversary.md`, `agents/foundation.md`, `agents/codex.md`, and — if the
user also wants the Codex-CLI-native skill — everything under `codex/`.

**Step 2 — figure out which tool you are and where things go:**

- **If you are Claude Code:** copy `commands/council.md` to
  `~/.claude/commands/council.md`, and copy each of `agents/jiro.md`,
  `agents/adversary.md`, `agents/foundation.md`, `agents/codex.md` into
  `~/.claude/agents/`. Use the global `~/.claude/` location (not a
  project-local `.claude/`) unless the user asks for this project only.
- **If you are Codex CLI:** copy the `codex/` folder's contents into this
  project's `.agents/skills/council/`, preserving its internal layout
  (`SKILL.md` at the top, `agents/openai.yaml`, everything else under
  `references/`). If this version of Codex CLI supports a user-level (not
  just per-project) skill location and the user wants that, check Codex
  CLI's own docs for the correct path rather than guessing at one.

Before writing any file, check whether it already exists. **This is also how
updates work** — someone re-pasting this file to pick up a change is running
the same steps against files that are already there. If a file exists and
differs from what you're about to write, show the user a diff so they can
see what's new, then ask whether to back it up (e.g. suffix the old file
with `.bak-<date>`) or overwrite. If it exists and is already identical,
just say so and move on — nothing to do.

**Step 3 — check for the OTHER model's CLI.** Each direction's fourth
reviewer is a *different* model than the one you're installing into, so
check for the other one:

- **If you installed the Claude Code path:** run `codex --version`. If it's
  missing and the user wants the fourth reviewer (the outside-model lane) to
  actually work, tell them plainly: they need to install the Codex CLI and
  authenticate it (a paid ChatGPT plan, or an OpenAI API key) — don't attempt
  to install it for them without asking, it's their choice of auth method.
  Without it, the council still runs fine with three reviewers.
- **If you installed the Codex-native path:** run `claude --version` instead
  — that skill's outside lane shells out to Claude Code, not Codex. Same
  logic: missing means three reviewers instead of four, tell the user how to
  add it, don't install it for them unasked.

If the relevant CLI is already present, just confirm the version and move
on.

**Step 4 — offer the automation policy.** Ask the user: "Do you want the
council to run automatically — before non-trivial plans and after real
implementation work — or only when you explicitly ask for it?" If they want
automatic, add this block to their `CLAUDE.md` (ask whether that's the
project's `CLAUDE.md`, shared with a team via git, or their global
`~/.claude/CLAUDE.md`, applying to every project):

```markdown
## Council policy
Run the council automatically: council any non-trivial plan before
implementing (iterate until findings thin out), and council the code after
significant implementation work. Skip it for trivial changes.
```

Create the file if it doesn't exist. If it does, append the section rather
than overwriting the rest of the file.

**Step 5 — confirm and explain.** Tell the user in plain language what you
just did (which files went where, whether the other CLI was found, whether
you added the automation policy). If you installed the Claude Code path,
they can now type `/council` in **any** project (it's a global install). If
you installed the Codex-native path into a project's `.agents/skills/`,
it's available in **this project only** — say so explicitly rather than
implying it's everywhere. Point them at this repo's `README.md` for what a
council run actually looks like.
