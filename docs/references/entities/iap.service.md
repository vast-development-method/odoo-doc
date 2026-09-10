# in-app purchase Service (`iap.service`)

**Transport name:** `iap.service`  
**Storage name:** `iap_service`  
**Kind:** persistent entity (one table)  
**Defined by package:** `iap`

Description: IAP Service

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `technical_name` | Technical Name | single line text |  | required; read only |
| `description` | Description | single line text |  | required; translatable |
| `unit_name` | Unit Name | single line text |  | required; translatable |
| `integer_balance` | Integer Balance | boolean |  | required |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_unique_technical_name` | Constraint | `UNIQUE(technical_name)` | Only one service can exist with a specific technical_name | `iap` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `iap` |
| `base.group_user` | no | yes | no | no | `iap` |

Machine-readable definition: `../../../schemas/data/entities/iap.service.json`.
