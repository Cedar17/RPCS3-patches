# RPCS3 patches

Small, documented RPCS3 patch collection. Current focus: **God of War HD** from **God of War Collection**, US release **BCUS98229**, game update **01.01**.

## Current status

Target PPU hash:

```text
PPU-645f0573d7438e37ddd231377adf9c1205e8a040
```

Included patches:

- **Unlimited Magic Use** — gameplay verified.
- **Infinite Mid-Air Jumps** — gameplay verified.
- **Infinite Health (experimental)** — the runtime HP pointer chain is strongly verified from RPCS3 memory dumps; the final hook implementation still needs one last gameplay test.

The patch file is [`imported_patch.yml`](./imported_patch.yml).

## Installation

Download `imported_patch.yml` and place/merge it into RPCS3's patch directory. Then open RPCS3's **Manage Game Patches** for God of War Collection and enable the desired entries.

When changing executable-code patches, clearing the game's PPU cache before retesting is useful so RPCS3 recompiles the affected code.

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

After the normal first/second jump, repeatedly pressing jump continues to produce additional mid-air jumps.

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

This is a very strong match for:

```text
player + 0x184 = current HP
player + 0x188 = max HP
```

which independently agrees with the old CodeFreak structure.

Therefore the runtime player-object base in that session was:

```text
0x30A3DC54 - 0x184 = 0x30A3DAD0
```

Searching the entire BEFORE memory dump for the big-endian pointer value `0x30A3DAD0` produced exactly one hit:

```text
0x0054DA9C -> 0x30A3DAD0
```

So for the tested BCUS98229 01.01 build, the reconstructed chain is:

```text
[0x0054DA9C]
    -> player object
    + 0x184 -> current HP
    + 0x188 -> max HP
```

That is the key result of the investigation.

### Current experimental implementation

The current `Infinite Health (experimental)` patch does not hard-code `200.0`.

Instead it periodically performs the conceptual operation:

```c
player = *(u32*)0x0054DA9C;
if (player)
    player->current_hp = player->max_hp;
```

In PowerPC terms it loads the dynamic player pointer, reads `+0x188`, and writes those same 32 bits to `+0x184`.

This is preferable to writing a fixed constant because it should naturally follow HP upgrades.

The patch preserves `r11`, `r12`, CR and the stack, then replays the original instruction displaced by the hook. It also checks for a null player pointer and a zero max-HP value before writing.

The **pointer chain itself is verified from runtime data**. The final code-hook cadence still needs one gameplay damage test before this entry should be promoted from experimental to stable.

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

Test environment during development included RPCS3 0.0.42 master builds on Windows 11 with BCUS98229 update 01.01.

Known verified behavior:

- Infinite Mid-Air Jumps: works in actual gameplay.
- Unlimited Magic Use: works; blue meter may visibly reach zero and casting still works.
- Infinite Health pointer mapping: verified by two guest-memory snapshots.
- Infinite Health final hook: pending one last live damage test.

If an entry behaves differently on another dump/update/region, first verify the game's PPU hash. These addresses are not intended as universal offsets for every release of God of War Collection.

## Why keep the reverse-engineering notes?

Most old cheat lists preserve only the final addresses. That is enough until a port, game update, emulator memory model or code conversion breaks them.

Documenting the pointer chain, actual runtime values, failed assumptions and observed effects makes the result reproducible instead of leaving the next person with another opaque hexadecimal patch.
