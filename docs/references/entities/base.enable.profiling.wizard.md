# Enable profiling for some time (`base.enable.profiling.wizard`)

**Transport name:** `base.enable.profiling.wizard`  
**Storage name:** `base_enable_profiling_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `base`

Description: Enable profiling for some time

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `duration` | Enable profiling for | selection |  |  |
| `expiration` | Enable profiling until | date and time |  | computed by rule `_compute_expiration` and stored |

## Selection values

### `duration` (Enable profiling for)

| Value | Label |
|---|---|
| `minutes_5` | 5 Minutes |
| `hours_1` | 1 Hour |
| `days_1` | 1 Day |
| `months_1` | 1 Month |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_expiration` | computation | self | `base` | depends: `duration` |  |
| `submit` | operation | self | `base` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_system` | yes | yes | yes | no | `base` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.enable_profiling_wizard` | form |  | `duration`, `expiration` | `Cancel`, `Enable profiling` |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/base.enable.profiling.wizard.json`; views: `../../../schemas/interfaces/views/base.enable.profiling.wizard.json`.
