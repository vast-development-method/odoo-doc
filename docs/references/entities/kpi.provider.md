# key performance indicator Provider (`kpi.provider`)

**Transport name:** `kpi.provider`  
**Storage name:** `kpi_provider`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `base_setup`  
**Extended by packages:** `account`

Description: KPI Provider

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `get_kpi_summary` | operation | self | `account`, `base_setup` | model | Other modules can override this method to add their own KPIs to the list. This method will be called by the databases module to retrieve the data displayed on the databases list. The return value shall be a list of dictionaries with the following keys:  - id: a unique identifier for the KPI - type: the type of data (`integer` or `return_status`) - name: the translated name of the KPI, as displayable to the current user - value: either the numeric value (for `type=integer`) or one of the statuses (for `type=return_status`):   - late       one return of this type should have been done already    |
| `get_account_kpi_summary` | operation | self | `account` | model |  |

Machine-readable definition: `../../../schemas/data/entities/kpi.provider.json`.
