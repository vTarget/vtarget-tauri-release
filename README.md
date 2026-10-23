# vTarget Tauri release artifacts

Distribution-only repository for the Tauri desktop app. It holds the signed
bundles and the update manifest that `vtarget-desktop`'s updater reads. No source
code lives here.

## Why this repository is separate

`vTarget/vtarget-desktop` is private, and the Tauri updater fetches its manifest
and artifacts with a plain unauthenticated HTTP request. It cannot read the
releases of a private repository without a token embedded in the application, and
shipping a token there is worse than shipping no updater at all. Publishing the
artifacts here keeps the source private and the update channel reachable.

**This repository has to be public for updates to work.** While it is private, the
updater's `check()` answers with an HTTP error, which the app treats as "no update
available" and the boot continues (see `checkUpdate` in
`vtarget-ui/src/engine/app/tauri.ts`).

## What gets published

Produced by the release workflow of `vtarget-desktop`; never uploaded by hand.

| Asset | Purpose |
| --- | --- |
| `latest.json` | Update manifest: version, notes, and per-platform URL plus signature |
| `*.nsis.zip`, `*.msi.zip`, `*.app.tar.gz` | The bundles the updater downloads |
| `*.sig` | Detached minisign signature for each bundle |

The endpoint the application reads is:

```
https://github.com/vTarget/vtarget-tauri-release/releases/latest/download/latest.json
```

## Signature verification

Each download is verified against the minisign public key pinned in
`vtarget-desktop/tauri.conf.json` under `plugins.updater.pubkey`. A release signed
with any other key is rejected, so the private key must stay out of this repository
and out of the source repositories: keep it in the CI secret
`TAURI_SIGNING_PRIVATE_KEY`.

## Never commit here

- The minisign private key, or its password.
- Unsigned bundles.
- Build output from a local machine.
