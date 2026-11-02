# euro-office-desktop

An OpenCloud web extension that announces the logged-in portal to the
[Euro-Office desktop app](https://github.com/Euro-Office/desktop-apps), so the
portal shows up (and persists) in the app's **Clouds** list.

The desktop app only lists a connected cloud when the portal page itself calls
`AscDesktopEditor.execCommand('portal:login', ...)`. On Nextcloud this is done
by a connector app. OpenCloud needs no connector for the editors themselves —
WOPI and the built-in collaboration service cover that natively — so this small
extension provides just the missing announcement.

Outside the desktop app (no `window.AscDesktopEditor` present) the extension
does nothing.

> **ONLYOFFICE desktop app:** Euro-Office is based on the ONLYOFFICE desktop
> editors and shares the same `portal:login` mechanism, so this extension
> should work there as well. The ONLYOFFICE app will, however, first need the
> `opencloud` provider added in the same way Euro-Office did (proposed in
> [ONLYOFFICE/DesktopEditors#2047](https://github.com/ONLYOFFICE/DesktopEditors/issues/2047)).
> Until then it is untested and unsupported there.

## Status

The desktop-side half of the integration — the `opencloud` connection provider
— has been merged into Euro-Office in
[Euro-Office/desktop-apps#42](https://github.com/Euro-Office/desktop-apps/pull/42)
(merged 2026-11-02). OpenCloud is therefore supported at the Euro-Office level:
builds from `main` onward list OpenCloud in the connect dialog and accept the
announcement from this extension.

## How it works

On startup, when running inside the desktop app's browser shell, the extension
waits for the authenticated user to land in the runtime's user store and then
announces the portal once:

```js
AscDesktopEditor.execCommand(
  'portal:login',
  JSON.stringify({
    displayName, // user's display name, falling back to the account name
    domain, // window.location.origin
    provider: 'opencloud'
  })
)
```

The payload deliberately carries no `email` field: the desktop app's start
page compares that field against its stored portal entry and silently drops
the event on mismatch.

## Requirements

- An OpenCloud instance where you can install web extensions.
- A Euro-Office desktop build that includes the `opencloud` provider
  (`common/loginpage/providers/opencloud/`, added in
  [#42](https://github.com/Euro-Office/desktop-apps/pull/42)). Without it, the
  start page drops the announcement as coming from an unknown provider and this
  extension is a no-op.

For an older installed build, the provider can be added by copying that
directory into `<install dir>/providers/` — the app reads the providers
directory at startup, so no rebuild is needed.

## Installation

Build the extension and drop the `dist` folder into your OpenCloud web apps
directory (see the
[OpenCloud docs](https://docs.opencloud.eu/docs/admin/configuration/web-applications)):

```sh
pnpm install
pnpm build
# copy dist/ to <WEB_ASSET_APPS_PATH>/euro-office-desktop
```

## Development

```sh
pnpm test:unit
pnpm lint
pnpm check:types
```

`docker-compose up -d` starts a local OpenCloud with the built extension
mounted.

## License

[AGPL-3.0](LICENSE)
