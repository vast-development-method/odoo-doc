# Contract Template Wizard (`hr.version.wizard`)

**Transport name:** `hr.version.wizard`  
**Storage name:** `hr_version_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `hr`

Description: Contract Template Wizard

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `contract_template_id` | Contract Template | many to one | `hr.version` | required; visible only to groups `hr.group_hr_user`; restricted by domain `lambda self: [('company_id', '=', self.env.company.id), ('employee_id', '=', False)]`; Help: Select a contract template to auto-fill the contract form with predefined values. You can still edit the fields as needed after applying the template. |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `action_load_template` | user action | self | `hr` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_hr_user` | yes | yes | yes | no | `hr` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr.hr_version_wizard_view_form` | form |  | `contract_template_id` | `Load`, `Discard` |  | `hr` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr.hr_version_wizard_action` | Contract Template Load | form |  |  | new | `hr` |

Machine-readable definition: `../../../schemas/data/entities/hr.version.wizard.json`; views: `../../../schemas/interfaces/views/hr.version.wizard.json`.
