# Paper Format Config (`report.paperformat`)

**Transport name:** `report.paperformat`  
**Storage name:** `report_paperformat`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`

Description: Paper Format Config

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (18)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `default` | Default paper format? | boolean |  |  |
| `format` | Paper size | selection |  | default `A4`; Help: Select Proper Paper size |
| `margin_top` | Top Margin (mm) | float |  | default `40` |
| `margin_bottom` | Bottom Margin (mm) | float |  | default `20` |
| `margin_left` | Left Margin (mm) | float |  | default `7` |
| `margin_right` | Right Margin (mm) | float |  | default `7` |
| `page_height` | Page height (mm) | integer |  | default  |
| `page_width` | Page width (mm) | integer |  | default  |
| `orientation` | Orientation | selection |  | default `Landscape` |
| `header_line` | Display a header line | boolean |  | default  |
| `header_spacing` | Header spacing | integer |  | default `35` |
| `disable_shrinking` | Disable smart shrinking | boolean |  |  |
| `dpi` | Output DPI | integer |  | required; default `90` |
| `report_ids` | Associated reports | one to many | `ir.actions.report` | inverse field `paperformat_id`; Help: Explicitly associated reports |
| `print_page_width` | Print page width (mm) | float |  | computed by rule `_compute_print_page_size` (not stored) |
| `print_page_height` | Print page height (mm) | float |  | computed by rule `_compute_print_page_size` (not stored) |
| `css_margins` | Use css margins | boolean |  | default  |

## Selection values

### `orientation` (Orientation)

| Value | Label |
|---|---|
| `Landscape` | Landscape |
| `Portrait` | Portrait |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_format_or_page` | validation | self | `base` | constrains: `format` |  |
| `_compute_print_page_size` | computation | self | `base` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_format_or_page` | ValidationError | You can select either a format or a specific page width/height, but not both. | `base` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_user` | no | yes | no | no | `base` |
| `group_system` | yes | yes | yes | yes | `base` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.paperformat_view_tree` | list |  | `name` |  |  | `base` |
| `base.paperformat_view_form` | form |  | `name`, `format`, `page_height`, `page_width`, `orientation`, `margin_top`, `margin_bottom`, `margin_left`, `margin_right`, `header_line`, `header_spacing`, `disable_shrinking`, `dpi`, `report_ids` |  |  | `base` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.paper_format_action` | Paper Format General Configuration | list,form |  |  |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/report.paperformat.json`; views: `../../../schemas/interfaces/views/report.paperformat.json`.
