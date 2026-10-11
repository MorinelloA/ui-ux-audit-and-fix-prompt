# UI/UX review and fix

## Inputs

Nothing here needs filling in. Work out each input from the checkout and list what you found at the top of the plan.
My message may also name problems to look at first, bugs to fix first and areas whose placement must not change; what
it says wins over what you find. Skip any step, row or note that doesn't apply to the platform or to what you found.

- **Repo and branch:** the repo you were started in, on its current branch. Work only on this branch. If my message
  names other repos (for example a separate web and native app), cover each.
- **Platform:** from the manifests: web (any framework), React Native / Expo, both, or another stack.
- **Design kit**, which governs screens and components: look in the repo and beside it for handoff notes, tokens, a
  component sheet or screen specimens (HTML/CSS, Figma exports, images or PDF). With none, use the no-kit rule under
  "Which source wins".
- **Brand guide**, which governs the mark, colour use, type roles, icons and its do/don't list: look the same way.
  With none, skip the brand guide checklist.
- **Native check:** the Android emulator if the Android SDK is installed, and the iOS Simulator on a Mac with Xcode.
  Otherwise, a checklist for my own devices.
- **Sizes** (defaults): phones 320×568, 360×740, 375×667, 390×844, 430×932; tablets 744×1133, 834×1194, 1024×1366,
  1194×834, 1366×1024; web desktop 1440×900, 1280×800, 1093×614 (1366×768 at 125%), 1024×768, 768×900, 700×900; plus
  one width either side of each breakpoint in the app's styles. Dark at one phone, one tablet and one desktop size.

Sizes in this prompt are CSS px on the web and pt or dp on native, never CSS pt.

## Goal

Review every screen, state and shared component at the sizes, themes and text sizes set above and in Step 2. Fix how
fields and controls are sized, placed and grouped so the product follows the design kit. Run a brand compliance check
before and after. Fix any bugs my message names. Nothing else about what the app does may change.

## How to work

- **Plan first.** Read the repo's own agent and contributor docs (for example CLAUDE.md, AGENTS.md, CONTRIBUTING.md)
  and raise any clash with this prompt in your questions. Do a read-only investigation, splitting the review in
  parallel by area if your tool can run subagents (for example one side of the app, the other side, the shared
  components and the kit). Then design the fix and write a plan for my approval before you edit anything.
- **The worked path.** This prompt was built on web apps and on React Native / Expo, captured with Playwright and adb.
  On another stack, use the equivalents and list the mapping in the plan.
- **Resume.** Once I approve the plan, save it as `plan.md` in the audit folder and log progress in `progress.md` there
  as you go. If both already exist when you start, read them and resume at the first unfinished step.
- **Ask me only what you can't decide.** Ask before planning:
  - the sizes, how the native check runs, and whether you may download or build a development build;
  - where the design kit and brand guide are, if you found none or more than one;
  - the touch minimum (44×44 by default; Material asks for 48dp, WCAG 2.2 AA for 24×24). Wherever this prompt says
    44, use the agreed minimum;
  - the text-size caps (a good default: body text to 1.5× the OS size, and dense chrome, meaning tab labels, chips,
    segments, pills and header titles, to 1.3×). Any cap under 2× trades WCAG 1.4.4 for layout stability, so say so
    when you ask. The web has no caps: large text there means a 200% root font size, and layouts must reflow;
  - the tablet layout (a good default: a centred 600pt column with full-width bars);
  - the field order (the prototype's, or today's);
  - how compound fields such as addresses are laid out at each size (for example Street / City / State + Zip on
    phones, and City 7 · State 2 · Zip 3 grid spans on the web);
  - which areas' placement must not change, if my message didn't say, and what "don't touch" means for them (it
    usually means the placement, not the sizing);
  - whether the brand guide is in scope.
- **Branches and commits.** Never create a branch or worktree without asking. Follow the repo's commit conventions (its
  agent and contributor docs, its recent history), with no AI attribution unless they call for it.
- **Which source wins:**
  - where the kit and a reference product I name differ, the kit wins; the reference product only breaks ties where
    the kit is silent;
  - the brand guide wins on its "never" rules (for example retired marks, filled icons, pure black or raw hex values)
    and wherever the kit is silent (for example a mono face with tabular figures for displayed IDs, dates and
    amounts);
  - with no kit, write the app's de facto kit to `kit.md` in the audit folder (type and spacing scales, field
    heights, radii, colours, shared components) and get it approved with the plan. Where the app is inconsistent or silent,
    follow its most common pattern, then WCAG 2.2 AA and the platform's guidelines (Apple HIG, Material 3). `kit.md`
    is then "the kit" everywhere in this prompt;
  - every other conflict goes in the compliance report with your recommendation, for me to decide.
- **Tokens only.** Where the kit is silent (tablets, touch areas, focus and error states, pickers, dialogs), extend it
  with its own tokens and pieces. List every deliberate departure from the kit for my veto.
- **No business-logic change:**
  - no field added, removed or hidden;
  - validation, roles, data flow and save payloads stay the same;
  - labels change only where they are cut off or wrong; check each change against any label-driven behaviour (for
    example a keyboard type picked from the label) and against the tests that query it;
  - accessibility roles stay;
  - prove the placement of any don't-touch areas is unchanged with a before/after geometry diff at every size.
- **Touch.** Every control gets a 44×44 touch area while keeping the kit's drawn size. On the web, use an invisible hit
  area (`::after`) scoped carefully. It can't attach to `<input>` or `<img>`, so put it on the label or a wrapper, and
  an ancestor with `overflow: hidden` clips it. On React Native, use `hitSlop` only where the parent has room (Android
  never extends a touch past the parent's bounds); otherwise make the pressable itself 44.
- **Never call a real API.** Answer every API call from synthetic data, including a long-data variant (50-character
  names, long addresses and emails, many rows), plus empty and error variants. Give the mock a switch that makes
  chosen endpoints fail. Abort every other non-local request except fonts: allow those or serve them locally, because
  fallback fonts fake truncation. Block service workers, and route WebSockets too. Request interception only covers
  pages the harness drives, so anything else that loads the app (a dev server's browser preview, your own browser,
  the emulator, server-side rendering) must reach the mock through the server's API URL. Hosted auth and backend SDKs
  (for example Firebase, Supabase, Auth0, Clerk) call their provider directly: seed the session or stub the SDK.
- **The audit folder.** Keep the tooling and evidence in a folder beside the repo (`../<repo>-ui-audit`, unless I name
  another), never in the repo, where the repo's own tools (Metro, Jest, ESLint, TypeScript, knip, Tailwind) would pick
  them up:
  - `harness/` and `evidence/before`, `evidence/after`;
  - the "before" copy of the app, and the native mirrors;
  - `plan.md` and `progress.md`;
  - `inventory.md`;
  - `compliance.md`;
  - `kit.md`, if there is no kit;
  - `gallery/`: the before and after captures side by side, and the kit comparison;
  - a `README.md` saying how to re-run everything.
- **Servers.** Start long-running servers in the background (or through your tool's preview feature), keep a list of
  what you started, and stop all of it at the end. Pin each port and fail if it is taken (for example Vite's
  `--strictPort`), so the "before" and "after" servers can't swap. Restart a server after edits if it has no watcher,
  and never edit app files mid-capture.
- **Commits:**
  - stage explicit paths only;
  - run the repo's full CI gate set locally, as its preflight script or CI config defines it, and get it green before
    committing, in one run per batch of commits. Skip deploy, release and publish jobs and anything that destroys
    local state (`git clean`, `git reset --hard`, dropping databases); report jobs that need secrets or services as
    not run;
  - green means no new failures: failures already on HEAD are reported, not fixed;
  - after the last commit, check `git status`: the gates test the working tree, but a commit holds only what's staged;
  - push only when I say;
  - if CI runs on a self-hosted runner on this machine, finish every local gate run before pushing, because a push
    starts CI jobs that compete with them for the machine.
- **Subagents.** Give any subagent that edits files these same rules, plus: it owns an explicit list of files; the
  shared building blocks are frozen for it (changes it needs go in its report); and it runs no git command that
  changes state, no servers and no gates.

## Step 1: baseline and harness

1. Start from a clean working tree on the current branch; if it isn't clean, ask. Install exactly what the lockfile pins, if
   there is one (`npm ci`, `pnpm install --frozen-lockfile`, `yarn install --immutable`). Run the full CI gates on HEAD
   so anything already red is known before any edit.
2. Make the "before" app: a `git archive` of HEAD, plus the ignored files it needs (for example `.env*` and generated
   code, pointed at the mock) and any submodules, which `git archive` leaves out. Link `node_modules` from the working
   tree (a symlink; a junction on Windows); with Yarn PnP or a workspace monorepo, install in the copy instead. Serve
   it on its own port, and the working tree on another.
3. For native, start one development build of the clean tree now (for example an EAS cloud build, or a local
   `npx expo run:android`), as agreed in the questions. Ask me before adding a native dependency later: it needs a new
   build.
4. Build the capture harness: headless Chromium (Playwright), from the repo if it has it, otherwise installed in the
   audit folder. For React Native, the harness runs the app's web build (react-native-web); if there isn't one, run
   the scenarios on the emulator only.
   - Scenarios: every route, tab, sub-panel, dialog, sheet and wizard step; every role; the empty, error and
     long-data variants; each detail screen scrolled to its end; the longest locale or a pseudo-locale if the app is
     translated, and right-to-left if it supports it.
   - Captures per state and size: the viewport, the full length (height set to the main scroller's scrollHeight,
     capped) and the end of the scroll, with the measures as JSON. Take the measures at the real viewport; the
     full-length shot is only for looking at, because a tall viewport stretches `vh` and `dvh` layouts.
   - Options to run a subset: `PHASE`, `SIZES`, `FILTER`.
5. For native, build an emulator harness:
   - adb, uiautomator dumps, taps by testID, text or content-desc, and deep links;
   - display configs: width through density or size, font scale, dark mode;
   - two app mirrors (before and after), pointed at a local mock server through `adb reverse`, with the emulator in
     airplane mode;
   - on a Mac, the iOS Simulator the same way: `xcrun simctl` for installs, deep links (`openurl`), dark mode
     (`ui … appearance`) and text size (`ui … content_size`), with a UI driver such as Maestro or idb for taps. The
     Simulator has no airplane mode and needs no `adb reverse`: it shares the Mac's network, so point the mirrors at
     the mock and check the mock's log for stray calls.

## Step 2: capture "before"

Capture every scenario at every size, plus dark, from the "before" app. On native, capture about 40 key screens:

| Config | Width | Text size | Theme |
|---|---|---|---|
| Small phone | 360dp | 1.0×, 1.5× | light |
| Default phone | 411dp | 1.0×, 1.3×, 1.5×, 2.0× | light, and dark at 1.0× |
| Wide window | 800dp | 1.0× | light |
| Tablet window | tablet size | 1.0×, and dark at 1.5× | light, dark |

On the iOS Simulator, use the matching device sizes and Dynamic Type steps.

## Step 3: measures (on every capture, visible elements only, because routers keep hidden screens mounted)

- **Layout:** overflow past the screen; controls clipped by an ancestor (scroll containers don't count); labels or
  button text cut off (measure the text, not just the control); inputs too narrow or truncating their value; tables
  wider than their card.
- **Rows and fields:** more than two fields in a row, or unequal halves, unless the kit draws them that way; tops not
  aligned; gaps other than the kit's; single-line fields of different heights on one screen; field edges drifting
  from the column (nested insets); empty grid columns.
- **Touch:** areas under 44×44, skipping disabled controls and links inside running text (WCAG 2.5.8 exempts them).
  Tap from the centre outward with `elementFromPoint`; inside shadow DOM it returns the host, so recurse into its
  `shadowRoot`. On react-native-web, read `hitSlop` from the React props and clip it to the ancestors as Android does.
  Skip controls under fixed chrome at that scroll position. Also flag controls covered at their centre (floating
  buttons, sticky bars).
- **Text:**
  - any text under 10px fails;
  - field text below the kit's size is flagged;
  - on web phones, inputs under 16px are flagged, because iOS Safari zooms in on smaller ones.
- **Brand:**
  - colours that aren't tokens;
  - contrast under 4.5:1 for body text, or under 3:1 for large text and for icons, borders and focus rings (the kit's
    own low-contrast pairs are reported separately);
  - displayed IDs, dates and amounts not in the kit's data face, if it has one;
  - icons that break the kit's icon rules (for example icon fonts, or filled icons in a line set).
- **Tablets:** the content sits in the chosen tablet layout, and floating pieces (for example toasts and chat buttons)
  sit inside it.
- **Errors:** page errors and crashes.

## Step 4: brand compliance check, before and after, into `compliance.md`

1. **Screen by screen.** Render or export each kit screen at its own size, light and dark (an HTML/CSS kit renders
   offline; Figma frames export). Capture the matching app screen with the kit's own sample data, before and after.
   Put them side by side in an HTML page and in image sheets. Every visible difference becomes an item: fixed, kept
   with a reason, or for me to decide. With no kit, check instead that each shared component looks the same on every
   screen.
2. **Rendered checks:** the brand measures from Step 3.
3. **Static scan of the code**, counted before and after:
   - raw colour literals outside the tokens (Tailwind arbitrary values such as `bg-[#0a0a0a]` count);
   - on native, text styles with no font family (CSS inherits it, so skip this on the web);
   - font weights on faces that already carry their weight;
   - radius and size literals off the kit's scale (including arbitrary values such as `text-[13px]`);
   - icon-font imports and glyphs that break the icon rules;
   - gradients, if the kit has none;
   - library widgets used outside the kit wrappers;
   - token values repeated in theme files.
4. **Brand guide checklist**, if there is a guide:
   - the mark: its sizes and clear space;
   - retired marks, if the guide retires any, anywhere: app icon, adaptive icon and its background, favicon, store
     graphics, splash in light and dark, unused legacy logo files (search for references before deleting);
   - the icon family rules;
   - colour roles and type roles;
   - plain language in errors and empty states.
5. **The kit's handoff rules**, as a checklist, if it has them.
6. **Conflicts.** List the known conflicts, each with your recommendation.

## Step 5: shared building blocks first

Each new piece lands in the same commit as its first user, so any dead-code check (for example knip) stays green.

- **Text and chrome:**
  - text-size caps, if any (on React Native, through wrapper components, because React 19 ignores `defaultProps` on
    function components);
  - the kit's header and tab bar.
- **Brand and theme:**
  - one icon set built from the kit's and the guide's own icons, with any missing glyph drawn to match;
  - the brand assets;
  - the UI library's theme built from the tokens.
- **Layout:**
  - the page column and the chosen tablet layout;
  - safe areas;
  - floating pieces (for example toasts) clear of the tab bar and other fixed bars;
  - touch constants.
- **Controls:**
  - buttons whose labels wrap;
  - a choice control: a segmented control when the labels fit, pills otherwise. On the web, prefer CSS that wraps on
    its own (flex-wrap, container queries). On React Native, which can't measure text before layout, the fit rule
    counts letter widths, calibrated on a device;
  - chips that wrap.
- **Fields:**
  - one field component (for example text, notes, masked date, read-only);
  - one select;
  - field rows: pair two fields only when both labels fit at the current text size, and always as equal halves unless
    the kit draws otherwise;
  - section eyebrows.
- **Containers and keyboard:**
  - dialogs and search;
  - keyboard-aware scrolling;
  - a save area that keeps Save clear of the last field;
  - any upload pieces.
- **Lint:** rules that ban the raw library pieces outside their wrappers, each message naming the kit piece to use (on
  React Native, for example, ban `allowFontScaling={false}`).

## Step 6: every form under the kit's rules

Write a table per form: its eyebrow groups, rows, pairs and controls. Keep the field order unless I decide otherwise.

## Step 7: the rest

- The key detail screens as the kit draws them.
- Lists, chrome and the remaining screens.
- Hard-coded colours moved to tokens.
- Copy changed only where it is wrong.

## Step 8: the bugs I named

Fix each one with tests that fail on the old code. Each is its own commit, first in the list.

## Step 9: capture "after" and review every capture

- Log every issue in `inventory.md` as screen | size | issue | fix | status.
- Fix, re-capture the affected scenarios with `FILTER`, and repeat until every capture is clean or the issue is
  explained.
- Re-run the compliance check.
- Before saying anything is done, check every line of the plan against the code. Earlier runs of this prompt slipped
  items logged as "fixed" that weren't, or close-enough values.

## Step 10: native pass (if mobile)

Run the matrix again on the same build, then check each of these:

- **Text size:** labels stop growing at the caps, if any.
- **Touch:** a tap halfway between a control's drawn edge and the edge of its 44 area still works, on each side where
  the control is drawn smaller than 44. Cover, for example, a segment, a chip, a pill, a delete button, an inline
  action, a notice's action and the password eye.
- **Keyboard:**
  - on long forms, the focused last field sits above the keyboard and the primary action stays reachable;
  - test a docked keyboard and a floating or hardware keyboard;
  - open and close the keyboard several times, closing it with its own button, with Back and with a tap outside.
- **Edges and brand:** nothing sits under the status bar or the gesture bar, the status bar follows the theme, and
  icons are crisp at every density.
- **The fixes:** each bug I named, on the device, and the main flows for each role (for example sign-in through the
  form).
- **iOS:** on a Mac, run what you can on the Simulator. Write a checklist for my own devices: Dynamic Type up to AX3,
  Bold Text, dark, iPad orientations and Split View.

## Step 11: tests and docs

- Unit tests for every new building block and fit rule, including tables across widths and text sizes.
- Update the tests that query changed labels.
- An e2e layout spec that steps the main screens through 320, 390, 834 and 1194×834 and fails on the Step 3 problems.
  Run it against the "before" app first, to prove it fails there. Use the repo's e2e runner; if it has none, ask
  before adding one, or keep the spec in the audit folder.
- A README section, "Layout and brand rules": the sources and which wins, the column and tablet layout, pairs, the
  touch minimum, text caps, the icon set, the kit pieces and the lint rules.
- Any coverage thresholds still pass.

## Step 12: commits and report

- Themed commits in a sensible order, with the bugs I named first. Gates green before committing.
- Stop the servers and the emulator.
- Report:
  - before/after numbers for every measure;
  - the device results;
  - what was found but not fixed;
  - the decisions for me, each with your recommendation;
  - what is mine to do: store graphics, the iOS checklist, a new build to see native assets;
  - links to the gallery, the kit comparison, `compliance.md` and `inventory.md`.
- Push only when I say.

## Traps from earlier runs (check for each before it costs you a re-run)

**Process**
- Grep every plan line before calling the plan done.
- Measure the text inside a control, not just the control. A segmented control passed every control check while its
  label ran 21px past its button.
- Fit rules must count letter widths (m and w are wide; i, l and t are narrow) and be calibrated on a device.
  Character counts are wrong.
- A dialog's exit animation leaves a remnant in the DOM. Wait for it before measuring the next state.
- Set the viewport before `goto`, and seed lists so they don't show the empty state.
- Dev servers without a watcher, or with a stale one (Metro under `CI=1`, Vite on Windows), serve old code. Restart
  them, and re-capture only what changed.
- Check that the brand fonts loaded (`document.fonts`) before any capture. A baseline captured in fallback fonts shows
  truncation that isn't there.
- Headless Chromium hides scrollbars. Run one pass with them shown: real ones take about 15px on Windows and Linux.
- Hide dev overlays (Expo LogBox, framework error overlays) in captures.

**Web CSS**
- If the app vendors the kit's CSS, those verbatim ports are not yours to edit. Override them in your own sheet, and
  mind the cascade: any `!important` rule beats normal rules; between two `!important` rules, specificity decides,
  then order, and the order reverses across `@layer`.
- Grid child rules such as `.grid:not(.x) > .field` beat a later `> *` rule.
- An invisible hit area on a class that also styles clickable table rows (`tr.link`) covered whole rows and ate their
  buttons' taps. Scope it to the elements you mean.
- Vite refuses files resolved through a linked `node_modules` (fonts, for example) until you allow the path in
  `server.fs.allow`.

**React Native** (seen on Expo SDK 57, React Native 0.86 with the New Architecture, React Native Paper 5, Reanimated 4,
react-native-screens 4 and react-native-keyboard-controller 1.21; check them against your versions)
- On Android, `maxFontSizeMultiplier` caps the font size but not `lineHeight` or letter spacing, so lines get looser
  above the cap.
- Never put letter spacing on a shrink-wrapped button label: Android dropped the last letter at some densities.
- React Native Paper hides a floating label until it is measured. Wait for two identical dumps before capturing on a
  device.
- Keyboard-aware scroll views keep a multi-line input's caret in view, not the whole box.
- Riding the keyboard:
  - `KeyboardStickyView` (react-native-keyboard-controller) adds the keyboard height to a fixed offset, so under a
    floating keyboard (whose inset is roughly the navigation bar) it pushes the view down. Clamp the rise to
    `max(0, keyboard − what the view already clears)`.
  - Don't re-render the screen when a docked keyboard shows or hides. A render as it closes can leave a Reanimated
    transform stuck at the raised value.
- Never set state after navigating away. With react-native-screens' removal transition and Fabric's view flattening,
  that crashes with "addViewAt … already has a parent". A view that dims while disabled needs `collapsable={false}`.
- A dead-code check flags a dependency the app no longer imports even when a UI library loads it at runtime (for
  example `@expo/vector-icons` for React Native Paper's own icons). Whitelist it with a reason; don't remove it.
- When the "before" copy and the working tree share `node_modules`, start Metro with `--clear`.

**Android emulator (headless)**
- `screencap` comes out blank: use the emulator console's screenshot (`adb emu screenrecord screenshot`).
- uiautomator can find no window mid-transition: retry.
- A stray Back on a root screen closes the app.
- Reloading inside the dev client leaks memory: cold-start instead.
- A snapshot boot can come up without your app: reinstall it.
- The first typing after boot can be dropped: type in small chunks and check the field.
- An emulated hardware keyboard makes Gboard show only a floating bar.

**Git**
- Pass paths with brackets or parentheses (file-based routers such as Next.js, SvelteKit, Nuxt and Expo Router) through
  `--pathspec-from-file`, with `:(literal)` entries.
- `git add -p` is interactive, so split one file across commits with a hunk patch and `git apply --cached`.
- A gate run tests the working tree; a commit holds only what's staged. A forgotten new file or deletion means CI
  tests a different tree from the one you checked, so run `git status` after the last commit.
- A push can fail silently behind a filtered pipe: confirm the remote ref.

**Windows**
- Git Bash `sed -i` rewrites CRLF files as LF, and an inserted line can leave mixed endings that fail prettier.
- Git Bash rewrites arguments that look like paths (`/settings`) into Windows paths: set `MSYS_NO_PATHCONV=1`.
- Stopping a dev server needs a tree kill (`taskkill /T /F`), or its child processes keep the port.

**macOS**
- BSD `sed -i` needs an empty suffix: `sed -i ''`.

**Brand**
- A retired mark hides in the app icon, the adaptive icon's background, the favicon, the store graphics, the splash
  (light and dark) and unused logo files.
- The brand guide's dark-background treatment of the mark can differ from the icon. Check it before matching the
  splash to the icon.
- The kit's faintest ink is often the guide's placeholder colour. Labels set in it fail contrast.
