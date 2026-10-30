# ChakChak user manual

[Documentation](README.md) · [Русский](ru/manual.md) · [Installation](installation.md) · [Integrations](integrations.md) · [Troubleshooting](troubleshooting.md)

ChakChak helps an existing code repository move to spec-driven development. It reads the implementation, agrees on the project's purpose with you, groups its files into capabilities, and writes ordinary OpenSpec specifications. You can then use the application to plan changes, review documents, run implementation tasks through your coding agent, and archive completed changes.

This manual describes the application window and browser interface. Both use the same local ChakChak process and the same repository files. Builds with **Help → User manual** include this text offline in the application's interface language. If your installed version only has a quick guide, read the full manual here.

## Contents

- [1. What ChakChak produces](#1-what-chakchak-produces)
- [2. Before the first run](#2-before-the-first-run)
- [3. Open a repository and choose an agent](#3-open-a-repository-and-choose-an-agent)
- [4. Find your way around the interface](#4-find-your-way-around-the-interface)
- [5. Start a migration and choose its languages](#5-start-a-migration-and-choose-its-languages)
- [6. Code map: decide what is in scope](#6-code-map-decide-what-is-in-scope)
- [7. Repository overview: brief, questions, and project description](#7-repository-overview-brief-questions-and-project-description)
- [8. Cut: group files into capabilities](#8-cut-group-files-into-capabilities)
- [9. Specs: inspect results and resolve problems](#9-specs-inspect-results-and-resolve-problems)
- [10. Stop, pause, and recover](#10-stop-pause-and-recover)
- [11. Read the report](#11-read-the-report)
- [12. Open an existing OpenSpec repository](#12-open-an-existing-openspec-repository)
- [13. Reconcile specifications after code changes](#13-reconcile-specifications-after-code-changes)
- [14. Create a change and write its planning artifacts](#14-create-a-change-and-write-its-planning-artifacts)
- [15. Implement and review tasks](#15-implement-and-review-tasks)
- [16. Verify, sync a change, and archive it](#16-verify-sync-a-change-and-archive-it)
- [17. Customize workflow schemas](#17-customize-workflow-schemas)
- [18. Settings, terminal access, and updates](#18-settings-terminal-access-and-updates)
- [19. Saved data, Git commits, and access](#19-saved-data-git-commits-and-access)
- [20. Practical questions](#20-practical-questions)

## 1. What ChakChak produces

A **capability** is an observable outcome or obligation of the system: for example, searching a catalog, checking out a basket, or storing a document. Its specification describes requirements and scenarios, rather than reproducing classes and methods. Code serving one outcome can belong to the same capability even when it spans UI, business logic, and storage.

The main deliverables are:

| File or folder | Purpose |
| --- | --- |
| `openspec/config.yaml` | Project context and rules used when preparing specifications and changes. |
| `openspec/specs/<capability>/spec.md` | Master specification of a capability. Names may include area folders. |
| `openspec/chakchak/report.md` | Migration or reconciliation results, including unfinished work and findings. |
| `openspec/chakchak/` | Code map, file assignment, saved progress, evidence linking specifications to code, and recovery data. |
| `openspec/changes/<name>/` | Documents and delta specifications for an active change. |
| `openspec/changes/archive/` | Archived changes. |

The specifications remain normal OpenSpec files. You can read them in an editor, review them in Git, and work with stock OpenSpec tools. ChakChak's evidence and saved progress add navigation and recovery; they do not replace the specifications.

Machine checks establish that declarations are accounted for and the specification has valid structure. A separate model checks scenarios against code. These checks help review the result; they do not prove that every statement is true or that the software passes its tests.

## 2. Before the first run

You need a local repository and an installed coding-agent CLI that is already signed in and can work from your terminal. ChakChak uses that program's account and model access. A release includes OpenSpec and its runtime; a development build may require `openspec` on `PATH`.

The release workflow currently publishes packages for Apple silicon and Intel Macs. Move `ChakChak.app` into Applications before opening it. Linux is supported through source builds and local packaging; a downloadable Linux release is not currently promised. For a Linux installation, put the binary on `PATH`; the desktop launcher is optional. The provided Linux packaging targets glibc 2.34 or newer.

Prepare the repository before asking an agent to implement changes:

1. Choose the repository root, rather than an arbitrary subfolder of a larger project.
2. Review its Git status and keep a recoverable baseline of the work you care about.
3. Add source files that should participate in the migration to Git. A file can be staged without being committed, but untracked files are outside the normal code map.
4. Check that the selected agent can read the repository. On macOS, the app and the agent can require separate Files & Folders permissions.

The migration reads source without building the project. Supported parsers cover Swift, Objective-C, Java, Kotlin, Python, Go, JavaScript, TypeScript, and TSX. Files outside the supported parsers can still appear in the inventory, but do not provide parsed declarations for specification coverage.

## 3. Open a repository and choose an agent

### Launch the application

Open ChakChak normally, or run:

```sh
chakchak app
```

The packaged macOS application opens its own window. Linux and terminal launches normally open the local interface in your browser. If the browser does not open, use the local address printed by the command.

The first screen contains repositories and the available agents. On **Repositories**, use **Choose folder**, where the platform offers a folder picker, or enter an absolute folder path under **Add repository** and press **Add**. A recent repository's **Open** button opens it directly; selecting its row shows its details first. **Forget** removes a remembered entry and leaves its folder on disk.

The destination depends on what the repository contains:

| Repository state | What opens |
| --- | --- |
| No local OpenSpec workspace | The start of a migration. |
| An unfinished ChakChak migration | Its saved migration stage. |
| A local OpenSpec workspace without an unfinished migration | The SDD workspace. |

The recent list is a convenience on this computer. A moved or cloned repository can still be recognized from its own ChakChak files.

### Select the executable and model

Open an agent's cell and press **Choose**. ChakChak can distinguish multiple installed copies by their executable paths. **Check** asks that copy for its model list; choosing an agent does not require this list to have loaded. The first-screen list of models is informational. Pick the model in the run's settings or the relevant composer.

For an additional provider, open **New provider**, enter its program name or absolute executable path, and press **Add**. Known providers do not need a custom command. An unfamiliar provider may require a command template using `{task}` and, if supported, `{model}`. Adding a provider can run it to determine how it accepts work. Provider support and available models depend on the installed CLI.

The chosen agent and executable path are saved on this computer. That choice also takes precedence when reopening a saved run; the run's recorded agent is the fallback when no saved choice exists or its executable is gone. In the application, **Settings → This run** allows switching agents when work is idle. The control explains why a change is unavailable. A switch to another agent preserves accepted work and starts a new conversation from saved context where necessary.

A model choice is held only in the running app, separately for each executable copy. It applies to subsequent calls, including when you select it while a call is already working; it does not change the call in flight. Restarting ChakChak clears these model choices, so check the model before continuing if you need a particular one. An empty choice uses the agent's own default.

A page started with `chakchak web` cannot change the agent in place: stop it and launch it with the desired agent and model. A page observing an externally driven migration is mainly read-only and explains that an agent is controlling it.

## 4. Find your way around the interface

The top bar identifies the repository and provides Help and Settings. The left **Hive** contains stages, specifications, changes, or workflow steps. The right **Workspace** shows the selected item.

- Select a stage or step to inspect it. Selecting the same stage again can open its model calls. Call details show the request, answer, timing, and outcome.
- Open a specification or file to read it in the Workspace. **Back** or Escape returns to the previous view.
- A linked Requirement or Scenario heading opens its supporting code. Scenario evidence may be the lines saved when it was checked; declaration views read current source and identify changes.
- Right-click a supported item, or use Shift+F10 on the focused item, for actions such as opening it, revealing its file, or copying its name or path.
- **Settings → Motion → Reduced** replaces the animated Hive with a sidebar and disables motion. The same fallback is used when the graphics view cannot run.

Several windows or browser tabs connected to the same app share the selected repository and main view. Navigation in one can move the others. Draft answers remain associated with their repository and question.

Opening a document does not start another model call. If the local process stops answering, the page keeps the last visible state, shows a connection notice, and disables actions that need the server.

## 5. Start a migration and choose its languages

On the start screen, inspect the repository tree and the description of what ChakChak found. Press **Start** (Start migration) when ready. Repository output is not created before this consent.

If the page offers **Init git**, that action initializes a Git repository, stages its current contents, and creates an initial commit using your Git identity. Review the folder and ignore rules first: the action covers the folder as it stands. Fix any identity or Git error reported by the page before continuing.

There are two independent language choices:

| Setting | Applies to | Can change during a run? |
| --- | --- | --- |
| Interface language | Buttons, navigation, messages, and this manual. | Yes. |
| Spec language | Discussion, project description, capability descriptions, specifications, and report. | Only before the run leaves Preparation. |

Set **Settings → This run → Spec language** before beginning. English and Russian are supported. Once locked, the language survives retries, restarts, and agent changes. Switching the interface does not translate existing documents. OpenSpec markers such as `Requirement`, `Scenario`, `WHEN`, and `THEN`, as well as paths and IDs, retain their technical form in Russian documents.

## 6. Code map: decide what is in scope

The conceptual migration sequence is **map → talk → picture → cut → specs → report**. In the application, talk and picture appear together as **Repository overview**.

The **Code map** stage inventories files and declarations. Review the production-file, test-file, unparsed-file, and code-element counts. Open the map or the unparsed-file list if the totals differ from what you expect. The estimate of model calls and time is approximate; model speed, corrections, and your review time affect the actual duration.

Use the folder checkboxes to choose the product being documented. Open a folder to narrow the selection inside it. Dependencies, generated files, caches, and build outputs start excluded where the default rules or the root `.gitignore` identify them. Examples include `vendor`, `node_modules`, `Pods`, `build`, `target`, `dist`, and `generated`. You can explicitly include a folder when it is part of your product.

Three rules explain most missing-file surprises:

1. In a Git repository, the map uses tracked files, including newly staged files. Run `git add` for a source file that is still untracked; merely creating it is insufficient.
2. Excluded folders contribute no mapped files or declarations to the migration's prepared tasks, cut, or trace.
3. Unsupported source and resources are not parsed into declarations. Selecting a folder does not add a parser for its language.

A folder without usable Git listing can be read as a working tree. A supported production file is required. If top-level folders exist, leaving every folder unchecked cannot be rescued by source files at the root: choose at least one product folder.

**Tests are product too** is useful for an automation or test-framework repository whose deliverable is its tests. Normally tests support production capabilities. With this switch on, mapped test files count as product code for coverage and specifications. ChakChak can enable it automatically when the selected folders contain tests but no production files; an explicit choice is preserved.

Press **Continue** after reviewing scope to open the repository overview. The choice is saved in `openspec/chakchak/scope.json`. Later reconciliation reuses it and does not automatically admit a new top-level folder. The application does not currently offer changing an established scope after migration. Older runs without a scope file retain their earlier behavior.

Scope controls the migration's inventory and prepared material. It is not a filesystem security boundary around the coding-agent process.

## 7. Repository overview: brief, questions, and project description

### Give the initial brief

After the map, the overview opens with a brief. Describe what the product does, who uses it, which terminology matters, and anything that is difficult to infer from code. Attach supporting files if useful. The brief may be empty; press **Start analysis** to let the model inspect the code first.

Useful input is concrete: “This service reserves inventory for partner orders; billing belongs to another system.” Intended future behavior should be distinguished from behavior that already exists.

### Answer or end clarification

The model can ask questions and propose answers. Select a proposal or write your own answer. Review the collected answers before sending them. **Finish talk** ends clarification using what has been established so far. Pressed from the initial brief, it runs the first analysis and takes the model's proposed answers. Unanswered questions are not evidence of your agreement.

If analysis fails, use the retry offered for that operation. Your brief, attachments, or submitted answers are retained. A failed reply is retried with the same words unless you explicitly replace them.

### Accept the project description

The resulting **Project description** is the picture stage: a concise account of the project, written into the `context` of `openspec/config.yaml`. Read it and correct mistaken boundaries, users, terms, or responsibilities. Review any specification rules alongside it.

Press **Continue** to accept the description. A normal interactive migration then waits for instructions before the cut; accepting the description does not itself start that model call.

If the existing workspace already has a project context, ChakChak can use it and skip the overview. During a later reconciliation, **Discuss again** explicitly reopens this discussion.

## 8. Cut: group files into capabilities

At the cut's instruction stop, describe boundaries you want preserved: what to keep together, what to separate, and which shared code is a product capability. Press **Start the cut**, with or without instructions. Nonempty instructions are saved as rules for that stage.

The model proposes a tree of capabilities and places mapped files under them or under `support`. Support contains files without a separate product behavior to specify. Tests normally follow the capability they test. Larger capabilities may receive extra model readings to decide whether they contain separate outcomes; “kept whole after reading” is an explained decision, not automatically an error.

Review the tree before accepting:

1. Open capabilities and inspect their files. Check whether the grouping matches outcomes understood by users or callers.
2. Read coverage issues, especially unplaced files and duplicate placement. Every mapped file must have one effective place before acceptance.
3. Move a file or folder where needed. Review the preview, then press **Apply**. A move can also carry associated tests.
4. For a broader correction, explain the desired change in the cut conversation. A new proposal uses the accepted description and your correction.

Some answer defects get one automatic repair before the proposal is shown. Remaining issues are visible for review. Accepting a cut with permitted issues keeps them in the report; it does not erase them. A cut with missing or duplicated file coverage cannot be accepted.

During additional readings, **Stop and review** keeps finished readings and opens the proposal. **Continue reading** resumes the unfinished ones. If you accept without completing required readings, affected capabilities can be set aside rather than receive a specification.

Accepting the cut in a normal interactive migration opens a second instruction stop before specifications. Add domain terms and writing rules if useful, then press **Start writing specs**. A reconciliation, autopilot, or unattended run can pass these extra instruction stops automatically.

## 9. Specs: inspect results and resolve problems

Each capability has a separate writing call. Several capabilities can run in parallel. The writer receives its code, project context, rules, and a checklist of declarations. ChakChak checks declaration accounting and OpenSpec structure, then a fresh model reader checks scenarios against the implementation. Findings can send the draft back for bounded correction rounds.

Use the **Specs** tab for the tree and **Chat** for calls and summaries. Filters expose work in progress, written specifications, leftovers, failures, and specifications not written. A completed model call is not the same as a completed specification.

| Result | How to interpret it |
| --- | --- |
| Written | A specification was saved. Inspect any advisory findings as well. |
| Written with leftovers | Saved work has remaining accounting or review information to inspect. |
| Needs fixing | The draft or its checks still have problems. |
| Not written / set aside | No completed specification was produced for that capability. Other work can continue. |
| Failed | The application encountered a failure for that capability. |
| Check pending | The specification was saved, but its reader check has not finished. |

For a problematic specification, open **Review** from its call summary or context menu. The review brings together form errors, declaration accounting, scenario verdicts, missing outcomes, and the calls that produced them. **Checked**, **contradicted**, and **not shown** describe the reader's evidence; “not shown” alone is advisory and does not prove that behavior is absent.

During the migration, use the remarks composer to address one specification with a correction, or save a rule for later specifications. A new global rule does not rewrite those already written. **Another round**, **Retry spec**, or **Run again** is offered according to the result; use the action on the affected capability instead of restarting successful work. The stage's **Retry** action (Retry failed) can retry failed capabilities together once the wave is idle.

After a migration's report is written, correction words and new rules through that migration's composer are closed. Wordless retries remain where offered. Use a new OpenSpec change to change behavior, or edit configuration for future work. A later reconciliation has its own correction workflow and does not share that migration-only closure.

## 10. Stop, pause, and recover

During specification writing, the **Stop** menu offers different outcomes:

| Action | What happens |
| --- | --- |
| Stop now | Cancels calls in flight and pauses on Specs. Completed files stay. **Continue** resumes remaining work. |
| Start no new specs | Lets started capabilities finish their writing and checks, then pauses if work remains. |
| End the stage | Cancels current calls, keeps completed work, and writes a report with unfinished specifications listed. |

**Continue** or **Resume** uses saved progress. A specification saved just before a stop remains written; its missing reader check runs when work continues. Deliberately stopped runs wait for you. Work interrupted by a process crash can be recovered when the repository is reopened.

A busy model is retried automatically with increasing delays, which can last roughly half an hour. **Retry now** bypasses the current wait where offered. A login, quota, or folder-access refusal needs the underlying problem fixed first. The page provides the action for its current state and keeps provider output under the diagnostic details.

If a saved provider conversation is unavailable and automatic recovery does not succeed, use **Restart** (Start the conversation again) in that refusal. It confirms a fresh conversation using saved context and preserves accepted artifacts. It is not the normal way to correct a specification.

If the accepted file assignment is missing, ordinary continuation is blocked to avoid guessing it from partial progress. Close the active process and back up the workspace. Restore a matching complete workspace, or use a valid cut-stage snapshot through the terminal's `chakchak stage cut` recovery. Keep the assignment and run checkpoint together.

Closing a browser tab alone does not stop the local process. **Settings → Quit** turns it off and keeps completed work. If the macOS window is only a viewer for a process started elsewhere, closing that viewer leaves the serving process running.

## 11. Read the report

The Report screen summarizes written specifications, files, model calls, and elapsed time. Open **Report** for the full Markdown account and **Code map** for the inventory. Findings and unfinished work are part of the result even when a report exists.

Read the report before treating migration as complete. Check specifications that were set aside, contradicted scenarios kept after review, accounting leftovers, and cut issues. Proposed file moves discovered while writing are proposals for you to review; they do not silently move files between capabilities. Applying a cut change can require rewriting affected specifications.

**Open SDD** opens the specification workspace. A report can also exist for a stopped run, so this button alone does not mean every capability has a specification.

## 12. Open an existing OpenSpec repository

ChakChak distinguishes existing specifications from an existing ChakChak migration:

- **Local workspace and ChakChak assignment:** the application recognizes saved progress or the completed migration. Moving or cloning the folder does not require migration again if the necessary files came with it.
- **Local workspace with specifications but no ChakChak assignment:** SDD can show the workspace. **Migration** builds ChakChak's map and assignment; existing specifications are reference material, not an accepted assignment. Existing project context may let the overview be skipped.
- **Workspace inherited from a parent folder or OpenSpec defaults:** ChakChak reports that the workspace is outside the selected folder and does not migrate it. Select the intended root, or explicitly create a local OpenSpec workspace when that is your intended project structure.

Current continuation expects ChakChak state under `openspec/chakchak/`. Earlier development layouts with its checkpoint or assignment directly under `openspec/` are not automatically converted.

## 13. Reconcile specifications after code changes

The SDD specification view compares the repository with its last saved reconciliation. This inspection asks no model. **In sync** means the tracked inputs and saved evidence match that record; it is not a new semantic review or a test run.

Differences can include changed production code, added or removed files, a hand-edited `spec.md`, an unwritten specification, or missing trace/check evidence. **Information** explains the reasons when synchronization is needed. Opening a changed specification can mark the Requirements and Scenarios touched by code changes.

To reconcile:

1. Inspect the reason and ensure new source files are tracked by Git and inside the saved scope.
2. Press the workspace's **Sync** to open the run's consent question.
3. Review which capabilities will be rewritten and the approximate cost. Choose **Sync**, or **Discuss again** if the project context must change first.
4. Review the cut. Deleted files are removed from the assignment; new or unplaced files may need a model to place them.
5. Let affected specifications be rewritten and inspect the resulting synchronization report.

The writer reads existing specifications as material, including hand edits, but reconstructs behavior from code. Hand-written requirements are not guaranteed to survive a sync. Changing only a tracked test or resource's contents does not trigger the same production-code comparison; adding or removing files can still matter. Code changed again after a synchronization began may need another sync.

Capabilities covered by active OpenSpec changes wait while those changes are active. This prevents ordinary reconciliation from overwriting planned work. After archive, affected capabilities can require reconciliation again. New top-level folders are not silently added to a saved scope.

## 14. Create a change and write its planning artifacts

Use an OpenSpec **change** for intended new behavior, corrections, or a planned refactoring. It keeps the proposal, delta specifications, design, and tasks together before implementation. The exact artifacts and dependencies come from the selected schema.

### Create the change

1. Open **Changes**, or press **Change** from an in-sync specification view.
2. Enter a unique name such as `add-catalog-search`: lowercase Latin letters, digits, and single hyphens, starting with a letter. The limit is 100 characters; `archive` is reserved.
3. Select a schema. The project's configured schema is selected initially.
4. Provide any schema-specific inputs or attachments requested by the form.
5. Press **Change** (Create a new change). The first ready artifact opens in its **Chat** tab.

For an artifact with no dependencies, describe what needs to be done and why; empty words are not enough. For subsequent artifacts, the prior documents supply the context and extra words can be optional. Attach files or images with the composer's attachment control when needed.

### Write, review, and confirm

Press **Write** for the selected artifact. The agent writes its configured output into the change folder. The **Artifact** tab shows the resulting document or documents; the chat shows the agent's report and available activity details.

Every artifact written through ChakChak waits for your review after the call finishes. This includes the tasks artifact. Read it, then choose:

- **Confirm** to accept it and move on. Unanswered questions may remain recorded; confirmation is allowed without answering all of them.
- Answer questions or add corrections and press **Refine**. The agent updates the artifact in another call, then waits for review again.

An existing file does not mean a running or interrupted write is complete. An unfinished artifact offers **Continue**, **Redo**, or **Accept as is**. Continue asks the agent to finish the partial output; Redo starts the writing again; Accept as is takes responsibility for the file as it stands. An unfinished or unconfirmed dependency blocks artifacts that need it.

**Back to this step** on an earlier written artifact previews the downstream documents that must be removed before it can be changed. Confirm that list deliberately. Chat history remains; implementation code already written for tasks is not rolled back by removing planning documents.

## 15. Implement and review tasks

ChakChak's task workflow uses numbered checklist lines in the change's top-level `tasks.md`. Once planning is complete, the implementation stage becomes one cell per task, in file order. Give each task a unique number. Later tasks can be inspected but wait until earlier tasks are marked complete.

Before generating tasks for added requirements, the **Examples** questionnaire can offer existing master requirements and code as examples. Choose relevant candidates, name repository-relative example files, or explicitly choose no example. Skipping a question keeps its default selection when a suitable candidate exists. An invalid path is refused rather than silently ignored.

For each task:

1. Open its cell and read the task, `Files`, `Example`, and `Check` instructions where present.
2. Add guidance if needed and press **Do task**. The agent can modify project code, run commands, and create task commits.
3. Inspect the agent's report and the **Changes** tab. It shows committed changes and, where available, changes still in the working tree. Large or binary diffs can be limited or omitted with an explanation.
4. Run or inspect the relevant verification for the work. A task's completion mark does not establish that its tests passed.
5. Press **Done** when satisfied. That changes the checkbox in `tasks.md` and allows the next task to run.

The first task needs a Git baseline: a repository with a first commit and no unrelated untracked project files outside `openspec/`. When a refusal names such files, decide which belong in Git or should be ignored, then establish the baseline yourself. ChakChak does not silently create it for a task.

**Done** becomes available after the task has a commit, or after an answered check task that changed no files. If the agent edited files but made no commit, ask it to commit its work or rerun the task. If a task already has a commit but is not ticked, review it and tick it; **Run again** explicitly authorizes another attempt. Untick a completed task before running it again.

Stopping a task leaves its existing changes in the working tree. It does not undo them. A change has one writing operation at a time; opening another step does not move the running agent there.

Some external schemas track a file other than `tasks.md`, or track no tasks. ChakChak then shows OpenSpec's progress read-only and does not offer its task cells, task chat, or checkbox editing. Use that schema's normal workflow outside ChakChak for implementation.

## 16. Verify, sync a change, and archive it

These are separate from the workspace's code-to-spec **Sync** described earlier.

| Change step | What to review or do |
| --- | --- |
| Verify | Inspect planning documents, task marks, source, delta specifications, and clashes. Check implementation and test evidence. |
| Sync | Review the delta specifications and their master destinations, including conflicting changes. |
| Archive | Review remaining work and finish the change through OpenSpec. |

The Verify and Sync panels do not launch an agent or tests just because you open them. “No recorded verification result” and an unknown sync result mean exactly that. File presence, a clear checklist, and the absence of reported clashes are useful evidence, but not proof that tests or a separate sync command ran.

When the panels have no blocking remarks, **Continue** records that you passed the step and advances. It does not execute verification or merge specifications. Changes to artifact/task state or task marks can clear those recorded passes; a manual edit outside that workflow still requires your own review.

On Archive, **Finish** runs stock OpenSpec archive. The operation moves the change to a dated archive folder and updates the master specifications as OpenSpec requires. A refusal remains visible with its reason. Successful archive returns to the specification workspace; if it reports code and specifications out of step, use the workspace reconciliation flow.

## 17. Customize workflow schemas

The **Schema** area lists workflows OpenSpec knows. A schema defines artifact IDs, dependencies, generated paths, templates, and implementation instructions. The default workflow is suitable for starting; custom schemas can add documents your team needs.

Built-in schemas are read-only. Create a project schema based on an existing one, or import a schema folder. Edit artifact instructions and templates, dependency order, rules, and apply/archive guidance. Field help explains each setting and the form reports validation errors before saving.

Schemas saved or imported through ChakChak must include specification output, a `tasks.md` artifact, and a valid apply step tracking `tasks.md`. Names use lowercase Latin letters, digits, and hyphens; paths must stay inside the change. Reserved operation names such as `apply`, `verify`, `sync`, and `archive` cannot be artifact IDs.

Changing a schema used by active changes can make their existing documents inconsistent with the new workflow. Review the warning before saving structural edits. Only a project schema unused by the config and active changes can be deleted. Deletion removes its schema and templates; use version control to recover them if necessary.

## 18. Settings, terminal access, and updates

Open the gear to inspect **Settings**. Close it or press Escape to return.

- **Preferences:** interface language, light/dark/system theme, and motion. These affect the interface on this computer.
- **This run:** repository, spec language and its lock, agent, model, and available folder or terminal actions.
- **About / Updates:** application version, **Check**, release notes, and **Update** when offered. Development builds do not perform release updates.
- **Terminal command:** the installed command or link and any shell `PATH` instructions needed to invoke `chakchak`.
- **Open in browser:** opens the same local interface in the default browser where offered.
- **Reset settings:** moves `settings.json` and custom `agents.json` aside as timestamped `.bak` files. Repository work, journals, and saved migrations remain. A restart may be needed to reload default agents.

The application has no autopilot switch. Autopilot is a terminal-run option; its presence in a saved run can explain automatic acceptance of clean review points.

**Open in Terminal** opens a terminal at the repository. Beside a displayed shell instruction, it can copy that instruction first. ChakChak does not paste or execute it for you.

An update downloads and checks the matching release, replaces the installed application, and restarts the local process. Wait until work is idle. If the app cannot replace itself, read the reason: for example, a macOS app still running from a temporary download location, a protected installation directory, or unavailable release files. You can install the release manually instead.

## 19. Saved data, Git commits, and access

### What stays in the repository

Besides standard OpenSpec output, `openspec/chakchak/` stores `run.json`, `codemap.jsonl`, `assignment.jsonl`, `scope.json`, `reconciliation.json`, the report, and per-capability `trace.jsonl` and `verify.jsonl` evidence under its `specs/` tree. Keep coherent saved state with the repository if colleagues should continue the migration and use code links.

Local conversations, journals, stage snapshots, and states saved before a restore are kept there too, with ignore rules excluding local history from ordinary commits. Brief attachments and change-chat attachments are copied into ignored ChakChak folders. Ignore rules are not encryption or access control.

Journals record model requests, answers, diagnostics, and timing. Launches retain the newest 20 journal folders and those modified in the last seven days. Conversations, snapshots, restore backups, and attachments do not have the same automatic age/count cleanup; review disk use when keeping long histories.

### What stays on this computer

`~/.chakchak/` holds settings, custom providers, the installed skill instruction, locks, runtime files, and journals for launches without a durable repository run. Temporary repository runtime is removed when its owner releases the repository. Interface drafts and some display preferences also use the browser's local storage.

### What gets committed

ChakChak can create commits on the current branch for its own OpenSpec writes: a new change, a completed artifact write, a task tick, schema save/import, archive, and migration output at Report. Those commits target the event's files and exclude `openspec/chakchak/**`; they do not publish a branch. The agent implementing a task makes its own task commits.

A failed commit does not undo the file write or archive. Read the commit notice and inspect Git status before retrying. ChakChak's automatic commits do not add its durable evidence for you: include the intended `openspec/chakchak/` files in your normal review/commit workflow while preserving its ignore rules.

### What the agent can access

The interface and saved workspace are local, but model work runs through the selected agent and its configured provider. Code, prompts, and attachments can be sent to that provider. The application also contacts release services when checking or downloading updates.

Migration calls request read-only work and use provider restrictions where available. This is not a universal sandbox guarantee across every supported CLI. SDD writing calls run with broad permissions, load the agent's normal configuration and available integrations, and do not stop for interactive tool approval. They can edit files, run commands, and use connected services according to that configuration. Configure the agent and its MCP servers accordingly before running a task.

The scope checkboxes do not restrict those permissions. Review material before attaching it. When sharing a diagnostic report, inspect the selected content; local logs can contain repository details and model messages even when displayed output masks some secret-looking values.

## 20. Practical questions

### Why is my new file missing?

Check Git tracking, the saved folder scope, and the parser support. A tree view can show an untracked file while the code map deliberately excludes it. A new top-level folder is not automatically included in a later reconciliation.

### Why are there zero test files?

The test folders may be excluded, the files may be untracked, or **Tests are product too** may count them as production instead. Inspect scope and the map before interpreting the number as test coverage.

### Why did the next stage not start?

A normal migration waits for your brief, description approval, cut instructions, cut review, and spec instructions. A change waits for artifact confirmation and task ticks. Open the current stage and follow its named action; an idle screen is often a deliberate review point.

### Can I leave the app while it works?

You can navigate to saved documents without restarting work. Closing only a browser tab leaves the process running. Use the stop controls when you intend to pause; use Quit to turn the app off. Already saved results remain available next time.

### Does green mean the software is correct?

No. A saved specification, structural validation, a scenario check, a task tick, and executed tests are different facts. Review the relevant evidence for the claim you need to make.

### Can I edit specifications by hand?

Yes, they are ordinary Markdown files. A later code reconciliation detects edits and can rewrite them from implementation evidence. Use a change to express intended behavior and Git to review or recover edits.

### Can I use the project without ChakChak afterwards?

Yes. Master specs, changes, schemas, and configuration are ordinary OpenSpec artifacts. Retain ChakChak metadata when you want its continuation, reconciliation, or evidence navigation.

### The provider says login, quota, or access failed. What now?

Fix the provider's sign-in, model entitlement, quota, or folder permissions using its own tools. Then retry the failed action in ChakChak. Changing the interface language or reopening a document cannot repair a provider refusal.

### How do I report a problem usefully?

Record the ChakChak version, operating system, selected agent and model, the action you pressed, expected behavior, actual message, and whether completed files remain. Use the app's developer-report controls if available and review the content before sending. Keep a copy of the repository's saved state when investigating an interrupted run.
