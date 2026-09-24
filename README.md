# The Council

A panel of reviewers you convene with **`/council`** to stress-test your
work before it bites you — either a **plan** (before you build) or a **diff**
(after you build). Three lenses, up to two outside models, one synthesized
verdict.

## When to use it

Two moments, both high-value:

1. **Before you build — review the plan.** Once you've sketched an approach, run
   `/council`. It pokes holes in the design. Revise, run it again, and repeat
   until the feedback thins out — when each round says less, you've hit
   *diminishing returns*, and that's when you implement. Fixing a problem on
   paper costs seconds; fixing it after it's built costs hours.
2. **After you build — review the code.** Once it's implemented, run `/council`
   again — it notices there's code now and reviews the actual diff, catching
   shipping-blocker bugs that a green test suite happily hides. A round or two
   here is worth it on anything touching correctness, money, auth, or data.

It auto-detects plan vs code. Force it with a word when needed: `/council plan`
or `/council code`.

## How often, and does it run itself?

By default, the council runs when you ask for it — type `/council`, or just
tell your agent "council this." Nothing runs in the background and nothing
runs without being asked, unless you turn on the optional automatic policy
below.

A typical cadence: 2-3 rounds on a plan (revise, re-run, repeat — stopping
when a round finds little to nothing new), then 1-2 rounds on the code once
it's built. There's no fixed number — `/council` is built to keep iterating
until findings thin out, then stop; that's the signal to move on, not a
round count.

If you don't want to remember to ask, see "Making it automatic" below —
`INSTALL.md` asks you about this directly during setup, so it's a choice you
make once up front rather than something to rediscover later.

## The reviewers

- **jiro — the craftsman.** Obsessed with simplicity. Asks "what can we remove?"
  Fights complexity and anything bolted-on.
- **adversary — the breaker.** Asks "how does this break?" Hunts edge cases,
  race conditions, partial failures, hostile inputs. Professionally paranoid.
- **foundation — the architect & roadmap guardian.** Takes the long view: "does
  this hold up the next ten things we build?" Checks the work against where the
  project is heading.
- **codex — the outside opinion.** Hands the work to a *different* model
  (OpenAI's Codex) for an independent second read. A different brain catches
  what one brain's blind spots miss. Optional — if the Codex CLI isn't
  installed, this lane returns `skip` and the others still run.
- **grok — the second outside opinion.** Hands the work to xAI's **Grok
  Build CLI**, a third model family, read-only. It runs beside codex so you can
  see, run after run, what each outside model catches that the other misses —
  every finding is tagged with the reviewers that raised it. Optional, and
  Claude Code-hosted only; without the CLI it returns `skip`.

The overall verdict is the most severe of the reviewers that ran (a partial
Grok read does not count toward it) (`ship it` / `revise` /
`rethink`), and the synthesis leads with anything **2+ reviewers agree on** —
those are the high-confidence findings.

## What a council run looks like

The reply is a short, plain-language summary — not a stack of engineering reports:
a one-line `Ran:` roster (which model and effort each reviewer used, so a
stale or unexpected model is visible every run), a bottom line, the handful
of merged findings, and — always last — a `🔔 NEEDS YOUR DECISION` block.
Anything that changes the spec, scope, or a major design direction lands
there with a recommendation and a brief why; on code reviews, so does any
place the implementation diverged from the plan. `🟢 Nothing needs your
input.` means you're clear. Ask for "the full council report" if you want
per-reviewer detail and the outside models' verbatim reviews.

## Installing

See [INSTALL.md](INSTALL.md) — the short version: paste its contents into a
Claude Code or Codex CLI session and let the agent set itself up.

## Staying up to date

There's no background auto-update — this is a handful of Markdown files, not
a running service. To pick up changes (a fixed bug, a sharper reviewer
prompt, a new capability), hand `INSTALL.md` to your agent again. It compares
what's already installed against the current files and shows you a diff
before touching anything — that diff is your "what's new," the same way it
would be on a first install.

There's no separate changelog file to fall out of sync with the code — this
repo's own commit history is the changelog. Each commit message says what
changed, in plain language, with nothing generated or copied from elsewhere.

## Tuning the roster

The public defaults depend on which tool starts the council:

| Council started from | Jiro, Adversary, Foundation | Outside reviewers |
| --- | --- | --- |
| Codex | Inherit the Codex session's model and reasoning effort | Claude Fable 5.1 / max |
| Claude Code | Inherit the Claude session's model and effort | GPT-6 Astra / max, and Grok 4.7 / high |

The outside reviewers use full model IDs and explicit effort flags, overriding
their CLI defaults for that review. Fable 5.1 requires Claude Code **2.1.251 or
newer**; run `claude update` if needed. Both outside lanes require authentication
and access to the pinned model. If a pin is unavailable, the lane reports
`skip` and the reason instead of silently switching models. The `Ran:` line
shows the model and effort used, with unverified settings labeled explicitly.

- **Claude reviewers:** edit `model:` / `effort:` in the installed agent files.
  Use `model: inherit` and omit `effort:` to inherit both session settings.
- **Codex reviewers:** set an explicit council roster in the project's
  `AGENTS.md` to override inheritance. For example, request GPT-6 Astra with
  `max` reasoning effort for all three. This keeps project preferences separate
  from the downloadable skill's defaults.
- **Outside reviewers:** change the model and effort in their invocation
  instructions, or ask for an override for one council run.
- **Seat swap (one run):** `/council grok=adversary` (or `jiro` /
  `foundation`) puts Grok in that seat with that reviewer's lens instead of the
  Claude agent. If Grok can't run, the Claude agent stands in — a seat is never
  left empty.

## Making it automatic (optional)

By default the council runs when you ask. To make it proactive, add this to
your project's `CLAUDE.md` (shared with your team via git) or your global
`~/.claude/CLAUDE.md` (every project):

```markdown
## Council policy
Run the council automatically: council any non-trivial plan before
implementing (iterate until findings thin out), and council the code after
significant implementation work. Skip it for trivial changes.
```

Delete the section to go back to on-request. Easiest toggle in either
direction: just ask Claude — "add the auto-council policy to this project" /
"remove the auto-council policy."

## Get the most out of it

- **Iterate, don't one-shot.** The value is in re-running until findings dry up.
- **Keep a roadmap doc.** `foundation` is sharpest when it knows your goals. A
  short `ROADMAP.md` or a note describing where the project is headed lets it
  catch "this shortcut will block that future feature." Without one it
  estimates trajectory from your code — still useful, just less sharp.
- **Frame the target.** Point it at the risky surface ("council the billing
  path", "review the concurrency in this branch") for a tighter review.

## The Codex outside lane (optional but recommended)

This reviewer calls OpenAI's **Codex CLI**. Install it and make sure
`codex` runs in your terminal. Auth is either a paid ChatGPT plan (includes
Codex usage) or an OpenAI API key (pay-per-token, no subscription). Without
it the council still runs without it.

The `codex/` folder is the mirror image: if the Codex CLI is your main tool,
it lets you run the council *from* Codex — its three reviewers plus Claude Code
as the outside lane (no Grok lane on that side). It assumes a Unix-like shell with `bash` and `perl`
(both present by default on macOS and Linux; Windows users can run it under
WSL).

## What outside reviewers can read

Everything an outside model reads leaves your machine, so each lane is
confined by its CLI's own flags. Grok and the Claude Code lane read only the
repository under review. Codex reads only the
repository on macOS and Linux; on Windows its sandbox keeps your home folder,
drives and network shares out but cannot hide files that every local user
can read, and the council labels that run as home-excluded rather than
repository-only.

## The Grok outside lane (optional)

The fifth reviewer calls xAI's **Grok Build CLI** (`grok`), pinned to
grok-4.7 / high. A review takes 10-15 minutes, so the orchestrator runs it as a
background task while the other reviewers work. Grok is an agentic CLI that can edit files, run commands,
read anywhere, and by default imports your Claude Code settings. So the lane
runs it from an isolated home (`~/.grok-council`, config rewritten every run,
every Claude/Cursor import off, API-key auth disabled) with only read-only
tools (`read_file`, `list_dir`, `grep` — no shell, no edits, no web, no
subagents) and denies every path outside the repository. Grok's own
`--sandbox` is not used. Everything Grok reads —
the diff or plan, and repository files — goes to xAI, so keep secrets and
regulated data out of reviewed repositories.

Setup, once per machine: install (`curl -fsSL https://x.ai/cli/install.sh |
bash`, or on Windows `irm https://x.ai/cli/install.ps1 | iex`), then sign the
isolated home in with a SuperGrok or X Premium+ account:
`GROK_HOME=~/.grok-council grok login --device-auth`. Without Grok the
council runs with the other four.

## Lessons from a few hundred runs

- **Council-on-code reliably finds shipping-blocker bugs a green test suite
  misses.** The most valuable single habit is running it again after you
  implement, not just before — and framing each reviewer with the specific
  files and attack surface you're worried about, rather than "review my
  branch."
- **The council catches correctness, not feel.** A design can pass every
  reviewer and still be clunky to actually use. For anything with a user
  interface, budget a separate pass of just using the thing for a while after
  it councils clean.
- **`rethink` is not a harder `revise`.** If a reviewer returns `rethink`,
  step back and reconsider the approach rather than iterating on the current
  one piece by piece — the two verdicts call for different responses.
- **When two reviewers disagree on a factual claim, measure it rather than
  asking a fifth opinion.** A short, real test usually settles it faster than
  more argument does.
- **Keep the Codex lane tightly scoped, including at max effort.** Point it at
  specific files or line ranges so it can finish within the review budget.

## License

MIT — see [LICENSE](LICENSE).
