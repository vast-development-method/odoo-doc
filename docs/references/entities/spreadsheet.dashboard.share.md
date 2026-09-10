# Copy of a shared dashboard (`spreadsheet.dashboard.share`)

**Transport name:** `spreadsheet.dashboard.share`  
**Storage name:** `spreadsheet_dashboard_share`  
**Kind:** persistent entity (one table)  
**Defined by package:** `spreadsheet_dashboard`

Description: Copy of a shared dashboard

## Identity and behavior

- Mixins (classical inheritance): `spreadsheet.mixin`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `dashboard_id` | Dashboard | many to one | `spreadsheet.dashboard` | required; on delete of the target: cascade |
| `excel_export` | Excel Export | binary |  |  |
| `access_token` | Access Token | single line text |  | required; default computed dynamically (lambda _x: str(uuid.uuid4())) |
| `full_url` | uniform resource locator | single line text |  | computed by rule `_compute_full_url` (not stored) |
| `name` | Name | single line text |  | related through path `dashboard_id.name` |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_full_url` | computation | self | `spreadsheet_dashboard` | depends: `access_token` |  |
| `action_get_share_url` | user action | self, vals | `spreadsheet_dashboard` | model |  |
| `_check_token` | validation | self, access_token | `spreadsheet_dashboard` |  |  |
| `_check_dashboard_access` | validation | self, access_token | `spreadsheet_dashboard` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | yes | `spreadsheet_dashboard` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| spreadsheet.dashboard.share: create uid | `[(4, ref('base.group_user'))]` | `[('create_uid', '=', user.id)]` | True | True | True | True |

Machine-readable definition: `../../../schemas/data/entities/spreadsheet.dashboard.share.json`.
