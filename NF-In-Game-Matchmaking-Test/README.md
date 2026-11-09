# NF In-Game Matchmaking — GitHub Update Test Kit

A **portable example** for mod creators who want to publish `.honmod` updates through a public GitHub repository and let Newerth Forge check for a newer version.

**This is a test release kit, not a new gameplay version.** The example `v1.0.20` has the same UI edits, scripts, icon, and sound as `v1.0.19`. Only the version number and GitHub update metadata differ.

You do **not** need a particular Windows drive letter, a specific game installation folder, or the original developer's computer. Python 3 is required only when rebuilding this example package; people installing an already published `.honmod` do not need Python.

## What's in this folder

| File | Purpose |
| --- | --- |
| `source-original-v1.0.19.honmod` | Unmodified input for reproducible builds; **not** the GitHub-enabled mod |
| `NF-In-Game-Matchmaking-v1.0.19.honmod` | GitHub-enabled starting version; import this to test detection |
| `NF-In-Game-Matchmaking-v1.0.20.honmod` | Test-only newer version to upload as a GitHub Release asset |
| `mods.json` | Update catalog placed at the root of the GitHub repository's default branch |
| `checksums.sha256` | SHA-256 checksums of the two built packages |
| `prepare_release.py` | Rebuild packages and the catalog with **your** GitHub repository |
| `validate_kit.py` | Verify the catalog, checksums, and contents without a game or Launcher installation |
| `release-notes.txt` | Suggested text for this test Release |

The initial generated assets refer to the placeholder repository `OWNER/NewerthForge-Honmods`. **Do not upload the placeholder builds as your final release.**

## Step 1 — Create a GitHub repository

Create a **public** GitHub repository using your GitHub account. For example, a user named `example-author` could create a repository named `honmod-updates`, giving the repository identifier:

```text
example-author/honmod-updates
```

A GitHub account is needed for the publisher; ordinary users of public Mod updates do not need to sign in.

## Step 2 — Rebuild with the correct repository

Extract this test kit and open a terminal **in the extracted folder**. Run:

```powershell
python prepare_release.py --repo "example-author/honmod-updates"
```

On Windows, `py -3` can be used instead of `python` if installed. On macOS/Linux, `python3` may be needed.

The script uses `source-original-v1.0.19.honmod` from the **same folder as the script**. It regenerates `v1.0.19`, `v1.0.20`, `mods.json`, `checksums.sha256`, and the test release notes. It does **not** read or write any installed game Mod.

For advanced use, `--source ./path/to/original.honmod` and `--output ./output-folder` are supported. Relative paths are resolved from your current terminal directory.

### What goes into mod.xml?

Inside each GitHub-enabled package the root `<modification>` element includes:

```xml
<modification
    application="Heroes of Newerth Reborn"
    name="NF In-Game Matchmaking"
    version="1.0.19"
    author="Kim22"
    updaterepo="example-author/honmod-updates"
    updateid="nf-in-game-matchmaking">
    <!-- Existing mod operations are preserved -->
</modification>
```

The two fields are supported by **Newerth Forge 1.8.1 Dev with GitHub Update**:

- `updaterepo`: the exact public GitHub `OWNER/REPO`.
- `updateid`: a permanent lowercase ID for **this mod**. Keep it unchanged between updates.
- `version`: bump for each new update.

Other creators should use their **own** mod name, author, repository and update ID when building their own packages. The name and author here describe this existing example Mod.

## Step 3 — Validate your generated files

Run in the same folder:

```powershell
python validate_kit.py
```

This verifies ZIP contents, the two versions, GitHub metadata, asset filename, file size, SHA-256, and that the gameplay files are identical. No internet or Newerth Forge installation is required.

## Step 4 — Publish the test update

1. In the **public** GitHub repository, create a Release with the **exact tag** `nf-in-game-matchmaking-v1.0.20`.
2. Attach `NF-In-Game-Matchmaking-v1.0.20.honmod` as a Release asset. Upload the `.honmod` itself, **not** the entire test-kit ZIP.
3. Add `mods.json` to the **root of the default branch** of that **same** repository, ideally **after** the Release and asset are published.
4. Confirm the entry's `version`, `releaseTag`, `asset`, `size` and `sha256` match your uploaded package exactly. Do not modify an archive after computing its checksum.

A single repository can host multiple Mod entries in `mods.json`, each with its own stable `updateid` and Release tag.

## Step 5 — Test with Newerth Forge

**Back up installed Mods before running any live tests.** Testing is optional: preparing this kit does not change the game or install Mods.

1. Use a test game installation/profile or a separate copy of the Mods folder.
2. Import the rebuilt GitHub-enabled `NF-In-Game-Matchmaking-v1.0.19.honmod` into Newerth Forge.
3. Open **Custom Mod (.honmod)** and click **CHECK UPDATES**.
4. If the Repository and Release are publicly accessible and configured correctly, an **UPDATE v1.0.20** button should appear.
5. Clicking UPDATE intentionally downloads the Release asset and verifies package identity and SHA-256 before replacing the installed package; the Launcher makes a backup.
6. Updating the package does not automatically Apply Mods to the game. Press Apply Mods separately if and when you wish to test that operation.

**Checking alone never installs or applies the Mod.**

## Common issues

| Message or symptom | What to check |
| --- | --- |
| No update source | The installed `.honmod` must contain `updaterepo` and `updateid`. The **original source file** does not. |
| GitHub 404 | Repository must be public, `mods.json` must be at the repository root, and tag/asset names must match exactly. |
| GitHub 403 / 429 | GitHub rate limit or access restrictions. Retry later; do not embed access tokens in Mods. |
| Update ID not listed | `updateid` in `mod.xml` must match a key in `mods.json`. |
| SHA-256 mismatch | Rebuild the catalog from the final uploaded bytes and verify the Release asset was not changed. |
| No UPDATE button | Check installed `version` against the catalog's `version`; the latter must be newer. |

## Security and compatibility

- Public GitHub repositories only. Never include tokens or secrets in `mod.xml` or `mods.json`.
- Existing Mods using `updatecheckurl` / `updatedownloadurl` are supported by Newerth Forge's separate legacy update mechanism.
- The index can be read by anyone. A SHA-256 checksum protects against accidental mismatch of the downloaded artifact against the catalog, but is **not** a substitute for trusting the repository owner.
- This kit is for GitHub Update support in **Newerth Forge 1.8.1 Dev**. Older Launchers may ignore these metadata fields.
- `v1.0.20` is **not** a feature release. Do not promote it as one.