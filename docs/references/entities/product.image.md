# Product Image (`product.image`)

**Transport name:** `product.image`  
**Storage name:** `product_image`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_sale`

Description: Product Image

## Identity and behavior

- Mixins (classical inheritance): `image.mixin`
- Default ordering: `sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (8)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `sequence` | Sequence | integer |  | default `10` |
| `image_1920` | Image 1920 | image |  |  |
| `product_tmpl_id` | Product Template | many to one | `product.template` | indexed; on delete of the target: cascade |
| `product_variant_id` | Product Variant | many to one | `product.product` | indexed; on delete of the target: cascade |
| `video_url` | Video uniform resource locator | single line text |  | Help: URL of a video for showcasing your product. |
| `embed_code` | Embed Code | rich text |  | computed by rule `_compute_embed_code` (not stored) |
| `can_image_1024_be_zoomed` | Can Image 1024 be zoomed | boolean |  | computed by rule `_compute_can_image_1024_be_zoomed` and stored |

## Operations (5)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_can_image_1024_be_zoomed` | computation | self | `website_sale` | depends: `image_1920`, `image_1024` |  |
| `_compute_embed_code` | computation | self | `website_sale` | depends: `video_url` |  |
| `_onchange_video_url` | on change | self | `website_sale` | onchange: `video_url` |  |
| `_check_valid_video_url` | validation | self | `website_sale` | constrains: `video_url` |  |
| `create` | lifecycle override | self, vals_list | `website_sale` | model_create_multi | We don't want the default_product_tmpl_id from the context to be applied if we have a product_variant_id set to avoid having the variant images to show also as template images. But we want it if we don't have a product_variant_id set. |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_valid_video_url` | ValidationError | Provided video URL for '%s' is not valid. Please enter a valid video URL. | `website_sale` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_public` | no | yes | no | no | `website_sale` |
| `base.group_portal` | no | yes | no | no | `website_sale` |
| `base.group_user` | no | yes | no | no | `website_sale` |
| `website.group_website_restricted_editor` | yes | yes | yes | yes | `website_sale` |
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `website_sale` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website_sale.view_product_image_form` | form |  | `sequence`, `name`, `video_url`, `image_1920`, `embed_code` |  |  | `website_sale` |
| `website_sale.product_image_view_kanban` | kanban |  | `video_url`, `sequence`, `image_1920`, `name` |  |  | `website_sale` |

Machine-readable definition: `../../../schemas/data/entities/product.image.json`; views: `../../../schemas/interfaces/views/product.image.json`.
