# Privacy

Agent Task Board runs on your PC and keeps your data there. It has no account, no telemetry and no analytics.

## What the board stores

- Your settings and templates, in `%LOCALAPPDATA%\AgentBoard\settings.json`.
- Your Jira API token and the passwords of your connections, encrypted with Windows DPAPI, in
  `%LOCALAPPDATA%\AgentBoard\secrets`. Only your Windows account can decrypt them.
- Its own log, one file per hour, kept 48 hours, in `%LOCALAPPDATA%\AgentBoard\logs`.
- Your tasks, in the tasks root (`AgentTasks` in your user folder unless you change it): the task documents, the
  run logs, notes, attachments and memories, and a local git history of them.
- Worktrees and the knowledge base, in folders next to the tasks root.

## What goes online

The board itself makes these network calls. Each one happens only when you use the feature.

- **Update check.** Every 6 hours, the board asks the GitHub API for the latest release of
  [ilievici/AgentTaskBoard](https://github.com/ilievici/AgentTaskBoard/releases). When a newer version exists, it
  downloads the installer from GitHub. The request carries the board version in its `User-Agent` header and none of
  your data. Turn it off in **Settings > Updates**.
- **Jira.** When you fetch a ticket, the board calls the Jira site you set in **Settings > Jira**, with your account
  email and API token.
- **Agent library.** When you import a source of agents, the board clones that git repository.
- **Pull requests.** When you open a pull request, the board runs `git push` and the GitHub CLI with your own sign-in.

GitHub and Atlassian see your IP address, as with any website.

## What the agents send

The board starts the agent CLIs you set up, such as Claude Code or Cursor CLI. They run as you, with your sign-in.
They send your prompts, the task documents and the code they read to their providers, under their own terms:

- Claude Code: [Anthropic privacy policy](https://www.anthropic.com/legal/privacy).
- Cursor: [Cursor privacy policy](https://cursor.com/privacy).

**Continue on phone** starts Claude Code Remote Control, which shows the session in the Claude app through your
claude.ai account.

## Other PCs

The board listens only on your own PC (`localhost`). Other PCs on your network can't open it, and web pages on other
sites can't call its API.
