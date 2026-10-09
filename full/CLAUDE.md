# Working in this repository

XTernal, OpenMacro's free, open source memory based macro for Roblox Fisch. AutoHotkey v2, no build step: `Main.ahk` is the entry point, `shared/` holds the macro, `ui/` the window and dialogs, `library/` third-party code. Git remote `anorexc/openmacro-xternal`, branch `main`.

## Commands

There is no compiler, so a syntax or include error only shows up when a user starts the script. Validate before every commit:

```
$p = Start-Process 'C:\Program Files\AutoHotkey\v2\AutoHotkey64.exe' -ArgumentList '/ErrorStdOut','/validate','"<repo>\Main.ahk"' -Wait -PassThru -NoNewWindow -RedirectStandardError "$env:TEMP\ahkerr.txt"; $p.ExitCode; Get-Content "$env:TEMP\ahkerr.txt"
```

- Exit 0 means it loads. Exit 2 means an error, and the file prints the line.
- Run it from PowerShell. From Git Bash the exe prints nothing and returns 2 for every file, good or bad.
- `/validate` only proves the script parses and every `#Include` resolves. It does not run anything, so it does not catch a misspelled variable inside a function or a wrong offset. Say so in the summary when a change was only validated, not run against the game.

## Scale

XTernal is free and has about ten times Swift's userbase. Every release reaches all of them through the in-app updater within hours, and there is no paid support channel to absorb a mistake. A bug that would cost Swift a few tickets costs XTernal thousands of broken installs and a flood in the Discord. Treat every change to `Main.ahk`, `shared/` and `ui/` as shipping to that many machines at once.

- The updater (`shared/Update.ahk`, `ui/Dialogs/Update*.ahk`) is how every fix reaches users. A release that breaks it strands everyone on that version with no way to ship the repair. Change it only when the task is about it, and say so in the summary when you touch it.
- `ValidateStagedUpdate` rejects a staged build whose `FULL_VER` does not match the version it was published as. Keep that check intact.
- Server calls go through the `XTERNAL_API_BASES` and `OFFSETS_API_BASES` fallback chains, because some ISPs SNI-block openmacro.net. Never call a hard-coded URL for anything the macro depends on.
- Anything that runs on a timer runs on every install: telemetry heartbeats, update checks, offset fetches. A new periodic request multiplied by the userbase is load on the API. Cache it, keep the interval long, and say what it costs in the summary.

## Every user's machine

XTernal runs on tens of thousands of different PCs, not just the one you are developing on. Code that works here but not on theirs is a defect, and you own it.

- Memory: offsets change with every Roblox update. They live in `settings/offsets.json` (loaded by `shared/Memory.ahk`) and the offsets feed (`shared/OffsetsRemote.ahk`). Never hard-code an offset in a function; add it to the offsets set. A read that returns 0 or garbage after an update must stop the macro with a reason, never click on a bad value.
- FPS: the game can run anywhere from under 30 to over 360 FPS, and it drops in busy scenes. Never count frames or tune a delay to one machine's frame rate. Base timing on elapsed time (`A_TickCount`), and wait on what memory says rather than a fixed `Sleep`.
- PC strength: low-end CPUs, laptops on battery and machines running other software deliver input and memory reads more slowly. A loop must keep up or degrade gracefully when a tick takes longer than planned.
- Display: resolution, aspect ratio, DPI scaling, windowed or fullscreen and multiple monitors all vary. Where a click has to land on screen, derive it from the Roblox window and what memory reports, never from coordinates measured on one screen.
- Internet: the connection is best effort. A failed update check, offsets fetch or telemetry ping must never stop fishing or show an error dialog. The macro keeps running on what it already has.
- Other factors: Roblox client updates, Windows versions, keyboard layouts, system locale (number and date formats), user paths with spaces or non-ASCII characters, antivirus that quarantines or slows AutoHotkey, and 32 vs 64 bit AutoHotkey installs.
- When a value was measured on one machine, say so and state how it holds on a slower or faster one. If you cannot tell, say that in the summary instead of claiming it works.

## Dead code

- Never write code that is not used by the change you are making. No scaffolding "for later", no unused helpers, no settings keys for features that do not exist.
- Never comment out code. Delete it. Git has the history.
- Never leave unused globals, functions, includes, controls, settings keys or files. When the last user of a component goes, the component and its `#Include` go with it.
- Never leave unreachable branches, empty bodies, or a `return` followed by more code.
- When you touch a file and notice existing dead code, delete it in the same change and say so in the summary. Overlooking it is a defect.
- Before finishing any task, re-read the full diff once specifically hunting for dead code and duplicated logic.

## Where things are declared

- Declare a variable at the point of first use, in the narrowest scope that needs it. Prefer locals and parameters to globals.
- Constants go in `shared/Constants.ahk` or at the top of the file that owns them. No magic numbers inside functions.
- No new globals for per-run state when a parameter or a local would do. The existing settings maps (`SETTINGS`, `MAIN`, `APPEARANCE`, `OFFSETS`) are the shared state; do not add parallel copies of them.
- A function's inputs come through its parameters. It does not reach into globals when it could be handed the value.

## Messy code

- One function does one thing. Roughly 40 lines is the ceiling. If it needs a section comment inside, split it. `GetGui` is the known exception and is not a model to copy.
- At most 3 parameters. No boolean flag parameters that switch behaviour. Write two functions.
- Return early on guard conditions. Never nest deeper than 3 levels.
- No logic duplicated in two places. The second time you write it, extract it.
- No clever one-liners or nested ternaries. Plain is better than compact.
- Names say what a thing is. `rodSlot`, not `val`, `data`, `info`, `tmp`, `result2`. Functions are verbs.
- Keep comments to a minimum. Write one only where the why is not obvious from the code: a non-obvious constraint, a workaround, a rule measured in the game. Most functions need none.
- Never write a comment that restates the code or a section banner. Delete redundant comments when you see them.
- A `TODO` names a concrete follow-up or is not written.
- Never swallow an error with an empty `catch` in the macro itself; stop with a reason the user can see. Best-effort network calls (telemetry, update checks, offset fetches) are the exception, and that is the only place an empty `catch` belongs.

## Licence headers

- XTernal is AGPL-3.0-only. Every new source file under `Main.ahk`, `shared/`, `ui/` and `tools/` starts with the standard header (copy it from any file in `shared/`). Never remove or alter an existing header, `LICENSE` or `NOTICE`.
- `library/` is third-party code under its own licences. Do not restamp it, and do not edit it unless the task is about it.

## Working style

- Do the task asked. Do not widen scope, add "nice to have" files, or introduce a library without saying why.
- Prefer editing an existing file over creating a new one.
- If a rule here conflicts with what the code already does, follow the rule and fix the code you touch. Do not imitate legacy code.
- Never use em dashes (U+2014) in any writing: code, comments, UI strings, docs, commit messages. Use a comma, colon, parentheses, a plain hyphen, or two sentences.

## User-facing text

- UI and changelog copy is written for the user of the product, who never saw the task. It never echoes, answers or narrates the prompt.
- State what happened or what to do next, as a professional product would, in as few words as that needs. Do not announce that something happens automatically, reassure the user that they need not do something, or explain how a feature works behind the scenes.
- The changelog tab (`changelog` in `ui/Gui.ahk`) names only what a user can see and use. Never mention telemetry, accounts, identity or data.
- XTernal is the free product. Upsell to Swift lives in the More Features tab only; do not add banners, popups or nags elsewhere.

## Commits

- Commit after each logical change that leaves the script loading: a feature, a fix, a dialog, a release bump. Do not batch a whole task into one commit, and do not commit half-done work.
- Small commits with an imperative subject line. No "WIP", "fix", or "misc" commits.
- Subjects follow the existing log: `type(scope): sentence in lower case`. Add a body only when the why is not obvious, and keep it to a sentence or two. No bullet lists, no file inventories, no restating the diff.
- Push all commits when the implementation is done and `/validate` passes. Do not push mid-task, and do not push a script that fails to load.
- Never leave finished work uncommitted. The release tool packages `git ls-files`, so an untracked file never ships and a modified one ships as it sits in the working tree.

## Releases

- A version bump is its own commit, `chore(release): bump to vX.Y.Z with its changelog`: `FULL_VER` in `shared/Constants.ahk`, `version.txt` (no trailing newline), and the date and `changelog` entries in `ui/Gui.ahk`, which replace the previous version's.
- Pushing is not releasing. `git push` moves nothing to users. A release is the tool in `tools/release` (`xternal release --version vX.Y.Z`, `xternal status` to see what the server has), and it refuses a version that does not match `FULL_VER`. Push first, then release from the tree that was pushed.
- Never publish a release unless the owner asks for it. Once it is up, every install updates to it.
- `tools/release/.env` holds the upload token and Cloudflare credentials. Never print, log or commit it.

## README

- `README.md` is the source of truth for what XTernal is and how it is licensed. Keep it correct.
- After any change that affects what it states, update it in the same change.
- Keep it short and factual. No roadmap, no restating what the code already documents.
