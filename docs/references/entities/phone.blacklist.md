# Phone Blacklist (`phone.blacklist`)

**Transport name:** `phone.blacklist`  
**Storage name:** `phone_blacklist`  
**Kind:** persistent entity (one table)  
**Defined by package:** `phone_validation`

Description: Phone Blacklist

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`
- Display name field: `number`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `number` | Phone Number | single line text |  | required; searchable through a search rule; changes are tracked in the message thread; Help: Number should be E164 formatted |
| `active` | Active | boolean |  | default `True`; changes are tracked in the message thread |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_unique_number` | Constraint | `unique (number)` | Number already exists | `phone_validation` |

## Operations (9)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `create` | lifecycle override | self, vals_list | `phone_validation` | model_create_multi | Create new (or activate existing) blacklisted numbers.         A. Note: Attempt to create a number that already exists, but is non-active, will result in its activation.         B. Note: If the number already exists and it's active, it will be added to returned set, (it won't be re-created) Returns Recordset union of created and existing phonenumbers from the requested list of numbers to create |
| `write` | lifecycle override | self, vals | `phone_validation` |  |  |
| `_search_number` | search rule | self, operator, value | `phone_validation` |  |  |
| `add` | operation | self, number, message | `phone_validation` |  |  |
| `_add` | internal rule | self, numbers, message | `phone_validation` |  | Add or re activate a phone blacklist entry.  :param numbers: list of sanitized numbers |
| `remove` | operation | self, number, message | `phone_validation` |  |  |
| `_remove` | internal rule | self, numbers, message | `phone_validation` |  | Add de-activated or de-activate a phone blacklist entry.  :param numbers: list of sanitized numbers |
| `phone_action_blacklist_remove` | operation | self | `phone_validation` |  |  |
| `action_add` | user action | self | `phone_validation` |  |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `create` | UserError | %(error)s Please correct the number and try again. | `phone_validation` |
| `write` | UserError | %(error)s Please correct the number and try again. | `phone_validation` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| all internal users | no | no | no | no | `phone_validation` |
| `base.group_system` | yes | yes | yes | yes | `phone_validation` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `phone_validation.phone_blacklist_view_tree` | list |  | `create_date`, `number` |  |  | `phone_validation` |
| `phone_validation.phone_blacklist_view_form` | form |  | `number` | `Unblacklist`, `Blacklist` |  | `phone_validation` |
| `phone_validation.phone_blacklist_view_search` | search |  | `number` |  | `Archived` | `phone_validation` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `phone_validation.phone_blacklist_action` | Blacklisted Phone Numbers |  |  |  |  | `phone_validation` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `mass_mailing_sms.phone_blacklist_menu` | Blacklisted Phone Numbers | `mass_mailing_sms_menu_configuration` | `phone_validation.phone_blacklist_action` | 1 | `mass_mailing.group_mass_mailing_user` |

Machine-readable definition: `../../../schemas/data/entities/phone.blacklist.json`; views: `../../../schemas/interfaces/views/phone.blacklist.json`.
