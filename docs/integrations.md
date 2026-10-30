# Agents, terminal commands and MCP

[Documentation](README.md) · [Русский](ru/integrations.md) · [Installation](installation.md) · [Troubleshooting](troubleshooting.md)

There are two agent roles. A **provider** answers ChakChak's prepared model tasks. A **driving agent** uses ChakChak's MCP tools to take the person's seat and make decisions. `chakchak mcp` can use the driving session as the provider too, or delegate model work to a named local CLI.

## Providers

The table below lists built-in integrations. It does not promise that every version of every external CLI has been tested. Install and authenticate a provider independently; ChakChak invokes the installed program rather than managing credentials.

| Provider | Program | Call transport | Default configuration folder under your home | Skill loader |
|---|---|---|---|---|
| Claude Code | `claude` | CLI/stream JSON | `.claude` | Yes |
| Codex | `codex` | CLI; app-server for write sessions/catalog | `.codex` | Yes |
| Gemini CLI | `gemini` | ACP | `.gemini` | Yes |
| Qwen Code | `qwen` | CLI; stream JSON write grants | `.qwen` | Yes |
| OpenCode | `opencode` | ACP | `.config/opencode` | Yes |
| Crush | `crush` | CLI | `.config/crush` | Yes |
| Cursor CLI | `cursor-agent` | ACP | `.cursor` | Yes |
| GitHub Copilot CLI | `copilot` | ACP | `.copilot` | Yes |
| Kimi CLI | `kimi` | ACP | `.kimi-code` | Yes |
| Qoder CLI | `qodercli` | ACP | `.qoder` | Yes |
| Kiro CLI | `kiro-cli` | CLI | `.kiro` | Yes |
| Cline CLI | `cline` | ACP | `.cline` | Yes |
| Kilo Code CLI | `kilo` | ACP | `.config/kilo` | No declared folder |
| iFlow CLI | `iflow` | CLI | `.iflow` | Yes |
| Factory Droid | `droid` | CLI | `.factory` | Yes |
| Auggie | `auggie` | ACP | `.augment` | Yes |
| CodeBuddy | `codebuddy` | ACP | `.codebuddy` | Yes |
| Amp | `amp` | CLI | `.config/amp` | Yes |
| Goose | `goose` | ACP | `.config/goose` | No declared folder |
| Blackbox CLI | `blackbox` | CLI | Not declared | No declared folder |
| Grok CLI | `grok` | ACP | `.grok` | No declared folder |

ACP means Agent Client Protocol over a local child process. A provider's ACP transport is distinct from the MCP interface that ChakChak exposes to clients.

Configuration-folder precedence is the provider's environment override, then its `homeUnderConfig` under `XDG_CONFIG_HOME`, then the default above. Declared environment overrides are `CLAUDE_CONFIG_DIR`, `CODEX_HOME`, `COPILOT_HOME` and `KIMI_CODE_HOME`. `homeUnderConfig` is declared for Claude Code, OpenCode, Crush, Kilo, Amp and Goose. A loader goes at `<resolved configuration folder>/skills/chakchak/SKILL.md` where the row declares `skills`.

### Program discovery and selection

Each launch reads the login-shell environment once with `$SHELL -lic`, bounded to ten seconds. It fills missing environment variables and appends login `PATH` directories to the launch's path. Logs record added variable names, not their values. This lets a desktop launch find tools configured in your shell.

The app shows discovered agent copies and their paths; choose the copy you intend to run. The choice persists by absolute executable path. When opening a saved run in the app, that saved application choice wins; the run's former agent is a fallback if there is no saved choice or its program is gone. The talk continues from its saved outcome if the provider changes. The CLI accepts an agent program or path after `cli`, `web` or `mcp`. At process launch, symlinks are resolved so helpers installed beside the real executable can be found; the UI retains the familiar path. The journal records both given and resolved paths.

ChakChak does not guess updated flags by running arbitrary `--help` or login probes. Model lists come from the catalog protocol declared in the row: Claude stream JSON, Codex app-server, Qwen-family stream JSON or ACP. A provider with no catalog takes a typed model name. Catalog failure is shown separately from program discovery.

### Models, effort and parallelism

`--model <name>` explicitly selects a model for a CLI/web/MCP-provider launch. Otherwise each call uses the current process's in-memory pick for that executable, or the provider's default if there is no pick. Model picks are not persisted in `settings.json` or restored from the old run's model attribution; after restarting, choose the model again if you require a particular one. The saved agent choice and a run's recorded model are separate things. No reasoning-effort flag is added by ChakChak.

The concurrency ceiling is eight calls unless `--parallel <1-8>` sets a smaller maximum. Actual parallelism starts from the pace learned for that agent, or one call, increases after successful calls, and halves on overload. The learned pace and call duration estimates persist in settings. A higher ceiling does not immediately start eight calls. Each call records the model reported by the provider; spec traces and the report retain that attribution.

## Permissions and external services

Every agent process runs in the subject repository. Its task is delivered on stdin, or through `{task}` for a row that requires a command argument. Migration calls (talk, cut, spec writing and reading) use **read mode**. Change and task chats use **write mode** and the provider's broad permission mode.

| Provider | Migration/read mode | Change/task write mode |
|---|---|---|
| Claude Code | `--safe-mode --permission-mode dontAsk --disallowedTools Write,Edit,MultiEdit,NotebookEdit,Bash` | `--dangerously-skip-permissions` |
| Codex | `--sandbox read-only --ignore-user-config --ignore-rules` | Full-rights app-server session; row also declares `--dangerously-bypass-approvals-and-sandbox` for write CLI calls |
| Qwen Code | `--approval-mode plan` | `--approval-mode auto-edit`, preapproved built-in/MCP tools and stream permission grants |
| ACP providers | Read-only tool permission requests allowed; other requests rejected | Allow-once permission option selected |
| Other CLI rows | Row's `read` flags, if any | Row's `write` flags or base invocation |

Read protection depends on the provider's declared flags/protocol. A custom or shipped CLI row with no read-specific restriction relies on the task's read-only instruction; ChakChak does not impose an independent filesystem sandbox. Some rows put permissive flags in their base invocation. Inspect a row before choosing it for a sensitive repository.

Write calls load the person's provider configuration, project instructions, hooks, skills and MCP servers. They can change files, run tools and affect external services with the rights the provider has. The task prompt asks the agent to commit a completed task with `Task: <change> <number>`; ChakChak itself reads those commits. Stopping a write call leaves the agent's edits in the tree. Git cannot undo external service actions, leaked secrets, changes outside the repository or destruction of `.git`.

SDD write calls run unattended. Provider questions are answered by the integration rather than pausing for the person. Codex MCP questions may be answered by a short model call; questions requesting a password, key, token or secret are declined. Permission notices appear in the chat. Qwen grant handling preapproves tools and answers permission-control requests; a deny rule in the provider's own settings can still refuse a tool. The table's grant metadata explicitly marks parts of Qwen support as unverified; it should not be read as a universal compatibility guarantee.

Code and task material can be sent to the selected model provider. The loopback web interface does not make the model local. Agent login, billing, data retention and connected services remain those of the agent/provider you chose.

## Custom providers and overrides

For an unlisted executable, the terminal can ask how it answers one task; the app can add a program and example call. Put `{task}` where the task belongs, or omit it to send the task on stdin. These custom calls are stored in settings. Use a structured provider row when you need explicit read/write modes, a non-stdout answer or a catalog.

`~/.chakchak/agents.json` accepts an array of rows or an object with a `known` array. A row with an existing `program` **replaces the whole shipped row**, not individual fields; other rows append. Unknown JSON fields and invalid call definitions are refused. Keep one override per program for clarity, and back up an existing override before changing it.

Example for a hypothetical CLI that reads stdin and prints its answer:

```json
{
  "known": [
    {
      "program": "my-agent",
      "title": "My Agent",
      "home": ".my-agent",
      "skills": "skills",
      "run": ["ask", "--model", "{model}"],
      "read": ["--read-only"],
      "write": ["--allow-writes"],
      "answer": "stdout"
    }
  ]
}
```

The sample flags belong to the hypothetical program; replace them with its actual noninteractive interface. The supported provider-row fields are:

| Field | Meaning |
|---|---|
| `program`, `title` | Executable name and display name |
| `run` | Argument array; supports `{model}`, `{task}`, `{answer}` |
| `read`, `write` | Extra arguments for the corresponding mode |
| `answer` | `stdout`, `file` (requires `{answer}`), or `json:<field>` |
| `acp` | ACP startup arguments, instead of `run`/answer/read/write |
| `resume` | JSON-stream conversation continuation, exactly one `{thread}` |
| `home`, `homeEnv`, `homeUnderConfig`, `skills`, `installs` | Discovery/registration locations |
| `catalog` | Model-list protocol, arguments and the supporting `why` reference |
| `mcp` | Provider's own server-list command and output format (`lines` or `json:<field path>`) |
| `appserver`, `grant` | Specialized session/permission behavior; use existing rows and implementation as the contract |

An absent model drops `{model}` and its preceding argument. Do not put credentials into this file or argument arrays; use the provider's own authentication mechanism. Restart ChakChak to reload provider overrides.

## Commands and flags

Run repository commands from the intended repository root. A bare `chakchak` is a terminal run; flags require the explicit `cli`, `web` or `mcp` word.

| Command | Effect |
|---|---|
| `chakchak cli [program-or-path] [--model name] [--parallel 1-8] [--lang en\|ru] [--strict]` | Start/resume terminal migration or sync |
| `chakchak web [program-or-path] [--model name] [--parallel 1-8]` | Serve this repository in the browser interface |
| `chakchak app` | Open the repository chooser/application |
| `chakchak mcp [program-or-path] [--model name] [--parallel 1-8]` | Serve MCP on stdin/stdout |
| `chakchak specs [capability]` | Read the workspace or one complete spec; no model call |
| `chakchak stage <map\|talk\|picture\|cut\|specs\|report> [capability\|--all]` | Repeat a stage; the last argument is valid only for `specs` |
| `chakchak fix <capability-or-number> ["correction"]` | Rewrite a saved spec; ask for correction if omitted |
| `chakchak rule "words"` | Add a rule for future spec writes |
| `chakchak accept <n…>\|all`, `chakchak reject <n…>\|all` | Mark post-run cut proposals; no rewrite yet |
| `chakchak apply [<n…>\|all]`, `chakchak dismiss <n>` | Apply accepted proposals together, or dismiss one |
| `chakchak new-conversation [--confirm]` | Archive the current checkpoint and begin a fresh provider conversation with saved context; asks for confirmation otherwise |
| `chakchak install`, `doctor`, `uninstall` | Install, inspect or remove provider skill loaders |
| `chakchak skill`, `help`, `version` | Print instructions, usage or build identity without creating a journal |

`--lang` is CLI-only and sets discussion/spec/report language before preparation is locked; it does not change terminal UI language. Web uses Settings, and MCP follows the screen language. `--strict` is CLI-only: ordinary completion exits 0 even if some specs remain open/failed; strict completion returns 3 when any spec is unwritten/open/failed/pending. A stopped run returns 2; startup/refusal or failed/blocked stages return 1. Read the final summary as well as the status code.

Inside the terminal, `help` lists actions at the current question, `show` repeats its text and `auto` toggles acceptance of defaults. Cut review also supports manual `move`; spec writing accepts `stop`, `end`, `fix` and `rule`. Use the displayed choices, since available commands depend on the current stage. `NO_COLOR=1` disables color. Piped stdout or `TERM=dumb` disables escape sequences and uses self-contained event lines.

## Install the agent skill

```sh
chakchak install
chakchak doctor
```

Install lists agents, exact destination paths and loader text, then asks for numbers, `all` or `none`. It creates only the selected loaders. Start a new agent session for the registration to take effect.

The loader runs the installed executable by **absolute path** with `skill` at each skill invocation and reads the full instruction from that binary. Updating in place therefore keeps the loader useful. Moving the binary or app requires another install. A conversation that already loaded the instruction is not guaranteed to refresh midway.

Install can replace a recognized ChakChak skill with matching complete frontmatter, including an edited body, without making a backup. Foreign/unrecognized files are preserved. `uninstall` removes only an exact unchanged current loader. `doctor` reports registration and does not change provider skill files; it is a working command and can create ChakChak's own diagnostic/settings state.

**Install does not configure MCP.** The skill and the client's MCP connection are separate. `chakchak skill` prints the full current instruction without writing it; ordinary working launches also keep a copy in `~/.chakchak/SKILL.md`.

## Connect MCP

Configure a **stdio command server** in your MCP client with these values:

| Setting | Value |
|---|---|
| Executable | Absolute path to `chakchak`, for example `/Applications/ChakChak.app/Contents/MacOS/chakchak` |
| Arguments, session mode | `mcp` |
| Arguments, provider mode | `mcp`, followed by an installed provider such as `codex` |
| Working directory | Absolute path to the subject repository |
| Environment | The environment needed by your installed provider; retain your normal home/config variables |

ChakChak has no `--cwd` argument and no HTTP MCP endpoint. Set the client's working directory or use a wrapper that changes directory and then `exec`s the binary. Client config syntax and file locations belong to that client; ChakChak does not maintain or rewrite them. Do not point an HTTP MCP client at the browser URL. Stdout is reserved for protocol messages; diagnostics and the observer-page address go to stderr.

Use the handoff text from `chakchak help`: save/read the skill from the absolute executable's `skill` command, connect its `mcp` command over stdio in the repository, and migrate that repository. A client that cannot configure cwd can use an executable shell wrapper such as:

```sh
#!/bin/sh
cd '/absolute/path/to/repository' || exit 1
exec '/absolute/path/to/chakchak' mcp
```

Give the MCP client the wrapper path as its executable. Use a separate connection for each repository. Skill loaders always name the binary, so keep the MCP configuration and skill pointed at the same installation.

### MCP lifecycle and tools

Connecting, listing tools and reading idle status do not claim the repository. `start` takes the repository lease and prepares recovery; a competing owner returns `busy` while the MCP connection remains available for retry. The lease lasts until the session closes. A started session serves an observer page on loopback, with only Stop and End available to the person.

| Tool group | Tools and purpose |
|---|---|
| Session | `start`, `status`; inspect the question, revision, progress and pending commands |
| Migration decisions | `accept`, `say`, `move`, `fix`, `rule`, `proposal`, `stage`, `stop`, `restart_conversation` |
| Session-model queue | `tasks`, `answer`; take and answer prepared model tasks |
| Workspace reading | `specs`; list freshness, active changes, unplaced/new files or one complete spec |
| OpenSpec changes | `schema` lists workflow/configuration; `change` creates a scaffold with optional answers/attachments |
| Prompt/explanation reading | `artifact`, `task`, `examples`; return prompts or code examples without starting an agent |

Read the schemas returned by `tools/list` from the installed binary for exact parameters. In particular, terminal `accept <n>` marks proposals; MCP `accept` answers the current question, while MCP `proposal` handles proposal marks and application.

Read `status` before deciding. Revision-bearing tools refuse a stale revision with the current revision/question-changed response; re-read the question before retrying. `status` with `wait`/`since` waits for a change, task or command completion; `unchanged` permits another wait. `idle` means only a client action can move the run. `fix` and `rule` return before their background model work completes; inspect `status.commands` and continue draining session tasks. `stage` is refused while competing work runs.

With **no named provider**, `start` accepts a `model` attribution for the session, and model work arrives via `tasks`. Answer each task's own requested format in full with `answer`; report quota/login/network/overload as a structured refusal rather than invented prose. After a lost worker/session, `tasks` with `retake: true` reissues taken but unanswered work. `stop` cancels unfinished tasks. With a **named provider**, ChakChak launches that CLI and queue tools report that they are not available in this mode.

`start` resumes an interrupted/failed mid-stage run directly. A fresh migration or finished repository needing sync can present the Start question; follow its words and obtain the person's consent where the skill requires it. In-step specs need no new model work. `restart_conversation` requires the current revision, proof from status and explicit `confirmed: true`; it archives the saved checkpoint before replacing the conversation.

`artifact` and singular `task` return prompt text; they do not execute it, acquire a write hold or validate the caller's later edits. `tasks` is the separate migration queue. A named artifact can be refused because an earlier or unfinished step blocks it. `examples` only reads relevant code. `change` writes a new stock OpenSpec scaffold and supplied answers/attachments; `schema` and `specs` are read-only. The driving agent records its decisions and evidence in `chakchak-decisions.md` at repository root, outside stage-restored `openspec/`.

## Skills and MCP servers used by a change

The application can list the chosen provider's personal and project skills; a project skill of the same name takes precedence, and ChakChak's own loader is excluded. This inventory does not install those skills.

Provider MCP server discovery uses that provider's declared list command, such as `codex mcp list --json` or `claude mcp list`; only server names are retained. Some provider rows mark their list command unverified. A list can take up to 20 seconds because a provider may health-check its servers. Configure/authenticate these servers in the provider, not in ChakChak. A schema can name an MCP server and ask for hints/attachments; inspect `schema` and supply the optional values when creating a change. If your driving agent lacks a server required by an artifact, connect it before executing that prompt.
