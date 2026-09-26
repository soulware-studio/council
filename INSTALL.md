# Installing the council

This file is meant to be **handed to an agent**, not read by a human line by
line. Copy this entire file's contents and paste it into a Claude Code or
Codex CLI session, in the project (or with the global scope) where you want
`/council` available. The agent will do the rest.

---

## Agent instructions

You are installing "the council" — a multi-reviewer plan/code review workflow —
from https://github.com/soulware-studio/council. If the user gave you a
different URL, a local clone, or a downloaded/extracted copy instead, use
that.

**Step 1 — get the files.** Retrieve (via fetch, or by reading a local clone)
the files for the tool you are installing into. Claude Code needs
`commands/council.md` and `agents/{jiro,adversary,foundation,codex,grok}.md`. Codex
needs everything under `codex/`. Retrieve both sets if the user wants both.

**Step 2 — figure out which tool you are and where things go:**

- **If you are Claude Code:** copy `commands/council.md` to
  `~/.claude/commands/council.md`, and copy each of `agents/jiro.md`,
  `agents/adversary.md`, `agents/foundation.md`, `agents/codex.md`,
  `agents/grok.md` into
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

Preserve explicit user/project roster overrides during updates unless the user
asks to reset them. The public defaults are inherited model and effort for the
three native reviewers, with pinned model and effort for the outside reviewers.

**Step 3 — check for the OTHER models' CLIs.** Each direction's outside
reviewers are *different* models than the one you're installing into, so
check for them:

- **If you installed the Claude Code path:** run `codex --version`. If it's
  missing and the user wants the codex reviewer (an outside-model lane) to
  actually work, tell them plainly: they need to install the Codex CLI and
  authenticate it (a paid ChatGPT plan, or an OpenAI API key) — don't attempt
  to install it for them without asking, it's their choice of auth method.
  Without it, the council still runs fine without that lane. This outside
  lane invokes `gpt-6-astra` with `max` reasoning effort; its CLI/account must
  support those settings. Then run `grok --version` for the optional grok
  fallback lane. If it's missing and wanted, tell them: install xAI's Grok Build CLI and
  sign the council's isolated Grok home in once with a SuperGrok / X Premium+
  account (`GROK_HOME=~/.grok-council grok login --device-auth`). Don't install
  it or sign in for them unasked. The lane configures its own isolation.
- **If you installed the Codex-native path:** run `claude --version` instead
  — that skill's outside lane shells out to Claude Code, not Codex. Same
  logic: missing means the council runs without its outside lane; tell the user how to
  add it, don't install it for them unasked. This outside lane invokes
  `claude-fable-5-1` at `max` effort and requires Claude Code **2.1.251 or newer**.
  If older, explain that `claude update` is needed before this lane can run.

Confirm the CLI version and pinned model requirements. A working `--version`
does not prove authentication or model access. If the requested model or effort
is unavailable at runtime, report a skipped lane with the reason; never silently
substitute a different model or effort.

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
you added the automation policy), including the roster: three native reviewers
inherit model and effort; the outside reviewer uses Fable 5.1 / max from Codex;
from Claude Code the outside reviewers are GPT-6 Astra / max and Grok 4.7 / high. If you installed the Claude Code path,
they can now type `/council` in **any** project (it's a global install). If
you installed the Codex-native path into a project's `.agents/skills/`,
it's available in **this project only** — say so explicitly rather than
implying it's everywhere. Point them at this repo's `README.md` for what a
council run actually looks like.
