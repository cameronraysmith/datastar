# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Datastar is a lightweight hypermedia framework for building reactive web applications using declarative `data-*` attributes in HTML.
The framework communicates via Server-Sent Events (SSE) from backend to browser, and JSON payloads from browser to backend.

## Build commands

Build the library (generates bundles in the repo-root `bundles/` directory):
```bash
cd library && pnpm i && pnpm build
```

Note: `library/package.json` is not included in the public source release. The build commands above require the full private repository.

Or via task runner:
```bash
task library
```

Lint TypeScript files:
```bash
npx biome check library/
npx biome check --write library/  # with auto-fix
```

## SDK test suite

The `sdk/test/` directory contains shell-based SDK conformance tests, while `sdk/tests/` contains Go-based tests.

Run shell tests against an SDK server:
```bash
cd sdk/test && ./test-all.sh http://localhost:7331
```

Run Go tests:
```bash
cd sdk/tests && go test -v
# Or against custom server
TEST_SERVER_URL=http://localhost:8080 go test -v
```

## Architecture

### Library structure (`library/src/`)

The core library uses a plugin architecture with three plugin types:

**Engine (`engine/`)** contains the core runtime:
- `engine.ts` - Plugin registration (`attribute`, `action`, `watcher`), DOM mutation observer, expression parser for `$signal` syntax
- `signals.ts` - Reactive signal system with computed values and effects
- `consts.ts` - SDK constants (event names, default values, header keys)
- `types.ts` - TypeScript type definitions for plugins and contexts

**Plugins (`plugins/`)** implement framework functionality:
- `attributes/` - DOM attribute handlers (e.g., `data-bind`, `data-on`, `data-text`, `data-show`, `data-signals`)
- `actions/` - Callable actions prefixed with `@` (e.g., `@post`, `@get`, `@setAll`)
- `watchers/` - Global event handlers for SSE events (`patchElements`, `patchSignals`)

**Bundle sources (`library/src/bundles/`)** define different build targets:
- `datastar.ts` - Full bundle with all plugins
- `datastar-core.ts` - Minimal bundle without action plugins
- `datastar-aliased.ts` - Bundle with `ds-` prefix aliasing

The build compiles these into the repo-root `bundles/` directory as `.js` files with source maps.

**Utilities (`utils/`)** provide shared helpers used across the engine and plugins.

### Path aliases

TypeScript uses these import aliases defined in `library/tsconfig.json`:
- `@engine` → `./src/engine/engine.ts`
- `@engine/*` → `./src/engine/*`
- `@plugins/*` → `./src/plugins/*`
- `@utils/*` → `./src/utils/*`

### SDK specification (`sdk/`)

The `sdk/ADR.md` defines the Architecture Decision Record for implementing SDKs in other languages.
Key concepts:

- `ServerSentEventGenerator` - Core class for sending SSE events
- `PatchElements` - Send HTML to morph/replace DOM elements
- `PatchSignals` - Update reactive signals using JSON Merge Patch (RFC 7386)
- `ExecuteScript` - Inject and execute JavaScript
- `ReadSignals` - Parse incoming signals from GET query params or POST body

Configuration constants are in `sdk/datastar-sdk-config-v1.json` (with companion `.schema.json`).

Language SDK implementations exist under `sdk/` for: clojure, dotnet, go, haskell, java, php, python, ruby, rust, typescript, and zig.

### IDE tooling (`tools/`)

- `vscode-extension/` - VS Code extension for `data-*` attribute completions
- `intellij-plugin/` - IntelliJ plugin (Gradle-based)

## Key conventions

Expression syntax in attribute values:
- `$signalName` accesses reactive signals (transformed to `$['signalName']` internally)
- `@actionName()` calls registered actions
- Escaped values use `$$` prefix/suffix

SSE event types:
- `datastar-patch-elements` - DOM manipulation
- `datastar-patch-signals` - Signal store updates

Element patch modes: `outer` (default), `inner`, `replace`, `prepend`, `append`, `before`, `after`, `remove`

## Contributing

Pull requests go to the `develop` branch.
The project is maintained in a private repo; this public repo contains versioned source releases.
A GitHub Actions workflow (`enforce-branch-policy.yml`) enforces the branch targeting policy.
