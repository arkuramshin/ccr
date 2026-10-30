# Installation and updates

[Documentation](README.md) · [Русский](ru/installation.md) · [Integrations](integrations.md) · [Troubleshooting](troubleshooting.md)

ChakChak runs locally and uses an agent CLI that you install and sign in to separately. Packaged apps include stock OpenSpec and its Bun runtime; you do not need Node, Bun or a global OpenSpec installation to use them.

## Choose a distribution

| Distribution | Requirements | Availability |
|---|---|---|
| macOS app, Apple silicon | macOS 12 or later, Apple M-series Mac | `ChakChak-<version>-macos-arm64.zip` |
| macOS app, Intel | macOS 12 or later, Intel Mac | `ChakChak-<version>-macos-amd64.zip` |
| Linux binary/archive | Linux amd64 or arm64; packaged binary must stay within the glibc 2.34 floor | Ready-made Linux archives are not currently published |
| Windows | — | No supported build or release path |

The macOS downloads are separate apps, not a universal binary. The [release repository](https://github.com/arkuramshin/ccr/releases) is the default download and update source; custom builds can name another repository or disable update checks. Choose your architecture's archive on the [latest release page](https://github.com/arkuramshin/ccr/releases/latest). GitHub's automatic **Source code** archives contain this documentation repository, not the application.

Use a repository you can read and write. Git is recommended: ChakChak maps tracked files, including newly staged files, and Git lets you review generated specs and restore changes. A folder outside Git is mapped from its working tree instead. An agent account and network access to its provider are needed for model work; mapping and the workspace comparison do not ask a model.

## Install the macOS app

1. Download the ZIP matching your Mac and `SHA256SUMS` from the same release.
2. Verify the downloaded archive. In a folder containing only the assets you downloaded, the command below checks the manifest; entries for the other architecture can report missing files. Confirm that the ZIP you will install says `OK`.
3. Unpack it and move `ChakChak.app` to `/Applications` or your own `~/Applications` folder before using updates.
4. Open the app. A Finder launch opens ChakChak's own window. Choose an installed agent and then your repository.

```sh
shasum -a 256 -c SHA256SUMS
```

The executable is `ChakChak.app/Contents/MacOS/chakchak`. OpenSpec and `THIRD_PARTY_NOTICES.md` are in `Contents/Resources/`. The app uses the bundled OpenSpec regardless of another `openspec` on `PATH`.

Signed release apps are checked by macOS. Archives with `-unsigned` in the name are local/ad-hoc packages and are not equivalent to the signed, notarized release. See [startup troubleshooting](troubleshooting.md#the-app-does-not-start) if the system refuses the app.

## Enable the terminal command

On launch from the macOS app bundle, ChakChak attempts to create a `chakchak` symlink to its executable. It tries writable `/usr/local/bin` first and `~/.local/bin` next; Linux prefers `~/.local/bin`. Settings → **Terminal command** shows the path and offers to create the link if the initial attempt could not.

A symlink pointing to an older copy can be updated. An ordinary file named `chakchak` is preserved, so inspect an existing installation before replacing it. The settings panel also provides a command for an administrator-managed `/usr/local/bin` link.

If the link is in `~/.local/bin`, add that directory to your shell's `PATH`. For zsh, put this line in your login profile (`~/.zprofile`) and open a new terminal:

```sh
export PATH="$HOME/.local/bin:$PATH"
```

Then check the command actually selected:

```sh
command -v chakchak
chakchak version
```

The version includes the release and source commit, for example `chakchak <version> (<commit>)`; a working-tree build can have a `-dirty` suffix. The app's settings show the same identity. `version`, `help` and `skill` do not create a home directory or journal.

## Linux installation from a built archive

Ready-made releases are currently available for macOS only. If a build distributor has supplied you with a Linux archive, use the steps below. Application sources are not in this repository; its **Source code** archive cannot be used to build the app.

A Linux package contains one directory with `chakchak`, `THIRD_PARTY_NOTICES.md`, `chakchak.desktop` and `chakchak.png`. The binary includes OpenSpec. The package checks a glibc 2.34 ceiling for required symbols; musl-only distributions are not the packaging target.

For an archive and matching manifest, substitute the actual version and architecture:

```sh
sha256sum -c --ignore-missing SHA256SUMS
tar -xzf chakchak-<version>-linux-amd64.tar.gz
cd chakchak-<version>-linux-amd64
mkdir -p "$HOME/.local/bin"
install -m 0755 chakchak "$HOME/.local/bin/chakchak"
install -Dm 0644 chakchak.desktop "$HOME/.local/share/applications/chakchak.desktop"
install -Dm 0644 chakchak.png "$HOME/.local/share/icons/hicolor/256x256/apps/chakchak.png"
chakchak version
```

Keep the notices with the distribution. The desktop entry runs `chakchak app`, so the desktop environment must find `chakchak` on `PATH`; a shell-only path change may not affect an existing desktop session. Running `chakchak app` from a terminal opens the local browser interface.

## First working launch

Install and authenticate the agent with its own tools first. ChakChak does not install the agent or sign in for you. The [integration guide](integrations.md#providers) lists the built-in provider definitions and explains custom programs.

Choose one of these entry points:

```sh
chakchak app                         # choose a repository in the application
cd /absolute/path/to/your/repository
chakchak web codex                   # browser interface for this repository
chakchak cli codex                   # terminal interface
chakchak cli claude --lang ru        # discussion, specs and report in Russian
```

Replace `codex` or `claude` with the program you use. The app requires an explicit agent choice. Terminal runs can use a named agent or the available/saved choice. Settings remember the agent and selected executable; model picks in the app live in memory. See [commands and defaults](integrations.md#commands-and-flags) for model, language and concurrency behavior.

The default local address is `127.0.0.1:7757`. Use the address the launch actually prints. It is a loopback service intended for this machine. `app` opens the browser from a terminal; launching the macOS bundle from Finder uses a native window around the same interface.

Review the repository and the scope before starting migration. Generated, dependency and build folders begin excluded according to the shipped scope hints and `.gitignore`. Git-untracked files are outside the map until staged. Work through the [application manual](manual.md) for the migration and later changes.

## Where files live

| Location | Purpose |
|---|---|
| `~/.chakchak/settings.json` | Recent repositories, chosen agent/program, custom calls, language, learned pace/timing, update check cache; not provider credentials |
| `~/.chakchak/agents.json` | Optional provider overrides/additions |
| `~/.chakchak/SKILL.md` | Full current agent instruction, refreshed on working launches |
| `~/.chakchak/logs/<launch>/` | Launches before a repository becomes durable, machine commands and application startup |
| `~/.chakchak/runtime/openspec/` | OpenSpec unpacked by a binary built with `BUNDLED=1`; `.app` runs the copy inside its bundle |
| `~/.chakchak/runtime/from/<name>-<key>/` | Temporary per-repository runtime/answer files, removed on normal exit; adjacent `.repository` file identifies the source |
| `<repository>/openspec/` | OpenSpec config, master specs, changes and ChakChak's durable migration state |
| `<repository>/openspec/chakchak/logs/` | Journals after consent/durable state; the launch prints the new location when it moves |
| `<repository>/openspec/chakchak/{conversations,snapshots,before-restore}/` | Local run history, excluded from commits by ChakChak's `.gitignore` |

The repository is identified by its saved content, not a home-directory project registry. A clone or moved repository with `openspec/chakchak/run.json` is recognized; commit the durable OpenSpec output needed by colleagues. Local journals and snapshots do not normally travel in a clone. Nothing is written into a new subject repository before consent.

Old installations can import settings and history from earlier config/cache layouts. Close older processes before upgrading. Imports preserve damaged/partial sources and refuse unsafe replacement; old cache directories are not automatically deleted. See [recovery](troubleshooting.md#saved-state-or-settings-cannot-be-read).

## Update a packaged installation

A release with an update repository checks GitHub for a newer release at most daily during `app`; Settings also offers a manual check. Development builds and builds made with `RELEASES=` do not check. Network failures and rate limits do not stop repository work; failed transient requests have two retries, after 2 and 5 seconds.

The **Update** action downloads the matching archive and `SHA256SUMS`, verifies the checksum and executable version, replaces the installed program/app, and restarts at the same local address. On macOS it also requires `codesign --verify --deep --strict` and the same signing Team ID as the running app. A failed bundle swap restores the old app. Linux replacement uses the checksum check without the macOS signing check.

An update is refused while work runs, from a translocated macOS app, into an unwritable location, or when a matching archive or checksum manifest is missing. Move the app to Applications, stop/finish active work, and use an installation location you can update. A Linux build cannot update from a release that has no Linux asset. Inspect the refusal instead of replacing files during a running migration.

## About source builds

This repository contains documentation and packaged releases. Building ChakChak requires separate access to its application source and the instructions in that repository. There is no `App` directory or application Make target here.

A development binary without bundled OpenSpec needs `openspec` on `PATH`; packaged applications include it. Custom builds may disable release-update checks: follow the instructions of the person who supplied your build.

## Remove an installation

Run `chakchak uninstall` before removing the executable if you installed agent skill loaders. It removes only exact, unchanged loaders owned by that installation and reports modified or foreign files as preserved. It does not remove client MCP configuration, the app, terminal links, provider history, your settings or repository specs.

Close ChakChak, remove the app/binary and any terminal symlink that points to it, and remove its MCP entry in each client you configured. Keep `openspec/` if you want to continue using the specs. Back up histories before removing `~/.chakchak/` or repository-local diagnostic/history folders. Uninstall does not erase these data; the separate [journal retention policy](troubleshooting.md#collect-evidence-without-leaking-secrets) only prunes eligible old launch folders.
