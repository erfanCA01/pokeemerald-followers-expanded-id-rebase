# Build Rules

`spritesheet_rules.mk` was deleted by upstream and is not restored. Its custom
follower sprite compilation rules have been ported into the modern build system
through the `INCGFX_*` macros.

## How the modern pipeline works

- `INCGFX_U32/U16/U8(source, extensions[, arguments])` is parsed by
  `tools/preproc` and `tools/scaninc`.
- `scaninc` generates a Makefile rule of the form
  `$(GFX) $< $@ <arguments>` for each graphics asset, so the `-mwidth`/`-mheight`
  (and any other `gbagfx`) arguments no longer need to be hand-listed in a
  `.mk` file.
- The generic Makefile rules still handle argument-less conversions:
  `%.4bpp: %.png`, `%.gbapal: %.pal`, `%.gbapal: %.png`, `%.lz: %`.

## Ported follower rules

| Asset class                          | Arguments passed via `INCGFX`      |
| ------------------------------------ | ---------------------------------- |
| Followers (species, `pokemon/*.png`) | `-mwidth 4 -mheight 4` + `.4bpp.lz` |
| Pokéballs (`misc/ball_*.png`)        | `-mwidth 2 -mheight 4`             |
| Pokéball palettes (`misc/ball_*.png`)| `.gbapal` (no args)                |
| Emotes (`graphics/misc/emotes.png`)  | `-mwidth 2 -mheight 2`             |
| People / misc / dolls / cushions     | unchanged from upstream rules      |

## Files affected

- `src/data/object_events/object_event_graphics.h`: all `INCBIN_*` graphics
  entries converted to `INCGFX_*` (follower species + ball palettes preserved).
- `src/trainer_see.c`: `sEmotion_Gfx` (custom follower emotes) converted to
  `INCGFX_U8("graphics/misc/emotes.png", ".4bpp", "-mwidth 2 -mheight 2")`.
- No explicit `.mk` rules were required; `scaninc` derives them from the
  `INCGFX` arguments automatically.

## Pokéball palettes are generated from the PNG

There are no `graphics/object_events/pics/misc/ball_*.pal` source files. Every
ball palette (`gObjectEventPal_*Ball`) must therefore be generated from the
ball image, not from a `.pal` file:

```c
const u16 gObjectEventPal_MasterBall[] = INCGFX_U16("graphics/object_events/pics/misc/ball_master.png", ".gbapal");
```

The same applies to the Substitute palette (`substitute.png`, no `.pal`).
Lighting palettes (`light.pal`, `light2.pal`, `neon_light.pal`), the emotes
palette (`graphics/misc/emotes.pal`), and the Castform form palettes
(`graphics/pokemon/castform/*/normal.pal`) *do* have dedicated `.pal` sources.

## Follower species that collide with vanilla overworld sprites

Eighteen event Pokémon exist in both upstream (as uncompressed overworld
sprites) and the follower branch (as compressed followers) under the same
`gObjectEventPic_Xxx` name. In the merged result the `gObjectEventPic_Xxx`
symbol is the **compressed follower** (`.4bpp.lz`, `-mwidth 4 -mheight 4`),
while the vanilla overworld sprite lives under a `...Old` suffix:

Azumarill, Azurill, Deoxys, Dusclops, Groudon, Kecleon, Kirlia, Kyogre,
Lugia, Mew, Pikachu, Poochyena, Rayquaza, Skitty, Sudowoodo, Vigoroth,
Wingull, Zigzagoon.

(`gObjectEventPic_Rayquaza` = compressed follower; the 8x8 cutscene sprite is
`gObjectEventPic_RayquazaCutscene`.) When converting `INCBIN_COMP`, always emit
`.4bpp.lz -mwidth 4 -mheight 4` *before* consulting upstream's `INCGFX` line,
otherwise these symbols will be wrongly treated as uncompressed vanilla art.
