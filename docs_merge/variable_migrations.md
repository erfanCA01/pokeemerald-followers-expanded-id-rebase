# Variable Migrations

This file records identifier and macro renames performed while resolving the
`upstream/master` merge conflicts. The follower branch (HEAD) carried older
asset/type conventions that have been migrated to modern pret naming.

## Asset macros: `INCBIN_*` -> `INCGFX_*`

The follower branch referenced pre-converted binary graphics (`*.4bpp`,
`*.gbapal`) through `INCBIN_U8/U16/U32/COMP`. Upstream generates these assets at
build time from source images (`.png` / `.pal`) via the `INCGFX_*` macros plus
`scaninc`. All graphics declarations were migrated accordingly.

| Old (follower)                       | New (upstream)                                          |
| ------------------------------------ | ------------------------------------------------------- |
| `INCBIN_U32("...X.4bpp")`            | `INCGFX_U32("...X.png", ".4bpp"[, "-mwidth W -mheight H"])` |
| `INCBIN_U16("...X.gbapal")`          | `INCGFX_U16("...X.pal" \| "...X.png", ".gbapal")`       |
| `INCBIN_U8("...X.4bpp")`             | `INCGFX_U8("...X.png", ".4bpp"[, args])`                |
| `INCBIN_COMP("...X.4bpp")`           | `INCGFX_U32("...X.png", ".4bpp.lz", "-mwidth 4 -mheight 4")` |

The `#define INCBIN_COMP INCBIN_U32` / `#if !(OW_GFX_COMPRESS)` conditional was
removed: follower graphics are now compressed unconditionally via the `.lz`
extension (matching the default `OW_GFX_COMPRESS TRUE` build).

## Type/parameter renames (upstream modern types)

`include/event_object_movement.h`:

| Old (follower)                               | New (upstream)                                  |
| -------------------------------------------- | ----------------------------------------------- |
| `GetWalkInPlace*MovementAction(u32)`         | `GetWalkInPlace*MovementAction(u8)`             |
| `GetSlideMovementAction(u32)`                | `GetSlideMovementAction(u8)`                    |
| `GetJumpMovementAction(u32)`                 | `GetJumpMovementAction(u8)`                     |
| `GetJump2MovementAction(u32)`                | `GetJump2MovementAction(u8)`                    |
| `GetCollisionAtCoords(..., u32 dir)`         | `GetCollisionAtCoords(..., u8 direction)`       |

Retained follower-only declarations: `GetObjectObjectCollidesWith`,
`MovementType_FollowPlayer`, and `CopySprite` (see below).

`include/field_player_avatar.h`:

| Old (follower)                                | New                                        |
| --------------------------------------------- | ------------------------------------------ |
| `GetJumpSpecialMovementAction(u32)`           | `GetJumpSpecialMovementAction(u8)`         |
| `GetPlayerAvatarGraphicsIdByStateId` (u16)    | kept as `u16` to match the merged `.c` definition |

## Renamed function

Upstream renamed `CopySprite` -> `CreateCopySpriteAt` (and changed its
allocation strategy). The follower code still uses `CopySprite` in
`ReallocSpriteTiles` for `LARGE_OW_SUPPORT`, so **both** functions are declared
and defined; `CreateCopySpriteAt` is upstream's version and `CopySprite` is the
retained follower helper.

## Script macro

Upstream introduced `applywaitmovement` as a replacement for
`applymovement ...` + `waitmovement 0`. All conflicted scripts now use
`applywaitmovement`, while retaining follower logic (`hidefollower`,
`FLAG_SAFE_FOLLOWER_MOVEMENT`, `EventScript_FollowerFieldMove`).

## Preprocessor tools

`tools/preproc/c_file.cpp` and `tools/scaninc/c_file.cpp` were reduced to
upstream's `{ "INCBIN_U8", "INCBIN_U16", "INCBIN_U32" }` identifier set; the
signed (`INCBIN_S8/S16/S32`) and `INCBIN_COMP` handling was removed because no
remaining source uses those macros (they only remain as IDE-only shims in
`include/global.h`).
