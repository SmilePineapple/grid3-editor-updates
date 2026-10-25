# AGENTS.md — reginald-updates

## What this is

The update feed for **Reginald** (formerly "Grid 3 editor" / Grid3Editor), a Windows desktop app (Python,
packaged as an exe) for editing Grid 3 gridsets. This repo holds no app source code and no user data:
the git tree is just `README.md`. **The actual content lives in GitHub Releases**, one per version, each
with two assets:

- `manifest.json` — tells the app which version is available and where to download it
- `Reginald-code-<version>.zip` — the app's Python code (older: `Grid3Editor-code-<version>.zip`)

The app's source (including `publish_update.py`, which builds the zip and manifest) lives in a
**separate repo that is not in this session**. Don't look for build tooling here.

## How the app consumes it

From `updater.py` inside the v1.2.1 zip:

- The app fetches
  `https://github.com/SmilePineapple/reginald-updates/releases/latest/download/manifest.json`
  (`MANIFEST_URL`). So **the release GitHub marks as "Latest" is the one users get**.
- Users can override the URL in the app's "Check for updates" screen; it's saved as `update_url` in
  the app's settings file.
- It offers the update if `manifest.version` > the running `VERSION`, comparing dotted integers
  (`1.10.0` > `1.9.0`).
- It downloads `manifest.url`, checks SHA-256 against `manifest.sha256`, and requires the zip to contain
  `app.py` and `version.py` at the zip root, with no absolute or `..` paths.
- **The install folder name comes from `version.py` inside the zip** (`VERSION = "x.y.z"`), not from the
  manifest. It installs to `%LOCALAPPDATA%\Reginald\code\<version>` (older exes: `...\Grid3Editor\code`).
  The launcher runs the newest version the next time the app starts.

## manifest.json schema

```json
{
 "version": "1.2.1",
 "url": "https://github.com/SmilePineapple/reginald-updates/releases/download/v1.2.1/Reginald-code-1.2.1.zip",
 "sha256": "<lowercase hex sha256 of the zip>",
 "notes": "One plain-English sentence for users."
}
```

All four keys are used. `version` must match `version.py` in the zip and the release tag (minus the `v`).

## Current releases (checked 2026-10-25)

| Tag | Assets | Notes |
|---|---|---|
| v1.2.1 (Latest) | `manifest.json`, `Reginald-code-1.2.1.zip` | URL points at `reginald-updates` |
| v1.2.0 | `manifest.json`, `Reginald-code-1.2.0.zip` | manifest URL still says `grid3-editor-updates` |
| v1.1.6 | `manifest.json`, `Grid3Editor-code-1.1.6.zip` | manifest URL still says `grid3-editor-updates` |

All three zips download and match their manifest SHA-256s. The old `grid3-editor-updates` URLs still work
only because GitHub redirects renamed repos.

## Conventions (from history)

- Tag `vX.Y.Z`; release title `Reginald X.Y.Z`; release body = the same sentence as `notes`.
- Releases are created on `main`. All tags point at the initial commit; nothing gets committed per
  release. The git tree only changes for README/docs.
- Commits are rare and terse (`Initial commit`, `Rename`). The repo was renamed from
  `grid3-editor-updates` on 2026-10-21 (v1.2.0 → v1.2.1).

## How to add a new update

1. In the **app's source repo**, bump `version.py` and run its `publish_update.py`. It writes
   `Reginald-code-<version>.zip` and `manifest.json`. (That script isn't here and hasn't been verified.)
2. Check the outputs:
   ```sh
   sha256sum Reginald-code-X.Y.Z.zip            # must equal manifest.json "sha256"
   unzip -l Reginald-code-X.Y.Z.zip             # app.py + version.py at the zip root
   unzip -p Reginald-code-X.Y.Z.zip version.py  # VERSION = "X.Y.Z"
   python3 -m json.tool manifest.json
   ```
   The manifest `url` must be
   `https://github.com/SmilePineapple/reginald-updates/releases/download/vX.Y.Z/Reginald-code-X.Y.Z.zip`.
3. Create a GitHub Release with tag `vX.Y.Z` on `main` and title `Reginald X.Y.Z`. Use the notes sentence as
   the body, upload both files, and **mark it as Latest** (not a pre-release or draft, or users won't get it):
   ```sh
   gh release create vX.Y.Z Reginald-code-X.Y.Z.zip manifest.json \
     --repo SmilePineapple/reginald-updates --title "Reginald X.Y.Z" --notes "..." --latest
   ```
   Cloud agent sessions may not have `gh` or permission to create releases. If so, give the owner the exact
   command.
4. Verify what users will see:
   ```sh
   curl -sSL https://github.com/SmilePineapple/reginald-updates/releases/latest/download/manifest.json
   curl -sSL <url from manifest> | sha256sum
   ```

The asset must be named exactly `manifest.json`, since the `latest/download/manifest.json` URL depends on it.

## Gotchas

- Publishing is a live push to every user's PC on their next update check. Never edit or replace assets on a
  published release. Ship a new version instead (the app ignores anything not strictly newer).
- Rolling back means publishing a *higher* version number with the old code; an older "Latest" is ignored.
- Never delete or rename this repo again. Old exes and saved `update_url`s depend on the URL (and on
  GitHub's redirect from `grid3-editor-updates`).
- Keep the zip code-only. Since 1.1.6 Grid 3's own sounds aren't bundled (the app finds them on each PC).
  Never include user data.
- No CI, build, tests, env vars or secrets live in this repo.
