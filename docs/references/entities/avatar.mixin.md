# Avatar Mixin (`avatar.mixin`)

**Transport name:** `avatar.mixin`  
**Storage name:** `avatar_mixin`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `base`

Description: Avatar Mixin

## Identity and behavior

- Mixins (classical inheritance): `image.mixin`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `avatar_1920` | Avatar | image |  | computed by rule `_compute_avatar_1920` (not stored) |
| `avatar_1024` | Avatar 1024 | image |  | computed by rule `_compute_avatar_1024` (not stored) |
| `avatar_512` | Avatar 512 | image |  | computed by rule `_compute_avatar_512` (not stored) |
| `avatar_256` | Avatar 256 | image |  | computed by rule `_compute_avatar_256` (not stored) |
| `avatar_128` | Avatar 128 | image |  | computed by rule `_compute_avatar_128` (not stored) |

## Operations (10)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_avatar` | computation | self, avatar_field, image_field | `base` |  |  |
| `_compute_avatar_1920` | computation | self | `base` | depends: |  |
| `_compute_avatar_1024` | computation | self | `base` | depends: |  |
| `_compute_avatar_512` | computation | self | `base` | depends: |  |
| `_compute_avatar_256` | computation | self | `base` | depends: |  |
| `_compute_avatar_128` | computation | self | `base` | depends: |  |
| `_avatar_generate_svg` | internal rule | self | `base` |  |  |
| `_avatar_get_placeholder_path` | internal rule | self | `base` |  |  |
| `_avatar_get_placeholder` | internal rule | self | `base` |  |  |
| `_get_avatar_128_access_token` | preparation rule | self | `base` |  | Return a scoped access token for the `avatar_128` field. The token can be used with `ir_binary._find_record` to bypass access rights.  :rtype: str |

Machine-readable definition: `../../../schemas/data/entities/avatar.mixin.json`.
