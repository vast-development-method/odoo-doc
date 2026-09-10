# Profiling results (`ir.profile`)

**Transport name:** `ir.profile`  
**Storage name:** `ir_profile`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`

Description: Profiling results

## Identity and behavior

- Default ordering: `session desc, id desc`

## Fields (16)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `create_date` | Creation Date | date and time |  |  |
| `session` | Session | single line text |  | indexed |
| `name` | Description | single line text |  |  |
| `duration` | Duration | float |  | precision `[9, 3]`; Help: Real elapsed time |
| `cpu_duration` | CPU Duration | float |  | precision `[9, 3]`; Help: CPU clock (not including other processes or SQL) |
| `init_stack_trace` | Initial stack trace | multi line text |  |  |
| `sql` | Sql | multi line text |  |  |
| `sql_count` | Queries Count | integer |  |  |
| `traces_async` | Traces Async | multi line text |  |  |
| `traces_sync` | Traces Sync | multi line text |  |  |
| `others` | others | multi line text |  |  |
| `qweb` | Qweb | multi line text |  |  |
| `entry_count` | Entry count | integer |  |  |
| `speedscope` | Speedscope | binary |  | computed by rule `_compute_speedscope` (not stored) |
| `speedscope_url` | Open | multi line text |  | computed by rule `_compute_speedscope_url` (not stored) |
| `config_url` | Open profiles config | multi line text |  | computed by rule `_compute_config_url` (not stored) |

## Operations (13)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_gc_profile` | background operation | self | `base` | autovacuum |  |
| `_has_memory_traces` | internal rule | self | `base` |  | Return True if all profiles have RSS memory measurements in traces_async. |
| `_get_memory_data` | preparation rule | self | `base` |  | Return baselined RSS memory data points for the chart. |
| `_compute_config_url` | computation | self | `base` |  |  |
| `_compute_speedscope` | computation | self | `base` | depends: `init_stack_trace` |  |
| `_default_profile_params` | preparation rule | self | `base` |  |  |
| `_parse_params` | internal rule | self, params | `base` |  |  |
| `_generate_speedscope` | internal rule | self, params | `base` |  |  |
| `_add_outputs` | internal rule | self, sp, suffix, params | `base` |  |  |
| `_compute_speedscope_url` | computation | self | `base` | depends: `speedscope` |  |
| `_enabled_until` | internal rule | self | `base` |  | If the profiling is enabled, return until when it is enabled. Otherwise return ``None``. |
| `set_profiling` | operation | self, profile, collectors, params | `base` | model | Enable or disable profiling for the current user.  :param profile: ``True`` to enable profiling, ``False`` to disable it. :param list collectors: optional list of collectors to use (string) :param dict params: optional parameters set on the profiler object |
| `action_view_speedscope` | user action | self | `base` |  |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_generate_speedscope` | UserError | All profiles must have the same initial stack trace to be displayed together. | `base` |
| `set_profiling` | UserError | Profiling is not enabled on this database. Please contact an administrator. | `base` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_system` | yes | yes | yes | yes | `base` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.ir_profile_view_search` | search |  | `name`, `session` |  | `Session` | `base` |
| `base.ir_profile_view_list` | list |  | `create_date`, `session`, `name`, `entry_count`, `sql_count`, `speedscope_url`, `config_url`, `duration`, `cpu_duration` | `View in speedscope` |  | `base` |
| `base.ir_profile_view_form` | form |  | `name`, `session`, `cpu_duration`, `entry_count`, `sql_count`, `speedscope_url`, `config_url`, `qweb` |  |  | `base` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.action_menu_ir_profile` | Ir profile | list,form |  | `{'search_default_group_session': 1}` |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/ir.profile.json`; views: `../../../schemas/interfaces/views/ir.profile.json`.
