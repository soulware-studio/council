---
description: Convene a council of four reviewers (jiro, adversary, foundation, codex) to stress-test a plan (pre-implementation) or a diff (post-implementation) in parallel
allowed-tools: Read, Edit, Glob, Grep, Agent, AskUserQuestion, Bash
---

# Council

Convene the council. Four reviewers, four lenses, one synthesis.

- **jiro** — the craftsman. Removes parts. Fights complexity.
- **adversary** — the breaker. Hunts failure modes.
- **foundation** — the structural engineer and roadmap guardian.
- **codex** — the outside opinion. A different model, a different brain.

The council catches problems the author can't see — and the same four lenses
work whether the artifact is a **plan** (before you build) or a **diff** (after
you build). Both are high-value:

- **Plan mode** — catch problems while the design is still molten, before a
  line is written. The cheapest possible moment to change course.
- **Code mode** — catch shipping-blocker bugs in committed work that the test
  suite passed straight over. Council-on-implementation reliably finds real
  bugs that green tests miss.

Iterate either way: re-convene after each round of amendments (plan) or fixes
(code) until the convergent findings dry up, then stop.

---

## Step 0: Determine the mode

Resolve **plan** vs **code** by the first rule that matches:

1. **Explicit** — the user passed `code` or `plan` as an argument. Honor it; stop.
2. **Named intent** — "review the plan / council this design" → plan. "review
   the code / this branch / what I just built / this diff" → code.
3. **Conversation** — just co-designed a plan with nothing implemented yet →
   plan. Just finished implementing or debugging → code.
4. **Content of the pending change** (only if 1-3 don't settle it) — classify
   *what is in* the working tree, not merely that a diff exists:
   - markdown design doc(s) — under `docs/plans/`, `~/.claude/plans/`, or reads
     as a plan → **plan**
   - source files → **code**
   - both → ambiguous
5. **Ambiguous** — do NOT silently pick; both wrong directions are costly. State
   the call in one line with the one-word correction — "Reading this as a *code*
   review of the 4 changed source files; say `plan` if you meant the design
   doc" — and proceed. Hard-stop and ask only when it is genuinely 50/50.

A freshly-written plan file is itself a diff — never read "a diff exists" as
"code mode." Rules 3 and 4 exist to catch exactly that.

## Step 1: Identify the artifact

**If mode is `plan`**, the artifact is one of the three kinds below (a-c). **If
mode is `code`**, skip to *Code artifact* at the end of this step.

### a) Conversation context (preferred)
If a plan file was created or discussed *in this conversation*, that's the
artifact — you already know its path. Use it directly.

### b) An explicit plan file path
If the user named a specific plan file or slug, use that. If multiple plan
files in `~/.claude/plans/` could match, ask which one — do not guess.

**Do NOT default to "most recent .md in ~/.claude/plans/".** That directory is
cross-repo and accumulates stale plans; "most recent" silently picks the wrong
artifact. Only fall back to filesystem search if conversation context is empty
AND the user implied a plan exists.

### c) An informal proposal in conversation context
If there is no plan file, look at the recent conversation. Has the user
described something they want fixed, built, or changed? Has the assistant
proposed an approach that hasn't been written down yet? **That informal
proposal is the artifact.** You don't need a plan file to convene the council.

In this case, write a brief synthesis of the proposed approach (3-10 bullet
points) into a temporary scratchpad in the conversation so the council members
have something concrete to review. State it back to the user as: "I'm
convening the council on this proposal: [bullets]. Let me know if I've
mischaracterized any of it before they begin."

Then proceed without waiting for a response unless the user corrects you.

### d) Nothing to review
If there is neither a plan file nor a clear proposal in context, ask the
user what they want reviewed. Do not invent something to review.

### Code artifact
When mode is `code`, the artifact is the **change under review** — the actual
diff, not a description of it. Scope it:

- Default to the branch vs its base: `git diff main...HEAD` plus uncommitted
  working-tree changes (`git status`, `git diff`). If the base isn't `main`,
  infer it or ask.
- If the user named something narrower (a commit range, one file, "the changes
  I just made"), scope to that.

Pull the **originating plan in as context if one exists** — the plan file from
this conversation, or a matching doc under `docs/plans/`. The richest code
review is "here is what we intended, here is what shipped — review the gap."
The reviewers read the diff and the coupled neighbors it touches.

## Step 2: Fan out in parallel

Spawn all four subagents **in a single message with four tool calls**. They
run concurrently. Each gets:

- **The mode, as an explicit first line** — "This is a CODE review of committed
  work" or "This is a PLAN review before any code is written." This single line
  reframes their entire read; never omit it.
- The artifact:
  - Plan mode: the plan file path, or the synthesized proposal.
  - Code mode: the diff itself. **jiro and adversary have no Bash** — run
    `git diff <base>...HEAD` yourself and paste the diff into their prompts;
    they Read the changed files for surrounding context. foundation and codex
    have Bash + git, so point them at the branch and base and let them
    self-serve. (Pasting the diff to the two Claude reviewers is fine — same
    model, no blind spot introduced. Only Codex must read the diff with its own
    eyes; never paste code into the Codex prompt.)
- Brief context: what the user is trying to accomplish — plus the originating
  plan, when this is a code review that has one.
- Code mode, when an originating plan exists: instruct each reviewer to also
  flag anywhere the implementation **diverged from the plan** — built
  differently than planned, features added or dropped, behavior changed — even
  if the divergence looks fine. Divergences feed the decision block (Step 4).
- A reminder of their lens (just their name — they know their job).

Do NOT spawn them sequentially. Parallel is the entire point.

## Step 3: Collect verdicts

Wait for all four to return. Each will respond with:

```
VERDICT: ship it | revise | rethink | skip
[body]
```

`skip` only comes from `codex` if the CLI is unavailable. Other agents do not
return skip.

## Step 4: Synthesize — write it for the user, not for engineers

The synthesis is the product. The person reading it owns the project but may
not have deep engineering context; a wall of per-reviewer reports does not get
read. Default output is SHORT and in plain language:

```
## Council: [one plain sentence — what was reviewed, and in which mode]
Ran: jiro (<model>/<effort>) · adversary (<model>/<effort>) · foundation (<model>/<effort>) · codex (<model from its MODEL line>)

**Bottom line:** [ship it | needs fixes | wrong approach] — one sentence why.
[Overall = the most severe individual verdict: any rethink → wrong approach;
any revise → needs fixes; ship it only if all four agree.]

**What they found** — merged across all reviewers, deduplicated, ranked by
severity. Each item is one or two plain-language sentences: what's wrong and
what happens if it isn't fixed. Typically 3–6 items; fold minor nits into a
single closing line. No per-reviewer sections. No jargon. No file:line
references unless the user asks.

**What happens next:** one or two sentences on what you will do with the
findings (e.g. "Applying fixes 1–3 now; 4 is cosmetic, skipping unless you
want it.")

━━━━━━━━━━━━━━━━━━━━━━
🔔 NEEDS YOUR DECISION
━━━━━━━━━━━━━━━━━━━━━━
[ALWAYS the final section of the message — nothing may come after it.]
```

Build the `Ran:` line by reading the `model:`/`effort:` frontmatter of the
three Claude agent files (one grep) plus the `MODEL:` line codex returns. For
a reviewer on `inherit`, print the actual session model instead of the
literal word "inherit". This line is the user's drift alarm — a stale codex
model or an unexpected reviewer model should be visible on every single run.

### The 🔔 NEEDS YOUR DECISION block

Always last, so it's the final thing on the user's screen. Each item is a
plain-language question or notice, followed by **Recommendation:** — one
sentence stating what you'd do and a brief why. It contains:

- Any amendment that changes the **spec, scope, or a major design direction**.
  These are never silently applied in plan mode and never glossed over in
  code mode.
- **Code mode:** anywhere the implementation **diverged from the originating
  plan/spec** — built differently than planned, features added or dropped,
  behavior changed — even when every reviewer thinks the divergence is fine.
  This block is how the user tracks spec fidelity.
- Reviewer disagreements that require the user's judgment.

If there is nothing: exactly one line — `🟢 Nothing needs your input.`

### Full detail on request

Keep the four raw verdicts in hand but do not paste them by default. If the
user asks ("show me the full council report"), give per-reviewer verdicts with
Codex's response verbatim.

## Step 5: Act on the verdict

### Plan mode

If the artifact is a **plan file** and the verdict is **revise**:
- Apply the mechanical amendments (correctness fixes, clarifications, missing
  steps that don't change what's being built) directly to the plan file, and
  summarize what changed in a sentence or two.
- Do NOT apply amendments that change the spec, scope, or a major design
  direction — those go in the 🔔 block with a recommendation, and you wait.

If the artifact is a **plan file** and the verdict is **rethink**:
- Do NOT edit the plan automatically. Use AskUserQuestion to present the
  alternative direction and ask whether to rewrite.

If the artifact is an **informal proposal** (no plan file):
- Present the synthesized verdict in chat. Do not create a plan file unless
  the user asks. The council's job here is to inform the discussion, not
  to formalize it.

### Code mode

The findings feed the **What they found** list in plain language (keep
`file:line` detail for the on-request full report). Lead with anything 2+
reviewers converged on — those are the high-confidence shipping-blockers.

- Do NOT auto-edit the code. Say in **What happens next** whether you propose
  fixing all, a subset, or deferring — and ask. (If the user already said "fix
  what you find," apply them and report what changed.)
- Every divergence from the originating plan/spec goes in the 🔔 block, even
  when the reviewers consider the divergence harmless.
- After fixes land, offer to re-convene on the new diff — the iterate-until-dry
  loop applies to code exactly as it does to plans.

### Ship it (either mode)

Even a clean run uses the Step 4 format — header, `Ran:` line, bottom line,
and the 🔔/🟢 closer. It will just be short.

## Adjusting the roster (models & effort)

- **Persistent:** each reviewer's `model:` and `effort:` are two frontmatter
  lines in its agent file (`~/.claude/agents/{jiro,adversary,foundation}.md`).
  When the user asks to change them ("set all council reviewers back to
  inherit", "put adversary on opus"), edit those lines directly.
- **Per-run:** if the user asks for a different model for this council only
  ("council this with everyone on opus"), pass the `model` override on each
  Agent call. Effort has no per-run override — frontmatter only.
- **codex** follows `~/.codex/config.toml` (`model = ...`), not Claude Code
  config. Its actual model appears in the `Ran:` line every run — if you know
  a newer OpenAI model exists than what's shown, say so.

## Council Discipline

- **Parallel always.** Sequential defeats the purpose.
- **Write for the user.** The synthesis is read by the project owner, not an
  engineering panel — plain language beats completeness; detail lives in the
  on-request full report.
- **No padding.** If the council is fast and clean, the report is short.
- **Surface disagreements.** Don't average opposing views into mush — they go
  in the 🔔 block with a recommendation.
- **Codex verbatim on request.** The default synthesis may summarize codex like
  any other reviewer, but the full report must carry its response verbatim.
- **Question budget is per-reviewer, not per-council.** Each reviewer has a
  hard cap of 2 questions and is instructed to use 0-1 in practice. Do not
  add questions of your own at the orchestrator level — you are the
  synthesizer, not a fifth reviewer.
