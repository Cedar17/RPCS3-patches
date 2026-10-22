# RPCS3 Patches

A small, personally gameplay-tested collection of custom [RPCS3](https://rpcs3.net/) patches. It currently covers the two remastered games in the **US God of War Collection**. Each included patch was tested in-game against a specific executable; this is not a universal cheat pack or an official RPCS3 patch set.

**[Download the combined imported_patch.yml](./imported_patch.yml)** · [God of War HD technical notes](./docs/god-of-war-hd.md) · [God of War II HD technical notes](./docs/god-of-war-ii-hd.md)

## Supported games and versions

| Game | Edition / Title ID | Installed app version | Verified PPU hash |
| --- | --- | --- | --- |
| **God of War HD** | God of War Collection, US, `BCUS98229` | `01.01` | `PPU-645f0573d7438e37ddd231377adf9c1205e8a040` |
| **God of War II HD** | God of War Collection, US, `BCUS98229` | `01.01` | `PPU-9cd8d28963ebc29b59c79819cb56f3d92e857a37` |

Both executables share the collection's Title ID, but **their PPU hashes are different**. The game update number alone does not establish compatibility. No other regional builds, update versions, or PPU hashes have been verified.

### Gameplay-tested patches

| Game | Patch | Verified effect |
| --- | --- | --- |
| God of War HD | **Infinite Health** | Current health is restored to the current maximum, including after taking damage. |
| God of War HD | **Unlimited Magic Use** | The blue magic meter may reach zero, but spells remain usable. |
| God of War HD | **Infinite Mid-Air Jumps** | Additional mid-air jumps can be performed by repeatedly pressing X. |
| God of War II HD | **Infinite Magic** | Casting spells does not decrease the blue magic meter. |

All four entries are independently selectable. The sole distributed patch file is [`imported_patch.yml`](./imported_patch.yml), using RPCS3 patch format `Version: 1.2`.

**Gameplay caveat:** God of War HD's final Ares phase uses a shared, tug-of-war health mechanic. **Disable Infinite Health before that phase** to avoid interfering with the encounter. Repeated infinite jumps can also bypass level triggers. See the individual game documents for limitations and testing evidence.

## Install

1. Download [`imported_patch.yml` as a raw file](https://raw.githubusercontent.com/Cedar17/RPCS3-patches/master/imported_patch.yml). It should remain a plain-text `.yml` file.
2. In RPCS3, open **Manage → Game Patches**, or right-click **God of War Collection** → **Manage Game Patches**.
3. Drag the downloaded YAML into the Patch Manager and select **Import** in the dialog. Importing a custom patch does not require replacing your entire local patch database.
4. Locate the **God Of War: HD (Custom QoL)** and/or **God Of War: II HD (Custom QoL)** entries for `BCUS98229`, app version `01.01`. Enable only the patches you want, then select **Save**.
5. Completely stop and relaunch emulation to test the changes.

If no matching entry appears or a patch has no effect, check the game's actual **PPU executable hash** in `RPCS3.log` first. The addresses here are intended only for the explicitly listed build.

### Upgrading from older versions of this repository

Earlier revisions provided separate God of War II experimental YAML files. **Downloading the new combined YAML does not automatically remove entries already imported into RPCS3.**

- Back up your current local `patches/imported_patch.yml` before changing anything.
- In RPCS3's Patch Manager, disable obsolete **TEST**, **A/B**, previous God of War II **Infinite Magic**, and nonworking God of War II **Infinite Mid-Air Jumps** entries.
- If duplicates remain after importing the new file, remove only the stale game-specific entries from the backed-up local imported patch file, or otherwise reset those obsolete imports. Do not discard unrelated imported cheats.
- Import the current single file and enable the **God Of War: II HD (Custom QoL) → Infinite Magic** entry. Avoid enabling multiple variants that hook `0x00090A38` or write to the same code cave.

RPCS3's imported-file storage and enablement state may differ from what is present in this GitHub repository. Review both when troubleshooting.

## Technical documentation

The docs contain primary gameplay observations, original instruction bytes, code-cave implementation details, failed approaches and attribution:

- [**God of War HD — patches and reverse-engineering notes**](./docs/god-of-war-hd.md): Wulf2k patterns; health pointer-chain reconstruction from before/after guest-memory dumps; final Ares caveat.
- [**God of War II HD — infinite magic and investigation**](./docs/god-of-war-ii-hd.md): Artemis/Markin_Mods starting point, hook isolation, PowerPC register/stack correction, final tested implementation and discarded infinite-jump attempt.

### Credit

This collection builds on historical community findings from **Wulf2k**, **CodeFreak / PS3UserCheat**, **Markin_Mods**, and the **ChiDreams** Artemis-to-RPCS3 conversions. **Cedar17** carried out build-specific verification, the God of War HD health pointer-chain port, and the God of War II HD register-safe magic revision. Upstream authors are credited per patch in the YAML and linked individually in the technical notes; not all underlying ideas originated in this repository.

Research and tooling links: [ArtemisPS3](https://github.com/Dnawrkshp/ArtemisPS3) · [ChiDreams' Artemis RPCS3 patches](https://github.com/chidreams/Artemis-Patch-Collection-RPCS3) · [RPCS3 Game Patches guide](https://wiki.rpcs3.net/index.php?title=Help:Game_Patches).

## Scope and distribution

This is an **as-tested** collection, not a guarantee that every game state, boss phase, or future emulator version behaves identically. Please report regressions with the game Title ID, app version, **PPU hash**, selected patch names and the observed behavior.

The repository also retains `GOW1.self` and `GOW2.self` as supplied binary research artifacts. **They are not required to install or use the YAML patches and are not covered by any open-source grant from this repository.** Users are responsible for obtaining game software they are entitled to use. This repository does not include a LICENSE file.
