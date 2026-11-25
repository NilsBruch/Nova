# Nova — Claude Code Guide

## What This Project Is

Nova is a UI skin for the [Decent DE1](https://decentespresso.com/) espresso
machine, running on the **Decent.app** (Decaid) gateway. It is a Vue 3 app built
with Vite into one self-contained `index.html`. The repo is an npm-workspaces
monorepo (`packages/*`): a headless shared core and the skin.

The older NSX skin is deprecated and lives, frozen, in `NilsBruch/NSX`.

## Project Structure

```
packages/
  core/                 # shared, DOM-FREE package (SOURCE OF TRUTH)
    README.md           # full NSXCore API docs — read this for core
    src/                # config, api, core, store, push, domains/*
    test/               # node --test unit tests
  nova/
    index.html
    vite.config.js      # single-file build; writes dist/manifest.json
    src/
      main.js           # boot order
      views/            # Espresso, Diary, Steam, Hot water, Cleaning, Settings
      components/       # incl. mobile/ (phone layout) and settings/
      composables/      # state and skin logic (use*.js)
      i18n/en.js        # UI strings (English)
      styles/           # app.css, phone.css
      core/             # GENERATED copy of packages/core/src (git-ignored)
scripts/sync-core.mjs   # packages/core/src -> packages/nova/src/core
tests/mock-gateway/     # gateway stand-in, see its README
```

**Edit core only in `packages/core/src`**, then `npm run sync-core`.

## Commands

- `npm test` — unit tests for the core
- `npm run dev:nova` — Vite dev server (port 5173), talks to the gateway on 8080
- `npm run build:nova` — production build into `packages/nova/dist`
- `npm run dev:gateway` — build and serve against the mock gateway (port 8080;
  `PORT=8090` plus `?gateway=http://localhost:8090` if the real app owns 8080)

## Releases

Push a tag `nova-v<version>` on `main`. The workflow builds and publishes
`Nova-v<version>.zip` as a GitHub release of this repository. Decaid installs and
updates Nova from the repository's **latest** release, so every release here must
be a Nova package with skin id `Nova-skin`.

## Core-first rule (for every new function)

1. **Check core first** (`packages/core/README.md`, `grep -rin <concept> packages/core/src`).
   If it exists, use `NSXCore.<fn>` — never reimplement it.
2. **Pure logic** (data transformation, business rule, formatting, gateway call)
   goes in **core**, with a test in `packages/core/test`.
3. Only **DOM-fused / UI-shaped** code goes in the skin.

## Key Conventions

- **Recipes** are the one source of truth for the Diary and the lists, and only
  the user creates them. A recipe is bean + grinder + profile (ids where known).
- **Shot API:** only write `annotations` (not `metadata`/`shotNotes`). `extras`
  merges key by key — to clear a key, send it as `null`.
- **Cross-device freshness:** list reads go through `getWithEtag` in `api.js`.
- **Settings** are per-field KV keys (`nova_*` in the `Nova` namespace); never
  write the whole object at once.

## Working With Claude on This Project

1. **Ask, don't assume.** If intent or architecture is unclear, ask first.
2. **Simplest solution first.** No abstractions that weren't requested.
3. **Don't touch unrelated code.**
4. **Flag uncertainty explicitly.**
5. **Suggest better approaches** when there is one with longer-lasting value.
