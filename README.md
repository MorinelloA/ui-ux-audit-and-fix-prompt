# UI/UX Audit and Fix Prompt

A prompt for AI coding agents that checks every screen of a web or React Native app at real phone, tablet and desktop
sizes, in light and dark, at normal and large text, and then fixes what it finds. Oddly sized fields, rows that don't
line up, labels cut off at 320px, touch targets too small to hit, colours that have drifted from the design kit: the
agent repairs them through shared components and proves each fix with before-and-after screenshots. What the app does
stays exactly as it was.

The prompt itself is in [prompt.md](prompt.md). This README is for you; the prompt is what you give the agent.

It came out of two real runs, one on a web app and one on an Expo app. The traps list at the end of the prompt records
what went wrong in them, so your run can skip those mistakes.

## What a run does

1. Reads your code, design kit and brand guide without changing anything, asks the few questions only you can answer,
   and writes a plan. Nothing is edited until you approve it.
2. Builds a capture harness in a folder outside your repo: Playwright for the browser, adb for an Android emulator.
   Every API call gets mock data, including long names, empty lists and errors. Your real backend is never called.
3. Captures every screen, state, dialog and role at every size from an untouched copy of your current code. That's
   the "before".
4. Measures each capture for overflow, clipped or cut-off text, uneven field rows, touch areas under 44×44, text under
   10px, phone inputs under 16px (iOS Safari zooms in on those), colours that aren't tokens and low contrast.
5. Checks brand compliance: kit screens side by side with your app, a static scan of the code, the brand guide's
   rules.
6. Fixes from the bottom up. Shared building blocks come first (fields, rows, selects, choice controls, columns,
   text-size caps, an icon set), with lint rules that keep code using them; then every form; then the rest.
7. Captures "after" and loops until each capture is clean or explained. Areas you marked "don't touch" get a geometry
   diff to prove they didn't move.
8. For native apps, checks on a device: text-size caps, touch areas, the keyboard, safe areas.
9. Writes tests, including a layout spec that fails on the old code, runs your CI checks until they pass, commits in
   themed batches and reports back with before/after numbers and the decisions it needs from you. It doesn't push.

## What it covers

Layout, sizing and grouping, touch targets, text scaling, light and dark themes, and compliance with your design kit
and brand guide.

It isn't a usability study, an information-architecture review, a copy edit, a screen-reader or keyboard-focus audit,
a cross-browser check (it captures in Chromium) or a performance pass. Use tools built for those.

And it won't change business logic. No field is added, removed or hidden; validation, roles, data flow and save
payloads stay as they are. It won't call a real API, create a branch or push without you.

## Is it for you?

| Your app | How it fits |
|---|---|
| Web, any framework (React, Vue, Angular, Svelte, plain HTML/CSS) | Fully. Playwright drives it. With server-side rendering (Next.js, Nuxt, SvelteKit, Remix), the server's own API calls go to a local mock server. |
| React Native or Expo | The path it was built on. The wide size matrix runs on the app's web build, and the device pass on an Android emulator or, on a Mac, the iOS Simulator. With no web build, everything runs on the emulator. |
| Flutter | The device pass carries over. Flutter's web build draws to a canvas the DOM can't measure, so the emulator harness does the work. Text caps come from `MediaQuery.withClampedTextScaling`. |
| Jetpack Compose | The emulator harness works once `testTagsAsResourceId` is on. Material's 48dp targets come from `minimumInteractiveComponentSize()`. |
| SwiftUI | Needs a Mac and the iOS Simulator. Text caps come from `.dynamicTypeSize(...)`. |

No design kit or brand guide? Write "none" for them. The agent then writes down your app's own system (type and
spacing scales, field heights, colours, shared components) in `kit.md` for you to approve, holds the app to it, and
falls back on WCAG 2.2 AA and Apple's or Google's guidelines where your app is silent.

## What you need

- The repo checked out locally on the branch you want changed, with your own work committed or stashed.
- Its dependencies installable, and its CI checks runnable, on your machine.
- Git, Node.js and the repo's package manager. If the repo doesn't have Playwright, the agent installs it (and a few
  hundred MB of Chromium) in the audit folder.
- For Android: the Android SDK, an emulator image and hardware acceleration (WHPX on Windows, an arm64 image on Apple
  Silicon). A development build also needs either a JDK for a local build or `eas login` done beforehand, because the
  agent can't sign in for you.
- For iOS: a Mac with Xcode.
- Your design kit and brand guide as files the agent can read. Export Figma frames first, or connect a Figma MCP
  server.
- Around 16 GB of RAM (emulator, bundler, Chromium and agent at once) and free ports for about five local servers.
- Hours of agent time on a plan or budget that can carry them. See [Recommended setup](#recommended-setup).

Screenshots, your kit and your brand guide reach your model provider like any other file the agent reads.

## Recommended setup

Last checked: October 2026. Model names change fast; if one below has been retired, use the provider's current pick
for long agentic coding.

For either tool: run it on your own machine (the cloud versions can't reach your emulator or devices), start in plan
mode, use the provider's top coding model at extra-high reasoning, give the agent write access to the audit folder,
and keep pushing in your own hands.

### Claude Code

| Setting | What to use |
|---|---|
| Where | The CLI, the desktop app (a Local session) or the VS Code or JetBrains extension, in an interactive session. Not Claude Code on the web. Headless `claude -p` time-limits background commands such as dev servers. |
| Model | **Opus 5.5** (`opus`, the default), which Anthropic describes as particularly good at codebase-wide migrations and audits. For the most thorough run, **Fable 5.1** (`fable`), which Anthropic positions for agent sessions that run for hours. Fable bills usage credits from the first request on Pro and Team Standard seats; on Max and Team Premium it can use up to half the weekly limit first. |
| Effort | **`xhigh`**, with `/effort xhigh` or `--effort xhigh`. Claude Code runs Opus 5.5 at `medium` by default, which is light for this. Drop to `high` to save usage. Skip `max`, which Anthropic says can overthink. |
| Plan phase | Plan mode: `--permission-mode plan`, Shift+Tab in the CLI, or the mode menu in the desktop app. |
| Long run | Approve the plan with **Yes, and use auto mode**: a classifier then approves routine actions and blocks risky ones. Auto mode allows `git push` by default, and limits you set in chat can be lost when the conversation is compacted, so deny pushes outright (below). Bypass permissions belong inside a container or VM only. |
| Audit folder | `--add-dir ../my-app-ui-audit`, or `/add-dir` in a running session. In the desktop app, open the folder that holds both the repo and the audit folder. |
| Long sessions | Current models have a 1M-token context, and compaction is safe because the plan, progress log and inventory live on disk. If you hit a usage limit, Claude Code can wait for the reset and carry on (`/rate-limit-options`). |
| Leave off | Fast mode. It bills usage credits at twice the price, and speed hardly matters over hours. |
| Attribution | Claude Code adds its own trailer to commits unless you change its `attribution` setting. `{{COMMIT_RULES}}` can't turn that off. |

From inside the repo:

```bash
claude --model opus --effort xhigh --permission-mode plan --add-dir ../my-app-ui-audit --disallowedTools "Bash(git push *)"
```

In the desktop app or an IDE, pick the model, effort and Plan mode from the menus, and deny pushes in
`.claude/settings.local.json`:

```json
{ "permissions": { "deny": ["Bash(git push *)"] } }
```

### OpenAI Codex

| Setting | What to use |
|---|---|
| Where | The Codex CLI, the IDE extension or the ChatGPT desktop app, working locally. Not Codex cloud. |
| Model | **GPT-6.1 Sol** (`gpt-6.1-sol`), OpenAI's pick for complex coding and agentic work. GPT-6 Astra (`gpt-6-astra`) is stronger, but its usage limits are tight for a run this long. Pin the model with `-m`. |
| Reasoning | **Extra High** (`xhigh`), which OpenAI recommends for long, agentic, reasoning-heavy work. Set it for Plan mode as well (`plan_mode_reasoning_effort`), since Plan mode has its own setting and defaults to medium. Max and Ultra aren't needed. |
| Permissions | Before you start, `/permissions` → **Approve for me**. A reviewer model then answers the requests to step outside the sandbox, and there will be some: Chromium, adb, the emulator and package caches all write outside the workspace. Full access belongs inside a VM or container only. |
| Plan phase | `/plan` (or Shift+Tab), then the prompt. Approve with **Yes, implement this plan**. Plan mode is an instruction to the model, not a sandbox; for a hard read-only start, launch with `--sandbox read-only` and change `/permissions` before you approve. |
| Network | Off in the sandbox by default. Turn it on for installs with `-c sandbox_workspace_write.network_access=true`. |
| Audit folder | `--add-dir ../my-app-ui-audit`. Launch inside the repo so its `AGENTS.md` loads. |
| Git | `.git` stays read-only in the sandbox, so commits come to you (or the reviewer) for approval. Push yourself. |
| Long sessions | `/goal` keeps Codex working for hours and pauses when it needs a decision. Goals are capped at 4,000 characters, so point one at the saved plan rather than pasting the prompt. Keep the machine awake (desktop app: Prevent sleep while running). |
| Plans | ChatGPT Plus allows roughly 15 to 160 GPT-6.1 Sol messages per five hours, so a full run will probably stall and wait. Pro (no five-hour limit at the time of writing) or an API key suits it better. On Enterprise and Edu, an admin has to enable GPT-6.1 Sol. |
| Windows | Codex's recommended Windows sandbox stops a command's leftover child processes when the command exits. Check that a dev server started in the background survives before the long run; if it doesn't, run Codex in WSL2 (adb and the emulator then live on the Windows side). |
| Leave off | `/fast`: double the speed for more usage. |

From inside the repo:

```bash
codex -m gpt-6.1-sol -c model_reasoning_effort=xhigh -c plan_mode_reasoning_effort=xhigh -c sandbox_workspace_write.network_access=true --add-dir ../my-app-ui-audit
```

### Other agents

Cursor, GitHub Copilot's agent mode, Gemini CLI and the like can run it as well. Use the strongest model and highest
reasoning setting you can afford, make the agent plan before it edits, give it write access to the audit folder and
keep pushes manual.

## How to use it

1. Commit or stash your own work, and check out the branch you want changed.
2. Pick an audit folder outside the repo, such as a sibling `../my-app-ui-audit`. The harness, screenshots and reports
   go there, out of reach of the repo's own tools (Metro, Jest, ESLint and TypeScript would otherwise pick them up).
3. Copy [prompt.md](prompt.md) into the audit folder. On GitHub, the "Copy raw file" button keeps the Markdown intact.
4. Fill in the `{{…}}` fields in its Fill in section (see [The fields](#the-fields)). Write "none" for any optional
   field you don't need, and leave the rest of the prompt alone.
5. Start the agent inside the repo with the setup above, in plan mode, and send
   `Read ../my-app-ui-audit/prompt.md and follow it.` Pasting the whole filled prompt works too.
6. Answer its questions: sizes, the touch minimum, text-size caps, the tablet layout, field order and a few more. Then
   read the plan, push back on anything you don't want, and approve it.
7. Let it run. Stay around for the first stretch (questions, plan, permission to download a build); after that it
   works alone. If it stops early to report progress, tell it to carry on with the plan, or set a goal with `/goal`
   (both tools have one).
8. Read the report, decide the open items and look through the before/after gallery. Then push.

## The fields

| Field | What to write | Example |
|---|---|---|
| `{{PROJECT_NAME}}` | The app's name | Acme Portal |
| `{{REPO_PATH}}` and `{{BRANCH}}` | Where the repo is, and the branch to work on. List each repo if the web and native apps live apart. | `~/code/acme-portal` on `main` |
| `{{PLATFORM}}` | Web (any framework), React Native / Expo, both, or another stack | React Native / Expo |
| `{{DESIGN_KIT_PATH}}` | The kit: handoff notes, tokens, component sheet, screen specimens, as HTML/CSS, Figma exports, images or PDF. Or none. | `~/design/acme-kit/` |
| `{{BRAND_GUIDE_PATH}}` | The brand guide, or none | `~/design/acme-brand.pdf` |
| `{{REFERENCE_PRODUCT}}` | A sibling product that already follows the kit, used for tie-breaks: a repo path, screenshots or a read-only URL. Or none. | `~/code/acme-web` |
| `{{KNOWN_PROBLEMS}}` | What looks wrong today | Fields oddly sized and placed, worst on Settings and Checkout |
| `{{EXTRA_FIXES}}` | Bugs to fix first, each as its own commit, or none | The date picker crashes on Android |
| `{{DO_NOT_TOUCH}}` | Areas whose placement must not change, or none | The dashboard's summary cards |
| `{{AUDIT_DIR}}` | A folder outside the repo | `~/code/acme-portal-ui-audit` |
| `{{COMMIT_RULES}}` | Your commit conventions | The repo's conventions; no AI attribution in commits, PRs, code or docs |
| `{{EDIT_SIZES}}` | Changes to the default size list, or none | Drop 320×568, add 412×915 |
| `{{NATIVE_CHECK}}` | How the device pass runs, or none for a web-only app | One development build on the Android emulator; iOS on my own devices |

## What you get

In the audit folder:

```
my-app-ui-audit/
  prompt.md        your filled copy
  plan.md          the approved plan
  progress.md      how far the run has got
  harness/         capture scripts; re-run a subset with PHASE, SIZES and FILTER
  evidence/
    before/        screenshots and measures for each screen, state and size
    after/
  gallery/         before and after side by side, plus the kit comparison
  inventory.md     every issue as screen | size | issue | fix | status
  compliance.md    kit and brand compliance, before and after, with conflicts for you to decide
  kit.md           your app's own system, when there is no kit
  README.md        how to re-run everything
```

It also holds the "before" copy of the app and, for native runs, the app mirrors.

In the repo, on your branch:

- themed commits, your listed bug fixes first, each batch made after your CI checks pass;
- shared building blocks (field, row, select, choice control, page and tablet columns, text-size caps, an icon set)
  and lint rules that steer code towards them;
- unit tests, updated label queries and a layout e2e spec proven to fail on the old code;
- a "Layout and brand rules" section in your README.

Then the final report: before/after numbers for every measure, device results, what it found but didn't fix, the
decisions it needs from you (each with a recommendation), and what's left for you to do, such as store graphics and an
iOS device checklist.

For scale: the two runs this prompt came from each produced about 0.5 GB of evidence (5,000 to 6,000 screenshots) and
an inventory of 130 to 150 issues. The mobile run took two days.

## Long runs

The plan, progress log and findings live in files, so a run survives compaction, a crash or a usage limit. To resume,
start a new session with the same setup, in the long-run mode rather than plan mode (the plan is already approved),
and send the same first message. The prompt tells the agent to carry on from the first unfinished step in
`progress.md`.

Keep the machine awake, because sleep kills the emulator and the dev servers mid-capture. And budget for it: a full
run uses several subscription usage windows, or a real API bill.

## Platform notes

- Windows: the "before" copy links `node_modules` with a junction (true symlinks need Developer Mode). Git Bash
  rewrites path-like arguments and line endings, and the prompt's traps cover both. For Codex, see the sandbox note
  above.
- macOS: the only place the iOS Simulator runs.
- WSL: if the agent runs inside WSL, the Android emulator and adb run on the Windows side.

## Adapting it

Some parts carry the method, so keep them: the untouched "before" copy, synthetic data in place of real APIs,
measuring instead of eyeballing, compliance checked before and after, every plan line checked against the code, and
building blocks backed by lint rules.

Others are safe to cut for a smaller run: the native pass, the compliance check when there's no kit or brand guide,
and sizes you don't support.

Found a trap the prompt doesn't warn about? Add it to the traps section and open a pull request.

## Glossary

- Specimen: the kit's reference drawing of a screen or component.
- Handoff notes: the designer's written rules that come with the kit.
- Chrome: headers, tab bars and toolbars around the content.
- Eyebrow: the small label above a group of fields.
- Pills and segments: two styles of choice control.
- Data face: the typeface the kit uses for IDs, dates and amounts, often monospaced with tabular figures.
- Gates: the checks your CI runs, such as lint, type checks, tests and builds.
- Development build: a debug build of a native app that loads its JavaScript from your machine.

## License

[MIT](LICENSE)
