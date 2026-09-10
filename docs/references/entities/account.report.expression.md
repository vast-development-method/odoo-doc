# Accounting Report Expression (`account.report.expression`)

**Transport name:** `account.report.expression`  
**Storage name:** `account_report_expression`  
**Kind:** persistent entity (one table)  
**Defined by package:** `account`  
**Extended by packages:** `l10n_it`

Description: Accounting Report Expression

## Identity and behavior

- Display name field: `report_line_name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (12)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `report_line_id` | Report Line | many to one | `account.report.line` | required; indexed; on delete of the target: cascade |
| `report_line_name` | Report Line Name | single line text |  | related through path `report_line_id.name` |
| `label` | Label | single line text |  | required |
| `engine` | Computation Engine | selection |  | required |
| `formula` | Formula | single line text |  | required |
| `subformula` | Subformula | single line text |  |  |
| `date_scope` | Date Scope | selection |  | required; default `strict_range` |
| `figure_type` | Figure Type | selection |  |  |
| `green_on_positive` | Is Growth Good when Positive | boolean |  | default `True` |
| `blank_if_zero` | Blank if Zero | boolean |  | Help: When checked, 0 values will not show when displaying this expression's value. |
| `auditable` | Auditable | boolean |  | computed by rule `_compute_auditable` and stored |
| `carryover_target` | Carry Over To | single line text |  | Help: Formula in the form line_code.expression_label. This allows setting the target of the carryover for this expression (on a _carryover_*-labeled expression), in case it is different from the parent line. |

## Selection values

### `engine` (Computation Engine)

| Value | Label |
|---|---|
| `domain` | Odoo Domain |
| `tax_tags` | Tax Tags |
| `aggregation` | Aggregate Other Formulas |
| `account_codes` | Prefix of Account Codes |
| `external` | External Value |
| `custom` | Custom Python Function |

### `date_scope` (Date Scope)

| Value | Label |
|---|---|
| `from_beginning` | From the very start |
| `from_fiscalyear` | From the start of the fiscal year |
| `to_beginning_of_fiscalyear` | At the beginning of the fiscal year |
| `to_beginning_of_period` | At the beginning of the period |
| `strict_range` | Strictly on the given dates |
| `previous_return_period` | From previous return period |

## Database constraints and indexes (2)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_domain_engine_subformula_required` | Constraint | `CHECK(engine != 'domain' OR subformula IS NOT NULL)` | Expressions using 'domain' engine should all have a subformula. | `account` |
| `_line_label_uniq` | Constraint | `UNIQUE(report_line_id,label)` | The expression label must be unique per report line. | `account` |

## Operations (16)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_carryover_target` | validation | self | `account` | constrains: `carryover_target`, `label` |  |
| `_check_formula` | validation | self | `account` | constrains: `formula` |  |
| `_compute_auditable` | computation | self | `account` | depends: `engine` |  |
| `_validate_engine` | validation | self | `account` | constrains: `engine`, `report_line_id` |  |
| `_get_auditable_engines` | preparation rule | self | `account` |  |  |
| `_strip_formula` | internal rule | self, vals | `account` |  |  |
| `_create_tax_tags` | internal rule | self, tag_name, country | `account` |  |  |
| `create` | lifecycle override | self, vals_list | `account` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `account` |  |  |
| `_unlink_archive_used_tags` | internal rule | self | `account` | ondelete | Manages unlink or archive of tax_tags when account.report.expression are deleted. If a tag is still in use on amls, we archive it. |
| `_compute_display_name` | computation | self | `account` | depends: `report_line_name`, `label` |  |
| `_expand_aggregations` | internal rule | self | `account` |  | Return self and its full aggregation expression dependency |
| `_get_aggregation_terms_details` | preparation rule | self | `account` |  | Computes the details of each aggregation expression in self, and returns them in the form of a single dict aggregating all the results.  Example of aggregation details: formula 'A.balance + B.balance + A.other' will return: {'A': {'balance', 'other'}, 'B': {'balance'}} |
| `_get_matching_tags` | preparation rule | self | `account` |  | Returns all the signed account.account.tags records whose name matches any of the formulas of the tax_tags expressions contained in self. |
| `_get_tags_create_vals` | preparation rule | self, tag_name, country_id, existing_tag | `account` | model |  |
| `_get_carryover_target_expression` | preparation rule | self, options | `account`, `l10n_it` |  |  |

## Validation and error messages (9)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_carryover_target` | UserError | You cannot use the field carryover_target in an expression that does not have the label starting with _carryover_ | `account` |
| `_check_carryover_target` | UserError | When targeting an expression for carryover, the label of that expression must start with _applied_carryover_ | `account` |
| `_check_formula` | ValidationError | Invalid formula for expression '%(label)s' of line '%(line)s': %(formula)s | `account` |
| `_validate_engine` | ValidationError | Groupby feature isn't supported by '%(engine)s' engine. Please remove the groupby value on '%(report_line)s' | `account` |
| `_expand_aggregations` | UserError | In report '%(report_name)s', on line '%(line_name)s', with label '%(label)s', The format of the cross report expression is invalid.  Expected: cross_report(<report_id>\|<xml_id>)Example:  cross_report(my_module.my_report) or cross_report(123) | `account` |
| `_expand_aggregations` | UserError | In report '%(report_name)s', on line '%(line_name)s', with label '%(label)s', Failed to parse the cross report id or xml_id. | `account` |
| `_expand_aggregations` | UserError | You cannot use cross report on itself | `account` |
| `_get_aggregation_terms_details` | UserError | Cannot get aggregation details from a line not using 'aggregation' engine | `account` |
| `_get_carryover_target_expression` | UserError | Could not determine carryover target automatically for expression %s. | `account` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_basic` | no | yes | no | no | `account` |
| `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.group_account_manager` | yes | yes | yes | yes | `account` |

Machine-readable definition: `../../../schemas/data/entities/account.report.expression.json`.
