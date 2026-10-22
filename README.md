# RPCS3 patches

Small, documented patches for **God of War Collection** (US serial **BCUS98229**, update **01.01**). God of War I HD and II HD are separate executables with different PPU hashes even on the same disc.

## God of War II HD — Infinite Magic confirmed; Infinite Jumps investigation

**Download:** [`God_of_War_II_HD_BCUS98229_01.01_patch.yml`](./God_of_War_II_HD_BCUS98229_01.01_patch.yml) | [Raw file](https://raw.githubusercontent.com/Cedar17/RPCS3-patches/master/God_of_War_II_HD_BCUS98229_01.01_patch.yml)

- Game: **God of War II HD**, BCUS98229, app version **01.01**.
- Target executable hash: `PPU-9cd8d28963ebc29b59c79819cb56f3d92e857a37`.
- **Infinite Mid-Air Jumps (NOT WORKING):** the old Markin_Mods/ChiDreams PPC NOP at `0x0008CDB4` has **no visible effect** in the owner\'s BCUS98229 01.01 RPCS3 gameplay test. It is preserved in YAML for provenance but should be **left disabled**.
- **Infinite Magic (WORKING):** two hooks at `0x00090A38` / `0x00090C74`, with code caves at `0x004C1214` / `0x004C1230`. **User-tested 2026-10-22** on BCUS98229 01.01: repeated spells do not reduce the blue HUD bar. Unlike GoW I\'s unlimited-casting patch, which allows the meter to drain to zero, this GoW II patch leaves the meter full.
- **Status:** GoW II Infinite Magic **verified working by the user** on 2026-10-22, while Infinite Mid-Air Jumps **failed the same game test**. The magic instruction writes are left unchanged. Do not advertise the one-NOP jump entry as verified.
- **Scope:** these two cheats only. No GoW II health, rage, orbs, unlock-all changes.

### Install without replacing existing patches (Windows RPCS3)

1. Download the **Raw file** and save it as `God_of_War_II_HD_BCUS98229_01.01_patch.yml`. Keep the filename ending in `patch.yml`.
2. In RPCS3, right-click **God of War Collection** → **Manage Game Patches**, or open **Manage → Game Patches**.
3. **Drag the downloaded YAML onto the Patch Manager window**, then select **Import** in the dialog (not merely Validate). The Import action is not necessarily a visible toolbar button.
4. Expand **God Of War: II HD (Custom QoL)** → **BCUS98229** → **01.01**. Tick **Infinite Magic** and click **Save**. Leave **Infinite Mid-Air Jumps** **unchecked**, because user testing found no effect.
5. Fully stop and restart the game. Check that repeated spells do not decrease the blue meter; test a checkpoint reload. The jumping candidate does not presently work and needs a separate reverse-engineering cycle.
6. If the patch does not show, boot GoW II and inspect `RPCS3.log` for `PPU executable hash`. It must match `PPU-9cd8d28963ebc29b59c79819cb56f3d92e857a37`. Do not force another hash into this file.

**Important:** Do not replace your RPCS3 `patches/imported_patch.yml` if it has your GoW I patches or other custom patches. The drag-and-drop **Import** procedure adds entries to the managed custom patch file. Do not enable competing patches that write the same GoW II code cave. If the game becomes unstable, disable the suspect entry and restart. Even a future *working* infinite jump can bypass trigger volumes; keep a normal save. If the executable patch is not picked up after changes, consider clearing the game's PPU cache.

### Sources and provenance

- [ChiDreams, Artemis-Patch-Collection-RPCS3 — BCUS98229 v1.01](https://github.com/chidreams/Artemis-Patch-Collection-RPCS3/blob/main/God%20Of%20War%20Collection%20%5BBCUS98229%5D%20v1.01): source of the exact 01.01 US two-code-cave magic patch and infinite-jumps NOP.
- [PaauloSilva Artemis-Patch-Collection-RPCS3_PKG](https://github.com/PaauloSilva/Artemis-Patch-Collection-RPCS3_PKG): mirrors the relevant code identically; a consistency cross-check, **not** independent runtime testing.
- [Ali-mk original PS3-era demo, July 2021](https://www.youtube.com/watch?v=EN6j6DhRb5U): describes GoW II HD Infinite Magic and Infinite Jumps for BCUS98229.
- [bucanero ArtemisPS3, GoW II European BCES00791 cheats](https://github.com/bucanero/ArtemisPS3/blob/master/docs/codes/God%20Of%20War%202%20BCES00791%20v01.00%20av01.01.ncl): alternative magic NOP sites `0x0008D000`, `0x0008D030`, `0x0008D038` belong to the **European** build, so they have **not** been reused for BCUS98229.
- [RPCS3 Game Patches documentation](https://wiki.rpcs3.net/index.php?title=Help:Game_Patches): YAML versioning and import methodology.

## Infinite jump investigation (2026-10-22)

**Observed failure:** On BCUS98229, app version 01.01, the user imported both GoW II entries successfully. The two-hook magic patch worked: casting magic did not decrease the blue HUD bar. The one-word infinite-jump patch, at `0x0008CDB4 -> 0x60000000`, produced no observable additional jumps. Since both entries matched the GoW II executable and magic was effective, "YAML not imported" is unlikely to explain the jump failure. The jump write's runtime execution and exact original instruction have **not** been confirmed yet.

**Source quality:** [Markin_Mods' original 2020 post](https://ps3hackingbyme.blogspot.com/2020/06/god-of-war-2-hd-unlock-all-cheats.html) advertises GoW II HD infinite jumps (BCUS98229 01.01) but does not publish the underlying original disassembly or test protocol. [ChiDreams' Artemis port](https://github.com/chidreams/Artemis-Patch-Collection-RPCS3) provides the single NOP; [PaauloSilva's mirror](https://github.com/PaauloSilva/Artemis-Patch-Collection-RPCS3_PKG) reproduces it, but copying is **not** independent working proof.

**Different platform, useful conceptual lead (do NOT paste PS2 addresses into PS3):** The PS2 NTSC-U GoW II raw cheat `2022662C 00000000` is documented as *Infinite Double Jumps* in [2016 GameHacking forum discussions](https://forum.gamehacking.org/forum/video-game-hacking-and-development/retro-hacking/14624-god-of-war-two-codes-ps2) and on [Almar's Guides](https://almarsguides.com/Retro/Walkthroughs/Ps2/Games/GodofWarII/Codebreaker/). It is a MIPS/EE game and that is **not** a PPU patch offset. A [GoW III RPCS3 cheat author](https://github.com/Luckisugar/rpcs3-gow3-patches) likewise succeeds by stopping the used-double-jump flag from being stored. This supports investigating **jump-used state writes** rather than blindly NOPing unrelated instructions.

**2026-10-22 owner RPCS3 debugger disassembly, jump cheat DISABLED:** The actual original instruction was captured in a live GoW II HD session, BCUS98229 01.01, RPCS3 0.0.42-20103:

```asm
0x0008CD78: lfs    f0,0x60(r8)
0x0008CD7C: fsubs  f0,f0,f31
0x0008CD80: lfs    f12,-0x69D0(r2)
0x0008CD8C: lhz    r0,0(r9)
0x0008CD90: extsh  r0,r0
0x0008CDA0: fcfid  f11,f11
0x0008CDA4: frsp   f13,f11
0x0008CDA8: ble    cr7,0x8CDD8
0x0008CDAC: fcmpu  cr7,f0,f13
0x0008CDB0: bge    cr7,0x8CDD0
0x0008CDB4: stfs   f0,0x60(r8)  # original 0xD0080060
0x0008CDD0: fmr    f0,f13
0x0008CDD4: b      0x8CDB4
0x0008CDD8: fmr    f0,f12
0x0008CDDC: b      0x8CDB4
```

The site is a **bounded floating-point update**: compute `f0 = [r8 + 0x60] - f31`, clamp between `f12` and a signed-16-bit-table-derived `f13`, then store `f0` back to `[r8 + 0x60]`. This looks like a clamped meter/resource or parameter update, **not an obvious "used double jump" flag**. The Artemis `nop` removes the store, which is logically insufficient evidence of an infinite X-jump mechanism. **Do not assume** the exact resource identity from the offset alone; `r26+0x60` in the working magic cave is suggestive, not proof both registers point at the same object. This direct in-game disassembly replaces previous uncertainty about the original opcode. It does not yet show whether the execution path is reached in a given action.

**Next diagnostic:** Keep the broken jump entry disabled, add a temporary execution breakpoint at `0x0008CDB4`, resume a safe gameplay scene and record whether/when the breakpoint fires during idle, jumping, and magic use, including `r8`, `f0`, `f12`, `f13`, and `f31` if stopped there. Remove the breakpoint before ending/restarting emulation (some RPCS3 builds have had stop-with-breakpoint regressions). Even if this site is actively executed, that alone does not prove it gates double-jump input; finding and patching the correct airborne-jump-used state writer is the goal. No speculative byte changes have been committed.

**Minimal evidence needed from user's executable:**
1. Uncheck only GoW II **Infinite Mid-Air Jumps** in RPCS3 Patch Manager, click Save, completely restart the game, keep verified Infinite Magic enabled.
2. While GoW II is running, open RPCS3 **Debugger**, select a PPU thread and use **Go to address** `0x0008CDA0`. Take a screenshot or copy disassembly covering `0x0008CD80` through `0x0008CE00`, especially the original word at `0x0008CDB4`. This distinguishes wrong-site/harmless NOP from a plausible but insufficient hook.
3. Optionally inspect the same area with the old jump patch on: `0x0008CDB4` should display `nop` / `60000000` if patched. Do not infer actual runtime execution merely from seeing a patched instruction.
4. Capture one on-ground / airborne / after-double-jump sequence of game-state values with RPCS3's guest-memory dumps. Search candidate changing flags (0→1 after second jump, reset on landing) and the **code writing them**. Compare with unpatched values, then test one hypothesis at a time using separately named experimental patch entries.
5. Do not transplant the PS2 `0x0022662C` or the first GoW HD `0x00080B14` offsets. They belong to different executables/CPU ISAs.

**Alternative without a binary cheat:** The speedrunning [GoW II Glitch FAQ](https://gamefaqs.gamespot.com/ps2/932295-god-of-war-ii/faqs/54994) describes Barbarian Hammer Infinite Jump and other timing-based game glitches, but these are **not** substitutes for a repeat-X jump patch.

**Current recommendation:** Use the verified Infinite Magic entry only. The jump patch remains recorded for archaeology; mark it failed rather than replacing it with guessed opcodes.

## God of War HD (first game) — gameplay-verified

First-game PPU hash: `PPU-645f0573d7438e37ddd231377adf9c1205e8a040`.

Cedar17 previously verified **Infinite Health**, **Unlimited Magic Use**, and **Infinite Mid-Air Jumps** in GoW I HD gameplay on BCUS98229 01.01. The existing [`imported_patch.yml`](./imported_patch.yml) remains **unchanged**. Prefer drag-and-drop Import of the separate GoW II file above so you keep these entries.

## Infinite Health

### Actual gameplay effect

Verified behavior on BCUS98229 01.01:

- Loading a save with low HP immediately restores the green bar to full.
- Enemy hits do not visibly reduce the green bar.
- The patch follows the game's current maximum HP rather than forcing a hard-coded constant.

### Runtime structure

The final pointer chain was reconstructed from RPCS3 guest-memory dumps:

```text
[0x0054DA9C]
    -> player object
    + 0x184 -> current HP
    + 0x188 -> max HP
```

One captured runtime instance was:

```text
[0x0054DA9C] = 0x30A3DAD0

BEFORE opening a green-health chest:
0x30A3DC54 (+0x184) = 40.5
0x30A3DC58 (+0x188) = 200.0

AFTER opening the chest:
0x30A3DC54 (+0x184) = 200.0
0x30A3DC58 (+0x188) = 200.0
```

The player base in that session was therefore:

```text
0x30A3DC54 - 0x184 = 0x30A3DAD0
```

Searching the entire BEFORE guest-memory dump for the big-endian pointer value `0x30A3DAD0` produced exactly one hit:

```text
0x0054DA9C -> 0x30A3DAD0
```

This independently matches the old CodeFreak / PS3UserCheat structure, which also treated HP as player-relative fields at `+0x184` and `+0x188`.

### Implementation

The patch does not write `200.0` directly. Conceptually it does:

```c
player = *(u32*)0x0054DA9C;
if (player && player->max_hp != 0)
    player->current_hp = player->max_hp;
```

That makes it naturally follow health upgrades.

The patch uses a small PowerPC code cave, preserves `r11`, `r12`, CR and the stack, replays the displaced original instruction, and returns to the original flow.

### Final Ares battle warning

The final Ares encounter has special health mechanics that differ from ordinary combat. In the last phase, Kratos and Ares use a shared / tug-of-war health bar rather than two completely independent ordinary HP pools. A patch that continuously forces Kratos' current HP back to maximum can therefore interfere with the intended boss mechanic.

For normal final-boss behavior, **disable Infinite Health before the final Ares phase**. Unlimited Magic Use and Infinite Mid-Air Jumps do not modify the HP fields and can remain enabled if desired.

The earlier warning that “Ares steals health” was imprecise wording. The important technical point is the special shared-health mechanic in the final phase, not a generic life-steal effect throughout the fight.

## Unlimited Magic Use

### Actual gameplay effect

The blue magic meter still decreases normally and can reach zero. However, Kratos can continue casting magic even with an empty blue bar.

This is intentional. The patch does **not** freeze or refill the displayed magic resource; it bypasses the game's "is magic available?" gate.

### Patch

```text
0x00071E60: 7D6307B4 -> 38600001
```

`0x38600001` is PowerPC:

```asm
li r3, 1
```

The next instruction is `blr`, so the patched function immediately returns true.

Original Wulf2k AoB from ArtemisPS3:

```text
7D6307B4 4E800020 89280411
        ->
38600001 4E800020 89280411
```

This AoB was also found at the expected location in the BCUS98229 01.01 executable used for testing.

## Infinite Mid-Air Jumps

### Actual gameplay effect

After the normal first/second jump, repeatedly pressing **X** continues to produce additional mid-air jumps.

This is useful for traversal and for bypassing some timing/platforming sections. It can also sequence-break the game, so very aggressive use may skip camera or gameplay trigger volumes.

### Patch

```text
0x00080B14: 60000018 -> 60000000
```

`0x60000000` is the canonical PowerPC NOP (`ori r0,r0,0`).

Original Wulf2k AoB:

```text
60000018 540902D6
        ->
60000000 540902D6
```

This patch was verified directly in gameplay on BCUS98229 01.01.

## Infinite Health — reverse-engineering notes

This was the difficult one.

### What older cheats told us

The old ArtemisPS3 God of War 1 list contains this CodeFreak/PS3UserCheat entry:

```text
Infinite HP
6 0053AE1C 00000184
0 00000000 42C80000
6 0053AE1C 00000188
0 00000000 42C80000
```

The important structural information is that HP lives behind a player-object pointer, with fields at offsets:

```text
player + 0x184
player + 0x188
```

The original 01.00 pointer root `0x0053AE1C` is **not valid as the player pointer root in the tested 01.01 build**. Runtime inspection showed:

```text
[0x0053AE1C] = 0x3FABC57F
```

which is not a plausible heap pointer.

### Failed approaches

Several approaches were tested and rejected:

1. **Hard-coding an old 0x30xxxxxx HP address**
   - Failed because the player object is dynamically allocated and heap addresses move between runs.

2. **Assuming all 01.00 data addresses shifted by one constant delta in 01.01**
   - A known magic-related address suggested a `+0x18E28` shift, but applying that globally produced false candidates.

3. **Reusing a Markin-style code cave without reconstructing the real player pointer chain**
   - The original conversion used scratch RAM around `0x00BFFFFC`, which is unmapped in RPCS3 and caused an access violation.
   - Repaired variants stopped crashing but did not actually protect HP.

4. **Assuming a register at a convenient hot hook was directly the CodeFreak player object**
   - Writing `r31+0x184/+0x188` produced no health effect, so that object assumption was wrong.

These failures are why the final approach switched to **runtime memory snapshots** instead of continuing to guess static addresses.

### Runtime memory-dump method

RPCS3's guest-memory dumper was used to take two snapshots of the same save:

- BEFORE: Kratos had less than one-third HP.
- AFTER: a nearby green-health chest was opened and HP became full.

A big-endian float32 diff over writable guest memory produced one standout candidate:

```text
0x30A3DC54: 40.5 -> 200.0
```

This matched the observed visual state very well: about 20% health before the chest and full health afterward.

The adjacent field was then inspected:

```text
BEFORE
0x30A3DC54 = 40.5
0x30A3DC58 = 200.0

AFTER
0x30A3DC54 = 200.0
0x30A3DC58 = 200.0
```

This strongly identifies:

```text
player + 0x184 = current HP
player + 0x188 = max HP
```

which independently agrees with the old CodeFreak structure.

Therefore the runtime player-object base in that session was:

```text
0x30A3DC54 - 0x184 = 0x30A3DAD0
```

Searching the entire BEFORE memory dump for the pointer value `0x30A3DAD0` produced exactly one hit:

```text
0x0054DA9C -> 0x30A3DAD0
```

That reconstructed the stable pointer root for the tested BCUS98229 01.01 executable.

### Final live verification

After implementing the runtime-derived pointer chain in the RPCS3 patch:

- A save that normally loaded at half/low health loaded with a full green bar.
- Enemy attacks no longer reduced the green bar.
- No VM access violation occurred.

At that point the patch was promoted from experimental to stable.

## Historical / upstream references

This work builds on older community cheat research rather than starting from zero.

### ArtemisPS3 original cheat database

https://github.com/Dnawrkshp/ArtemisPS3

Relevant GoW1 list:

https://github.com/Dnawrkshp/ArtemisPS3/blob/master/ArtemisPS3-GUI/pkgfiles/USRDIR/USERLIST/God%20Of%20War%201%20BCES00800%20BCUS98229%2001.00.ncl

That file contains the CodeFreak pointer-layout HP cheat and Wulf2k's Infinite Magic / Infinite Mid-Air Jumps AoBs used here as historical reference.

### Artemis patch collection for RPCS3

https://github.com/chidreams/Artemis-Patch-Collection-RPCS3

This project ports a large amount of Aldo/Artemis cheat material into RPCS3's patch format and was useful for comparison.

### PKG-oriented fork

https://github.com/PaauloSilva/Artemis-Patch-Collection-RPCS3_PKG

Also useful when comparing per-game historical conversions.

### ARTEMIS RPCS3 Cheat Manager

https://github.com/chidreams/ARTEMIS-RPCS3-Cheat-Manager

Useful tooling for managing/importing RPCS3 cheat data.

### RPCS3

https://github.com/RPCS3/rpcs3

The emulator's Debugger, Memory Viewer and Guest Memory Dump features were essential for verifying the runtime HP layout without relying on unstable hard-coded heap addresses.

## Testing notes

Development/testing used RPCS3 0.0.42 master builds on Windows 11 with BCUS98229 update 01.01.

Verified behavior:

- Infinite Mid-Air Jumps: repeatedly pressing X in mid-air keeps jumping.
- Unlimited Magic Use: blue meter drains normally, but casting still works at zero.
- Infinite Health: low-HP saves are restored to full and enemy hits do not reduce the green bar.

If an entry behaves differently on another dump/update/region, first verify the game's PPU hash. These addresses are not intended as universal offsets for every release of God of War Collection.

## Why keep the reverse-engineering notes?

Most old cheat lists preserve only the final addresses. That is enough until a port, game update, emulator memory model or code conversion breaks them.

Documenting the pointer chain, actual runtime values, failed assumptions and observed effects makes the result reproducible instead of leaving the next person with another opaque hexadecimal patch.
