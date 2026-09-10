# Onboarding Step (`onboarding.onboarding.step`)

**Transport name:** `onboarding.onboarding.step`  
**Storage name:** `onboarding_onboarding_step`  
**Kind:** persistent entity (one table)  
**Defined by package:** `onboarding`  
**Extended by packages:** `account`

Description: Onboarding Step

## Identity and behavior

- Default ordering: `sequence asc, id asc`
- Display name field: `title`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (15)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `onboarding_ids` | Onboardings | many to many | `onboarding.onboarding` |  |
| `title` | Title | single line text |  | translatable |
| `description` | Description | single line text |  | translatable |
| `button_text` | Button text | single line text |  | required; default computed dynamically (lambda s: s.env._("Let's do it")); translatable; Help: Text on the panel's button to start this step |
| `done_icon` | Font Awesome Icon when completed | single line text |  | default `fa-star` |
| `done_text` | Text to show when step is completed | single line text |  | default computed dynamically (lambda s: s.env._('Step Completed!')); translatable |
| `step_image` | Step Image | binary |  |  |
| `step_image_filename` | Step Image Filename | single line text |  |  |
| `step_image_alt` | Alt Text for the Step Image | single line text |  | default `Onboarding Step Image`; translatable; Help: Show when impossible to load the image |
| `panel_step_open_action_name` | Opening action | single line text |  | Help: Name of the onboarding step model action to execute when opening the step, e.g. action_open_onboarding_1_step_1 |
| `current_progress_step_id` | Step Progress | many to one | `onboarding.progress.step` | computed by rule `_compute_current_progress` (not stored); Help: Onboarding Progress Step for the current context (company). |
| `current_step_state` | Completion State | selection |  | computed by rule `_compute_current_progress` (not stored) |
| `progress_ids` | Onboarding Progress Step Records | one to many | `onboarding.progress.step` | read only; inverse field `step_id`; Help: All related Onboarding Progress Step Records (across companies) |
| `is_per_company` | Is per company | boolean |  | default `True` |
| `sequence` | Sequence | integer |  | default `10` |

## State fields

State machine fields of this entity: `current_step_state`. Transitions are specified in the domain documents.

## Operations (15)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_current_progress` | computation | self | `onboarding` | depends_context: `company`; depends: `progress_ids`, `progress_ids.step_state` |  |
| `check_step_on_onboarding_has_action` | validation | self | `onboarding` | constrains: `onboarding_ids` |  |
| `write` | lifecycle override | self, vals | `onboarding` |  |  |
| `action_set_just_done` | user action | self | `onboarding` |  |  |
| `action_validate_step` | user action | self, xml_id | `onboarding` | model |  |
| `_get_placeholder_filename` | preparation rule | self, field | `onboarding` | model |  |
| `_create_progress_steps` | internal rule | self | `onboarding` |  | Create progress step records as necessary to validate steps.  Only considers existing `onboarding.progress` records for the current company or without company (depending on `is_per_company`). |
| `action_open_step_company_data` | user action | self | `account` | model | Set company's basic information. |
| `action_open_step_base_document_layout` | user action | self | `account` | model |  |
| `action_validate_step_base_document_layout` | user action | self | `account` | model | Set the onboarding(s) step as done only if layout is set. |
| `action_open_step_bank_account` | user action | self | `account` | model |  |
| `action_open_step_create_invoice` | user action | self | `account` | model |  |
| `action_open_step_fiscal_year` | user action | self | `account` | model |  |
| `action_open_step_chart_of_accounts` | user action | self | `account` | model | Called by the 'Chart of Accounts' button of the dashboard onboarding panel. |
| `action_open_step_sales_tax` | user action | self | `account` | model |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `check_step_on_onboarding_has_action` | ValidationError | An "Opening Action" is required for the following steps to be linked to an onboarding panel: %(step_titles)s | `onboarding` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| all internal users | no | no | no | no | `onboarding` |
| `base.group_user` | no | no | no | no | `onboarding` |
| `base.group_system` | yes | yes | yes | yes | `onboarding` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `onboarding.onboarding_onboarding_step_view_tree` | list |  | `sequence`, `title`, `onboarding_ids`, `current_step_state`, `is_per_company` |  |  | `onboarding` |
| `onboarding.onboarding_onboarding_step_view_form` | form |  | `title`, `current_step_state`, `panel_step_open_action_name`, `is_per_company`, `onboarding_ids`, `description`, `button_text`, `done_text`, `done_icon`, `step_image_filename`, `step_image`, `step_image_alt` |  |  | `onboarding` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `onboarding.action_view_onboarding_step` | Onboarding Steps | list,form |  |  |  | `onboarding` |

Machine-readable definition: `../../../schemas/data/entities/onboarding.onboarding.step.json`; views: `../../../schemas/interfaces/views/onboarding.onboarding.step.json`.
