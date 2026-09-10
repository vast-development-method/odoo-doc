# E-Commerce Extra Info Shown on product page (`website.sale.extra.field`)

**Transport name:** `website.sale.extra.field`  
**Storage name:** `website_sale_extra_field`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_sale`

Description: E-Commerce Extra Info Shown on product page

## Identity and behavior

- Default ordering: `sequence`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `website_id` | Website | many to one | `website` | indexed (btree_not_null) |
| `sequence` | Sequence | integer |  | default `10` |
| `field_id` | Field | many to one | `ir.model.fields` | required; on delete of the target: cascade; restricted by domain `[["model_id.model", "=", "product.template"], ["ttype", "in", ["char", "binary"]]]` |
| `label` | Label | single line text |  | related through path `field_id.field_description` |
| `name` | Name | single line text |  | related through path `field_id.name` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_public` | no | yes | no | no | `website_sale` |
| `base.group_portal` | no | yes | no | no | `website_sale` |
| `base.group_user` | no | yes | no | no | `website_sale` |
| `website.group_website_restricted_editor` | yes | yes | yes | yes | `website_sale` |

Machine-readable definition: `../../../schemas/data/entities/website.sale.extra.field.json`.
