# Analytic Mixin (`analytic.mixin`)

**Transport name:** `analytic.mixin`  
**Storage name:** `analytic_mixin`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `analytic`

Description: Analytic Mixin

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `analytic_distribution` | Analytic Distribution | structured document |  | computed by rule `_compute_analytic_distribution` and stored; searchable through a search rule |
| `analytic_precision` | Analytic Precision | integer |  | default computed dynamically (lambda self: self.env['decimal.precision'].precision_get('Percentage Analytic')) |
| `distribution_analytic_account_ids` | Distribution Analytic Account | many to many | `account.analytic.account` | computed by rule `_compute_distribution_analytic_account_ids` (not stored); searchable through a search rule |

## Operations (17)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `init` | lifecycle override | self | `analytic` |  |  |
| `_query_analytic_accounts` | internal rule | self, table | `analytic` |  |  |
| `_get_analytic_account_ids_from_distributions` | preparation rule | self, distributions | `analytic` | model |  |
| `_compute_distribution_analytic_account_ids` | computation | self | `analytic` | depends: `analytic_distribution` |  |
| `_search_distribution_analytic_account_ids` | search rule | self, operator, value | `analytic` |  |  |
| `_compute_analytic_distribution` | computation | self | `analytic` |  |  |
| `_search_analytic_distribution` | search rule | self, operator, value | `analytic` |  |  |
| `_read_group_groupby` | internal rule | self, alias, groupby_spec, query | `analytic` |  | To group by `analytic_distribution`, we first need to separate the analytic_ids and associate them with the ids to be counted Do note that only '__count' can be passed in the `aggregates` |
| `_read_group_select` | internal rule | self, aggregate_spec, query | `analytic` |  |  |
| `_get_count_id` | preparation rule | self, query | `analytic` |  |  |
| `filtered_domain` | operation | self, domain | `analytic` |  |  |
| `write` | lifecycle override | self, vals | `analytic` |  | Format the analytic_distribution float value, so equality on analytic_distribution can be done |
| `create` | lifecycle override | self, vals_list | `analytic` | model_create_multi | Format the analytic_distribution float value, so equality on analytic_distribution can be done |
| `_validate_distribution` | internal rule | self, **kwargs | `analytic` |  |  |
| `_sanitize_values` | internal rule | self, vals, decimal_precision | `analytic` |  | Normalize the float of the distribution |
| `_modifiying_distribution_values` | internal rule | self, old_distribution, new_distribution | `analytic` |  |  |
| `_merge_distribution` | internal rule | self, old_distribution, new_distribution | `analytic` |  |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_search_analytic_distribution` | UserError | Operation not supported | `analytic` |
| `_validate_distribution` | ValidationError | One or more lines require a 100% analytic distribution. | `analytic` |

Machine-readable definition: `../../../schemas/data/entities/analytic.mixin.json`.
