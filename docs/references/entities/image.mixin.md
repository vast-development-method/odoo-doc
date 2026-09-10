# Image Mixin (`image.mixin`)

**Transport name:** `image.mixin`  
**Storage name:** `image_mixin`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `base`

Description: Image Mixin

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `image_1920` | Image | image |  |  |
| `image_1024` | Image 1024 | image |  | related through path `image_1920` and stored |
| `image_512` | Image 512 | image |  | related through path `image_1920` and stored |
| `image_256` | Image 256 | image |  | related through path `image_1920` and stored |
| `image_128` | Image 128 | image |  | related through path `image_1920` and stored |

Machine-readable definition: `../../../schemas/data/entities/image.mixin.json`.
