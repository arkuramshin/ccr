# Troubleshooting and recovery

[Documentation](README.md) · [Русский](ru/troubleshooting.md) · [Installation](installation.md) · [Integrations](integrations.md)

Start with the refusal or error shown by ChakChak. It usually names the blocked action and next step. Keep the launch's journal folder before changing configuration or retrying a difficult case; `journal.log` alone does not contain the full evidence.

## Basic checks

```sh
command -v chakchak
chakchak version
chakchak help
```

These identify the installation without starting a migration. From the intended repository, `chakchak specs` inspects the workspace without asking a model. `chakchak doctor` inspects provider skill registration; it does not repair loaders or configure MCP. Working commands, including doctor/specs, can create ChakChak's own journals.

Record the version/commit, OS and architecture, launch command, selected agent executable and model, repository state, exact action and visible result. Do not include API keys, tokens or provider authentication files in a report.

## The app does not start

| Symptom | Check and recovery |
|---|---|
| Wrong architecture or executable format | Use `macos-arm64` for Apple silicon and `macos-amd64` for Intel. Linux archives are separate executables. |
| macOS refuses the app | Verify it came from the intended release and that its archive matches `SHA256SUMS`. A `-unsigned` package is ad-hoc, not notarized; obtain a signed release or build locally. Do not treat deleting quarantine as release verification. |
| Linux reports a missing GLIBC version | Use a distribution compatible with the packaged glibc floor (2.34) or build for your system. The package is not a musl-native build. |
| Startup reports an unreadable home/settings/import path | Keep the named file and follow [saved-state recovery](#saved-state-or-settings-cannot-be-read). Close older processes before an import. |
| Browser does not open | Use the loopback address printed by the process. Check whether another ChakChak owns the application/page or another program owns the port. Do not terminate an unidentified process. |

The normal address is `127.0.0.1:7757`; it is not a remote service address or MCP URL. A macOS Finder launch uses its own window, whereas `chakchak app` from a terminal uses a browser. A stale tab can point to an earlier process; compare its address with the active launch.

## `chakchak` or an agent is not found

For the terminal command, check Settings → **Terminal command**, `command -v chakchak` and your `PATH`. An ordinary file at a desired link location is deliberately preserved. After moving the app, refresh its terminal link and explicitly run `chakchak install` if you use agent loaders.

For an agent, first confirm its own program can start and sign in in your terminal. ChakChak does not install or authenticate it. The app can choose a specific discovered executable copy. Compare that path with `command -v <program>`; the journal records given and resolved executable paths for symlinked installations.

Desktop launches import missing login-shell environment variables using `$SHELL -lic` once, with a ten-second limit. A shell profile that waits for input or takes too long can hide tools from the app. Keep interactive prompts out of startup files, reopen ChakChak, and inspect the environment-import event. Do not paste `env` output into a bug report: it can contain secrets.

On macOS, if the only Git is `/usr/bin/git` and `xcode-select -p` cannot find a developer directory, ChakChak avoids triggering Apple's install dialog and skips that Git. A separately installed Git can still be used. Git-backed listing/source stamps are unavailable when Git cannot run; a non-Git listing uses the working tree.

## The agent fails, hangs or returns an unreadable answer

| Reported kind | Meaning and next step |
|---|---|
| `login` | Provider authentication is missing/expired/refused. Sign in with that provider, then resume. |
| `quota` | Usage was exhausted. Wait for the reset if one is shown or resolve account limits, then resume. |
| `overload` | Provider is temporarily busy. ChakChak retries with growing pauses and reduces its learned pace. |
| `unknown model` | Check the model catalog or type a model supported by that installed provider/account. |
| `network` | Check provider connectivity, proxy and DNS without exposing credentials; resume when restored. |
| Unreadable/invalid answer | Inspect the provider's complete output and current row. Unsupported flags, a CLI banner or a changed JSON shape can break the contract. |
| Program missing | Reinstall/select the executable and relaunch; an absent program cannot be recovered by resuming the same process. |

During spec writing, login, quota, network and repeated identical unexplained refusals pause the wave instead of discarding completed specs. The terminal offers **Enter = resume writing specs** / `q`; the page offers **Resume**. The paused state persists in `run.json`. Relaunching continues remaining work. A missing agent/OpenSpec executable instead fails the affected specs and allows a report naming the problem.

Read calls with streaming output have a ten-minute inactivity limit and all read calls have a one-hour ceiling. Change artifact writes are limited to 20 minutes and task writes to 60 minutes. These are ceilings, not expected durations. A working child can be slow without being stuck; inspect its process stream and journal progress. Failed stages pause autopilot for an explicit retry rather than retrying endlessly.

A failed-call diagnosis includes executable, masked arguments, exit/signal, stream sizes and last output lines. The journal links complete streams captured while the process was running. When an external CLI update changes flags, use a complete override row in `~/.chakchak/agents.json` and restart; see [provider overrides](integrations.md#custom-providers-and-overrides). Do not blindly add permission-bypass flags to solve a read-call failure.

## MCP connects but work does not move

Check that the client starts an absolute executable with `mcp` over **stdio**, in the intended repository directory. `chakchak install` does not add this connection. The browser's loopback URL is not an MCP endpoint. An old loader or client entry may still point at a moved app.

An idle connection does not start work: call `start`, then read `status`. `busy` means another working session owns the repository; close/finish that owner and retry `start` on the same connection. Merely listing tools in a second client does not claim the repository.

In session mode (`mcp` without a provider), the driver must take `tasks` and return each `answer`. `tasks_taken` with no worker still active usually means lost work; `tasks {retake: true}` returns unanswered tasks. In provider mode (`mcp codex`, for example), the external CLI does that work and the queue tools are intentionally unavailable.

After a stale revision refusal, read `status` again and reassess its question. Do not resend the old decision blindly. `fix` and `rule` return before their work finishes: inspect `commands` and drain tasks. `status.wait` returning `unchanged` permits another wait; `idle` requires a client action. Singular `task` and `artifact` only return prompts and never launch the work themselves.

## A repository is busy or an action is refused

One working launch owns a repository. A change/task write also holds a writer boundary: migration, sync, retries and agent changes can be refused while it runs. Use the running session's Stop or finish it before retrying. Stop does not revert the agent's edits; inspect `git diff` and task commits.

After a process crash, reopen normally. ChakChak sweeps an orphaned writer before using that repository. Do not delete lock files or checkpoints to force a competing process through. A schema, artifact or task action can also be refused because an earlier step is unfinished, awaiting confirmation, or its underlying text changed. Re-read the workspace and resolve the indicated step first.

If an `openspec/` folder belongs to a parent rather than the subject repository, ChakChak refuses it. Work in the intended Git repository root. When experimenting with a nested sample, copy it outside the parent repository and initialize the copy as its own Git repository.

## Specs stay red, files are missing, or sync does nothing

The workspace compares production-code and spec hashes with the **last reconciliation**, not only file timestamps. A changed/missing spec, changed production file, missing trace/check, or unreadable reconciliation can make it out of step. Read `chakchak specs` and the report for the affected capabilities.

| Observation | Expected behavior or recovery |
|---|---|
| New file is absent | Git-untracked files are ignored; stage it with Git when it should become part of the project. Also check saved scope. |
| A new top-level folder is ignored | Sync retains the original scope and does not automatically widen it. Changing scope after migration is not offered yet. |
| Edited tests/resources do not trigger sync | Their content is not the production-code hash basis; additions/removals can still be seen. |
| Capability says it is in an active change | It waits while an unarchived change names it; sync does not overwrite it. Finish/archive or otherwise resolve that change through the normal workflow. |
| Rule/context edits did not rewrite specs | They do not themselves trigger freshness drift. Use `chakchak stage specs --all` when you intend to apply them to all specs. |
| A failed migration resumes instead of syncing | A run without a completed report is still unfinished. Resume and review it before a later sync. |
| All specs become out of step | Check whether `reconciliation.json`, trace/check files or saved state were removed. Restore the intended committed files if appropriate; a full rewrite costs model calls. |

Do not delete `openspec/chakchak/` as a general reset: it removes migration identity and can turn the next launch into a new migration over existing specs. Restore accidentally removed committed state through Git after reviewing what will be restored. Keep a backup of current changes first. A hand-edited spec is material for a later writer, not an authority stronger than the code; review the resulting diff.

## Continue a conversation or start a fresh one

A clone, new machine or switched provider can continue the talk from its saved outcome in `run.json`. If the provider's old conversation is missing, ChakChak seeds a replacement once and records `talk continued` with `reason=session-missing`; switching providers records `reason=agent-changed`. The visible talk outcome also appears in `talk.md`; edit project context in `openspec/config.yaml`, not that generated transcript.

For an intentional fresh conversation, use:

```sh
chakchak new-conversation
```

It asks for confirmation; `--confirm` supplies explicit confirmation for a scripted invocation. The old exact checkpoint is archived under `openspec/chakchak/before-restore/`, and stage, talk, specs and results remain. If archiving or the new provider call fails, the active checkpoint is preserved. MCP `restart_conversation` needs status revision/proof and `confirmed: true`.

## Saved state or settings cannot be read

Close older ChakChak processes before the first working launch of a newer build. The importer uses earlier settings/history only as sources; a valid destination takes precedence. Required damaged/partial data is kept and opening fails with its path. Best-effort copies of old logs may warn while leaving the source for another launch.

Preserve the named source and destination before repair. For malformed `~/.chakchak/settings.json`, restore a known-good backup or correct its JSON while ChakChak is closed; writes deliberately refuse to overwrite invalid existing bytes. Do not replace a damaged file with `{}` before keeping a copy. Unknown settings keys are preserved by the writer.

Older caches can remain at `~/Library/Caches/chakchak/` on macOS or `~/.cache/chakchak/` on Linux. They are not the current runtime and are not automatically deleted, because they may contain incomplete imports or client-owned history. Repository histories previously under `~/.chakchak/projects/` move into the repository after it is durable. Repeatedly deleting the home/cache obscures evidence and does not repair a damaged repository checkpoint.

## Updating fails

Finish active work, move a translocated Mac app into Applications, and check write permission on its parent directory. The updater refuses missing platform archives, missing/mismatched `SHA256SUMS`, a wrong executable version, invalid macOS signatures or a different Team ID. Use a matching signed distribution instead of bypassing the checks.

Development builds may have automatic release checks disabled. Ready-made Linux archives are not currently published, so a Linux installation may correctly find no compatible update. Refresh a custom build according to its distributor's instructions. After a replacement, quit and reopen a still-running old copy.

## Collect evidence without leaking secrets

The launch prints its journal path at startup/close and again if it moves. Look in:

```text
~/.chakchak/logs/<launch>/
<repository>/openspec/chakchak/logs/<launch>/
```

Keep the **whole launch folder** and referenced snapshots/before-restore state for a private investigation. It can include:

| Evidence | What it explains |
|---|---|
| `journal.log` | Actions, state transitions, refusals, process details, evidence-file links |
| `stdout.txt`, `stderr.txt` | Exact terminal output, partial prompts and control bytes |
| `calls.jsonl`, task/answer files and process stdout/stderr | Model attribution, timing, full requests/answers and diagnostics |
| Numbered verification artifacts | Findings for each saved spec hash, including earlier versions |
| Repository `trace.jsonl`, `verify.jsonl`, `report.md` | Current spec basis, scenario verification and open work |

Raw diagnostic folders can contain source code, prompts, answers, typed input, local paths and secrets echoed by tools. They are not sanitized support bundles. Copy important evidence promptly: when a log location is opened or entered, journal pruning keeps the newest 20 launch folders and every launch changed within the last seven days, plus the current/origin launch and folders protected by a live journal lock. Other eligible old launch folders can be removed. The rule applies to launch folders containing a journal, not arbitrary directories, snapshots or conversations. Archive or remove other historical material only after preserving what you need and closing the associated launch.

The application's developer-report screen builds a **previewable cleaned text report** for selected groups: environment, problems, timeline, model calls and crash. It removes model/user/file-text fields, substitutes local identity/path values, and masks known secret formats. The preview is the text sent, with a separately cleaned comment. Review both before sending: arbitrary names or secrets cannot be guaranteed recognizable by pattern matching. Do not paste full raw logs into the comment.

Sending is an explicit action to the configured feedback endpoint; it is not automatic uploading of the raw launch folder. The default endpoint is configured in the application build; `CHAKCHAK_FEEDBACK_URL` can override it for a custom deployment. Provider calls and GitHub update checks are separate network activity.

For a useful issue, include a minimal reproduction, expected/actual behavior, version and selected executable, the visible refusal, and a reviewed report/excerpt. Keep raw evidence privately until a specific need for it is agreed. A completed migration can still contain unshown scenarios or unfinished verification; report those as open findings rather than treating exit 0 as proof every claim was verified.
