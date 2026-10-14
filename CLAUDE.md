# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

This is the documentation site for Zigbee2MQTT (https://www.zigbee2mqtt.io), built with VuePress v2 (webpack bundler) and Quasar. Package manager is pnpm (version pinned in `package.json`).

## Commands

```bash
pnpm i --frozen-lockfile   # install
pnpm run dev               # dev server with hot reload; EXCLUDES the ~4000 device pages (fast)
pnpm run dev:devices       # dev server including all device pages (slow, needs 8 GB heap)
pnpm run build             # production build to dist/
pnpm run pretty:check      # lint (Prettier) — CI fails on this
pnpm run pretty:write      # auto-format
pnpm run test              # docgen tests (see below)
pnpm run docgen            # regenerate device pages + supported-devices data
```

Dev-server environment variables (read in `vuepress.config.ts`):

- `INCLUDE_DEVICE=<file name without .md>` — include just one device page in `dev` mode, e.g. `npx cross-env INCLUDE_DEVICE=TS011F_plug_1 pnpm run dev`
- `DEV_PORT=<port>` — change the dev port (default 8080)
- `DEVELOP_BRANCH=1` — build with base `/develop/` instead of `/` (see `getBase.ts`)

CI (`.github/workflows/build-deploy.yml`) uses Node 24 and runs `pretty:check`, then `build` (only on `master`/`dev`), then `test`. `master` deploys to the `gh-pages` branch and is auto-merged into `dev`.

### Tests

`pnpm run test` runs `docgen/tests/index.ts`, which runs four checks in parallel: missing device images, device image size, broken internal links, and the Notes comment markers in device pages. **`check-links` reads the built VuePress output, so run `pnpm run build` first** or it will fail. Each check is an exported async function in `docgen/tests/`; to run one on its own, call it from a ts-node one-liner, for example `npx ts-node -e "require('./docgen/tests/check-notes-commet').checkDeviceNotesCommentLines()"`.

## Architecture

- `docs/` — Markdown content. `docs/.vuepress/` holds the client config (`client.ts`), custom components and styles.
- `vuepress.config.ts` — site config. It controls which pages are built (device-page include and exclude logic), plugins (docsearch, sitemap, google analytics, component registration), and the `base` path.
- `navbar.ts` / `sidebar.ts` — top navigation and per-section sidebars. Add new guide pages here or they won't appear in navigation.
- `supported-devices-component/` — the Vue component for the `/supported-devices/` overview page. It filters a generated device list (`devices.js`), with filter state kept in the URL hash.
- `public/` — static assets, including `public/images/devices/` (device pictures).

### Device pages are generated (docgen)

`docs/devices/*.md` (~4000 files) are **generated** by `docgen/` from the device definitions in the `zigbee-herdsman-converters` npm package. The page content follows the version pinned in `package.json`.

- `docgen/index.ts` removes pages for devices that no longer exist, regenerates every device page (`generate_device.ts`), and regenerates the supported-devices list (`generate_supported-devices.ts`).
- Generated sections come from `device_page_exposes.ts`, `device_page_options.ts`, and `device_page_notes.ts`. The last one holds vendor- or model-specific boilerplate notes (for example Philips Hue reset instructions), applied by matching on the definition.
- **Only the text between `<!-- Notes BEGIN ... -->` and `<!-- Notes END ... -->` in a device page is hand-written.** Docgen keeps it across regenerations, and everything else is overwritten. Do not use h1/h2 headings inside the Notes section, except the `## Notes` heading itself. To change generated text for many devices, edit the docgen source, not the `.md` files.
- Device page file names come from `normalizeModel(model)` in `docgen/utils.ts`. Device images are resolved and downloaded via `docgen/missing_device_images.ts` (`download-missing-device-images` / `prepare-…` / `move-…` scripts).

## Code style

Prettier config (`.prettierrc`): 4-space indent, single quotes, semicolons, trailing commas, `printWidth` 150, no bracket spacing. Markdown is formatted too, so run `pnpm run pretty:write` before committing.
