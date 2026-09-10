# Wizard to Split Multiple Productions (`mrp.production.split.multi`)

**Transport name:** `mrp.production.split.multi`  
**Storage name:** `mrp_production_split_multi`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `mrp`

Description: Wizard to Split Multiple Productions

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `production_ids` | Productions To Split | one to many | `mrp.production.split` | inverse field `production_split_multi_id` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mrp.group_mrp_user` | yes | yes | yes | no | `mrp` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mrp.view_mrp_production_split_multi_form` | form |  | `production_ids`, `production_id`, `product_id`, `product_qty`, `production_capacity`, `product_uom_id` | `action_prepare_split`, `Discard` |  | `mrp` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mrp.action_mrp_production_split_multi` | Split productions | form |  |  | new | `mrp` |

Machine-readable definition: `../../../schemas/data/entities/mrp.production.split.multi.json`; views: `../../../schemas/interfaces/views/mrp.production.split.multi.json`.
