# Module 1: Steer the Work

## Steering Long Sessions

### 🎯 What's the problem?

Short tasks with Claude are easy — you ask, it does the work, you check it. But **long tasks** (like refactoring many files or building a big feature) can take hours. The longer the task, the more you need to guide ("steer") Claude so it doesn't go off track.

There are 2 main habits to handle long sessions:

1. **Scope the work before Claude starts**
2. **Steer Claude while it's running**

### 1️⃣ Scope the work first — "Plan Mode"

- Before Claude writes any code, ask it to make a **plan** first.
- In Plan Mode, Claude only _reads_ your code (doesn't change anything yet), figures out what needs to be done, and shows you a plan.
- **Important:** Actually read the plan carefully, don't just skim it. A detailed plan = fewer surprises later.
- If something is missing or wrong in the plan, tell Claude to fix it _before_ it starts coding. Fixing a plan is much faster than fixing a mess after Claude has already written code.

### 2️⃣ Steer Claude while it works

#### 🔹 Compact

- `/compact` takes your whole conversation, summarizes it, and replaces the old messages with that summary — this frees up space (context window) so Claude can keep working.
- **Risk:** Something important might get left out of the summary, causing Claude to lose track of what matters.
- **Fix:** Don't run `/compact` alone — add instructions telling Claude what to focus on while summarizing. Example:

```
/compact Focus on the --version flag implementation
```

#### 🔹 Rewind

- If Claude goes in the wrong direction, you don't need to argue with it — just **rewind** back to an earlier checkpoint.
- Every prompt you send creates a checkpoint automatically.
- To open the rewind menu: double-tap **Escape** on an empty prompt.
- Options in the rewind menu:
  - **Restore code and conversation** – undo both code and chat
  - **Restore conversation** – undo only the chat
  - **Restore code** – undo only the files
  - **Summarize from here** – compress everything _after_ this checkpoint
  - **Summarize up to here** – compress everything _before_ this checkpoint (good for skipping a long setup phase but keeping the real work)

### 4️⃣ Running multiple Claude sessions at once — "Worktrees"

- If you run more than one Claude session on the same codebase at the same time, they can conflict with each other (like two drivers fighting over one steering wheel).
- **Worktrees** solve this by giving each session its own separate copy of the file tree, so they can't overwrite each other's changes.
- When a session ends, its worktree is automatically cleaned up.
- Special file: `.worktreeinclude` — placed at the root of your repo, it lists git-ignored files (like `.env` files or local configs) that should still be copied into every new worktree.

### ✅ Summary — Habits for Long Sessions

| Habit                                | Tool                         |
| ------------------------------------ | ---------------------------- |
| Plan before starting                 | Plan Mode                    |
| Control what's kept when summarizing | `/compact <instructions>`    |
| Undo/correct mistakes                | Rewind menu (double-tap Esc) |
| Run multiple Claude sessions safely  | Worktrees                    |

## A CLAUDE.md That Follows

### ⭐ Core idea (most important)

`CLAUDE.md` is **guidance, not enforced rules**. Every line competes for Claude's attention — the longer the file, the less reliably each rule gets followed. So keep it **short and tight**.

### ⭐ Hard rules vs soft rules (most important)

- **Soft conventions** → fine to put in `CLAUDE.md`.
- **Hard "never break this" rules** (e.g. "never push to main") → don't trust `CLAUDE.md` for these. Put them in a **pre-tool-use hook**, which can actually _block_ the action — real enforcement.

### ⭐ Phrasing makes rules stick (most important)

- Be **specific & checkable**: "Put API routes in `src/api/handlers`" ✅ vs "follow best practices" ❌.
- **Name the replacement**: When you tell Claude not to do something, say what to do instead. Don't just ban: "Use named exports, not default exports" ✅.
- **Emphasis is a budget** — `IMPORTANT`/`YOU MUST` only work if used sparingly on 2–3 critical rules, not everywhere.

### 🔹 Four file locations (minor detail)

`Managed policy` (org-wide) → `User` (personal, all projects) → `Project` (shared, in repo) → `Local` (gitignored, just you, this repo). All load together, always.

### 🔹 Imports (minor detail)

Split long files with `@path/to/file.md`. Helps organize only — Claude still loads everything, so it **doesn't reduce context**.

### 🔹 Keep revising (minor detail)

Treat mistakes as a "bug report" on `CLAUDE.md`. You can just tell Claude "add that to CLAUDE.md" to update it.

### ✅ Summary — CLAUDE.md Best Practices

- Move hard rules to hooks, where they're actually enforced.
- Organize long files with imports (just remember they don't reduce context).
- Keep revising CLAUDE.md as you find gaps or mistakes — it's a living document, not a one-time setup.
- Make rules specific, checkable, and actionable. Avoid vague "best practices" or "follow your instincts" guidance.

## Verification Skills

### ⭐ Core idea (most important)

Checking Claude's work manually depends on you _remembering_ to check. A **verification skill** removes that dependency — it fires automatically whenever a matching change is made, and always runs the same checks.

### ⭐ What a verification skill does (most important)

Triggered automatically (by its description matching the situation), it:

1. Runs the test suite
2. Reads the diff
3. Checks tests weren't weakened just to force a pass
4. Reports pass/fail with evidence

⚠️ Key point: "Done" ≠ tests look green. "Done" = the checks were actually run and the results are shown — not just eyeballing the diff.

### ⭐ Rule of thumb (most important)

If you've typed the same multi-step instruction **twice**, turn it into a skill (release checklists, migration steps, pre-PR checks, etc. all qualify).

### 🔹 A skill folder ≠ just one file (minor detail)

- `skill.md` = short main file (what to do).
- `reference.md` = optional deeper details, linked from skill.md, loaded only when needed.
- Scripts (e.g. `check.sh`) = Claude _runs_ them instead of reading them into context — keeps things efficient.

### 🔹 Where should a rule live? (minor detail)

| Rule type                                        | Goes in                                        |
| ------------------------------------------------ | ---------------------------------------------- |
| Always-true conventions (naming, file locations) | `CLAUDE.md`                                    |
| Task-specific procedures                         | a **Skill**                                    |
| Must-never-be-skipped rule                       | a **Hook** (actual code, not just instruction) |

### 🔹 Cost of adding skills (minor detail)

Only the skill's _description_ loads into context by default — full content loads only when triggered. So it's cheap to add many skills.

### ✅ Takeaway

Build a verification skill first, save it in `.claude/skills`, and your whole team automatically gets the same consistent checks on every change.

## Permission Modes

### ⭐ Core idea (most important)

Permission modes let you decide **once** what Claude can run without asking you each time — instead of approving every single action. Cycle through the everyday ones with **shift-tab** (status bar shows current mode).

### ⭐ The 6 modes (most important)

| Mode                   | What it does                                                                      |
| ---------------------- | --------------------------------------------------------------------------------- |
| **Manual**             | Only reads without asking; everything else needs approval                         |
| **Accept edits**       | Reads, file edits, common bash file commands run freely; you review after         |
| **Plan**               | Read-only — researches & proposes, edits nothing                                  |
| **Auto**               | Accepts everything, but a **classifier model** reviews each action before it runs |
| **Don't ask**          | Only pre-approved tools run; everything else auto-denied, no prompt               |
| **Bypass permissions** | Skips all checks — ⚠️ only use inside an isolated container/VM                    |

### ⭐ Auto mode explained (most important)

- A separate **classifier** checks _intent_ before each action runs.
- **Blocks:** production deploys/migrations, force pushes, piping downloaded code into a shell, sending sensitive data out, deleting session files.
- **Allows:** local edits, installing deps from lock file, read-only actions, pushing to your own branch.
- ⚠️ **Limitation:** classifier only checks if an action is _dangerous_, not if the code is _correct_. Broken code can still pass through.
  → Pair Auto mode with a **stop hook** that runs tests, so: classifier guards intent (before), stop hook guards correctness (after).

### 🔹 Don't ask mode (minor detail)

Best for unattended runs (CI, scheduled/overnight jobs) — no human around to approve, so only pre-approved tools run and the pipeline never hangs.

### ✅ Takeaway

Match the mode to the job: **Auto** for hands-off work (with a stop hook), **Don't ask** for unattended pipelines, **Bypass** only in isolated VMs/containers.

# Module 3 : Automate Repeat Work

## GitHub Actions and Code Review

### ⭐ Core idea (most important)

Two ways to put Claude to work on pull requests: **Code Review** (a managed service you just turn on) and the **GitHub Action** (DIY, for custom CI work beyond reviewing).

### ⭐ Code Review — the managed path (most important)

- Anthropic-hosted, works through the Claude GitHub app — nothing to build.
- Admin enables it in Claude Code admin settings → installs GitHub app → picks repos → sets timing: on PR open, on every push, or only when someone comments `@claude review`.
- Analyzes the **full diff against the whole codebase** (not just changed lines), posts **inline comments** tagged by severity + a summary table. Also deduplicates/ranks findings so you don't get flooded with nitpicks.
- ⚠️ **Limits:** never approves/blocks a PR (human decides), no auto-fix in the service itself, still a research preview (team/enterprise plans).
- To actually apply a finding: run `/code-review --fix` locally. Since there's no autofix in the service, applying a finding is a local move. From your own terminal, the /code-review command reviews a diff, and its --fix flag applies the findings to your working tree.

### ⭐ GitHub Action — the DIY path (most important)

Use when the job is more than review (e.g. implementing a requested change, scheduled reports, custom CI).

- Setup: run `/install-github-app` in Claude Code (needs repo admin) — installs the GitHub app and sets the API key secret.
- Action used: `anthropics/claude-code-action@v1`.
- Key inputs: `github_token`, `trigger_phrase` (default `@claude`), `prompt`, `claude_args` (extra CLI args passed to Claude Code), plus `use_bedrock`/`use_vertex` if on those providers.
- **Two common workflows:**
  - Comment-triggered: listens for `@claude` on PR/issue comments (e.g. `@claude implement the spec...`) → Claude pushes commits + comments.
  - Scheduled: cron trigger (e.g. daily) for rollups/reports; can add `workflow_dispatch` to also trigger manually.

### ⭐ Tuning with claude_args (most important)

- `--max-turns 5` → caps the agent loop so it can't run forever.
- Set a **permission mode** that doesn't stop to ask (no human present in CI).
- **Allowed tools** → give only what's needed (e.g. read-only for reports).

### ✅ Decision guide

- **PR reviews** → use managed **Code Review**, fix locally with `/code-review --fix`.
- **Anything beyond review** (implementing changes, scheduled jobs) → use the **GitHub Action**.

# Module 4 : Verify and Share

## Trust It: Verifying Unsupervised Runs

### ⭐ Core idea (most important)

**Verify in proportion to how much you _didn't_ watch.** A quick glance is fine for a session you watched live; an unattended/CI run with nobody watching needs a real, thorough check — you have to reconstruct what happened after the fact.

### ⭐ Keep unattended runs in Auto mode (most important)

Use **Auto**, not Bypass permissions — the classifier still screens each action for danger. But remember: the classifier only catches _dangerous_ actions, not _incorrect_ code. So your own verification bar doesn't change just because a classifier is watching.

### ⭐ Start with the diff, not the summary (most important)

- Don't trust Claude's tidy summary — it can look clean while the diff touched something unexpected.
- Run `/code-review` to flag issues, **and** personally read `git diff`.
- Check files that were part of the plan first, then look for anything outside the plan.

### ⭐ Turn tests into a gate, not a promise (most important)

Don't just trust Claude's claim that tests passed — enforce it with hooks:

- A **Stop hook** that runs tests and refuses to end the turn on failure.
- A **PostToolUse hook** that lints/type-checks after every edit.
- Key detail: exit with **exit code 2** so the failure is fed back to Claude automatically, and it fixes it without you having to ask. The key detail is the exit code. A hook that exits with exit 2 feeds the failure straight back to Claude. Claude reads that failure and fixes it without you asking.

## Plugins

### ⭐ Core idea (most important)

A **plugin** packages a whole `.claude` setup (skills, subagents, hooks, MCP configs, settings, etc.) into **one installable unit**, so your team doesn't have to copy-paste files between machines to stay in sync.

### ⭐ Installing plugins (most important)

- Install directly: `/plugin install org-name@plugin-name`, then /reload-plugins` to apply.
- For teams, add a shared **marketplace** once: `/plugin marketplace add your-org/claude-plugins` — gives centralized discovery, version tracking, and updates for everyone. Browse via the **Discover** tab.

### ⭐ Read before you install (most important — safety)

A plugin **runs code on your machine with your privileges**. Its hooks fire on every matching tool call — even if you only wanted its skills.

- Example risk: a hidden Stop hook could call an external endpoint every time, with no warning in your config.
- Claude Code shows what a plugin will install + estimated context cost, and warns Anthropic doesn't control third-party plugin content.
- "Reviewed" (community marketplace, automated review) ≠ "trusted" — only install plugins/marketplaces from sources you actually trust.

### ⭐ How plugin components interact with yours (most important)

- **Hooks stack** — a plugin's PreToolUse/Stop hooks run _alongside_ yours, not replacing them (another reason to check hooks first).
- **Skills/agents/commands** are namespaced under the plugin name, so no clashes with your own.
- Manage/uninstall anything from the plugin panel.

### 🔹 Packaging your own plugin (minor detail)

Uses the **same `.claude` folder structure** you already have (one folder per skill, one `.md` per subagent under `agents/`, `hooks/hooks.json`, `.mcp.json`) — Claude Code discovers components by convention.

Optional manifest at `.claude-plugin/plugin.json`:

```json
{
  "name": "svg-splitter-review",
  "version": "0.1.0",
  "description": "Reviews the SVG Splitter repo",
  "author": { "name": "Lewis Menelaws" }
}
```

- `name` is the only required field — namespaces skills as `company-name:skill-name`.
- Version it like any dependency, for update tracking.

### ✅ Takeaway

**Using** plugins → read before you install (hooks/agents/MCP run with your privileges). **Building** plugins → package your working `.claude` folder with a manifest, so the whole team gets one trusted install.
