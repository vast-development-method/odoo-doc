# Snooze Orderpoint (`stock.orderpoint.snooze`)

**Transport name:** `stock.orderpoint.snooze`  
**Storage name:** `stock_orderpoint_snooze`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `stock`

Description: Snooze Orderpoint

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `orderpoint_ids` | Orderpoint | many to many | `stock.warehouse.orderpoint` |  |
| `predefined_date` | Snooze for | selection |  | default `day` |
| `snoozed_until` | Snooze Date | date |  |  |

## Selection values

### `predefined_date` (Snooze for)

| Value | Label |
|---|---|
| `day` | 1 Day |
| `week` | 1 Week |
| `month` | 1 Month |
| `custom` | Custom |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_onchange_predefined_date` | on change | self | `stock` | onchange: `predefined_date` |  |
| `action_snooze` | user action | self | `stock` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `stock.group_stock_user` | yes | yes | yes | yes | `stock` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `stock.view_stock_orderpoint_snooze` | form |  | `orderpoint_ids`, `predefined_date`, `snoozed_until` | `Snooze`, `Discard` |  | `stock` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `stock.action_orderpoint_snooze` | Snooze | form |  |  | new | `stock` |

Machine-readable definition: `../../../schemas/data/entities/stock.orderpoint.snooze.json`; views: `../../../schemas/interfaces/views/stock.orderpoint.snooze.json`.
