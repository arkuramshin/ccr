# ChakChak: a tool for SDD

English | [Русский](README.ru.md)

- High-quality migration of code repositories to SDD
- Real-time sync of code and spec.md
- Easy builder for schema.yaml
- Configurable change steps (MCP, attachments, custom rules)
- Clean, strictly controlled harness
- ~~Vibecoding~~ Deepcoding
- WEB, MCP or CLI mode, your choice

## Models

**Any model with a CLI.** Every agent known today works out of the box; any other is one line in the settings.

## Migration

**Easy migration.** Tick the folders to migrate. The model distributes the files across specs, and you steer it with your own instructions and accept or reject its suggestions; there is an autopilot. You see which spec every file went to. Nothing goes unnoticed: every file, every line and every signature has its own address. The rules, the terms and the language of the specs are yours.

**Works even with code that does not compile.** ChakChak reads code without building it: Swift, Java, Kotlin, Python, Go, Objective-C, JavaScript, TypeScript.

**Multi-stage checks.** Every spec goes through several checks: for completeness, for form and against the code. What is found gets fixed on its own, without you. Code that never landed in any spec is highlighted, not hidden.

## Sync

**Real-time check of specs against code**
ChakChak watches at runtime that the code and the specs match, checks every fact against lines of code and shows which parts of a spec the changed code touched.

**Drift shown precisely.** A deterministic check: no model, no cost, the same result on every run. It finds where the code moved away from the spec and shows exactly what changed, down to the function.

## Schemas

**For any project and any team.** Full-stack, backend, frontend, test automation, scripts and more: ChakChak adapts to your project and to the way your team works. You build the process: steps, instructions, templates and dependencies. For example: analysis → mockups → proposal → specs and design → tasks → implementation. Start from a copy of a ready schema or import your own from a folder.

**Your own MCP, attachments, skills and rules for any step.** Attach an MCP server to a step, for builds or for tests, and the step gets only its tools. Fields, files and folders go into the task of the right step, a placeholder hints what to fill in ("link to the mockup"), and the step's own instruction sets how it runs.

## Change run

**~~Vibecoding~~ Clean Deepcoding.** A change goes through the steps of the schema; every step has its own chat and a tab with what it wrote. Tasks run one at a time, each with its own commit and its own diff, not one pile of edits for the whole change. So you can review all the written code thoroughly. You mark it "Done", and you can go back to any step.

## Convenient

**The ideal harness for models.** The model works at its best: every step is a separate chat with exactly its own context, its own MCP server and your rules. In the migration the agent works by your rules, with unrelated settings turned off. In the change steps your skills and MCP work.

## Economical

**Interrupted? Pick up where you left off.** Every step is saved to disk, finished work is not redone. Out of limits: pause, then continue.

## Vanilla!

**Delete ChakChak and everything stays.** Vanilla OpenSpec in your repository, no formats of its own.

## Install (macOS, Apple silicon)

1. Download `ChakChak-<version>-macos-arm64.zip` from the [latest release](https://github.com/arkuramshin/ccr/releases/latest).
2. Unpack it and drag `ChakChak.app` to Applications.
3. Open it. The app is signed and notarized by Apple.

ChakChak checks this repository for a new version once a day and offers the update in the app.
