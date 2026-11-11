# Agent Task Board

Agent Task Board is a local control panel for work you hand to coding agents such as Claude Code. Paste a Jira ID or
describe a task. The board creates a task folder, the agent writes a plan and a step list, you approve it, and the
board runs the agent and shows each step as it finishes. It runs on your PC and opens in your browser.

## Download

1. Open the [latest release](https://github.com/ilievici/AgentTaskBoard/releases/latest).
2. Under **Assets**, download `AgentBoard-Setup-<version>.exe`.
3. Run it and accept the admin prompt.

The installer isn't signed, so SmartScreen might warn you. Click **More info**, then **Run anyway**.
Then open **Agent Task Board** from the Start menu. The board opens at <http://localhost:5290>.

You need:

- Windows 10 version 1809 or later, 64-bit. The installer brings its own .NET runtime.
- [git](https://git-scm.com/downloads).
- [Claude Code](https://code.claude.com/docs/en/setup), installed and signed in. Run `claude` once in a terminal to
  check. Cursor CLI works too.
- Optional: the [GitHub CLI](https://cli.github.com/) to open pull requests from a task.

The board runs as you, not as admin, because the agent uses your sign-in, your PATH and your git settings.

## Features

- Create a task from a Jira ticket, from a chat, or by hand. Each task is a folder of plain Markdown and JSON.
- Let the agent write a plan and a step list, then approve, edit or reject it.
- Run the agent headless and watch each step, run, edited file and question on the task page.
- Answer the agent's questions and approve the commands it wants to run, from the board.
- Queue runs, set cost budgets, and start the queue at night.
- Work in a git worktree per task, and open a pull request when the work is done.
- Keep a memory of each task and build a knowledge base from all of them.
- See metrics for the last 7, 30 or 90 days.
- Use light, dark or system themes. Read the built-in **Docs** page for every feature.

## Updates

The board checks this page for new versions every 6 hours. When it finds one, it downloads the installer in the
background and shows an **Update** button. Your settings, templates and tasks stay as they are, and agent runs keep
running while the board restarts.

To turn the check off, open **Settings > Updates** and turn off **Check for updates**. To update by hand, download
the new installer and run it.

## Your data

The board keeps its settings in `%LOCALAPPDATA%\AgentBoard` and your tasks in `AgentTasks` in your user folder. You
can change the tasks folder in **Settings > Paths**. Uninstalling the board keeps both.

The board sends none of your data to its author. The agents you run send your prompts and code to their own
providers. See [Privacy](PRIVACY.md) for every network call.

## Documentation

- [User guide](docs/user-guide.md): the board, the task page, the lifecycle and the settings.
- The **Docs** page inside the board has one article per feature, with search.

## License

Agent Task Board is freeware. See the [license](LICENSE.md).

## Support

To report a problem or ask for a feature, open an
[issue](https://github.com/ilievici/AgentTaskBoard/issues). Add your version from **Settings > About**.
