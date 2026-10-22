# God of War II HD — infinite magic patch and investigation

[Back to project overview](../README.md) · [Download combined patch YAML](../imported_patch.yml)

## Compatibility and verified outcome

| Field | Value |
| --- | --- |
| Game | God of War II HD (God of War Collection, second game) |
| Region / Title ID | US / `BCUS98229` |
| Installed game update | `01.01` |
| Executable PPU hash | `PPU-9cd8d28963ebc29b59c79819cb56f3d92e857a37` |
| Emulator / platform | RPCS3 0.0.42-series on Windows |
| Patch shipped | **Infinite Magic**, verified in gameplay on 2026-10-22 |

**Observed result:** casting spells does not deplete the blue magic meter. The final tested revision replaced the earlier community two-hook implementation with a one-hook implementation that preserves the PowerPC floating-point register `f30`. It was tested by the repository owner. Testing one specific build does not prove compatibility with other game regions or emulator versions.

## Research lineage and attribution

- **Markin_Mods:** historical original cheat implementation.
- **ChiDreams:** conversion of historical cheats into RPCS3 YAML format; the initial two-hook `Infinite Magic` port was used as the starting point. [Artemis-Patch-Collection-RPCS3 — God Of War Collection BCUS98229 01.01](https://github.com/chidreams/Artemis-Patch-Collection-RPCS3/blob/main/God%20Of%20War%20Collection%20%5BBCUS98229%5D%20v1.01).
- **PaauloSilva:** [collection mirror](https://github.com/PaauloSilva/Artemis-Patch-Collection-RPCS3_PKG) that reproduces the original port's instructions, useful as a consistency check, but not independent proof of runtime behavior.
- **Cedar17:** RPCS3 gameplay testing, isolated A/B analysis, revised PPC code-cave prologue/epilogue, final gameplay verification and documentation.
- [RPCS3 Game Patches documentation](https://wiki.rpcs3.net/index.php?title=Help:Game_Patches) for imported YAML syntax.

## First working implementation: original two-hook community port

The upstream `Infinite Magic` patch used two modifications:

| Hook | Original patch address | Injected cave | Principal effect |
| --- | --- | --- | --- |
| A | `0x00090A38` | `0x004C1214` | Derive a float from a halfword and store it to `0x60(r26)` |
| B | `0x00090C74` | `0x004C1230` | Store a constant/load result into `0x68(r26)` |

Hook A's original cave:

```asm
0x004C1214: lhz   r0,0(r9)
0x004C1218: std   r0,-0x20(r1)
0x004C121C: lfd   f30,-0x20(r1)
0x004C1220: fcfid f30,f30
0x004C1224: stfs  f30,0x60(r26)
0x004C1228: b     0x00090A3C
```

**Gameplay observation:** blue magic did not decrease, but the full yellow Amulet of the Fates HUD gauge rendered too long compared to an unmodified game. The owner confirmed the yellow gauge was full in *both* reference screenshots, so this was a presentation regression rather than normal resource refilling.

The old cave also overwrote nonvolatile `f30` and used temporary storage beneath `r1` without creating a stack frame. Both are ABI-sensitive choices; these facts suggested a focused repair, but the A/B experiments did **not** independently prove which detail caused the HUD artifact.

## A/B isolation

Tests were run with only one entry enabled per game relaunch:

| Enabled hooks | Magic bar | Yellow gauge rendering |
| --- | --- | --- |
| A + B | No decrease on casting | Incorrect full-bar length |
| A only | Same result as A+B | Same rendering defect |
| B only | Vanilla / no observable patch effect | Normal |
| Final revised A (register-safe) | **No decrease on casting** | **Normal in gameplay test** |

**Conclusion:** Hook A alone was sufficient for the unlimited-magic behavior. Hook B was unnecessary for the observed effect and was removed. Removing Hook B by itself did not address the display regression.

## Final implementation: preserve f30 and use an explicit stack frame

The successful revision keeps the `0x00090A38` hook and the magic assignment but correctly preserves `f30`, allocates and releases a `0x60`-byte stack frame, replays the displaced `lhz` instruction, and branches back to `0x00090A3C`. The exact 11 writes, including entry hook, are in the canonical [`imported_patch.yml`](../imported_patch.yml).

```asm
0x00090A38: b     0x004C1214
0x004C1214: stdu  r1,-0x60(r1)
0x004C1218: stfd  f30,0x28(r1)
0x004C121C: lhz   r0,0(r9)         ; replay original instruction
0x004C1220: std   r0,0x30(r1)
0x004C1224: lfd   f30,0x30(r1)
0x004C1228: fcfid f30,f30
0x004C122C: stfs  f30,0x60(r26)   ; preserve unlimited-magic behavior
0x004C1230: lfd   f30,0x28(r1)
0x004C1234: addi  r1,r1,0x60
0x004C1238: b     0x00090A3C
```

The absolute patch jump targets were checked:

- `0x00090A38` → `0x004C1214`
- `0x004C1238` → `0x00090A3C`

The revision uses the cave address range previously occupied by the original two-hook patch. **Do not activate the original port and final patch together**; their hook and cave writes overlap.

### Why this is a bounded claim

The final A-only, register-safe code was tested in-game and produced the requested unlimited-magic behavior without the earlier UI artifact. That does not prove the precise root cause was solely `f30` rather than another interaction with stack usage. The contribution is a reproducible final fix supported by differential testing, not a claim of complete source-level understanding of the game's resource system.

## Investigation of unrelated failed infinite-jump candidate

The old Artemis `Infinite Jumps` candidate used:

```asm
0x0008CDB4: stfs f0,0x60(r8)  ; original (0xD0080060)
               ->
0x0008CDB4: nop              ; 0x60000000
```

In the tested build this modification **did not enable further mid-air jumps**. Debugger inspection showed it sits inside a bounded floating-point calculation (loads, subtraction, comparisons, clamp and a store), not an obvious used-double-jump state flag. It was **excluded** from the final YAML, although the address and negative result may help future researchers avoid retracing a dead end.

Historical PS2 cheats and other God of War patches are not assumed compatible across executables or CPU architectures.

## ELF / SELF analysis limitation

The supplied `GOW2.self` was inspected as a PS3 SCE/NPDRM encrypted SELF with an embedded big-endian PowerPC64 ELF header. A standard free-license NPDRM decryption attempt did not yield a readable decrypted code segment. The above verified result comes from known original patch machine code, PPC instruction/branch checks and **live RPCS3 A/B gameplay observations**, not a claim to have fully reverse-engineered the encrypted executable.

## Installation, troubleshooting and scope

Use the project's [single combined patch YAML](../imported_patch.yml). Import through RPCS3 **Manage Game Patches**, enable this game's `Infinite Magic` entry and restart emulation. If upgrading from an earlier experiment, first disable/remove the old separate `Infinite Magic`, `TEST 1`, `TEST 2`, and A/B entries to avoid overlapping instruction writes.

Only US `BCUS98229` app `01.01` and the explicit PPU hash above are verified. A successful test of infinite magic does not establish other cheat functionality, completion-wide compatibility, or support for different dumps/updates.
