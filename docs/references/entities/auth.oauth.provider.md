# OAuth2 provider (`auth.oauth.provider`)

**Transport name:** `auth.oauth.provider`  
**Storage name:** `auth_oauth_provider`  
**Kind:** persistent entity (one table)  
**Defined by package:** `auth_oauth`

Description: OAuth2 provider

## Identity and behavior

- Default ordering: `sequence, name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (10)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Provider name | single line text |  | required |
| `client_id` | Client identifier | single line text |  |  |
| `auth_endpoint` | Authorization uniform resource locator | single line text |  | required |
| `scope` | Scope | single line text |  | default `openid profile email` |
| `validation_endpoint` | UserInfo uniform resource locator | single line text |  | required |
| `data_endpoint` | Data Endpoint | single line text |  |  |
| `enabled` | Allowed | boolean |  |  |
| `css_class` | CSS class | single line text |  | default `fa fa-fw fa-sign-in text-primary` |
| `body` | Login button label | single line text |  | required; translatable; Help: Link text in Login Dialog |
| `sequence` | Sequence | integer |  | default `10` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `auth_oauth` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `auth_oauth.view_oauth_provider_form` | form |  | `name`, `client_id`, `enabled`, `body`, `css_class`, `auth_endpoint`, `scope`, `validation_endpoint`, `data_endpoint` |  |  | `auth_oauth` |
| `auth_oauth.view_oauth_provider_tree` | list |  | `sequence`, `name`, `client_id`, `enabled` |  |  | `auth_oauth` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `auth_oauth.action_oauth_provider` | Providers | list,form |  |  |  | `auth_oauth` |

Machine-readable definition: `../../../schemas/data/entities/auth.oauth.provider.json`; views: `../../../schemas/interfaces/views/auth.oauth.provider.json`.
