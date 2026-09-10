# Print Pre-numbered Checks (`print.prenumbered.checks`)

**Transport name:** `print.prenumbered.checks`  
**Storage name:** `print_prenumbered_checks`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `account_check_printing`

Description: Print Pre-numbered Checks

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `next_check_number` | Next Check Number | single line text |  | required |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_next_check_number` | validation | self | `account_check_printing` | constrains: `next_check_number` |  |
| `print_checks` | operation | self | `account_check_printing` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_next_check_number` | ValidationError | Next Check Number should only contains numbers. | `account_check_printing` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_user` | yes | yes | yes | no | `account_check_printing` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account_check_printing.print_pre_numbered_checks_view` | form |  | `next_check_number` | `Print`, `Cancel` |  | `account_check_printing` |

Machine-readable definition: `../../../schemas/data/entities/print.prenumbered.checks.json`; views: `../../../schemas/interfaces/views/print.prenumbered.checks.json`.
