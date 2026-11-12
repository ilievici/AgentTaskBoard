# User guide

Agent Task Board turns a Jira ticket into a task folder, has an agent plan and implement it, and shows every task
on one board. This page explains each part.

## Start the board

On Windows, install `AgentBoard-Setup-<version>.exe` from the
[latest release](https://github.com/ilievici/AgentTaskBoard/releases/latest), then open **Agent Task Board** from
the Start menu. It opens <http://localhost:5290> in your browser.

From source:

```bash
dotnet run --project src/AgentBoard.Service
```

Open <http://localhost:5291>. To open the browser on start, set `AgentBoard__OpenBrowser=true`.

Before your first task, open **Settings** and check these:

1. **Paths > Tasks root.** Where task folders go. Default: `AgentTasks` in your user folder.
2. **Paths > Repositories.** The local clones you work on. They show as one-click choices on new tasks.
3. **Jira.** Site URL, account email, default project such as `SFM`, then **Save token** with an
   [API token](https://id.atlassian.com/manage-profile/security/api-tokens).
4. **Executors > Claude Code.** The command (`claude`, or a full path), the model and the permission mode.

## Profiles

A profile sets the board up for one kind of work: its task types, board columns, house rules and prompts. On a new
board, the welcome page opens once so you pick one. Settings saved before profiles existed use Software development.

The board ships the example profiles of the roadmap: Software development, Writing and content, Research and
analysis, Legal and compliance, Finance and reporting, Recruiting and HR, Teaching and training, and Support and
operations. The profiles other than software use prompts for work without code: no commits, a source for every
claim, and drafts only. The agent never sends or publishes anything.

Profiles are data, not code. Each one is a folder with a `profile.json` and one Markdown file per template it changes,
such as `prompt-plan.md`. A profile can take the prompts of another with `"basedOn"`. Its `"tools"` list sets which
programs System health checks and how much it needs each one, so a writing board never warns about the GitHub CLI.
The shipped profiles are in `profiles/` next to the program. Put your own in `profiles/` in the home folder. One with
the ID of a shipped profile replaces it.

**Settings > Profile** changes the profile, exports your setup as a profile file, imports one, and saves your setup as a
new profile. Changing the profile replaces the task types and columns. A column that still holds tasks stays on the
board, and your other settings don't change. A template you saved in Settings wins over the profile's.

## The lifecycle

| Step | You click | What happens | Column |
| --- | --- | --- | --- |
| Create | **New task** | Folder, `scope.md` and the spec-kit skeleton are written. | Backlog |
| Plan | **Plan** | The agent reads the scope and the code, then writes `plan.md` and `tasks.md`. | Backlog, then Planned |
| Review | edit steps | Change, add, delete or reorder steps. Each edit writes `tasks.md`. | Planned |
| Approve | **Approve plan** or **Reject with comment** | Reject starts a re-plan with your comment. | Planned |
| Run | **Run N steps** | The agent implements the open steps and ticks them in `tasks.md`. | In progress |
| Finish | | All steps done goes to Review. A failure or Stop goes to Blocked. | Review or Blocked |
| Answer | **Send answer and continue** | The agent asked a question and stopped. Your answer continues its session. | Awaiting input, then In progress |
| Resume | **Resume from T00X** | Runs again from the first unfinished step. | In progress |
| Close | **Close and write memory** | The agent writes `summary.md` (what was delivered) and `memory.md` (learnings). | Done |

You can also drag a card to any column. That only changes the status, it starts nothing.

## The board

- Each card shows the ticket ID (click it to open Jira when the ID is a Jira key), the title, repositories, tags,
  pull requests the agent opened, step progress, the next step, the last activity and the executor.
- The ticket ID is optional and works with any system: `SFM-1234`, `#482`, `INC0012345`.
- Click the arrow on a card to see all its steps.
- A blue border and a spinning badge mean an agent runs on the task now.
- Each column has its own color. Change it in Settings > Board.
- Search with free text, or narrow with `tag:dapr`, `repo:planner`, `status:review`, `component:planner`.
- The board updates by itself when files change, including edits you make outside the tool.

### Archive

The Done column fills up over time. To clear it, archive the tasks you no longer need on the board:

- To archive one task, hover over its card and click the archive icon, or click **Archive** on the task page.
- To archive every task in Done, click the archive icon in the Done column header.

The **Archive** page lists archived tasks, the most recently archived first. Search it with the same filters as the
board. Click a title to open the task, or click **Restore** to put it back in its column.

Archiving only hides the task from the board. Its folder, memory and logged time stay as they were. You can't
archive a task while a run is active or queued.

### New task from chat

Click **Chat** on the board and describe the task in your own words, in any language. Name the app, the
problem and what done looks like. Enter sends.

- The card shows at once, with your first line as a provisional title and an **Intake** run badge.
- The agent reads your text, the task types, the known repositories and the tags in use. It writes `intake.json`.
- The board then sets the title, type, ticket ID, tags, repositories, description and acceptance criteria.
  Your text stays in `request.md` and under "My notes" in `scope.md`.
- The agent only picks repositories from Settings > Repositories or from earlier tasks. A path that doesn't
  exist is dropped with a warning.
- If the run fails, the card stays and the timeline says to fill it in by hand.
- The chat panel keeps its messages until you close the browser tab. Each answer links to its task.

### The board as an MCP server

The board serves MCP at `http://localhost:5290/mcp`. Every Claude Code run connects to it for its own task, so
the agent can report progress with tools instead of editing files:

- `step_start`, `step_done`, `step_failed` and `list_steps` for the steps in `tasks.md`.
- `add_note`, `add_link` and `ask_question`. A question goes to the Awaiting input box as before.
- `move_card` moves the card, but never to Done. Only you do that. When the run ends, the board still sets the
  column from the result.

**Command approvals.** When the agent wants a tool that isn't on the allowed list, the run waits and the task page
shows the call with **Allow** and **Deny**. Add a note to tell the agent why. With no decision in 10 minutes the
call is denied. Blocked tools, such as `git push`, are refused without asking.

**Use it from your own Claude Code.** Run `claude mcp add --transport http agentboard http://localhost:5290/mcp`.
Without a task the server offers `list_tasks`, `get_task` and `create_task`, so you can ask "what's on the board?"
or create a task from a chat.

### Plan changes

Before a plan or re-plan run replaces the plan, the board keeps a copy in `plan-history/`. The **Plan changes**
tab compares any earlier version with the current plan, or two earlier versions. Added lines are green, removed
lines are red, and the reason (your rejection comment) is shown on top. Ticked checkboxes don't count as changes.

### Run queue

Plan and Run go through a queue. When a slot is free the run starts at once. Otherwise the card shows
"Plan queued" or "Implement queued", and the task page says why it waits.

- **Agents at once** in Settings sets the limit. 3 by default.
- **Waits for** on the Details tab lists tasks that must be done first, such as a schema change before the endpoint
  that uses it. The board refuses a circle.
- With worktrees off, two tasks on the same repository never run together.
- When a run ends blocked or with a question while other runs wait, the queue pauses. It goes on by itself once
  that task moves on. The board header shows the queue, with a button to pause or resume it.
- **Take out of the queue** on the task page cancels a queued run.
- **Cost budget per task** and **Cost budget per day** in Settings stop the queue at a limit, in US dollars. When a
  task's runs reach its budget, the queue starts no more runs for that task. When the runs that started today reach
  the daily budget, the queue starts nothing until midnight. Running agents finish. The timeline says when a run
  crossed a budget, and the board header shows what today cost. Empty means no limit.
- **Start at 22:00** on the board holds Plan and Run requests until that time, then starts them. Click it before you
  leave, queue the tasks, and they run overnight. Set the time in Settings > Start the queue at. Resume starts them
  at once.

### Worktrees

By default the agent works in the repository folder you named on the task, on the task branch. The first implement
run creates and checks out that branch. Uncommitted changes stay in the folder, and the agent keeps them out of its
commits. Two tasks on the same repository never run together.

With **A git worktree per task** on, the first implement run adds a worktree for each repository of the task, on the
task branch. The agent works there, not in your folder, so you can keep coding in your folder and two tasks on the
same repository can run at once.

- The worktrees live next to the tasks root, in `AgentTasks-worktrees/<task>/<repo>`. Set another folder in
  Settings.
- The prompt, the file list and the diffs all use the worktree.
- Close removes each clean worktree. The branch stays, so you can push it or open a pull request from it.

The agent never puts the step IDs from `tasks.md` (T001) or the board's task ID in commit messages, branch names or
pull request text. It names the ticket key, such as `[ABC-123]`, when the repository's commits do.

## The task page

- **Steps.** Edit text and estimates inline, set a status, move steps up and down or drag them by the grip,
  delete them, or run one step on its own with the play button.
- **Documents.** Read `scope.md`, `spec.md`, `plan.md`, `tasks.md`, `notes.md`, `review.md` and `summary.md`
  rendered with **Preview**, or switch to **Edit** (or double-click the preview). The board remembers your choice. Ctrl+S saves.
  `summary.md` shows its word count against the limit from Settings. If the agent changed
  a file while you edited it, saving fails and you reload.
- **Runs.** Every run with its output, the exact prompt sent, the command, the exit code, the cost and the tool events.
  The output streams while the run is active. Turn off **Readable** to see the raw stream-json.
- **Timeline.** Every status change, run, step change and note in order. Add notes here.
- **Files.** One tree per repository the task touches, with folders you can fold, line counts per file, folder and
  repository, and a diff per file. Repositories with no changes yet still show.
- **Memory.** `memory.md` for this task.
- **Details.** Title, tags, skills and subagents, external links, repositories, executor, auto-approve, and time you
  log by hand. The skill and subagent boxes suggest what Claude Code finds in `~/.claude` and in `.claude` of the
  task's repositories. You can type any other name, such as a plugin skill. Every run of the task tells the agent to
  use them.

### When the agent asks a question

The agent runs without a person to approve prompts, so it can't stop and wait in the middle of a run. When it
needs a decision or a fact that only you can give, it writes the question to `question.md`, marks the step
`- [?]` and ends the run. Then:

- The task moves to **Awaiting input** and the card shows **Needs your answer**.
- The task page shows the question and a box for your answer.
- **Send answer and continue** writes the question and your answer to `notes.md` and starts the next run.
  With Claude Code and Cursor the run resumes the session that asked (`--resume`), so the agent keeps everything it
  already read. Other executors start a new run with the question and the answer on top of the prompt.
- **Run** stays off while a question is open.

### Continue on your phone

While a plan or implementation run is in progress with Claude Code, **Continue on phone** moves the session to
[Claude Code Remote Control](https://code.claude.com/docs/en/remote-control). The board stops the headless run and
starts `claude --resume <session> --remote-control "<key> <title>"`, so the agent keeps its context, repositories,
tool lists and the board's MCP tools.

On your phone:

1. Open the Claude app (iOS or Android) and tap **Code**. The session shows under the task's key and title, with a
   computer icon and a green dot while it runs.
2. Follow the agent's messages and tool calls as they happen. Answer its questions and approve permission prompts
   there. Turn on push notifications in Claude Code (`/config`) to hear when it needs you.
3. When the work is done, send `/exit`. The run ends and the board moves the task as for any run: plan to
   **Planned**, implementation to **Review** or **Blocked**.

The run shows a phone icon on the card. The Runs tab shows the session transcript, and **Stop** still works.

Before you use it:

- Sign in to Claude Code with your claude.ai account (Pro, Max, Team or Enterprise). Remote Control doesn't work
  with `ANTHROPIC_API_KEY`.
- Run `claude --remote-control` once in a terminal and accept its confirmation. Trust the tasks root folder once
  too, so the task folders don't ask.
- On Windows the session opens in a console window. Leave it open: closing it ends the run. On Linux and macOS it
  runs under `script` without a window.

### External links

Add links to a task when you create it or on the Details tab: a design, a doc, a support case. They show under the
title and go into `scope.md`, so the agent sees them.

When a run ends, the board also adds links by itself. It looks for addresses in the run output, `summary.md` and
`notes.md` that match the **link rules** in Settings. By default these are pull requests on GitHub, GitLab and Azure
DevOps. These links carry a spark icon. The timeline records each one.

Only the pull requests the agent opened show under the title and on the card: the ones in the output of
`gh pr create`, `glab mr create`, `az repos pr create` or a `create_pull_request` tool. Pull requests it only read
during an investigation stay on the Details tab, marked **read by the agent**. The pin button there moves a link to
the top, or back.

### Attachments

Attach screenshots, logs or specs when you create a task: pick files with **Attach files** or drop them on the box.
Add more later on the Details tab. Each upload takes up to 30 MB.

The board saves them in `attachments/` in the task folder. A name that is taken gets a number, such as
`log-2.txt`, so nothing is overwritten. Every prompt of the task lists the files with their full paths, so the agent
reads them. A custom template can place the list itself with `{{task.attachments}}`.

Click a file to open it. Images, PDFs and text open in the browser. Other files download.

**Folder** opens the task folder in Explorer. **Terminal** opens a terminal there, so you can run `claude` yourself.
**VS Code** opens one window with the task folder and the worktree of each repository. It writes
`workspace.code-workspace` in the task folder for that. Set another editor command in Settings > Paths, such as `cursor`.

## Memory

When you close a task, the agent writes `memory.md`: what worked, what failed, decisions, gotchas and snippets. The
front matter lists tags, repositories and Jira components.

When you plan a new task, the plan dialog lists memories from tasks that share a tag, repository, component or Jira
project. Tick the ones you want, and they go into the prompt. Search all memories on the **Memory** page.

## Knowledge

The Knowledge page turns what tasks learned into something you can browse and reuse.

- **Tasks and files** is a graph the board builds by itself: tasks, their repositories, tags, Jira components and
  the files their runs edited. Two tasks that edited the same file share the file node, so you see where work
  overlaps. Click a node to see its links, and **Open task** for a task node.
- **Build graph** copies every `memory.md` to `memories/` in the knowledge folder and runs
  [graphify](https://github.com/safishamsi/graphify) on it through Claude Code. Install it once with
  `pip install graphifyy` and `graphify install`. The page then shows **Graphify concepts**, the
  **Graph report** (`GRAPH_REPORT.md`) and a link to graphify's own `graph.html`.
- **Compact memories** has an agent merge every memory into `KNOWLEDGE.md`: grouped by repository and topic, without
  duplicates, each bullet with the tasks it comes from. Every plan run gets it under the learnings from earlier
  tasks, up to the word limit in Settings.
- **Suggest skills** has an agent read every memory and what happened during the tasks: the questions it asked,
  your answers, the commands it asked to run, warnings and rejected plans. For a lesson that shows up in at least
  three tasks, it drafts a skill. The **Skills** tab lists the suggestions with their source tasks. Click
  **Create skill** to edit the draft and save it, or **Dismiss** to never see that name again. A suggestion can
  update a board skill instead of adding a new one.
- **Board skills** live in `skills/<name>/SKILL.md` in the board's home folder, not in `~/.claude` or a repository.
  Every Claude Code run gets them as `agentboard:<name>`, through the same plugin as the agent library. Edit or
  delete them on the Skills tab. On the **Memory** page, **Promote to skill** starts a skill from one task's memory.
- The knowledge folder sits next to the tasks root (`<tasks root>-knowledge`). Change it, the executor and the word
  limit in Settings > Paths.

## The task folder

```
SFM-1234-flight-plan-polling/
  task.json          status, repositories, tags, executor, Jira data
  scope.md           the ticket plus your notes
  spec.md            spec-kit spec
  plan.md            the plan the agent wrote
  tasks.md           the steps as checkboxes
  notes.md           notes with times
  memory.md          learnings, written at Close
  summary.md         what the task delivered, per repository, written at Close
  question.md        the agent's open question, removed when you answer
  attachments/       files you attached to the task
  timeline.jsonl     the timeline, one JSON line per entry
  .steps.json        the step states the board last saw, to find changes made while it was off
  .claude/settings.json   hooks that report tool use to the board
  runs/
    2026-11-07T10-15-00.json         the run record
    2026-11-07T10-15-00.prompt.md    the exact prompt
    2026-11-07T10-15-00.cmd          the launcher script (.sh on Linux and macOS)
    2026-11-07T10-15-00.log          the output
    2026-11-07T10-15-00.exit         the exit code
    2026-11-07T10-15-00.events.jsonl tool events
```

Everything is plain text. With the board off, open the folder and run `claude` there: the files still work.

Step checkboxes in `tasks.md`:

| Marker | Status |
| --- | --- |
| `- [ ]` | To do |
| `- [~]` | In progress |
| `- [x]` | Done |
| `- [-]` | Skipped |
| `- [!]` | Failed |
| `- [?]` | Waiting for your answer |

Add `(est: 2h)` to a step for an estimate. Add `(after: T002)` when a step needs an earlier step that is not
right before it. The steps list then shows "waits for T002" until that step is done, and "ready" after. The
board refuses to run such a step alone before its dependencies, but runs them together.

### History

The board keeps the tasks root in a local git repository and commits each changed task folder every 30 seconds.
The commit message is the task ID and its latest timeline entry, such as `my-task: T003 Fix it: in progress → done`.
Use it to see how a plan, its steps, the notes or the findings changed:

- In VS Code, open a file of the task and look at **Timeline**, or use **Source Control** on the tasks root.
- On the command line, run `git log -p -- <task id>` in the tasks root.

Run logs, run scripts, `scratch/`, `.claude/` and `workspace.code-workspace` stay out of the history. The
`.gitignore` in the tasks root lists them, and you can add your own rules to it.

The board creates the repository only when the tasks root isn't inside one. If you already keep the tasks root in
your own repository, the board leaves it alone and writes a warning to its log. To turn the history off, clear
**Keep a git history of the tasks folder** in Settings. The repository stays; delete the `.git` folder to remove it.

## Settings

Every setting is saved to `settings.json` in the home folder (`%LOCALAPPDATA%\AgentBoard` on Windows) and applies
without a restart.

- **Naming rules.** Placeholders `{key}`, `{slug}`, `{title}`, `{type}` and `{date}`. Empty values drop out with their
  dash, so a manual task gets `flight-plan-polling`. Add `:lower` or `:upper` to change the case:
  `task/{key:lower}/{slug}` gives `task/abc-123/fix-login`.
- **Templates.** The plan, re-plan, implement and close prompts, and the files a new task starts with. Placeholders
  such as `{{scope}}`, `{{steps}}`, `{{memory}}` and `{{jira.description}}` are listed on the page. A changed template
  is used by the next run. **Reset to default** brings back the built-in text.
- **Executors.** Claude Code, and Cursor CLI (off until you turn it on). Add any other agent CLI. See
  [executors.md](executors.md) for Cursor, Codex, Gemini CLI, Aider and how to write an adapter.
- **Board.** Add, rename, reorder and remove columns, set WIP limits, and choose the column for each lifecycle step.
- **Global rules.** Rules for every task, in Markdown, such as "never push to main". They go on top of every prompt
  the agent gets. Put `{{systemRules}}` in a template to place them somewhere else.
- **Language of the task files.** The agent writes `plan.md`, `tasks.md`, `notes.md` and `summary.md` in this
  language, whatever the language of the ticket. It is English by default. Type any language, such as Romanian, to
  change it. If you clear the field, the board saves English. In the
  repositories the agent keeps the language the code and commits already use. Templates can place `{{language}}`.
- **Task types.** Feature, Bug, Chore and Investigation come built in, and you can add your own. Each type has
  instructions for the agent, a step skeleton, the Jira issue types that pick it, and its expected parts.
  - All built-in types are **lightweight**: no `spec.md`, and `plan.md` is a one-page delta plan. Turn the switch off
    on a type or a single task when you want the full spec.
  - **Expected parts** are files or sections, such as `plan.md` or `scope.md#Acceptance criteria`. While one is empty,
    the card shows a warning count and the task page lists them under "Still empty". It is a reminder and never
    blocks a run. Plan files count only after a plan run, and `summary.md` only after Close. An investigation expects
    only a description, because its answer goes to `notes.md`. A lightweight task shows no such warnings, because
    its documents are optional.
- **Link rules.** Which addresses in the agent's output become links of the task. Each rule has a regular
  expression and a title with named groups, such as `PR {repo}#{number}`.
- **Useful links.** Pages you open often, such as Nexus repositories or Jenkins jobs. The link button in the header
  lists them, and every prompt lists them with their note, so the agent knows where to look. Put `{{services}}` in a
  template to place that section somewhere else.
- **Connections.** Services the agent may call, such as Nexus or Jenkins. Each has an ID, an address, a user and a
  password or token. Every run gets them as environment variables named after the ID: `NEXUS_URL`, `NEXUS_USER` and
  `NEXUS_PASSWORD` for the ID `nexus`. The agent decides when it needs them.
  - The password comes from an environment variable you name, such as `NEXUS_PASSWORD`, or you save it in Settings,
    where it is stored encrypted like the Jira token. A saved password wins. On Windows the board also reads your
    user and system variables, so a variable you set after the board started works without a restart.
  - Write `%NAME%` in the address or user to read them from an environment variable, such as `%NEXUS_USER%`.
  - The values go to the agent's process only. They are not in the run script, the prompt or `settings.json`. The
    prompt names the variables and tells the agent never to print them. The run log on the page shows `***` in
    place of the user and password, but the log file on disk keeps what the agent printed.
  - For Jenkins, use an API token, not your account password.
- **Agent library.** Paste the link of a git repository with Claude Code subagents, such as
  `https://github.com/codewithmukesh/dotnet-claude-kit/tree/main/agents`, and click Import. The board clones it into
  the `agent-sources` folder of its home folder and lists every agent. Tick the agents that every Plan, Implement and
  Review run calls, then save. **Check for updates** lists the agents that changed upstream, and **Update** takes them.
  - When two sources have an agent with the same name, both get the source as a prefix, such as
    `dotnet-claude-kit-code-reviewer`.
  - A run gets its agents as a plugin in its run folder, so Claude Code lists them as `agentboard:<name>`. Nothing
    goes into `~/.claude` or your repositories, and hooks, MCP servers and commands from a source never load.
  - On a task, the Details tab lists the library agents too. One you pick there joins every plan, implement and
    review run of that task.
- **Summary word limit.** The most words the agent may write in `summary.md`. 2000 by default. The timeline warns
  when a summary is longer.
- **Look.** The palette button in the header sets the theme (system, light, dark), the size (compact, default,
  large, extra large) and the accent color, the same way as in HourLog. The choice applies at once and is kept in
  this browser. The Settings page has the same controls.
- **Menu click sound.** A short click plays when you pick a header link or a dropdown menu item. Turn it off
  in Settings, under the theme controls. The choice is kept in this browser.
- **Import and export.** One JSON file with all settings and templates. The Jira token and the connection passwords
  stay out of it.

## Where the tool's own files are

- Settings and templates: `%LOCALAPPDATA%\AgentBoard` (Windows), `~/.local/share/agentboard` (Linux and macOS).
- Its log: the `logs` folder there, one file per hour, kept for 48 hours.
- The encrypted Jira token: the `secrets` folder there. Only your Windows account can decrypt it.
- Tasks: the tasks root from Settings > Paths, `~/AgentTasks` by default.
- Knowledge: `<tasks root>-knowledge`, unless you set another folder.
- The agent library: the `agent-sources` folder in the home folder, one clone per source and `sources.json`.
- The program: `C:\Program Files\AgentBoard`. Only `appsettings.json` there holds anything you might change: the port
  and the home folder.

Setup never writes to your folders above. A new version replaces only the program and keeps everything else.
It keeps a changed `appsettings.json` too. An uninstall keeps your files as well. Delete the folders by hand if you
want them gone.

A new version reads an older `settings.json` and fills in new settings with their defaults. Templates you changed
stay as you wrote them. Templates you never changed follow the new built-in text. Settings > Templates marks the ones
you changed with a dot, and **Reset to default** brings back the new text.

To use another folder, set `AgentBoard__Home`. To use another port, set `AgentBoard__Port`. Environment variables
don't need admin rights. Editing `appsettings.json` in Program Files does.
