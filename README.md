# onlyoffice-desktop

Announces the logged-in OpenCloud portal to the ONLYOFFICE desktop app, so the
portal shows up (and persists) in the app's **Clouds** list.

The desktop app only lists a connected cloud when the portal page itself calls
`AscDesktopEditor.execCommand('portal:login', ...)`. On Nextcloud and ownCloud
this is done by the ONLYOFFICE connector app. OpenCloud needs no connector for
the editors themselves — WOPI and the built-in collaboration service cover that
natively — so this small extension provides just the missing announcement.

Outside the desktop app (no `window.AscDesktopEditor` present) the extension
does nothing.

This extension was developed for the **Euro-Office desktop app**, which is based
on the ONLYOFFICE desktop editors. The `opencloud` provider it depends on (see
[Requirements](#requirements)) is currently being merged into Euro-Office's
main branch. Since the desktop apps share the same `portal:login` mechanism, it
also works with the upstream ONLYOFFICE desktop app once that app knows the
`opencloud` provider.

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

The payload deliberately mirrors the Nextcloud connector's payload — in
particular it carries no `email` field, since the desktop app's start page
compares that field against its stored portal entry and silently drops the
event on mismatch.

## Requirements

This extension is only half of the integration: **the desktop app must know
the `opencloud` provider**, i.e. have an `opencloud` entry in its
`providers/` directory. Without it, the desktop app's start page drops the
announcement as coming from an unknown provider, and this extension is
effectively a no-op.

No shipped desktop build includes that entry yet. Euro-Office is currently
merging it into its main branch, and it has also been proposed upstream to
ONLYOFFICE (ref. [ONLYOFFICE/DesktopEditors#2047](https://github.com/ONLYOFFICE/DesktopEditors/issues/2047)).
Until a release ships with it, it can be added to an installed app by dropping the
provider directory into `<install dir>/providers/` — the app enumerates that
directory at startup, no rebuild needed.

## Installation

Build the extension and drop the `dist` folder into your OpenCloud web apps
directory (see the
[OpenCloud docs](https://docs.opencloud.eu/docs/admin/configuration/web-applications)):

```sh
pnpm install
pnpm build
# copy dist/ to <WEB_ASSET_APPS_PATH>/onlyoffice-desktop
```

## Development

```sh
pnpm test:unit
pnpm lint
pnpm check:types
```

`docker-compose up -d` starts a local OpenCloud with the built extension mounted.

## License

[AGPL-3.0](LICENSE)
