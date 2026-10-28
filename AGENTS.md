# AGENTS.md

Notes for anyone, human or AI agent, changing this code. The README is only a title; this file describes what is
here and how to work on it.

## What this is

A playground for Electron (`package.json` describes it as "A Playground for Electron"). The current app is the
Electron "Hello World": one 800x600 window that shows the Node.js, Chromium and Electron versions and the platform.
An earlier experiment, a canvas data grid written in TypeScript, sits beside it but is not wired into the current
app. It runs wherever Electron does; the packaging config targets Windows, macOS and Linux.

## Commands

```bash
npm install          # Electron ^23.2.0 and electron-builder ^23.6.0 (dev dependencies)
npm start            # electron .
npm run build        # electron-builder: Windows portable, macOS dmg, Linux AppImage, into build/
```

`npm test` is the npm placeholder and exits with an error: there are no tests yet. `package-lock.json` is
ignored by git, so installs are not pinned.

## Layout

- `main.js`: the main process. Creates the window with `preload.js` and loads `index.html`. Quits when all
  windows close, on every platform.
- `preload.js`: fills the `*-version` spans in the page from `process.versions` and `process.platform`.
- `index.html`: the page, with a Content Security Policy that only allows scripts from the app itself.
- `render.js` and `styles.css`: empty.
- `css/styles.css`: styles used by the old page in `OldStuff/`, not by `index.html`.
- `scripts/LCTDataGrid.ts`: a canvas-drawn data grid class (title, header, scroll bars), with its compiled
  `LCTDataGrid.js` and source map. There is no `tsconfig.json`; the `.js` was compiled elsewhere and committed.
- `scripts/jquery-1.10.2.min.js`: jQuery, used by the old page.
- `OldStuff/`: an earlier version of the app ("Opus Tool experimental") that used `nodeIntegration: true` and
  `contextIsolation: false`. Its `index.html` loads `css/` and `scripts/` by relative path, so those paths no
  longer resolve from inside `OldStuff/`.
- `assets/`: a Tux icon (`.png`, `.icns`) and two arrow images.

## Rules that keep it working

- **Packaging file list.** The `build.files` list in `package.json` names `renderer.js`, which does not exist
  (the file is `render.js`), and leaves out `css/`, `scripts/` and `assets/`. Anything a packaged build needs
  must be added to that list.
- **Keep the preload model.** The current app uses a preload script and the default context isolation. Don't
  bring back `nodeIntegration` from `OldStuff/`; expose what the page needs from `preload.js` instead.
- **Edit the TypeScript, not the JavaScript.** `LCTDataGrid.js` is generated from `LCTDataGrid.ts`. Its first
  line references `node_modules/@types/jquery`, which is not in `package.json`.

## Making a change

- Run `npm start` to check a change; there are no tests.
- Update the README, which is currently only a title, if the app gains real features.
