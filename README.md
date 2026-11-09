# NF In-Game Matchmaking

GitHub-hosted test packages for the Newerth Forge 1.8.1 Dev **.honmod GitHub Update** feature.

## Test the update flow

1. Download the [GitHub-enabled v1.0.19 starter package](https://github.com/kim21357/NF-In-Game-Matchmaking/raw/refs/heads/main/test-assets/NF-In-Game-Matchmaking-v1.0.19.honmod). This is not the original unmodified package: its `mod.xml` includes GitHub update metadata.
2. Back up your existing installed mod first, then import the starter package into a **test installation** using Newerth Forge 1.8.1 Dev.
3. Open **Custom Mod (.honmod)** and press **CHECK UPDATES**. The launcher reads the root [mods.json](mods.json) and should show **UPDATE v1.0.20**.
4. When you choose **UPDATE**, the launcher downloads the actual [v1.0.20 Release asset](https://github.com/kim21357/NF-In-Game-Matchmaking/releases/tag/nf-in-game-matchmaking-v1.0.20), verifies its SHA-256, backs up the old archive, and replaces that installed mod.
5. Applying the updated package to HoN remains a separate manual action. Checking updates never installs or applies anything.

> **Test-only:** v1.0.20 changes only the version and GitHub update metadata. No gameplay, UI, or sound changes are included. Do not describe this as a new feature release.

## Update configuration in mod.xml

```xml
<modification
    application="Heroes of Newerth Reborn"
    name="NF In-Game Matchmaking"
    version="1.0.19"
    author="Kim22"
    updaterepo="kim21357/NF-In-Game-Matchmaking"
    updateid="nf-in-game-matchmaking">
    <!-- Existing editfile/copyfile operations -->
</modification>
```

`updaterepo` tells Newerth Forge where to look; `updateid` matches the entry key in [mods.json](mods.json). Keep the ID stable between releases and increment the version when publishing a new package.

## Publishing future versions

Create a GitHub Release with a unique tag, attach the final `.honmod`, and update the root `mods.json` with its exact `version`, `releaseTag`, `asset`, SHA-256 and byte size **after** attaching the Release asset. Use a valid 64-character lowercase SHA-256 of the final uploaded bytes. The package version and repository/ID must match. These instructions use relative repository paths, not machine-specific game directories.

### Test asset information

| Version | Purpose | Location |
| --- | --- | --- |
| v1.0.19 | Starting package with update metadata | [test-assets/](test-assets/NF-In-Game-Matchmaking-v1.0.19.honmod) |
| v1.0.20 | Metadata-only update used to test downloading | [GitHub Release](https://github.com/kim21357/NF-In-Game-Matchmaking/releases/tag/nf-in-game-matchmaking-v1.0.20) |

For normal users, only the packaged `.honmod` is necessary. No Python, local source tree, or particular drive letter is needed.
