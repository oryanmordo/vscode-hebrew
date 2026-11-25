# Hebrew and bidirectional text support

This unofficial Code - OSS branch adds Hebrew-aware editing and reading without mirroring the workbench. It applies to the shared editor and chat components used by both the normal window and the Agents window.

## Status

The source changes passed 271 repository tests in an isolated Chromium harness (two existing tests skipped), 11 Markdown-engine tests with minimal host adapters, and a strict scoped TypeScript check of the core editor/Markdown changes. Real Chromium layout checks also verified Hebrew/English list markers, quotations, table cells, and LTR code in the file preview. A full desktop build, full TypeScript check, real extension-host tests, and an authenticated agent conversation have **not** been verified: dependency installation on current upstream `main` requires access to Microsoft's VS Code npm feed. Do not treat this branch as a verified, ready-to-install application until those checks have passed.

## Behavior

- `editor.textDirection` defaults to `auto`: each logical line uses its first strong directional character. Hebrew lines render RTL, English lines render LTR, and wrapped continuations retain their logical line's direction.
- Set `editor.textDirection` to `ltr` or `rtl` to force a direction. Explicit model direction decorations still take precedence.
- Chat input uses automatic direction in both windows. Shared rendered Markdown handles paragraphs, headings, lists, quotations, and table cells independently, including streamed content.
- Fenced code and embedded chat code editors remain LTR. Hebrew inside English source code does not reverse the source line.
- Markdown file previews have independently directed table cells, LTR code blocks, and logical list/quotation spacing.
- The launcher uses separate local user-data and extension directories; it does not overwrite your installed VS Code or its profile.

Automatic direction follows the first strong character, rather than forcing every paragraph containing a Hebrew word to RTL. A paragraph beginning with an English product name is therefore LTR; force `rtl` for such a prose file if desired. This differs deliberately from Scout's presence-based heuristic so English source containing Hebrew strings remains LTR.

Example user settings for Hebrew prose:

```json
{
  "[plaintext]": {
    "editor.wordWrap": "on"
  },
  "[markdown]": {
    "editor.wordWrap": "on"
  },
  "editor.unicodeHighlight.allowedLocales": {
    "_os": true,
    "_vscode": true,
    "he": true
  }
}
```

Allowing the Hebrew locale avoids treating ordinary Hebrew as suspicious without disabling invisible-character or other Unicode safety checks.

This is text-direction support, not a Hebrew translation of menus. The integrated terminal and third-party extension webviews retain their upstream behavior. Mixed-direction visual cursor navigation, complex IME composition, accessibility, and actual agent-provider sign-in still require end-to-end desktop verification.

## Build and launch on macOS

Use the Node version from `.nvmrc`, Xcode command-line tools, Git LFS, and a current pnpm (11 or newer). The repository pins its pnpm version in `package.json`; older pnpm 9 cannot parse this upstream lockfile.

```sh
cd ~/git/vscode
nvm install
nvm use
npm install --global pnpm
pnpm install --frozen-lockfile
pnpm run build-fast
```

If dependency installation needs the authenticated feed, follow upstream's documented setup with an account authorized for that feed:

```sh
npm run setup-feed
npm run reauth-feed
pnpm install --frozen-lockfile
pnpm run build-fast
```

The feed setup changes your npm configuration; run it only when that feed is required. Never add credentials to the repository.

Launch the normal window or the Agents window:

```sh
./scripts/code-rtl.sh --new-window
./scripts/code-rtl.sh --agents
```

The first launch downloads the development Electron runtime and prepares built-in extensions through the upstream launcher. Sign in to your chosen agent provider in the application; this repository does not include credentials or guarantee access to proprietary Microsoft services or Marketplace extensions.

## Regression checks

After dependency installation and building:

```sh
pnpm run typecheck-client
pnpm exec playwright install chromium
pnpm run test-browser-no-install --browser chromium --runGlob '**/{strings,markdownRenderer,viewModelImpl}.test.js'
pnpm run test-browser-no-install --browser chromium --run vs/editor/test/common/viewLayout/viewLineRenderer.test.js --grep 'bidirectional text'
```

The Markdown extension additionally has a regression in `extensions/markdown-language-features/src/test/engine.test.ts`, to run with the upstream extension-host test workflow.

For manual acceptance in **both** windows, type Hebrew and English on separate lines, insert punctuation/numbers/niqqud and inline English, select and copy text, edit the beginning/middle/end, undo/redo, wrap a long paragraph, and test a streamed Hebrew response with nested lists, quotations, a mixed-language table, and a fenced code block. Check that copied text stays in logical order and that source code, controls, and file paths remain LTR.

## Repository and upstream updates

- `origin`: `https://github.com/oryanmordo/vscode-hebrew.git`
- `upstream`: `https://github.com/microsoft/vscode.git`
- `main`: an unmodified upstream history.
- `feature/hebrew-rtl`: the Hebrew changes, based on upstream `main`.

The personal repository is a history-preserving copy rather than a GitHub-network fork because the authenticated managed account could not fork public repositories.

Imported upstream GitHub Actions are disabled in this personal copy, as they are for a new fork by default. Enable only the workflows you intend to run after configuring their prerequisites.

Keep local work committed before merging:

```sh
git fetch upstream
git switch main
git merge --ff-only upstream/main
git push origin main
git switch feature/hebrew-rtl
git merge main
nvm install
nvm use
pnpm install --frozen-lockfile
pnpm run build-fast
# Re-run the regression checks and manual acceptance above.
git push origin feature/hebrew-rtl
```

Resolve any merge conflicts normally and re-test; future upstream changes cannot be guaranteed conflict-free. Merging rather than rebasing avoids rewriting the published feature branch.

