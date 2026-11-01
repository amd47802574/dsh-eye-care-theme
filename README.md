# DSH Eye Care Theme（DSH 护眼主题）

An eye-care theme for the **DeepSeek Harness (dsh) Web UI** — soft, low-glare
palettes for long working sessions, delivered in two forms:

1. **Persistent dsh bundle (recommended)** — `package.json` declares
   `dsh.bundle.patch` + `dsh.client`, so installing the package into a dsh
   profile makes the theme part of the host composition: it is served at
   `/plugins/dsh-eye-care-theme/client.js`, listed in `window.__DSH_BOOT__`,
   and **survives host restarts** (same mechanism as `@liustack/modlens`).
2. **Dynamic Cordis plugin (reference only)** — the legacy form
   (`plugin/client.js` + `plugin/host.js`), activated per-session via
   `cordis_define`/`cordis_run`; it dies with the process.

Both forms stack a token layer over the active theme — no rebuild of the
harness shell is needed.

- **Dark mode** — Community Material Theme *Palenight*: indigo `#292D3E`
  surfaces, periwinkle text `#A6ACCD`, teal accent `#80CBC4`.
- **Light mode** — warm neutral (no pure white): parchment `#F5F4F0`,
  warm grey text `#4F4B45`, teal accent `#00897B`.
- **Composer action button** — the send/stop button uses a deep-indigo fill
  with its white icon, so it stays clearly visible on the near-white input
  (fixed in v1.0.1; the icon color is hard-coded white by the app CSS).
- **Code blocks** get clear contrast in both modes.
- **~100 theme variables** covered (surfaces, tabs/headers, inputs, menus,
  text tiers, borders, buttons, interactive states, code, scrollbars).

## Preview

| Light | Dark |
| --- | --- |
| <img src="screenshots/light.jpeg" width="420" alt="Light mode"> | <img src="screenshots/dark.jpeg" width="420" alt="Dark mode"> |

> Screenshots predate the v1.0.1 composer-button contrast fix.

## How it works

The dsh web frontend exposes a `theme` client service. `overrideTokens(source,
tokens)` stacks a per-token CSS-variable layer on top of the active theme:

```js
theme.overrideTokens('dsh-eye-care-theme', {
  '--dsw-alias-bg-base': { light: '#F5F4F0', dark: '#292D3E' },
  '--dsw-alias-brand-primary': { light: '#00897B', dark: '#80CBC4' },
  // ... ~100 tokens total
})
```

The browser half (`plugin/client.js`) applies the layer and registers small UI
(run-card panel, composer status line). The host half (`plugin/host.js`)
registers two package-private RPC handlers (`demo/ping`, `demo/hello`) and a
model-visible demo tool (`demo_greet`) to prove both halves are live.

## Install (persistent bundle form — survives restarts)

The package is a dsh **bundle**: `package.json` declares `dsh.bundle.patch`
(its `cordis.patch.yml` row) and `dsh.client` (the web client bundle). Install
it so the dsh profile can resolve it:

1. **Resolve the package from the profile.** Either install it straight from
   GitHub with pnpm in the profile directory:

   ```sh
   dsh plugin --profile web add github:amd47802574/dsh-eye-care-theme
   ```

   or, for a local checkout, copy the package directory into the profile's
   dependency store and register it:
   - copy to `~/.dsh/profiles/node_modules/dsh-eye-care-theme/`
   - add `"dsh-eye-care-theme": "1.0.1"` to `dependencies` in
     `~/.dsh/profiles/web/package.json`
   - append `dsh-eye-care-theme` to `dsh.profile.bundles` in the same file
2. **Restart dsh.** The row `- id: dsh-eye-care-theme` composes into the host
   tree, and the client-modules registry adds
   `/plugins/dsh-eye-care-theme/client.js` to the boot manifest
   (`dsh.client.platform: web`, `immediately: true`).
3. **The theme applies at every boot.** Switch dsh Appearance to
   Light/Dark/System — both palettes are themed.

To remove: drop the dependency and bundle entry, then restart.

> **Activation gate.** The client bundle declares `inject: ['theme']`, so the
> plugin activates only after the `theme` service (provided by
> `@deepseek-ai/dsh-client-ui-theme`) is available. Do not remove it: with
> `inject: []` the `immediately: true` bundle runs before `ui-theme` provides
> the service, `ctx.get('theme')` is `undefined`, and the token layer silently
> never applies.
>
> **Hot updates.** `dsh-client-hmr` watches served bundle files, so replacing
> `plugin/client-module.js` (same package, new content hash) reaches the graph
> without a host restart.

## Activation (dynamic form, deprecated — dies on restart)

1. `cordis_define` a new Plugin (`kind: new`, any `idPrefix`), pasting
   `plugin/host.js` as `code.host` and `plugin/client.js` as `code.client`.
2. `cordis_run` the returned `pluginId`/`packageId` (`mode: run`).
3. Approve the browser-side `cordis/request-run` prompt (client code only runs
   after you approve it).
4. The theme applies immediately. Switch dsh Appearance to Light/Dark/System —
   both palettes are themed.

To revert: `cordis_stop <pluginId>` restores the default theme and removes all
effects (badge, cards, tools). `cordis_undefine` removes it permanently.

## Repository layout

```
dsh-eye-care-theme/
├── README.md            This file
├── README.zh-CN.md      中文说明
├── LICENSE              MIT
├── package.json         Bundle manifest: dsh.bundle.patch + dsh.client
├── cordis.patch.yml     Host row insert (- id: dsh-eye-care-theme)
└── plugin/
    ├── client-module.js Persistent bundle: browser half (token layer only,
    │                    module protocol, inject: ['theme'])
    ├── host-module.js   Persistent bundle: minimal host row (no-op)
    ├── client.js        Dynamic form: browser half (theme + demo UI)
    └── host.js          Dynamic form: host half (RPC handlers + demo tool)
```

## Changelog

- **1.0.1** — Composer send/stop button and attachment "remove" button are no
  longer white-on-white in light mode (`--dsw-alias-button-info-fill`,
  `--dsw-alias-button-info-hover`, `--dsw-alias-button-contrast-fill`).
- **1.0.0** — Initial release as `dsh-palenight-theme`; renamed to
  `dsh-eye-care-theme`.

## Notes

- Both halves are **plain JavaScript** (no TS/JSX/import). The client uses
  `React.createElement` only.
- `Theme.listTokens` exposes only 13 tokens; the frontend actually consumes
  ~100. `overrideTokens` accepts any token name (only the `{ light, dark }`
  pair shape is validated), so the full set is overridable.
- Dark palette values are taken from the Community Material Theme *Palenight*
  VS Code extension (`equinusocio.vsc-community-material-theme-1.4.7`), whose
  license governs those colors; the code here is MIT.

## License

MIT — see [LICENSE](LICENSE).
