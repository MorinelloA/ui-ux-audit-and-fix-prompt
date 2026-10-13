# UI/UX Audit and Fix Prompt

A drop-in prompt for AI coding agents. It reviews every screen of a web or React Native app at phone, tablet and
desktop sizes, in light and dark, at normal and large text. Then it fixes how fields and controls are sized, placed and
grouped so the app matches its design kit and brand guide. What the app does stays the same.

The prompt is [prompt.md](prompt.md). Nothing to fill in: the agent finds what it needs in your repo and asks you the
rest.

## Use it

1. Create an empty folder beside your repo for the audit's tooling and screenshots: `../my-app-ui-audit`, where
   `my-app` is your repo's folder name.
2. Start your agent in the repo with the setup below.
3. Paste all of [prompt.md](prompt.md), appendices included, as your first message. Optionally add a line after it
   naming problems to look at first, bugs to fix first or areas that must not move.
4. Answer its questions and approve its plan. It stops twice more to show you before/after screens and ask what only
   you can decide. It takes hours, commits on your current branch, never pushes on its own, and checks CI after you
   push.

## Setup

Last checked October 2026. Run it locally; the cloud versions can't reach your emulator.

| | Claude Code | OpenAI Codex |
|---|---|---|
| Model | Opus 5.5 (Fable 5.1 for the most thorough run) | GPT-6.1 Sol |
| Effort | `xhigh` | Extra High, including Plan mode |
| Plan phase | Plan mode | `/plan` |
| Long run | Approve with "Yes, and use auto mode" | `/permissions` → Approve for me |

```bash
claude --model opus --effort xhigh --permission-mode plan --add-dir ../my-app-ui-audit --disallowedTools "Bash(git push *)"
```

```bash
codex -m gpt-6.1-sol -c model_reasoning_effort=xhigh -c plan_mode_reasoning_effort=xhigh -c sandbox_workspace_write.network_access=true --add-dir ../my-app-ui-audit
```

The Claude command blocks `git push`, which auto mode would otherwise allow. In a desktop app, make the same choices
from its menus. A full run outgrows Claude Pro and ChatGPT Plus limits; Claude Max, ChatGPT Pro or an API key suits it.
If Codex on Windows kills background dev servers, run it in WSL2.

## What you need

- A clean checkout whose dependencies install and whose CI checks run locally.
- Node.js. For Android, the Android SDK and an emulator; for iOS, a Mac with Xcode.
- Optional: your design kit and brand guide as files in or beside the repo. Without them, the agent writes down your
  app's own design system and holds the app to it.

## What you get

- In the repo: themed commits with shared components, lint rules, tests and a layout end-to-end spec.
- In the audit folder: the plan, a progress log, before/after screenshots side by side, `inventory.md` (every issue
  and its fix) and `compliance.md` (kit and brand checks).
- A final report with before/after numbers and the decisions you made.

If a run stops (a crash or a usage limit), start it the same way and it resumes from the progress log.

Built for web apps in any framework and for React Native / Expo; other stacks work with adaptation. It doesn't cover
usability, copy, screen readers or performance.

## License

[MIT](LICENSE)
