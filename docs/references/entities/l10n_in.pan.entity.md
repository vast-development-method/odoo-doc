# Indian permanent account number Entity (`l10n_in.pan.entity`)

**Transport name:** `l10n_in.pan.entity`  
**Storage name:** `l10n_in_pan_entity`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_in`

Description: Indian PAN Entity

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `mail.activity.mixin`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (8)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | permanent account number | single line text |  | required; changes are tracked in the message thread |
| `type` | Type | selection |  | read only; computed by rule `_compute_type` and stored |
| `partner_ids` | Partners | one to many | `res.partner` | restricted by domain `[('l10n_in_pan_entity_id', '=', False), '\|', ('vat', '=', False), ('vat', 'like', name)]`; inverse field `l10n_in_pan_entity_id` |
| `tds_deduction` | tax deducted at source Deduction | selection |  | default `normal`; changes are tracked in the message thread |
| `tds_certificate` | tax deducted at source Certificate | binary |  | not copied on duplication |
| `tds_certificate_filename` | tax deducted at source Certificate Filename | single line text |  | not copied on duplication |
| `msme_type` | MSME/Udyam Registration Type | selection |  | not copied on duplication |
| `msme_number` | MSME/Udyam Registration Number | single line text |  | not copied on duplication |

## Selection values

### `type` (Type)

| Value | Label |
|---|---|
| `a` | Association of Persons |
| `b` | Body of Individuals |
| `c` | Company |
| `f` | Firms |
| `g` | Government |
| `h` | Hindu Undivided Family |
| `j` | Artificial Judicial Person |
| `l` | Local Authority |
| `p` | Individual |
| `t` | Association of Persons for a Trust |
| `k` | Krish (Trust Krish) |

### `tds_deduction` (tax deducted at source Deduction)

| Value | Label |
|---|---|
| `normal` | Normal |
| `lower` | Lower |
| `higher` | Higher |
| `no` | No |

### `msme_type` (MSME/Udyam Registration Type)

| Value | Label |
|---|---|
| `micro` | Micro |
| `small` | Small |
| `medium` | Medium |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_uniq` | Constraint | `unique (name)` | A PAN Entity with same PAN Number already exists. | `l10n_in` |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_pan_name` | validation | self | `l10n_in` | constrains: `name` |  |
| `create` | lifecycle override | self, vals_list | `l10n_in` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `l10n_in` |  |  |
| `_compute_type` | computation | self | `l10n_in` | depends: `name` |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_pan_name` | ValidationError | The entered PAN %s seems invalid. Please enter a valid PAN. | `l10n_in` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | yes | `l10n_in` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_in.l10n_in_pan_entity_view_form` | form |  | `name`, `type`, `partner_ids`, `tds_deduction`, `tds_certificate`, `tds_certificate_filename`, `msme_type`, `msme_number` |  |  | `l10n_in` |
| `l10n_in.l10n_in_pan_entity_view_tree` | list |  | `name`, `type`, `partner_ids` |  |  | `l10n_in` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `l10n_in.l10n_in_pan_entity_action` | PAN Entity | list,form |  |  |  | `l10n_in` |

Machine-readable definition: `../../../schemas/data/entities/l10n_in.pan.entity.json`; views: `../../../schemas/interfaces/views/l10n_in.pan.entity.json`.
