# Analytic Plan Fields (`analytic.plan.fields.mixin`)

**Transport name:** `analytic.plan.fields.mixin`  
**Storage name:** `analytic_plan_fields_mixin`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `analytic`

Description: Analytic Plan Fields

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `account_id` | Project Account | many to one | `account.analytic.account` | indexed; on delete of the target: restrict; must belong to the same company |
| `auto_account_id` | Analytic Account | many to one | `account.analytic.account` | computed by rule `_compute_auto_account` (not stored); writable through an inverse rule; searchable through a search rule |

## Operations (16)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_auto_account` | computation | self | `analytic` | depends_context: `analytic_plan_id` |  |
| `_compute_partner_id` | computation | self | `analytic` |  |  |
| `_inverse_auto_account` | inverse computation | self | `analytic` |  |  |
| `_search_auto_account` | search rule | self, operator, value | `analytic` |  |  |
| `_get_plan_fnames` | preparation rule | self | `analytic` |  |  |
| `_get_analytic_accounts` | preparation rule | self | `analytic` |  |  |
| `_get_distribution_key` | preparation rule | self | `analytic` |  |  |
| `_get_analytic_distribution` | preparation rule | self | `analytic` |  |  |
| `_get_mandatory_plans` | preparation rule | self, company, business_domain | `analytic` |  |  |
| `_get_plan_domain` | preparation rule | self, plan | `analytic` |  |  |
| `_get_account_node_context` | preparation rule | self, plan | `analytic` |  |  |
| `_check_account_id` | validation | self | `analytic` | constrains: |  |
| `default_get` | lifecycle override | self, fields | `analytic` | model |  |
| `fields_get` | lifecycle override | self, allfields, attributes | `analytic` | model |  |
| `_get_view` | lifecycle override | self, view_id, view_type, **options | `analytic` |  |  |
| `_patch_view` | internal rule | self, arch, view, view_type | `analytic` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_account_id` | ValidationError | At least one analytic account must be set | `analytic` |

Machine-readable definition: `../../../schemas/data/entities/analytic.plan.fields.mixin.json`.
