# Contributor guide

## Repository scope

This repository is a Wails v2 desktop demo that recreates a WeChat login screen with Go, React, and TypeScript. Login and account switching return demonstration strings; there is no authentication service, database, or WeChat API integration.

Run commands from this repository unless a frontend working directory is specified. The sibling profile and GitHub Pages repositories have separate build workflows; their generation commands do not apply here.

## Source map and configuration

- `main.go`: Go entry point. Embeds `frontend/dist`, creates `App`, and configures the native window, asset server, startup callback, and Go bindings.
- `app.go`: Backend `App` type, startup context, and the exported `LogInSuccess` and `SwitchAccountSuccess` methods.
- `frontend/index.html`: HTML entry point with the `root` element and the `src/main.tsx` script.
- `frontend/src/main.tsx`: Mounts React `App` inside `React.StrictMode` and imports global styles.
- `frontend/src/App.tsx`: Login screen, button handlers, and result state. `App.css`, `style.css`, and `src/assets/` provide styles, fonts, and images.
- `frontend/wailsjs/`: Generated Go method wrappers, TypeScript declarations, and Wails runtime files. Regenerate these with Wails when bindings change; do not edit them manually.
- `wails.json`: Application name, output filename, author metadata, and frontend install, build, and development commands.
- `go.mod` and `go.sum`: Go module requirements and checksums. Wails is the only direct third-party Go dependency. The trailing `replace` example is commented out.
- `frontend/package.json`: Frontend dependencies and npm scripts. `vite.config.ts` enables the React plugin; `tsconfig.json` and `tsconfig.node.json` configure TypeScript checking.
- `build/`: Application icon and packaging inputs. `build/windows/` contains Windows metadata, manifest, icon, and installer scripts. `build/README.md` explains platform assets.
- `README.md` and `screenshot.jpg`: Chinese tutorial and reference screenshot. Keep documented examples consistent with intentional behavior changes.

## Startup and component data flow

1. `main()` calls `NewApp()` and passes the instance to `wails.Run` through `Bind` and `OnStartup`.
2. Wails creates the `WeChat` window at 280 × 400 and invokes `app.startup`, which stores the application context.
3. The frontend loads `index.html`; `main.tsx` mounts `App.tsx`.
4. Clicking **Log In** calls the generated `LogInSuccess(name)` wrapper with the hardcoded name `除`. **Switch Account** calls `SwitchAccountSuccess()` without arguments.
5. The wrappers call `window.go.main.App`. Wails invokes the corresponding Go method and resolves a JavaScript promise with its returned string.
6. The promise handler updates React's `resultText` state. The screen displays that result, or the name when the result is empty.

Keep the Go method signatures, generated declarations, and frontend calls consistent. The current handlers have no rejection handling. Running Vite alone does not provide the Go bridge required by these buttons.

## Set up and develop

Use Go, Node.js/npm, the Wails v2 CLI, and the native build dependencies for your operating system. Read the checked-in manifests for dependency versions; keep the CLI aligned with the Wails version in `go.mod` when upgrading.

From the repository root, check the native environment and start development:

```sh
wails doctor
wails dev
```

`wails.json` configures `npm install`, `npm run dev`, and automatic frontend development-server discovery. Development can install dependencies and regenerate bindings. Review the resulting changes.

## Build pipeline and generated outputs

From the repository root, build the desktop application:

```sh
wails build
```

Wails orchestrates frontend installation, binding generation, the frontend build, and native compilation/packaging. The frontend build script runs `tsc && vite build`, producing `frontend/dist`. Go embeds those assets into the application; packaged output goes under `build/bin`.

For frontend-only validation, run these commands from `frontend/`:

```sh
npm install
npm run build
```

Build the frontend before invoking Go compilation directly: `main.go` requires `frontend/dist` to exist. Use the target platform's supported native toolchain when validating platform-specific packaging.

`frontend/dist`, `node_modules`, `build/bin`, and `build/darwin` are ignored build outputs. The repository also ignores `package-lock.json`, so frontend installs are not pinned by a committed npm lockfile. Preserve tracked packaging inputs and generated bindings; inspect their diffs after builds.

## Validate contributions

- Inspect `git status --short` before editing and preserve unrelated work. Follow existing Go and frontend conventions; format changed Go files with `gofmt`.
- For implementation changes, run the frontend build and `wails build`. Once frontend assets exist, use `go test ./...` and `go vet ./...` for Go checks. There are currently no checked-in automated tests or frontend lint/test scripts.
- Smoke-test the desktop window, image, initial name, and both buttons. Verify the login welcome message and account-switch confirmation. Check affected platforms separately when changing native dependencies or packaging.
- For documentation-only changes, verify paths, commands, and behavior against source; a desktop build is unnecessary.
- Finish with `git diff --check` and `git status --short`. Report checks actually run and any missing native tools or network limitations.
- Keep dependency upgrades scoped and review both manifests and generated changes. Do not stage, commit, push, or publish unless requested.
