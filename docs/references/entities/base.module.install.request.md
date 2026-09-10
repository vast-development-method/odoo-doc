# Module Activation Request (`base.module.install.request`)

**Transport name:** `base.module.install.request`  
**Storage name:** `base_module_install_request`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `base_install_request`

Description: Module Activation Request

## Identity and behavior

- Display name field: `module_id`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `module_id` | Module | many to one | `ir.module.module` | required; read only; on delete of the target: cascade; restricted by domain `[["state", "=", "uninstalled"]]` |
| `user_id` | User | many to one | `res.users` | required; default computed dynamically (lambda self: self.env.user) |
| `user_ids` | Send to: | many to many | `res.users` | computed by rule `_compute_user_ids` (not stored) |
| `body_html` | Body | rich text |  |  |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_user_ids` | computation | self | `base_install_request` | depends: `module_id` |  |
| `action_send_request` | user action | self | `base_install_request` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | no | `base_install_request` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base_install_request.base_module_install_request_view_form` | form |  | `module_id`, `user_ids`, `body_html` | `Request Activation`, `Cancel` |  | `base_install_request` |

## Email templates

| Template | Name | Subject |
|---|---|---|
| `base_install_request.mail_template_base_install_request` | Mail: Install Request | Module Activation Request for "{{ object.module_id.shortdesc }}" |

Machine-readable definition: `../../../schemas/data/entities/base.module.install.request.json`; views: `../../../schemas/interfaces/views/base.module.install.request.json`.
