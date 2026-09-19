# claude-vault-starter — assisted setup

You (Claude) are setting up **claude-vault-starter** for your user on their
machine. It is a small kit: a markdown vault that is simultaneously their
notes, a git repo, and the context every Claude Code session starts with —
plus optional automation around it. Everything below comes from the same site
this file is hosted on.

The point of this process is to hand the user a **working habit**, not a
parts list. People who installed the kit from a feature menu spent days
figuring out which pieces to use. So: look at their machine first, ask about
how they work (not which scripts they want), propose a shaped setup, install
it, and then walk them through one real loop so they feel it working before
you leave.

Ground rules:

- **Observe before asking.** Step 1 is commands, not questions. Every
  proposal in step 3 must cite something you saw on their machine.
- **Two question calls total.** One interview (step 2), one plan confirmation
  (step 3). Nothing else blocks on input. Their confirmation IS consent for
  the crontab lines the plan implies.
- **Silent by default.** Files are cheap; interruptions are not. Anything
  that messages the user requires the Slack bridge, and the plan never
  carries more than **two** messaging jobs unless they ask for more in
  step 3.
- **Merge, never clobber.** setup.sh merges hooks and writes `.bak` backups;
  do not hand-edit settings beyond what it does. If the user already has
  `~/vault` or another vault, setup.sh uses it as-is.
- **Mechanisms only.** Do not import anyone's preferences, coding-style
  files, or permission settings. This kit deliberately contains none.
- **Verify and report.** Actually run the checks in step 6; do not assume.

## 1 · Observe the machine (no questions yet)

Run these and keep the numbers — they drive the proposal:

```bash
ls -d ~/vault ~/Documents/*/.obsidian ~/*/.obsidian 2>/dev/null      # existing vault / Obsidian vaults
ls ~/.claude/projects 2>/dev/null | wc -l                             # distinct Claude Code project dirs
find ~/.claude/projects -name '*.jsonl' 2>/dev/null | wc -l           # past sessions (the archive's value)
find ~ -maxdepth 3 -name .git -type d -not -path '*/node_modules/*' 2>/dev/null \
  | sed 's#/.git$##' | head -20                                       # code repos → project-context candidates
ls ~/.claude/CLAUDE.md ~/.claude/settings.json 2>/dev/null; wc -c ~/.claude/CLAUDE.md 2>/dev/null
command -v git python3 crontab claude gh tmux obsidian 2>/dev/null
uname -s; crontab -l 2>/dev/null | wc -l
# --- already installed? (any hit ⇒ UPGRADE mode, see below) ---
ls ~/.vault/crontab.template ~/.vault/.is_primary 2>/dev/null
ls ~/.vault/slack.env ~/.vault/todoist.env ~/.vault/calendar.env ~/.vault/projects.json 2>/dev/null
grep -v '^\s*#' ~/.vault/crontab.template 2>/dev/null | awk 'NF>5{print $6,$7}'   # cron'd modules
ls ~/.local/bin | grep -E '^(vault-|save-convo|project-context|session-distill|task-|journal-prompt|reminder|todoist-tasks|calendar-sync|refine-prefs|test-fix|kssh|claude-slack)' 2>/dev/null
grep -c 'claude-vault-starter:start' ~/.claude/CLAUDE.md 2>/dev/null      # global stub present?
ls ~/vault/Meta/Vault\ Setup.md ~/vault/Distill 2>/dev/null
grep -H VAULT_DIR ~/.vault/crontab.template ~/.bashrc ~/.zshrc ~/.profile 2>/dev/null   # non-default vault path? reuse it everywhere
```

### Upgrade mode (they already ran an earlier version of this kit)

If the checks above find `~/.vault/crontab.template` or any kit scripts,
this is an **upgrade**, not an install. Differences, all of which keep the
flow to the same two question calls:

- **Start by telling them what they have** — a short inventory from the
  output above: which modules are cron'd, which env files exist, whether the
  hooks and the global stub are in place, and the date of `Meta/Vault
  Setup.md` if one exists. Earlier versions never wrote that note; you will.
- **Still run the interview (step 2)** — the answers drive what to *add*;
  in the plan table (step 3) mark every row **installed** / **add** /
  **update**, and never propose removing something they have unless they
  pick "trim". The things an earlier kit lacked and that almost every
  upgrade should add: the global `CLAUDE.md` stub, `vault-context`,
  `vault-doctor` (update — the new one audits installed modules), and for
  Scholar/Operator `project-context` + `session-distill`.
- **Re-run setup.sh with the same features they had** (`archive` if the
  Stop hook exists, `comment-loop` if `~/.vault/comment-loop` exists). It is
  idempotent: hooks already present are left untouched, the crontab
  template is never rewritten, the stub is merged, `vault-context` lands in
  `~/.local/bin`. Pass `--yes` — the crontab it re-installs is their own
  template, unchanged.
- **Re-download every extra they already have** — same filenames, newer
  code, no config changes. Then download the added ones.
- **Skip Slack setup if `~/.vault/slack.env` exists**; likewise Todoist /
  calendar env files. Never ask for values they already configured.
- **First loop (step 5): only the new pieces** — skip what they already
  use daily. `vault-context` is always worth showing; it is new to them.
- **Setup note (step 6): describe the whole current state**, not just
  today's additions.

Interpretation you will use later:

- **sessions ≥ 20** → the session archive + `RECENT.md` gives them
  cross-session memory on day one ("what was I doing last week" works
  immediately). Say so with the number.
- **≥ 2 code repos** → `project-context` is worth it: each repo gets a
  `CLAUDE.local.md` regenerated hourly from the vault notes, daily notes,
  and recent sessions that mention that project.
- **Obsidian vault found** → offer to use it as the vault instead of
  creating `~/vault` (`VAULT_DIR=<path>`). Never move their notes.
- **`~/.claude/CLAUDE.md` > 20 KB** → they already spend a lot of context
  per session; lead with `vault-context` so they can see the budget.
- **macOS** → `crontab` works but warn that the machine must be awake;
  `date -r`/`stat -f` differences are handled by the scripts.

## 2 · Interview (ONE question call, four questions)

Use your question tool — a single call with these four questions. If you
cannot ask (non-interactive), assume *Notes-first / Nothing / none / just
this machine*.

**Q1 — "How should Claude relate to this vault?"** (single select)

| answer | preset |
|---|---|
| **Notes I write, Claude reads** — keep a daily note, sync it, remember sessions | **Seed** |
| **Research / coding across repos** — Claude should know each project's state when I open it | **Scholar** |
| **A teammate that reaches out** — digests, nudges, does vault work while I'm away | **Operator** |

**Q2 — "What is allowed to interrupt you?"** (single select)

| answer | effect |
|---|---|
| **Nothing — files only** | no Slack bridge, no messaging jobs |
| **One morning digest** | Slack bridge + `vault-digest` |
| **Digest + nudges through the day** | bridge + `vault-digest` + `task-nagger` |
| **An agent that proposes work and waits for my yes** | bridge + `task-worker` + `vault-digest` (that is the cap of two) |

**Q3 — "Which outside sources should flow into the vault?"** (multi select)

| answer | module |
|---|---|
| Google/Apple/Outlook calendar (iCal link) | `calendar-sync` |
| Todoist | `todoist-tasks` |
| Meeting transcripts / other files I'll drop in | nothing to install — `vault-organizer` files them |
| None for now | — |

**Q4 — "How many machines?"** (single select)

| answer | effect |
|---|---|
| Just this one | primary marker set; done |
| Several — this is my main one | primary here; print the second-machine recipe at the end |
| Several — this is a secondary one | `rm ~/.vault/.is_primary` after setup; no messaging jobs here |

## 3 · Propose the plan (ONE confirmation call)

Build the module list from the preset, then *add or remove* based on what
you observed in step 1. Present it as a table — **module · why, for you ·
when it runs · what it writes** — where "why" cites a step-1 observation
("you have 140 sessions", "three repos under ~/code"). Then one question:
**Install this / Trim something / Show me the full menu**. On "trim", ask
what to drop in chat and proceed; on "full menu", show the catalogue at the
bottom of this file and let them pick, still honouring the two-messaging-job
cap unless they explicitly raise it.

### Presets

**Seed** — the base loop, nothing speaks:

| module | what it gives you | runs |
|---|---|---|
| vault + `vault-sync` | `~/vault` git repo, `*.md merge=union`; commit → pull --rebase → push | every 5 min |
| session archive | every finished session → `Conversations/` + one line in `RECENT.md` | Stop hook + 10-min safety net |
| daily-note injection | today's `YYYY-MM-DD.md` (head) + pointers to recent days into every session | SessionStart hook |
| global `CLAUDE.md` stub | 5-line block telling Claude where the vault, `RECENT.md`, daily notes, and `Distill/` are | once (marker-delimited, merged) |
| `vault-context` | shows what a session will load and how much — the context budget, on demand | on demand |
| `comment-loop` | shift+click pins on any locally-served page wake a Claude session that edits and replies | on demand |

**Scholar** = Seed +

| module | what it gives you | runs |
|---|---|---|
| `project-context` | per-repo `CLAUDE.local.md` (git-excluded) rebuilt from vault notes, daily notes, `RECENT.md`, and `Calendar.md` entries that mention that project | hourly |
| `session-distill` | last 7 days of `Conversations/` → `Distill/YYYY-MM-DD.md`: decisions, open threads, things learned, per project | weekly, Mon 07:00 |
| `vault-organizer` | Claude proposes filing loose root notes into existing folders; python validates every move; `Inbox/` when unsure | daily 05:00 |
| `vault-doctor` | broken links, unclosed frontmatter, orphans, **and a module audit** (what is installed, when it last wrote anything) → `Meta/vault_doctor.md` | daily 05:30 |
| + anything from Q3 | `calendar-sync` hourly · `todoist-tasks` every 30 min | |

Skip `project-context` if step 1 found no repos (say so). Skip
`vault-organizer` if the vault is brand new — nothing to file yet; mention
it in the setup note as the thing to add once root fills up.

**Operator** = Scholar + Slack bridge + the Q2 messaging jobs + `reminder`
(per-minute firing line; `reminder add "+30m" "stand up"`). Two messaging
jobs maximum by default.

Optional in any preset, only if the user asks or Q1 hints at it:
`refine-prefs` (weekly distillation of corrections into `coding_prefs.md`;
needs the archive), `vault-tokens` (per-day token report), `test-fix`
(bounded fix attempts on a failing test command), `journal-prompt` (11:55pm
question via Slack — counts as a messaging job).

## 4 · Install

```bash
BASE=https://jeremy-collins.com/vault
cd "$(mktemp -d)" && curl -fsSLO "$BASE/starter.tar.gz"
tar xzf starter.tar.gz && cd claude-vault-starter
```

Core (`STARTER_FEATURES` is a comma list of `archive` and/or `comment-loop`,
or `none`; `VAULT_DIR` only if they chose an existing vault):

```bash
STARTER_FEATURES=archive,comment-loop ./setup.sh --yes      # sync cron accepted in step 3
echo n | STARTER_FEATURES=archive,comment-loop ./setup.sh   # if they declined the sync cron
```

setup.sh is idempotent and prints everything it does, including the global
`CLAUDE.md` stub it merges (marker-delimited; re-runs replace the block only).

Extras — each is one stdlib-only python script; download exactly the chosen
ones:

```bash
for s in project-context session-distill vault-organizer vault-doctor; do   # ← the chosen subset
  curl -fsSL "$BASE/extras/$s" -o ~/.local/bin/$s && chmod +x ~/.local/bin/$s
done
```

Config before cron — ask the user for values, never invent them; `chmod 600`
the env files:

- **project-context** → `~/.vault/projects.json`. Seed it yourself from the
  repos found in step 1 (they can edit later); keywords = repo basename plus
  anything obvious from the repo's README title:

  ```json
  {
    "thesis": {"repos": ["~/code/thesis"], "vault_dirs": ["Projects/Thesis"], "keywords": ["thesis", "experiments"]}
  }
  ```

  Create any `vault_dirs` that do not exist yet (empty dirs are fine — they
  are where project notes should go from now on).
- **todoist-tasks** → `~/.vault/todoist.env` with `TODOIST_API_TOKEN=...`
  (Todoist → Settings → Integrations → Developer).
- **calendar-sync** → `~/.vault/calendar.env` with `ICAL_URL=...` (Google
  Calendar → Settings → your calendar → "Secret address in iCal format").
- **Slack bridge** (Operator only) — walk them through, in order:
  1. https://api.slack.com/apps → **Create New App → From a manifest** → pick
     the workspace → paste:
     ```json
     {
       "display_information": {"name": "vault-bot"},
       "features": {"bot_user": {"display_name": "vault-bot", "always_online": true}},
       "oauth_config": {"scopes": {"bot": ["chat:write", "im:write", "im:history"]}}
     }
     ```
  2. **Install to Workspace** → OAuth & Permissions → copy the **Bot User
     OAuth Token** (`xoxb-…`).
  3. Their member ID: Slack profile → **⋮ → Copy member ID** (`U…`).
  4. `printf 'SLACK_BOT_TOKEN=xoxb-...\nSLACK_USER_ID=U...\n' > ~/.vault/slack.env && chmod 600 ~/.vault/slack.env`
  5. Prove it: `claude-slack send "vault-starter: hello"`.
  `task-worker` refuses to run without the bridge; the other agents print to
  the cron log instead.

Cron wiring — append only the chosen lines to `~/.vault/crontab.template`,
then `deploy-crontab --install`. Every line is wrapped in
`vault-run-if-primary`, so an identical crontab on a second machine never
double-runs. Guard each append so re-runs and upgrades never duplicate a
line:

```bash
add_cron() { grep -qF "vault-run-if-primary $1 " ~/.vault/crontab.template || echo "$2" >> ~/.vault/crontab.template; }
add_cron project-context  '0 * * * * ${HOME}/.local/bin/vault-run-if-primary project-context >> ${HOME}/.vault/sync.log 2>&1'
add_cron session-distill  '0 7 * * 1 ${HOME}/.local/bin/vault-run-if-primary session-distill >> ${HOME}/.vault/sync.log 2>&1'
deploy-crontab --diff && deploy-crontab --install
```

The full set of lines (first word after the guard is the module name the
`add_cron` check keys on; `reminder check` keys on `reminder`):

```
0 * * * * ${HOME}/.local/bin/vault-run-if-primary project-context >> ${HOME}/.vault/sync.log 2>&1
0 7 * * 1 ${HOME}/.local/bin/vault-run-if-primary session-distill >> ${HOME}/.vault/sync.log 2>&1
0 5 * * * ${HOME}/.local/bin/vault-run-if-primary vault-organizer >> ${HOME}/.vault/sync.log 2>&1
30 5 * * * ${HOME}/.local/bin/vault-run-if-primary vault-doctor >> ${HOME}/.vault/sync.log 2>&1
0 * * * * ${HOME}/.local/bin/vault-run-if-primary calendar-sync >> ${HOME}/.vault/sync.log 2>&1
*/30 * * * * ${HOME}/.local/bin/vault-run-if-primary todoist-tasks >> ${HOME}/.vault/sync.log 2>&1
0 8 * * * ${HOME}/.local/bin/vault-run-if-primary vault-digest >> ${HOME}/.vault/sync.log 2>&1
0 9,14,19 * * * ${HOME}/.local/bin/vault-run-if-primary task-nagger >> ${HOME}/.vault/sync.log 2>&1
0 * * * * ${HOME}/.local/bin/vault-run-if-primary task-worker >> ${HOME}/.vault/sync.log 2>&1
55 23 * * * ${HOME}/.local/bin/vault-run-if-primary journal-prompt >> ${HOME}/.vault/sync.log 2>&1
* * * * * ${HOME}/.local/bin/vault-run-if-primary reminder check >> ${HOME}/.vault/sync.log 2>&1
0 7 * * 1 ${HOME}/.local/bin/vault-run-if-primary refine-prefs >> ${HOME}/.vault/sync.log 2>&1
```

`vault-context`, `vault-tokens`, `test-fix` are on-demand — no cron line.
Secondary machine (Q4): `rm ~/.vault/.is_primary` now.

## 5 · The first loop (do this with them — it is the point)

Ten minutes, in this order, narrating what each step proves:

1. **Today's note.** Ask in chat for one line about what they are working on
   today and append it to `~/vault/$(date +%F).md` (create with a `# date`
   heading if new). This is the only file they need to touch by hand.
2. **Show the budget.** Run `vault-context` in the vault and show the
   output: the daily-note hook's bytes against the 2 KB cap, the global
   `CLAUDE.md` size, `RECENT.md` length. Explain that every session starts
   by loading exactly these things and nothing else — so the vault makes
   Claude smarter only to the extent these stay small and current.
3. **Project context (Scholar/Operator).** Run `project-context` once, then
   show the generated `CLAUDE.local.md` in one repo. Point out which vault
   files fed it. If it is thin, say why (no notes mention the project yet)
   and that the daily note they just wrote is the fix.
4. **Archive this session.** Run `save-convo` and show the new top line in
   `RECENT.md`. Explain that the Stop hook will do this for every session
   from now on, and that the global `CLAUDE.md` stub tells future sessions
   to read `RECENT.md` when they say "like last time".
5. **Sync.** `vault-sync` — show the commit. If they have no remote, print
   the `gh repo create` line (below) and leave it.
6. **The habit, stated once:** write the daily note; open Claude sessions
   inside the vault or a configured repo; everything else is automatic. The
   next thing that will happen without them is listed in the setup note.

## 6 · Verify, write the setup note, report

Run each check; fix loudly if one fails (no silent fallbacks):

```bash
git -C ~/vault log --oneline -1          # setup commit exists
vault-sync                                # exits 0 (no origin yet is fine)
grep -l SessionStart ~/vault/.claude/settings.json
grep -l Stop ~/.claude/settings.json      # only if archive chosen
grep -c 'claude-vault-starter:start' ~/.claude/CLAUDE.md    # global stub present (1)
crontab -l | grep -c vault-sync           # only if sync cron chosen
crontab -l | grep -c vault-run-if-primary # ≥ number of chosen cron extras
command -v vault-context kssh claude-slack save-convo deploy-crontab
command -v project-context session-distill ...        # every chosen extra resolves
vault-context; vault-doctor                            # cheap smoke runs, chosen ones only
```

Write `~/vault/Meta/Vault Setup.md` — the user's map of what is running,
so they never have to reconstruct it:

```markdown
# Vault setup — <hostname>, <date>

Preset: <Seed|Scholar|Operator>. Everything below was chosen on <date>;
re-run the assisted setup any time to change it.

## Running
| module | when | writes | undo |
| vault-sync | every 5 min | git history | edit ~/.vault/crontab.template → deploy-crontab --install |
| ... one row per installed module, on-demand ones marked "on demand" ...

## Not installed (and when to add it)
- vault-organizer — once loose notes pile up at the vault root
- ... modules you skipped with the one-line reason ...

## Check-in — <date + 14 days>
Open a Claude session in the vault and ask: "read Meta/vault_doctor.md and
Meta/Vault Setup.md — which modules have not written anything I looked at?
Propose removals." Removing is one crontab line + one file.
```

Add the check-in line to the daily note for that date
(`~/vault/<date+14>.md`) so the daily-note hook surfaces it on the day.

Then report to the user, short: each installed piece in one line, the first
automatic thing that will happen and when, how to undo any of it (hooks:
remove the entry from the settings.json; cron: edit the template then
`deploy-crontab --install`; scripts: `rm ~/.local/bin/<name>`; stub: delete
the marker block in `~/.claude/CLAUDE.md`). End with the optional next
steps, printed not executed:

```bash
gh repo create vault --private --source ~/vault --push   # backup + multi-machine sync
# second machine:  git clone <repo> ~/vault && ./setup.sh && rm ~/.vault/.is_primary
```

If `~/.local/bin` is not on their PATH, say so and give the export line.

---

## Full catalogue (only when the user asks for "the full menu")

**Core**: auto-sync · session archive · daily-note injection · global
CLAUDE.md stub · vault-context · comment-loop · Slack bridge.

**Context layer**: `project-context` (hourly, per-repo `CLAUDE.local.md`) ·
`session-distill` (weekly `Distill/` notes) · `refine-prefs` (weekly
`coding_prefs.md`) · `vault-organizer` (daily filing) · `vault-doctor`
(daily health + module audit).

**Agents that message you** (need the bridge): `vault-digest` (08:00) ·
`task-nagger` (09/14/19) · `task-worker` (hourly, acts on "yes") ·
`journal-prompt` (23:55) · `reminder` (per-minute firing).

**Pipelines in**: `calendar-sync` (hourly) · `todoist-tasks` (30 min).

**On demand**: `vault-tokens N` · `test-fix "<cmd>" <tries>` ·
`reminder add "+30m" "…"` · `kssh <host>`.
