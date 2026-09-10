# Rules for the reconciliation model (`account.reconcile.model.line`)

**Transport name:** `account.reconcile.model.line`  
**Storage name:** `account_reconcile_model_line`  
**Kind:** persistent entity (one table)  
**Defined by package:** `account`

Description: Rules for the reconciliation model

## Identity and behavior

- Mixins (classical inheritance): `analytic.mixin`
- Default ordering: `sequence, id`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (10)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `model_id` | Model | many to one | `account.reconcile.model` | read only; indexed (btree_not_null); on delete of the target: cascade |
| `company_id` | Company | many to one |  | related through path `model_id.company_id` and stored |
| `sequence` | Sequence | integer |  | required; default `10` |
| `account_id` | Account | many to one | `account.account` | on delete of the target: cascade; restricted by domain `[('account_type', '!=', 'off_balance')]`; must belong to the same company |
| `partner_id` | Partner | many to one | `res.partner` |  |
| `label` | Label | single line text |  | translatable |
| `amount_type` | Amount Type | selection |  | required; default `percentage` |
| `amount` | Float Amount | float |  | computed by rule `_compute_float_amount` and stored |
| `amount_string` | Amount | single line text |  | required; default `100`; Help: Value for the amount of the writeoff line     * Percentage: Percentage of the balance, between 0 and 100.     * Fixed: The fixed value of the writeoff. The amount will count as a debit if it is negative, as a credit if it is positive.     * From Label: There is no need for regex delimiter, only the regex is needed. For instance if you want to extract the amount from R:9672938 10/07 AX 9415126318 T:5L:NA BRT: 3358,07 C: You could enter BRT: ([\d,]+)     If the label is "01870912 0009065 00115" and you need the amount in decimal     format (e.g. 90.65), you can use a regex with capturing groups, for example:         \s+0*(\d+?)(\d{2})(?=\s)     In this case:     • the first group captures the integer part     • the second group captures the decimal part (last two digits) |
| `tax_ids` | Taxes | many to many | `account.tax` | on delete of the target: restrict; must belong to the same company; association table `account_reconcile_model_line_account_tax_rel` |

## Selection values

### `amount_type` (Amount Type)

| Value | Label |
|---|---|
| `fixed` | Fixed |
| `percentage` | Percentage of balance |
| `percentage_st_line` | Percentage of statement line |
| `regex` | From label |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_onchange_amount_type` | on change | self | `account` | onchange: `amount_type` |  |
| `_compute_float_amount` | computation | self | `account` | depends: `amount_string` |  |
| `_validate_amount` | validation | self | `account` | constrains: `amount_string` |  |

## Validation and error messages (4)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_validate_amount` | UserError | The amount is not a number | `account` |
| `_validate_amount` | UserError | Statement line percentage can't be 0 | `account` |
| `_validate_amount` | UserError | Balance percentage can't be 0 | `account` |
| `_validate_amount` | UserError | The regex is not valid | `account` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.group_account_invoice` | yes | yes | no | no | `account` |
| `account.group_account_basic` | yes | yes | yes | yes | `account` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Account reconcile model_line template company rule | global (all users) | `[('company_id', 'parent_of', company_ids)]` | True | True | True | True |

Machine-readable definition: `../../../schemas/data/entities/account.reconcile.model.line.json`.
