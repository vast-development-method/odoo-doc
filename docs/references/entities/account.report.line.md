# Accounting Report Line (`account.report.line`)

**Transport name:** `account.report.line`  
**Storage name:** `account_report_line`  
**Kind:** persistent entity (one table)  
**Defined by package:** `account`

Description: Accounting Report Line

## Identity and behavior

- Default ordering: `sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (20)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `expression_ids` | Expressions | one to many | `account.report.expression` | inverse field `report_line_id` |
| `report_id` | Parent Report | many to one | `account.report` | required; computed by rule `_compute_report_id` and stored; indexed; on delete of the target: cascade; precomputed before insertion; recursive dependency |
| `hierarchy_level` | Level | integer |  | required; computed by rule `_compute_hierarchy_level` and stored; precomputed before insertion; recursive dependency |
| `parent_id` | Parent Line | many to one | `account.report.line` | indexed (btree_not_null); on delete of the target: set null |
| `children_ids` | Child Lines | one to many | `account.report.line` | inverse field `parent_id` |
| `groupby` | Group By | single line text |  | Help: Comma-separated list of fields from account.move.line (Journal Item). When set, this line will generate sublines grouped by those keys. |
| `user_groupby` | User Group By | single line text |  | computed by rule `_compute_user_groupby` and stored; precomputed before insertion; Help: Comma-separated list of fields from account.move.line (Journal Item). When set, this line will generate sublines grouped by those keys. |
| `sequence` | Sequence | integer |  |  |
| `code` | Code | single line text |  | Help: Unique identifier for this line. |
| `foldable` | Foldable | boolean |  | Help: By default, we always unfold the lines that can be. If this is checked, the line won't be unfolded by default, and a folding button will be displayed. |
| `print_on_new_page` | Print On New Page | boolean |  | Help: When checked this line and everything after it will be printed on a new page. |
| `action_id` | Action | many to one | `ir.actions.actions` | Help: Setting this field will turn the line into a link, executing the action when clicked. |
| `hide_if_zero` | Hide if Zero | boolean |  | Help: This line and its children will be hidden when all of their columns are 0. |
| `domain_formula` | Domain Formula Shortcut | single line text |  | writable through an inverse rule; Help: Internal field to shorten expression_ids creation for the domain engine |
| `account_codes_formula` | Account Codes Formula Shortcut | single line text |  | writable through an inverse rule; Help: Internal field to shorten expression_ids creation for the account_codes engine |
| `aggregation_formula` | Aggregation Formula Shortcut | single line text |  | writable through an inverse rule; Help: Internal field to shorten expression_ids creation for the aggregation engine |
| `external_formula` | External Formula Shortcut | single line text |  | writable through an inverse rule; Help: Internal field to shorten expression_ids creation for the external engine |
| `horizontal_split_side` | Horizontal Split Side | selection |  | computed by rule `_compute_horizontal_split_side` and stored; recursive dependency |
| `tax_tags_formula` | Tax Tags Formula Shortcut | single line text |  | writable through an inverse rule; Help: Internal field to shorten expression_ids creation for the tax_tags engine |

## Selection values

### `horizontal_split_side` (Horizontal Split Side)

| Value | Label |
|---|---|
| `left` | Left |
| `right` | Right |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_code_uniq` | Constraint | `unique (report_id, code)` | A report line with the same code already exists. | `account` |

## Operations (16)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_hierarchy_level` | computation | self | `account` | depends: `parent_id.hierarchy_level` |  |
| `_compute_report_id` | computation | self | `account` | depends: `parent_id.report_id` |  |
| `_compute_horizontal_split_side` | computation | self | `account` | depends: `parent_id.horizontal_split_side` |  |
| `_compute_user_groupby` | computation | self | `account` | depends: `groupby`, `expression_ids.engine` |  |
| `_validate_groupby_no_child` | validation | self | `account` | constrains: `parent_id` |  |
| `_validate_groupby` | validation | self | `account` | constrains: `groupby`, `user_groupby` |  |
| `_check_parent_line` | validation | self | `account` | constrains: `parent_id` |  |
| `_copy_hierarchy` | internal rule | self, copied_report, parent, code_mapping | `account` |  | Copy the whole hierarchy from this line by copying each line children recursively and adapting the formulas with the new copied codes.  :param copied_report: The copy of the report. :param parent: The parent line in the hierarchy (a copy of the original parent line). :param code_mapping: A dictionary keeping track of mapping old_code -> new_code |
| `_get_copied_code` | preparation rule | self | `account` |  | Look for an unique copied code.  :return: an unique code for the copied account.report.line |
| `_inverse_domain_formula` | inverse computation | self | `account` |  |  |
| `_inverse_aggregation_formula` | inverse computation | self | `account` |  |  |
| `_inverse_aggregation_tax_formula` | inverse computation | self | `account` |  |  |
| `_inverse_account_codes_formula` | inverse computation | self | `account` |  |  |
| `_inverse_external_formula` | inverse computation | self | `account` |  |  |
| `_create_report_expression` | internal rule | self, engine | `account` |  |  |
| `_unlink_child_expressions` | internal rule | self | `account` | ondelete | We explicitly unlink child expressions. This is necessary even if there is an ondelete='cascade' on it, because the @api.ondelete method _unlink_archive_used_tags is not automatically called if the parent model is deleted. |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_validate_groupby_no_child` | ValidationError | A line cannot have both children and a groupby value (line '%s'). | `account` |
| `_check_parent_line` | ValidationError | Line "%s" defines itself as its parent. | `account` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_basic` | no | yes | no | no | `account` |
| `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.group_account_manager` | yes | yes | yes | yes | `account` |

Machine-readable definition: `../../../schemas/data/entities/account.report.line.json`.
