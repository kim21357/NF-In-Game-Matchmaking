# .honmod GitHub Update — Implementation Reference

**Status:** Implemented in Newerth Forge 1.8.1 Dev (2026-11-09), including a successful real GitHub release/download test. This is a technical reference, not the step-by-step publishing procedure. **Developers should start with [SKILL.md](SKILL.md).**

## Public author contract

- Add `updaterepo="OWNER/REPO"` and `updateid="stable-mod-id"` to the root `<modification>` of a mod's `mod.xml`. Do not change its existing mod actions or the identity across versions.
- Create `mods.json` under the public GitHub repository default-branch root, with `schemaVersion: 1` and per-mod `mods[updateid]` entries.
- Each entry identifies `name`, `version`, exact published `releaseTag`, attached `asset`, required 64-character lowercase `sha256`, optional `size` and `notes`.
- Publish the Release and asset **before** pushing a catalog that advertises them; otherwise users can see an unavailable update.
- The starting/installed mod must already have `updaterepo` and `updateid`. Existing packages with no update metadata require an explicit migration.
- The legacy fields `updatecheckurl` + `updatedownloadurl` still work when the GitHub fields are not configured.

Example-only metadata and index: [mod.example.xml](mod.example.xml) / [mods.example.json](mods.example.json).

## Actual implementation

- `honmodGithubUpdate.js` reads repo-local `mods.json` through GitHub Contents API, caches repository indices for 15 minutes, validates metadata, resolves the exact Release Asset by tag and ID using the GitHub Releases API, downloads via the Release Asset API, verifies SHA-256 and optional size, rejects unexpected mod identity/version, validates archive structure, and backs up the old archive before replacement.
- `honmodManager.js` reads `updaterepo`/`updateid` and dispatches the existing CHECK UPDATES / UPDATE actions to either GitHub or the legacy URL updater.
- `modEditor.js`, `modEditorUI.js` and `honmodConverter.js` expose optional GitHub Repository + Update ID inputs.
- `src/renderer/31-honmod-workspace.js` retains existing buttons and adds update-source/error information plus a persistent Apply-required flag after package replacement.
- `src/main/70-honmod-settings.js` blocks updating while the game is running.
- `scripts/test-github-honmod-update.js` runs local network-mocked tests; integration test assets remain isolated in `NF-In-Game-Matchmaking-Test/`.

## Verified behavior

A public GitHub test repo successfully served a starter v1.0.19 and a metadata-only v1.0.20 release. A separate test directory verified: CHECK found newer version, UPDATE downloaded the actual GitHub asset through the API, the hash matched, old package was backed up, new package installed, and second CHECK found no update. This example does **not** require any other developer to use that repo or filename.

## Limits and compatibility

- Public repositories only. No credentials stored in mods.
- Download max **64 MiB**; archive max **5,000** entries and advertised expanded size **256 MiB**.
- GitHub Releases must be published (not draft/prerelease).
- Version comparison is numeric-component based, not complete prerelease SemVer.
- GitHub wins when both GitHub attributes are present. Invalid GitHub configuration does not silently fall back to legacy.
- CHECK UPDATES is manual. UPDATE only runs on user click. Apply Mods remains separate and manual.
- SHA-256 validates the final asset against the index but does not authenticate the publisher.
- The ZIP/.honmod format, game installation directory and author tools can differ per computer; **never hardcode another developer's local path into public instructions**.

See [SKILL.md](SKILL.md) for the actual release workflow and troubleshooting.