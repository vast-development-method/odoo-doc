# Board (`board.board`)

**Transport name:** `board.board`  
**Storage name:** `board_board`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `board`

Description: Board

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `id` | Identifier | identifier |  |  |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `create` | lifecycle override | self, vals_list | `board` | model_create_multi |  |
| `get_view` | lifecycle override | self, view_id, view_type, **options | `board` | model | Overrides orm field_view_get. @return: Dictionary of Fields, arch and toolbar. |
| `_arch_preprocessing` | internal rule | self, arch | `board` | model |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `board` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `board.board_my_dash_view` | form |  |  |  |  | `board` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `board.open_board_my_dash_action` | My Dashboard | form |  | `{'disable_toolbar': True}` |  | `board` |

Machine-readable definition: `../../../schemas/data/entities/board.board.json`; views: `../../../schemas/interfaces/views/board.board.json`.
