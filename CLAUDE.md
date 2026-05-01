# CLAUDE.md

> ⚠️ **IMPORTANT: This is a PUBLIC repository. NEVER commit or push any confidential information (API keys, tokens, credentials, personal data, internal/proprietary information, etc.).** ⚠️

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

A collection of small, self-contained browser mini-games hosted on GitHub Pages. The repo is the deployment artifact — there is **no build step, no package manager, no test runner, no linter**. Every file under `docs/` is served as-is.

Pages is configured to deploy from the `main` branch with root `/docs`. Anything outside `docs/` (e.g. `design-docs/`) is intentionally not published.

## Development workflow

This is a solo hobby repo — **commit and push directly to `main`**. Don't open PRs or feature branches unless the user explicitly asks. Each push to `main` is a deploy, so make sure the change actually works (open the affected page locally) before pushing.

### GitHub authentication in Codex

This repo is under `~/private-projects`, where GitHub credentials for `HirakuMori` are supplied by the parent `.envrc` as `GH_TOKEN`. In Codex's non-interactive shell, direnv is not always auto-loaded, so plain `gh` / `git push` may fall back to another machine-wide `gh auth` active account from the keyring.

For any GitHub-authenticated command in this repo, especially `git push` and `gh ...`, run it through direnv:

```sh
direnv exec . git push
direnv exec . gh auth status
```

Do not print or inspect the token value. It is enough to verify whether `GH_TOKEN` is set/non-empty.

## Commands

There is no toolchain. To preview locally, serve `docs/` with any static server, e.g.:

```sh
python3 -m http.server --directory docs 8000
# then open http://localhost:8000/
```

Open an individual game by visiting `http://localhost:8000/games/<game-name>/`.

## Architecture

### One game = one HTML file

Each game lives at `docs/games/<game-name>/index.html` as a **single self-contained file** with inline `<style>` and inline `<script>` — no external assets, no shared CSS/JS, no frameworks, no module bundling. Games communicate state to themselves only (a couple use `localStorage` for best scores, e.g. `loopForgeBest`). They do not share code; copy-paste between games is the accepted pattern.

This constraint is deliberate: a game can be opened directly from disk, ported elsewhere, or rewritten without touching anything else.

### Adding a game

1. Create `docs/games/<game-name>/index.html` (one file, vanilla JS, mobile-friendly viewport, link back to `../../index.html`).
2. Add a row to the `<tbody>` of the `.games` table inside `docs/index.html` with a `No.`, anchor, and short description in Japanese.

### Top page (`docs/index.html`)

A retro-BBS-styled (Win98/teal-and-navy) directory page in Japanese. The visitor counter is a cosmetic random number generated on load — not a real counter. When adding a game, just append a new `<tr>` with the next zero-padded `No.`.

### Penguin Blocks is special

`docs/games/penguin-blocks/index.html` is **not hand-written** — it is a pre-bundled artifact containing `<script type="__bundler/manifest">`, `<script type="__bundler/template">`, `<script type="__bundler/ext_resources">` blocks and an `__bundler_loading` / `__bundler_thumbnail` loader UI. Treat it as opaque output: do not edit its bundled payload by hand. Other games (21-or-curse, sushi-catch, loop-forge, post-penguin-sorter) are plain hand-written HTML and can be edited directly.

### Design docs

`design-docs/` holds non-deployed planning material (e.g. `21-or-curse-spec.md`, an MVP spec covering rules, scoring, UI, and acceptance criteria). When implementing a game from a spec there, the spec is the source of truth for game rules.

## Conventions

- **Language**: All player-facing copy is Japanese. Identifiers/variables in code are typically English.
- **Mobile first**: Games target mobile browsers with `touch-action: none`, pointer events, and large tap targets. Always keep the `<meta name="viewport">` tag and test that pointer events work on touch.
- **Vanilla only**: No build tooling, no npm dependencies, no CDN scripts. If a game needs a library, inline the minimum required code.
- **Style density**: Older games (`21-or-curse`, `sushi-catch`) use readable, expanded CSS/JS. `loop-forge` uses heavily-minified one-liners — match the style of the file you are editing rather than rewriting it.
- **Back-link**: Every game page links back to `../../index.html` so players can return to the directory.

## Intellectual property

The repo's README explicitly disclaims any affiliation with The Tetris Company. Falling-block / puzzle games here must avoid official Tetris assets (names, piece colors copied verbatim, sounds, logos). Originals only.
