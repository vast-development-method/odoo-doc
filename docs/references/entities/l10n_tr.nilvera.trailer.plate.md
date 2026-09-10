# GİB Plate numbers (`l10n_tr.nilvera.trailer.plate`)

**Transport name:** `l10n_tr.nilvera.trailer.plate`  
**Storage name:** `l10n_tr_nilvera_trailer_plate`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_tr_nilvera_edispatch`

Description: GİB Plate numbers

## Identity and behavior

- Default ordering: `name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | GİB Plate Number | single line text |  |  |
| `plate_number_type` | Plate Number | selection |  | required |

## Selection values

### `plate_number_type` (Plate Number)

| Value | Label |
|---|---|
| `vehicle` | Vehicle |
| `trailer` | Plate |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_uniq` | Constraint | `unique(name,plate_number_type)` | A Plate Number with that type already exists. | `l10n_tr_nilvera_edispatch` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `stock.group_stock_user` | yes | yes | yes | yes | `l10n_tr_nilvera_edispatch` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_tr_nilvera_edispatch.l10n_tr_nilvera_trailer_plate_view_tree` | list |  | `name`, `plate_number_type` |  |  | `l10n_tr_nilvera_edispatch` |
| `l10n_tr_nilvera_edispatch.l10n_tr_nilvera_trailer_plate_view_form` | form |  | `plate_number_type`, `name` |  |  | `l10n_tr_nilvera_edispatch` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `l10n_tr_nilvera_edispatch.action_l10n_tr_nilvera_trailer_plate` | GİB Plate Numbers | list,form |  |  |  | `l10n_tr_nilvera_edispatch` |

Machine-readable definition: `../../../schemas/data/entities/l10n_tr.nilvera.trailer.plate.json`; views: `../../../schemas/interfaces/views/l10n_tr.nilvera.trailer.plate.json`.
