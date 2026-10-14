# Struct Compatibility

The merge was performed with vanilla Emerald save interchangeability and
multiplayer link integrity in mind.

## SaveBlock / link packet layout

- **No changes** were made to `SaveBlock1` / `SaveBlock2` layout, field offsets,
  or alignment during this merge.
- **No changes** were made to any struct transmitted over the multiplayer link
  protocol.
- Follower state continues to live entirely in the isolated/padding regions
  previously established by the follower branch; none of those offsets were
  shifted.

## `include/global.h` / `include/event_objects.h`

- No struct definitions or `#pragma pack`/alignment pragmas were modified.
- `OW_GFX_COMPRESS`, `LARGE_OW_SUPPORT`, `OW_MON_POKEBALLS`,
  `OW_MON_SCRIPT_MOVEMENT`, and the expanded object-event graphics IDs
  (`OBJ_EVENT_GFX_SPECIES`, `OBJ_EVENT_GFX_MON_BASE`, `NUM_OBJ_EVENT_GFX`)
  remain unchanged.

## Function signature notes

- `GetPlayerAvatarGraphicsIdByStateId` and
  `GetPlayerAvatarGraphicsIdByStateIdAndGender` retain their `u16` return type
  in the merged translation unit (the `.c` file was not part of the conflict),
  so the header declaration was kept as `u16` to avoid a declaration/definition
  mismatch. This does not affect memory layout.
- `GetObjectObjectCollidesWith` remains `u32` (follower helper); upstream's
  `GetCollisionAtCoords` uses `u8 direction` as normal.

## Conclusion

The merged result preserves the vanilla byte boundary expectations for save
data and multiplayer packets.
