# NF In-Game Matchmaking

A Heroes of Newerth (HoN) `.honmod` project and a **test** of GitHub-based update detection in Newerth Forge 1.8.1 Dev.

## GitHub Update test

- Stable update ID: `nf-in-game-matchmaking`
- Repository: `kim21357/NF-In-Game-Matchmaking`
- Update catalog: [`mods.json`](mods.json)
- Planned test release tag: `nf-in-game-matchmaking-v1.0.20`
- Package: `NF-In-Game-Matchmaking-v1.0.20.honmod`

The **v1.0.20 package is a metadata-only test update** from v1.0.19. It does not change the game mod's features, patches, images, or sounds. The package must be attached to a published GitHub Release under the exact tag above before testing real update downloads.

## For mod creators

To enable GitHub updates, add `updaterepo="kim21357/NF-In-Game-Matchmaking"` and `updateid="nf-in-game-matchmaking"` to the root `<modification>` element of the `.honmod`'s `mod.xml`. The installed version is read from `version`. On CHECK UPDATES, Newerth Forge reads `mods.json` and compares the version. The user must click UPDATE to download and must manually Apply Mods afterward.

Keep `mods.json` SHA-256 and file size in sync with the exact `.honmod` asset published in the Release. No personal local paths or installation directories are required.
