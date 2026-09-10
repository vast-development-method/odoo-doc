# Italian Document Type (`l10n_it.document.type`)

**Transport name:** `l10n_it.document.type`  
**Storage name:** `l10n_it_document_type`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_it_edi`

Description: Italian Document Type

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable; Help: The document type name |
| `code` | Code | single line text |  | required |
| `type` | Type | selection |  |  |

## Selection values

### `type` (Type)

| Value | Label |
|---|---|
| `sale` | Sale |
| `purchase` | Purchase |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `l10n_it_edi` |  |  |
| `_check_code_unique` | validation | self | `l10n_it_edi` | constrains: `code` |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_code_unique` | ValidationError | Document Type code must be unique. | `l10n_it_edi` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | yes | `l10n_it_edi` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_it_edi.l10n_it_document_type_tree` | list |  | `code`, `name` |  |  | `l10n_it_edi` |
| `l10n_it_edi.l10n_it_document_type_form` | form |  | `name`, `code` |  |  | `l10n_it_edi` |

Machine-readable definition: `../../../schemas/data/entities/l10n_it.document.type.json`; views: `../../../schemas/interfaces/views/l10n_it.document.type.json`.
