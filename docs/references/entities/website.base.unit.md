# Unit of Measure for price per unit on eCommerce products. (`website.base.unit`)

**Transport name:** `website.base.unit`  
**Storage name:** `website_base_unit`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_sale`

Description: Unit of Measure for price per unit on eCommerce products.

## Identity and behavior

- Default ordering: `name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable; Help: Define a custom unit to display in the price per unit of measure field. |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_public` | no | yes | no | no | `website_sale` |
| `base.group_portal` | no | yes | no | no | `website_sale` |
| `base.group_user` | no | yes | no | no | `website_sale` |
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `website_sale` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website_sale.base_unit_action` | Base Units | list,form |  |  |  | `website_sale` |

Machine-readable definition: `../../../schemas/data/entities/website.base.unit.json`.
