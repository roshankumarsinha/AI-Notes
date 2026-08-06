# Module 1: What is Claude Code?

## What is Claude Code?

### 🧠 Simple Definition

Claude Code is a agentic coding tool that doesn't just talk about code — it actually **does the work for you**. It can look at your project, edit your files, run commands in your terminal, and use other developer tools, all to help you finish coding tasks faster.

You can use it in several places:

- Terminal
- Visual Studio Code
- Claude Desktop app
- On the web
- JetBrains IDEs

### 🤔 How is it different from regular Claude (Claude.ai)?

With Claude.ai, you usually copy code out, paste it into the chat, get a suggestion, then copy it back into your project yourself.

With Claude Code, there's no copy-pasting. It has **direct access** to your files, terminal, and whole codebase — so it just makes the changes itself.

The big reason for this difference: **Claude Code acts as an "AI Agent."**

### 🤖 What is an "Agent"?

An AI Agent is a program that can:

1. Look at its environment (like your codebase)
2. Take actions to reach a goal (like editing a file or running a command)

Under the hood, it's a large language model working in a loop — thinking, acting, checking results, and deciding what to do next — repeatedly, until the task is done. Agents can also use tools, connect to external services, or even work with other AI agents.

### 🛠️ What Can Claude Code Actually Do?

| Ability                | Example                                                |
| ---------------------- | ------------------------------------------------------ |
| Read & understand code | Explain a feature, or trace where a bug is coming from |
| Edit files             | Refactor a function and update every place it's used   |
| Run terminal commands  | Run your build script, execute tests, install packages |
| Search the web         | Look up docs or the latest API info when needed        |

### 🎯 3 Key Concepts to Use It Well

1. **Context Window (its "working memory")**
   Claude can't hold your _entire_ codebase in memory at once. Instead, it smartly searches for the relevant pieces it needs — this is part of what makes it "agentic" rather than just reading everything blindly.

2. **It Asks for Permission**
   By default, Claude Code checks with you before running commands or making changes. You stay in control — you can be hands-on (reviewing every step) or more hands-off, depending on your preference.

3. **It Can Make Mistakes**
   Claude Code isn't perfect. It might misunderstand what you want, introduce a bug, or overcomplicate a solution. That's why it's important to stay involved and review its work, especially early on.

## How Claude Code Works

### 🔁 The Agentic Loop

This is the core idea of how Claude Code operates. Think of it as a repeating cycle:

1. **You give a prompt** — you type what you want.
2. **Claude gathers context** — it talks to the model, which replies with either text or a "tool call" (a request to do something, like read a file).
3. **Claude takes action** — for example, editing a file or running a command.
4. **Claude checks its own work** — it verifies whether the result actually matches what you asked for.
5. If the result is good → **it stops and waits for your next prompt.**
   If not → **it loops back and tries again**, repeating until the task is truly done.

💡 The cool part: while this loop is running, _you_ can jump in anytime — add more context, interrupt it, or redirect it toward your actual goal.

### 🧩 Context (Working Memory)

Claude has a **context window** — basically a limit on how much conversation history, file content, and command output it can "remember" at once.

Once that limit is reached, Claude Code automatically **compacts** the conversation — meaning it summarizes or trims out less important parts so it still has room to keep working.

### 🛠️ Tools

Tools are what make Claude Code an "agent" rather than a basic chatbot.

- A normal AI assistant just takes text in and gives text out.
- Claude Code's **tools** let it actually _execute things_ — like reading a file, searching the web, or running a command — instead of only describing what to do.

Claude decides on its own when a tool is needed and how to use the result, based on understanding the meaning of your request (not just matching keywords).

### 🔐 Permissions (How Much Control Claude Has)

Claude Code has different **permission modes** you can choose from:

| Mode            | What Happens                                                                            |
| --------------- | --------------------------------------------------------------------------------------- |
| **Default**     | Claude asks you before editing files or running shell commands                          |
| **Auto-accept** | Files get edited automatically (no asking), but commands still need your approval       |
| **Plan mode**   | Claude only uses "read-only" tools to build a plan first — no actual changes happen yet |

⚠️ **Be careful** with skipping permissions — if you let Claude Code run commands freely without checking in, it becomes harder to catch a mistake before it actually happens. All of this is configurable in your settings file.

### ✅ Quick Recap

Claude Code works by combining four big ideas:

- an **agentic loop** (act → verify → repeat)
- a **managed context window** (its memory, with auto-compacting)
- **tools** (so it can actually do things, not just talk)
- **configurable permissions** (so you stay in control)

# Module 2: Your First Prompt

## Installing Claude Code

### 💻 Terminal Installation

**macOS / Linux / WSL:**

- Use the `curl` command to install it in one step.

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

**After installing:**

1. Run the `claude` command in your terminal (restart the terminal first if it doesn't work).
2. Navigate to your project folder, then run `claude`.
3. Go through first-time setup — pick a color theme and sign in (with Claude Pro, Max, Enterprise account, or an API key). If you're on an Enterprise account, make sure to select that option specifically.

⚠️ **Important:** Whatever folder you run `claude` in, it gets access to that folder _and all its subfolders_ — so run it from your actual project directory.

### 🧩 Visual Studio Code

1. Open the **Extensions** panel.
2. Search for "Claude Code" — look for the one made by **Anthropic** (has a blue verification checkmark).
3. Click Install, then restart VS Code if needed.
4. Open it via Command Palette (`Cmd/Ctrl + Shift + P`) → search "Claude Code Open in New Tab," or just click the Claude logo in your sidebar.

You can also skip the visual UI and just use the terminal-style experience instead, via your settings.

### 🖥️ Desktop App

- After installing and signing into the Claude Desktop app, you'll see a toggle at the top labeled **"Code."**
- It looks similar to the regular chat interface, but lets you work inside a specific folder, adjust permissions, and even run things in a cloud environment.

### 🌐 Web

- Go to **claude.ai/code**, or click "Code" in the sidebar of the chat app.
- Works like the desktop version, but only supports **GitHub repositories** (not local folders).

## 🤔 Which Should You Use?

| Option                        | Best For                                                        |
| ----------------------------- | --------------------------------------------------------------- |
| **Terminal**                  | Getting the newest features first (they ship here first)        |
| **IDE (VS Code / JetBrains)** | Working right inside your code editor                           |
| **Desktop**                   | Letting Claude work in the background while you do other things |
| **Web**                       | Working remotely on a GitHub repo                               |

## Your First Prompt

### ✅ Auto-Accept vs. Approval Mode

You get to choose how much Claude checks in with you before making changes.

Press **Shift + Tab** to cycle between modes:

| Mode                 | What Happens                                                                       |
| -------------------- | ---------------------------------------------------------------------------------- |
| **Approval mode**    | Claude asks permission every time before editing a file or running a command       |
| **Auto-accept mode** | File edits happen automatically (no asking), but commands still need your approval |

There's no "correct" mode — just pick whatever level of control feels comfortable to you.

### 📝 Plan Mode

Also found in the Shift + Tab menu is **Plan Mode**. Instead of jumping straight into making changes, Claude:

1. Uses only **read-only tools** to study your codebase.
2. Researches how your requested feature could be implemented.
3. Asks you clarifying questions along the way.
4. Returns a **detailed plan** before touching any code.

This is especially useful for complex, multi-step changes, or when you want a safer way to review what Claude intends to do before it actually does it.

### 💡 Example Walkthrough: Adding a Dark Mode Toggle

Here's how you might use Plan Mode in practice:

1. Open your project's root folder and run `claude`.
2. Press **Shift + Tab** a couple of times to enter Plan Mode.
3. Write a clear, descriptive prompt — for example, asking Claude to add a dark/light mode toggle to your app's header, matching your existing theme's contrast.
4. Review the plan Claude comes back with.
5. If it looks good, accept it — Claude will then ask for your approval at each step as it implements the plan.
6. At the end, you can see exactly what Claude did and why.

### ✅ Quick Recap

Two big takeaways from this lesson: try to be as **descriptive as possible** in your prompts, and use **Plan Mode** whenever you want Claude to fully understand the task and show you a plan before writing any code. You're always able to stay as involved as you want, at every step.

# Module 3: Daily Workflows

## The Explore → Plan → Code → Commit Workflow

### 🧠 Simple Definition

This is probably the **single most important habit** to take from the whole course. Instead of jumping straight to "write me this code," follow four steps: **Explore, Plan, Code, Commit**.

### 🔍 Step 1 & 2: Explore and Plan

The fastest way to do these two steps together is with **Plan Mode**.

- In Plan Mode, Claude **cannot edit any files** — it can only _read_ files to understand your codebase and figure out how it would approach the task.
- To enter it: press **Shift + Tab** until "Plan Mode" shows under the text box.
- Then give a clear prompt describing what you want to build or fix.

Claude will read relevant files, maybe do some web searches, and hand you back a **plan of action**. You review it — if something's off, ask Claude to revise specific parts.

💡 This is the best moment to catch problems, since no code has been written yet, so changes are cheap and easy.

(Side note: you can also just use the "explore" subagent without full Plan Mode if you only want a summary of your codebase, without planning to make changes.)

### 💻 Step 3: Code

Once the plan looks good, approve it and let Claude start working through it.

- You can choose auto-accept or manual approval for file edits, just like before.
- Claude will try to troubleshoot issues on its own, but sometimes you'll need to step in.

**Tips to make this phase smoother:**

- **Define success criteria** — be explicit about what "correct" looks like so Claude knows when it's actually done.
- **Add helpful tools** — e.g., installing the "Claude in Chrome" extension lets Claude control a browser tab directly to test web UIs itself.
- **Include a test suite** — give Claude tests to check its work against (it can even write these tests for you). Just make sure the tests are trustworthy, so you don't get false "it works!" results.
- **Quick tip:** if Claude keeps repeating the same mistake, ask it to save the fix into its `CLAUDE.md` file so it remembers going forward.

### ✅ Step 4: Commit

Once you've tested things yourself and you're happy with the results:

1. Run a **subagent code reviewer** to double-check your work before committing. A subagent looks at the code with "fresh eyes," without the bias the main agent may have built up during the session.
2. Ask Claude to **generate a commit message** in your usual style.
3. Commit, and repeat the whole cycle for your next feature.

## Context Management

### 📦 What is the Context Window?

Think of it as a container with limited room. Anything that happens during your session — your prompts, files Claude reads, tool calls and their results — all gets added into this space. Because it's finite, it's important to be thoughtful about how you fill it up.

### ⚠️ What Happens When It Fills Up

Once you're close to the limit, Claude Code automatically **compacts** the context — meaning it summarizes the important parts and drops less-necessary tool results to free up room. Keep in mind: this process _can_ lose some details along the way.

### ⌨️ Useful Commands

| Command    | What It Does                                                                                              |
| ---------- | --------------------------------------------------------------------------------------------------------- |
| `/compact` | Manually compacts everything so far — frees up space while keeping a summary/memory of your previous work |
| `/clear`   | Wipes everything — starts completely fresh with no memory of the previous session                         |
| `/context` | Shows an overview of your context size, what's taking up the most space, and a visual breakdown           |

### 🤔 When to Use Which

- Use **`/compact`** when you're still working on the _same_ feature and are running low on context space, but want to keep going without losing your progress memory.
- Use **`/clear`** when you're starting a completely _new_ feature — this avoids the old conversation accidentally biasing Claude's approach to the new task. If there's something important you want Claude to always remember across sessions, save it in your **`CLAUDE.md`** file instead of relying on context memory.

### 💡 Tips for Saving Context Space

1. **Be specific in your prompts.** A vague prompt might look "shorter," but it actually costs _more_ context overall — because Claude has to explore your codebase and reason more to fill in the gaps. Clear, detailed prompts save space in the long run.
2. **Manage your MCP servers.** MCP servers load all their available tools into context by default — even ones you're not currently using. If you have servers set up for unrelated things, turn them off for the current project. "Skills" are a similar alternative that don't load everything upfront.
3. **Use subagents.** Subagents run in parallel with your main agent, but have their _own separate_ context window. For simple lookup-style questions (like "where are the authentication endpoints?"), a subagent can do the digging and just report back a short summary — keeping your main context clean and uncluttered.

## Code Review

### 👀 Review with a Subagent

Before pushing a pull request (PR), ask Claude to use a **subagent** to review your changes.

- The subagent runs in its **own separate context window**, giving it "fresh eyes" — it isn't influenced by the bias the main agent might have built up while writing the code all session.
- When setting up a code-reviewer subagent, restrict it to **read-only tools**. Its job is to flag issues, not edit files.
- Save the subagent's configuration into your repo so your **whole team** uses the exact same reviewer setup.

### ⚡ The `/commit-push-pr` Skill

Instead of manually committing, pushing, and creating a PR as three separate steps, the `/commit-push-pr` skill does **all three in one go**.

- Bonus: if you have a **Slack MCP server** set up with channels listed in your `CLAUDE.md`, it will automatically post the new PR link to your team's Slack channel.

### 🔗 Resuming Work with `--from-pr`

When Claude creates a PR (using `gh pr create`), that session automatically gets **linked to the PR**.

- If you need to come back later — say, to address review comments or fix a failing build — just run:

```bash
claude --from-pr <PR_NUMBER>
```

- This picks the session back up exactly where you left off, so you don't have to re-explain context from scratch.

# Module 4: Customizing Claude Code

## The CLAUDE.md File

### ❓ The Problem It Solves

Without a `CLAUDE.md` file, every time you open Claude Code it starts completely fresh:

- It has to re-explore your codebase.
- It has to figure out what dependencies you're using.
- It has to guess what features already exist — sometimes making wrong assumptions, which makes it harder to steer in the right direction.

`CLAUDE.md` fixes this. It's a plain Markdown file placed in your project's root folder, and Claude Code automatically reads it at the start of every session. Think of it like an **onboarding script** for your codebase — its contents get added onto your prompt automatically.

### 📄 An Example

A typical `CLAUDE.md` might include:

- What tech stack the project uses (e.g., framework, styling tool, database ORM)
- Common commands (like how to start the dev server, run tests, or lint)
- Code style preferences (indentation rules, export style, folder conventions, etc.)

Once this is in place, if you ask Claude to build a new component, it already knows your conventions and styling approach without you repeating yourself.

### 👥 CLAUDE.md is for Teams Too

You should commit your `CLAUDE.md` to version control so your whole team benefits from it. There's actually a small hierarchy of memory files:

| Type                        | Location                    | Who It's For                                     |
| --------------------------- | --------------------------- | ------------------------------------------------ |
| **Project-level CLAUDE.md** | Root of your project        | Shared with your whole team                      |
| **User-level CLAUDE.md**    | Your personal config folder | Just for you, applies across _all_ your projects |

### 💡 Tips

1. **Save corrections to memory.** If you keep correcting Claude on the same thing (e.g., "always use server actions instead of API routes"), just ask Claude to save that rule to memory — next time, it'll already know.
2. **Reference project docs with `@`.** You can point Claude to existing documentation using the `@` symbol followed by the file path (e.g., referencing your `README.md`) so it reads that file for extra context.
3. **Start without one first.** It's recommended to begin a project _without_ a `CLAUDE.md`, so you can notice what you keep correcting Claude on. This keeps your eventual file compact and only full of necessary info. When ready, run `/init` and Claude will generate one for you automatically.

## Subagents

### 🧠 Simple Definition

Claude can hand off tasks to **subagents**, which break work into smaller pieces and run them separately — helping keep your main context window clean. Each subagent has its **own isolated context window**, completely separate from your main conversation.

### ⚙️ How It Works

A lot of your context window gets used up by things like exploring the codebase or running web searches — and honestly, most of that exploration "journey" isn't something you need to see. You just want the _answer_.

That's exactly what subagents solve:

1. Claude spawns a subagent to handle a specific task (e.g., "explore this codebase for me").
2. The subagent works in **parallel**, using its own separate context window to do all the digging.
3. Once done, it summarizes what it found and sends that summary back to the main Claude agent.

**Result:** you get the answer you needed, without all the messy exploration steps cluttering your main context.

### 🛠️ Creating Your Own Subagent

Subagents are defined in Markdown files with YAML frontmatter (a structured header). The easiest way to make one is to let Claude build it for you:

1. Run `/agents`
2. Select **"Create new agent"**
3. Walk through the setup: choose the agent's scope, define its purpose, pick which tools it can access, and even choose a color for it.

Claude will then generate a name, description, and prompt for the subagent — this also tells the main Claude _when_ it should automatically call this subagent based on your future prompts.

### 🎛️ Further Customization

A couple of extra options for subagents:

- **Persistent memory** — lets a subagent remember things across multiple conversations. Great if you use it repeatedly on the same project.
- **Preloaded skills** — you can add a `skill` key and list skills by name so they load automatically. Note: unlike skills used in your main conversation, here the _entire_ skill gets loaded into context upfront.

### ✅ Quick Recap

Keeping your context window clean is one of the best ways to stay productive in Claude Code. Subagents let you run background work — the heavy lifting — and only bring back the final answer into your main context.

## Skills

### 🧠 Simple Definition

**Skills** are a feature designed to stop you from repeating the same instructions to Claude over and over. A Skill is basically a Markdown file where you write down instructions or standards **once** — and Claude figures out on its own when to use that knowledge based on what you ask it to do.

### ⚡ How Skills Are Different from Slash Commands

- **Slash commands** (like `/compact`) need you to manually trigger them every time.
- **Skills** are automatic — Claude compares your request against the descriptions of all available skills, and activates the relevant one **on its own**, without you needing to call it directly.

### 🧩 Why Skills Help with Efficiency

Skills only load the specific information needed for a task **at the exact moment it's needed** — instead of stuffing everything into context upfront. This keeps your context window cleaner and less cluttered (similar in spirit to why subagents help keep context clean, but for reusable instructions instead of one-off delegated tasks).

### 📂 Where Skills Live

| Type                | Location                                       | Best For                                                                                                                                 |
| ------------------- | ---------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| **Personal Skills** | `~/.claude/skills` (your home directory)       | Follows you across _all_ your projects — good for personal preferences, like your preferred commit message style or documentation format |
| **Project Skills**  | `.claude/skills` folder inside the repo's root | Automatically shared with anyone who clones the repository — great for team-wide standards like brand guidelines or coding conventions   |

### 💡 Bottom Line

If you notice yourself explaining the **same instructions** to Claude again and again, that's a perfect sign it should become a Skill instead — write it once, and let Claude handle it automatically going forward.

### ✅ Quick Recap

Skills = reusable instructions written once in a Markdown file, automatically activated by Claude based on context (not manually triggered like slash commands), and stored either personally (just for you) or at the project level (shared with your team).

## MCP

### 🧠 Simple Definition

**MCP (Model Context Protocol)** is an open standard that lets Claude Code connect to external tools and data sources beyond your codebase. When you ask a question, Claude automatically figures out when it needs to use one of these connected tools to help answer it.

A lot of useful context lives _outside_ your codebase — in databases, productivity apps, or public repositories — and MCP is the bridge that connects Claude to all of that.

### 🛠️ What Can You Do With It?

First, a quick concept: **"tools"** give agents like Claude Code the ability to actually _perform actions_, not just respond with text — this is what separates agentic AI from a typical chatbot.

Some examples of what MCP servers let you connect to:

- A **Jira MCP server** — brings in details from your team's project management issues.
- You ↔ Claude ↔ [MCP Server] ↔ Jira's API ↔ Your Jira workspace.
- Once you connect it, the MCP server exposes a defined set of "tools" that Claude can call — things like:
  - search_issues — find tickets matching a filter (e.g., "assigned to me", "in sprint X")
  - create_issue — file a new bug/task with title, description, project, priority
  - update_issue — change status, reassign, add labels
  - add_comment — comment on a ticket
  - get_issue — pull full details of a specific ticket

So instead of you switching tabs to Jira, you can just say in chat: "Show me all open bugs assigned to me in the Payments project" or "Create a ticket in Jira for this login bug, priority high" — and Claude calls the right MCP tool behind the scenes, which talks to Jira's actual API, and returns real data or performs the real action.

### ➕ Adding an MCP Server

You add servers using the `claude mcp add` command. There are two main types:

| Type              | Use Case                                                               |
| ----------------- | ---------------------------------------------------------------------- |
| **HTTP servers**  | For remote services hosted by the provider, connected over the network |
| **Stdio servers** | For local processes running on your own machine                        |

Use `/mcp` inside a Claude Code session to see what's connected, check statuses, and disable servers you don't need.

### 🔍 Scoping Servers

MCP servers can be set up at three different levels:

- **Local** — only available in your current project, just for you.
- **User** — available across _all_ your projects.
- **Project** — configured via a `.mcp.json` file that's checked into version control, so everyone on the team automatically gets the same servers.

### 💰 Context Costs

MCP servers add their tool definitions into your context window — **even when you're not actively using them**. If you've got a lot of servers configured, this can eat into your available context space.

A few tips to manage this:

- Run `/mcp` regularly to check what's connected, and disable anything you're not actively using.
- If a tool has a **CLI equivalent** (like `gh` for GitHub, or `aws` for AWS), using the CLI is more context-efficient since it doesn't add persistent tool definitions.
- Consider using a **Skill** instead — a Skill only loads its name and description into context upfront, and only loads full contents when Claude actually decides it needs to use it.
- If your MCP tools take up more than **10% of your context window**, Claude Code automatically switches to a "tool search mode" that discovers the right tool on demand — though this may be a bit less reliable.

## Hooks

### 🧠 Simple Definition

**Hooks** let you run commands at specific points during Claude Code's process. The key thing that makes hooks different from everything else in this course: they are **deterministic** — meaning they _always_ run, no exceptions.

### 🤔 Why Use Hooks

You could tell Claude in your `CLAUDE.md` to run something like Prettier (a code formatter) after every file edit — and most of the time it will listen. But "most of the time" isn't the same as "every single time." A hook guarantees it happens **every time**, without fail.

Common use cases:

- Auto-formatting code after file edits
- Logging every command that gets run (useful for compliance)
- Blocking dangerous actions, like editing production files
- Sending yourself a notification when Claude finishes a task

### ⚙️ How They Work

Hooks are configured in your `settings.json` file. You choose:

- An **event** (when the hook should trigger)
- Optionally, a **matcher** (which tools it should apply to)
- A **command** to run when triggered

**Available events:**

| Event              | When It Runs                                         |
| ------------------ | ---------------------------------------------------- |
| `PreToolUse`       | Before a tool call happens                           |
| `PostToolUse`      | After a tool call finishes                           |
| `UserPromptSubmit` | When you submit a prompt, before Claude processes it |
| `Stop`             | When Claude finishes responding                      |
| `Notification`     | When Claude sends a notification                     |

You can set these up using the `/hooks` command inside Claude Code, or by editing `settings.json` directly.

### 💡 Practical Example: Auto-Formatting

The most common hook is auto-formatting after edits:

- Set a `PostToolUse` hook with a matcher like `"Edit|MultiEdit|Write"` so it triggers whenever Claude modifies any file.
- The command checks the file extension and runs the right formatter — Prettier for TypeScript, `gofmt` for Go, or whatever fits your project.

### 🚫 Blocking Actions with PreToolUse

`PreToolUse` hooks can actually **block** a tool call before it happens. The hook receives the tool name and its input as JSON, and the exit code decides what happens next:

| Exit Code      | Meaning                                                                                            |
| -------------- | -------------------------------------------------------------------------------------------------- |
| `0`            | Proceed normally                                                                                   |
| `2`            | Block the action — the error message gets sent back to Claude so it understands why and can adjust |
| Any other code | Non-blocking error — shown to you, but doesn't stop anything                                       |

This is how you enforce **hard rules** rather than just suggestions — for example: blocking writes to a production config folder, blocking dangerous bash commands (like `rm -rf`), or blocking commits directly to the `main` branch.

### 👥 Sharing Hooks with Your Team

Hooks set in `.claude/settings.json` are project-level, so you can check them into your repo — meaning your whole team automatically gets the same hooks. Use the `CLAUDE_PROJECT_DIR` environment variable in your commands to reference scripts stored in your project, so they still work no matter what Claude's current working directory happens to be.

💬 **Rule of thumb:** if something absolutely must happen every single time without fail, don't just ask for it in a prompt — put it in a hook.
