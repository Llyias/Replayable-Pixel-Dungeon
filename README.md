# Replayable Pixel Dungeon

[Replayable Pixel Dungeon](https://github.com/Llyias/Replayable-Pixel-Dungeon) is a work-in-progress fork of
[Shattered Pixel Dungeon](https://shatteredpixel.com/shatteredpd/) focused on
making seeded runs deterministic enough to record, replay, compare, and later
reuse in [Versus-Pixel-Dungeon](https://github.com/qkayqkay/versus-pixel-dungeon) speedrun races.

This is not a finished replay viewer yet. The current focus is deterministic
gameplay groundwork: separating RNG streams, saving their state, and reducing
avoidable divergence between runs that start from the same seed.

## Project Goals

- Preserve Shattered Pixel Dungeon gameplay unless a replay or fairness fix needs
  a targeted change.
- Make same-seed runs more reproducible even when players kill mobs in different
  orders.
- Record player decisions at the game-action level, not raw input frames.
- Detect replay divergence with deterministic checksums.
- Prepare these systems for VS-Pixel-Dungeon, where multiple players race on the
  same seed.

## Current Status

The first pass of loot RNG isolation is implemented, along with debug-only tools
for manual replay/fairness testing.

Implemented RNG work:

- Main gameplay RNG state is saved and restored.
- Visual RNG is separated so cosmetic effects do not affect gameplay RNG.
- Bones RNG is separated from normal gameplay RNG.
- Level mob spawn RNG is separated from the global gameplay stream.
- Mob drop RNG is managed through deterministic keyed streams in
  `DropRNGManager`.
- Basic mob drop checks and loot creation use per-mob-class drop streams.
- Swarm drop streams account for swarm generation.
- Special mob extra drops now use dedicated streams for count, generated loot, or
  placement where needed:
  - `Eye`
  - `CausticSlime`
  - `DM201`
  - `SpectralNecromancer`
  - `Goo`
  - `DM300`
  - `HermitCrab`
  - `GnollExile`

The practical goal is that killing the same class/order of mob under the same
seed should produce the same drop result even if other mob classes were killed
in a different order.

Debug/testing support:

- Debug starts grant local test equipment, including `CMD`, boosted rings, a
  wand of blast wave, and mind vision potions.
- `CMD` is a debug-only command item. It currently supports:
  - mob summoning by class name, such as `Eye` or `/summon Eye`;
  - main-path floor jumps with `/floor N`.
- `/floor N` uses the normal interlevel transition flow and pre-generates
  skipped main-path floors before loading the target floor. This preserves more
  of the level generation and global item-generation state than directly
  creating only the target floor.

## Important Notes

- This work separates RNG calls used for drops; it does not try to redesign
  vanilla item generation rules such as artifact uniqueness, limited drops, or
  generator deck behavior.
- Some non-loot systems still use main RNG and may affect full replay stability.
  Examples include Sacrificial Fire progress and some player-driven bonus
  effects.
- Debug tools such as `CMD` are development helpers gated behind debug mode.
  They are not intended to become normal release features.

## Building

The upstream build guides still apply:

- [Compiling for Android](docs/getting-started-android.md)
- [Compiling for desktop platforms](docs/getting-started-desktop.md)
- [Compiling for iOS](docs/getting-started-ios.md)

For quick desktop testing from the repository root:

```sh
./gradlew desktop:debug
```

On Windows PowerShell:

```powershell
.\gradlew.bat desktop:debug
```

To compile core Java code without launching the game:

```powershell
.\gradlew.bat :core:compileJava
```

## TODO

### Near Term

- Port the completed loot RNG isolation work into VS-Pixel-Dungeon.
- Decide which debug/test tools should also be ported into VS-Pixel-Dungeon for
  development builds.
- Add simple repeatable same-seed tests for mob loot consistency.
- Add manual test notes for `CMD` workflows, especially `/summon` and `/floor`.

### RNG Isolation

- Audit common bonus drop/effect systems:
  - Ring of Wealth bonus drops.
  - Lucky enchantment bonus loot.
  - Soul Eater death proc.
  - Trinkets or other effects that modify drop behavior.
- Decide whether Sacrificial Fire should get a dedicated RNG stream for its
  sacrifice multiplier.
- Continue separating RNG only where it protects determinism or fairness; avoid
  broad rewrites that change vanilla balance.

### Replay System

- Design and implement a replay file format.
- Store replay metadata: game version, git commit, seed, custom seed text,
  challenges, hero class, replay format version, and initial game state.
- Record normalized hero actions such as move, wait, attack, pickup, drop, use,
  zap, throw, choose option, interact, and descend.
- Add a playback mode that injects recorded actions only when the game is waiting
  for hero input.
- Add deterministic checksums after hero actions to detect the first divergent
  turn.
- Decide the item reference strategy for replay actions:
  - short term: class name plus container/location fallback;
  - long term: stable item replay IDs if action references need to survive stack
    merges, splits, inventory sorting, and save/load boundaries.

### VS-Pixel-Dungeon Integration

- Define the deterministic run data needed by VS-Pixel-Dungeon.
- Decide how race clients validate same-seed fairness.
- Expose replay/checksum data in a form that can be compared across players.
- Keep the replay system independent from networking so it can be tested in
  single-player first.

## Upstream Project

This project is based on Shattered Pixel Dungeon, an open-source traditional
roguelike dungeon crawler by Evan Debenham, itself based on Pixel Dungeon by
Watabou.

Official Shattered Pixel Dungeon repository:
https://github.com/00-Evan/shattered-pixel-dungeon

Official website:
https://shatteredpixel.com/shatteredpd/

This fork is not an official Shattered Pixel Dungeon release.

## License

This project follows the upstream Shattered Pixel Dungeon license. See
[LICENSE.txt](LICENSE.txt).
