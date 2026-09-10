# Wizard to mark as done or create back order (`mrp.production.backorder`)

**Transport name:** `mrp.production.backorder`  
**Storage name:** `mrp_production_backorder`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `mrp`

Description: Wizard to mark as done or create back order

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `mrp_production_ids` | Manufacturing Production | many to many | `mrp.production` |  |
| `mrp_production_backorder_line_ids` | Backorder Confirmation Lines | one to many | `mrp.production.backorder.line` | inverse field `mrp_production_backorder_id` |
| `show_backorder_lines` | Show backorder lines | boolean |  | computed by rule `_compute_show_backorder_lines` (not stored) |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_show_backorder_lines` | computation | self | `mrp` | depends: `mrp_production_backorder_line_ids` |  |
| `action_close_mo` | user action | self | `mrp` |  |  |
| `action_backorder` | user action | self | `mrp` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mrp.group_mrp_user` | yes | yes | yes | no | `mrp` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mrp.view_mrp_production_backorder_form` | form |  | `show_backorder_lines`, `mrp_production_backorder_line_ids`, `mrp_production_id`, `to_backorder` | `Create backorder`, `Validate`, `No Backorder`, `Discard` |  | `mrp` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mrp.action_mrp_production_backorder` | You produced less than the initial demand | form |  |  | new | `mrp` |

Machine-readable definition: `../../../schemas/data/entities/mrp.production.backorder.json`; views: `../../../schemas/interfaces/views/mrp.production.backorder.json`.
