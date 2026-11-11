# Executors

An executor is the agent CLI that does a run: plan, implement, review, close, intake and the knowledge runs.
Agent Task Board ships with two. You can add any other CLI in Settings, or write an adapter in C#.

## How a run starts

For every run the board:

1. Writes the prompt to `runs/<run>.prompt.md` in the task folder.
2. Builds the command from the executor settings and the adapter.
3. Writes `runs/<run>.sh` (or `.cmd` on Windows) and starts it detached. The script changes to the task folder,
   sends the output to `runs/<run>.log` and writes the exit code to `runs/<run>.exit`.
4. Reads the log when the run ends: cost, tokens, edited files, tool calls and the session ID.

The run keeps going if you close the board, and you can run the script again by hand.

The script sets these variables for the agent:

| Variable | Value |
|---|---|
| `AGENTBOARD_TASK_ID` | The task ID |
| `AGENTBOARD_RUN_ID` | The run ID |
| `AGENTBOARD_TASK_FOLDER` | The task folder |
| `AGENTBOARD_HOOK_URL` | Where Claude Code hooks post their events (Claude Code only) |

## Built-in executors

### Claude Code

- Command `claude`, arguments `-p --output-format stream-json --verbose`, prompt on standard input.
- The adapter adds `--model`, `--permission-mode`, `--allowedTools`, `--disallowedTools`, one `--add-dir` per
  repository, `--resume` when it answers a question, and the board's MCP server with `--mcp-config`.
- With **Approvals on the board**, a command outside the allowed tools waits on the task page for Allow or Deny.

### Cursor CLI

- Turn it on in Settings > Executors. You need `cursor-agent` on PATH and a Cursor login (`cursor-agent login`, or
  `CURSOR_API_KEY` in the executor's environment).
- Command `cursor-agent`, arguments `-p --output-format stream-json`, prompt as the last argument.
- The adapter adds `--model`, `--resume` when it answers a question, and `--force` when the permission mode is
  `force`. Without `--force`, Cursor asks before shell commands, and a headless run has nobody to answer.
- The board reads Cursor's stream-json: edited files, tool calls, the session ID and pull requests it opened.
  Cursor doesn't report cost, so the cost column stays empty.
- Cursor has no `--add-dir`. The prompt lists the repository folders, and the agent works on them by path.
- The board's MCP tools and approvals are Claude Code only for now. A Cursor run marks steps in `tasks.md` itself.

## Add another CLI

Settings > Executors > **Add executor** creates an entry with the adapter **Any command**. Fill in:

| Field | What it does |
|---|---|
| Command | The program, or a full path. |
| Arguments | One per line. `{{promptFile}}`, `{{taskFolder}}`, `{{model}}` and `{{permissionMode}}` are replaced. |
| Prompt via | **Standard input**: the prompt file goes to stdin. **File path**: the program gets the path through `{{promptFile}}`. **Last argument**: the prompt text is the last argument. |
| Environment | `NAME=value` lines, such as an API key the CLI reads. |

If the CLI prints Claude Code style stream-json (`assistant`, `tool_use`, `result` events), the board reads cost,
tokens and edited files from it. Any other output still shows in the run log, and the run counts as done when the
exit code is 0.

Examples to start from. Check `--help` of your version, since flags change between releases.

| CLI | Command | Arguments | Prompt via |
|---|---|---|---|
| OpenAI Codex | `codex` | `exec`, `--json`, `--full-auto`, `-` | Standard input |
| Gemini CLI | `gemini` | `--yolo`, `--output-format`, `stream-json` | Standard input |
| Aider | `aider` | `--yes-always`, `--message-file`, `{{promptFile}}` | File path |

Tips:

- A run has no person at the keyboard. Use the CLI's non-interactive mode and let it run commands without asking,
  or the run hangs until the board stops it.
- Point the agent at the repositories in the prompt, since the run starts in the task folder.
- Set **Language of the task files** and **Global rules** in Settings. They go into every prompt, whatever the
  executor.

## Write an adapter

When a CLI needs more than arguments, such as its own output format or flags built from settings, write an adapter:

1. Add a value to `ExecutorKind` in `src/AgentBoard.Core/Settings/ExecutorSettings.cs`.
2. Subclass `ShellAdapter` in `src/AgentBoard.Core/Executors/`. Override `Kind`, and `ExtraArguments` for flags.
   Override `ReadOutput` and `Pretty` when the output is not stream-json. `CursorAdapter` is a short example.
3. Register it in `src/AgentBoard.Service/Program.cs` with `AddSingleton<IExecutorAdapter, YourAdapter>()`, and in
   the test `Harness`.
4. Add the kind to `ExecutorKind` in `src/AgentBoard.Web/src/lib/types.ts` and to the Adapter list in
   `SettingsPage.tsx`.
5. Add tests like `CursorTests`: the command it builds, and the output it reads from a recorded log.
