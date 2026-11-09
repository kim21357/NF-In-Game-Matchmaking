---
name: honmod-github-updates
description: Publish and maintain GitHub-based updates for any Heroes of Newerth .honmod using mod.xml, GitHub Releases, mods.json, SHA-256 checksums, and Newerth Forge. Use when setting up an update source, preparing a new release, testing update detection, or troubleshooting an update.
---

# GitHub Updates for .honmod — Developer Skill

**For every HoN mod author.** This procedure is not bound to NF In-Game Matchmaking, a particular GitHub account, a person's Windows drive, or a specific game installation.

**Required client:** Newerth Forge 1.8.1 Dev with GitHub Update support. Earlier clients may ignore the new XML attributes.

## When to use this skill

Apply this workflow when a developer needs to enable GitHub updates for an existing or new `.honmod`, publish a new version, make a GitHub update index, validate integrity, or diagnose missing updates.

Inputs: an actual `.honmod` package, the public GitHub `OWNER/REPO`, a stable lowercase `updateid`, the installed version, and the intended new numeric version. Do not invent the repository, author, or download artifact.

**Do not modify live installed Mods or game files** while preparing a release. Work on a separate copy of the package. Checking updates, downloading a new archive, and applying its operations to HoN are separate steps.

## Step 1 — Prepare the public repository

One public repository can host one or many mods. Repository layout:

```text
GitHub repository (default branch)
├── README.md          optional human-facing documentation
└── mods.json          required at the ROOT of the default branch

GitHub Releases (NOT files under Code)
├── example-mod-v1.2.0 (tag)
│   └── Example-Mod_v1.2.0.honmod (attached Release asset)
└── another-mod-v2.0.0 (tag)
    └── Another-Mod_v2.0.0.honmod
```

Use a **separate release tag per mod version**. Do not rely on the repository-wide GitHub "Latest" Release for multiple mods. Publish the Release and its asset **before updating mods.json**, so users cannot discover a nonexistent download.

The client fetches `https://api.github.com/repos/OWNER/REPO/contents/mods.json`. The default branch must contain this file at root (not in an arbitrary subfolder).

## Step 2 — Add GitHub metadata in mod.xml

Open the original `.honmod` archive and update only the root `<modification>` attributes, preserving all existing `<editfile>`, `<copyfile>`, conditions, icons, sounds, and other files.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<modification
    application="Heroes of Newerth Reborn"
    appversion="*"
    mmversion="*"
    name="Example Mod"
    author="Example Author"
    version="1.2.0"
    updaterepo="OWNER/REPO"
    updateid="example-mod"
>
    <!-- Keep the mod's real editfile/copyfile operations here -->
</modification>
```

The sample above is **not a functional mod on its own**; it shows metadata to add to an existing package.

- `name`: existing mod name; must match the installed package's name when updating.
- `version`: the version actually packaged in this file. Increase it for each release.
- `updaterepo`: a public repository identifier `OWNER/REPO`, not a URL.
- `updateid`: a permanent ID for this mod, e.g. `example-mod`, matching a key in `mods.json` exactly.
- `author`: the actual creator's name, not the example name.

Current ID pattern: `^[a-z0-9][a-z0-9-]{0,79}$`. Both GitHub fields must be supplied together. When both are configured, the GitHub source takes precedence over old `updatecheckurl` and `updatedownloadurl`. A broken GitHub source does **not** automatically fall back to those legacy URLs.

**Migration requirement:** The **older installed package** must also contain the GitHub fields. Publishing a newer mod with GitHub metadata alone will not make older installations without metadata check GitHub. Provide a GitHub-enabled starting package or explicit migration instructions.

The Newerth Forge Mod Editor and Converter offer optional **GitHub Repository** and **GitHub Update ID** fields; manual XML editing is also supported.

## Step 3 — Build mods.json

Place a JSON file named `mods.json` in the repository's **default-branch root**:

```json
{
  "schemaVersion": 1,
  "mods": {
    "example-mod": {
      "name": "Example Mod",
      "version": "1.2.0",
      "releaseTag": "example-mod-v1.2.0",
      "asset": "Example-Mod_v1.2.0.honmod",
      "sha256": "REPLACE_WITH_REAL_64_CHAR_LOWERCASE_SHA256",
      "size": 12345,
      "notes": "Optional release summary"
    }
  }
}
```

**Replace all placeholders before publishing.** This sample `sha256` is deliberately invalid.

Fields:
- `schemaVersion` = 1, `mods` = object keyed by exact `updateid`.
- `name` = display string; downloaded mod identity must remain the same.
- `version` = advertised numeric version, e.g. `1.2.0`.
- `releaseTag` = exact **published** GitHub Release tag (unique per mod/version).
- `asset` = exact attached `.honmod` Release Asset filename, NOT a URL or a containing ZIP.
- `sha256` = **required** SHA-256 of final `.honmod` bytes, lowercase 64 hex characters.
- `size` = optional positive integer byte length (maximum 64 MiB in this client).
- `notes` = optional human-readable text.

To host additional mods, add distinct entries under `mods`; never erase the others.

## Step 4 — Package and calculate SHA-256

Finish and validate the new `.honmod` first. It must be a ZIP-compatible archive containing `mod.xml` at its root. Keep all mod data intact and verify the new `version`, `updaterepo` and `updateid`.

Windows PowerShell, from any folder containing the package:

```powershell
$file = ".\Example-Mod_v1.2.0.honmod"
(Get-FileHash -Algorithm SHA256 $file).Hash.ToLowerInvariant()
(Get-Item $file).Length
```

macOS/Linux:

```bash
shasum -a 256 ./Example-Mod_v1.2.0.honmod
wc -c < ./Example-Mod_v1.2.0.honmod
```

Copy the real digest to `sha256` and file length to `size`. **Never** edit/repack an archive after computing the checksum without recalculating and updating `mods.json`.

## Step 5 — Publish a GitHub Release

1. Open `https://github.com/OWNER/REPO` → **Releases** → create a new release.
2. Set its tag to the exact `releaseTag` from `mods.json` (example: `example-mod-v1.2.0`).
3. Attach the final `Example-Mod_v1.2.0.honmod` file directly under Release **Assets**. Do not attach the repository source ZIP instead.
4. Publish a normal Release (**not a draft or prerelease**).
5. Only then commit the new `mods.json` to the repository's default-branch root.
6. Confirm the Release and catalog are readable without authentication.

Do not embed GitHub passwords, personal access tokens, API secrets, or private links in XML/JSON. Public repositories do not require a user token for ordinary checks.

## Step 6 — Test update detection safely

Use a test game installation/profile or a temporary Mods directory; do not overwrite a user's existing mod merely to try the updater.

1. Install the **older GitHub-enabled** package (for example `1.1.0`).
2. In Newerth Forge go to **Custom Mod (.honmod)** → **CHECK UPDATES**.
3. Verify the Launcher finds the new version from the repository's `mods.json` and offers `UPDATE v1.2.0`.
4. If the developer explicitly approves the installation test, click **UPDATE**. The client resolves the exact GitHub Release Asset via the GitHub API, checks SHA-256 and (if present) size, verifies the package's mod identity/version, and stores a timestamped backup of the previous archive.
5. Check `version` inside the newly installed package. Run CHECK UPDATES again; expect no further update.
6. **Apply Mods** is an independent manual action when the game is closed. Checking or downloading never automatically applies the mod to HoN.

Current CHECK UPDATES is user-initiated and uses an approximately **15-minute per-repository cache**. No automatic update installation on opening the Launcher.

## Acceptance checklist

- [ ] GitHub repository is public, and matches `updaterepo` in **both older and newer packages**.
- [ ] Stable `updateid` and unchanged `name` match across versions.
- [ ] New package's `version` is higher than installed version.
- [ ] Repository root has valid JSON with `schemaVersion: 1` and matching `mods[updateid]`.
- [ ] Published Release tag exactly matches `releaseTag`; Asset filename matches `asset`.
- [ ] SHA-256 and optional size match the **published final file**.
- [ ] CHECK UPDATES finds the new version; UPDATE works only after clicking.
- [ ] Old package is backed up; no automatic game-side Apply happens.

## Troubleshooting

| Problem | What to inspect |
| --- | --- |
| No update source | Older installed `mod.xml` lacks both GitHub attributes. |
| Invalid repository / ID | Enter exactly `OWNER/REPO` and a lowercase stable ID. |
| GitHub 404 | Public repo, correct default branch, `mods.json` at root, correct release tag/asset. |
| GitHub 403 / 429 | GitHub rate limit or access issue; retry later. |
| HTTP 504 | Network/release download issue; current client uses GitHub Release Asset API. |
| Update ID not listed | `mod.xml updateid` and `mods.json` object key differ. |
| No UPDATE button | Catalog version is not newer; the client compares numeric version components. |
| Missing asset | Release tag exists but asset filename does not match `mods.json`. |
| SHA-256 mismatch | Catalog hash differs from uploaded final archive. Recalculate; do not bypass validation. |
| Identity/version mismatch | Downloaded `mod.xml` uses different name/repo/update ID/advertised version. |
| Apply required | Update downloaded the package, but game resource changes were not applied yet. |

## Current Newerth Forge 1.8.1 Dev constraints

- Public GitHub repositories only; no GitHub token embedded in the mod.
- Package download <= 64 MiB, at most 5,000 archive entries, advertised expanded size <= 256 MiB.
- Release must be published, not draft or prerelease.
- Numeric version comparison **is not full SemVer prerelease ordering**. Prefer simple versions like `1.0.0`, `1.0.1`.
- Legacy URL updates still work when no GitHub fields are present.
- No automatic download or Apply.
- SHA-256 detects archive mismatch with the catalog; it does **not** prove the publisher is trustworthy.

## Reference materials

- [Generic mod.xml example](mod.example.xml)
- [Generic mods.json example](mods.example.json)
- [Implementation notes](IMPLEMENTATION.md)
- [NF In-Game Matchmaking test case](NF-In-Game-Matchmaking-Test/README.md) — **case study only**, not a required repository, version, or folder.

**For every new mod:** substitute its actual author, repository, mod name, permanent update ID, version, release tag, final asset name, size, and digest. Never reuse the matchmaking sample identifiers for an unrelated mod.