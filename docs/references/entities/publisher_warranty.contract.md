# Publisher Warranty Contract (`publisher_warranty.contract`)

**Transport name:** `publisher_warranty.contract`  
**Storage name:** `publisher_warranty_contract`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `mail`  
**Extended by packages:** `website_mail`

Description: Publisher Warranty Contract

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_message` | preparation rule | self | `mail`, `website_mail` | model |  |
| `_get_sys_logs` | preparation rule | self | `mail` | model | Utility method to send a publisher warranty get logs messages. |
| `update_notification` | operation | self, cron_mode | `mail` |  | Send a message to Odoo's publisher warranty server to check the validity of the contracts, get notifications, etc...  @param cron_mode: If true, catch all exceptions (appropriate for usage in a cron). @type cron_mode: boolean |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `update_notification` | UserError | Error during communication with the publisher warranty server. | `mail` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `mail` |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `mail.ir_cron_module_update_notification` | Publisher: Update Notification | 1 weeks | `update_notification` | 1000 |

Machine-readable definition: `../../../schemas/data/entities/publisher_warranty.contract.json`.
