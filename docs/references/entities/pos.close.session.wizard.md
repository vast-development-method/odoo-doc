# Close Session Wizard (`pos.close.session.wizard`)

**Transport name:** `pos.close.session.wizard`  
**Storage name:** `pos_close_session_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `point_of_sale`

Description: Close Session Wizard

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `amount_to_balance` | Amount to balance | float |  |  |
| `account_id` | Destination account | many to one | `account.account` |  |
| `account_readonly` | Destination account is readonly | boolean |  |  |
| `message` | Information message | multi line text |  |  |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `close_session` | operation | self | `point_of_sale` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `point_of_sale.group_pos_user` | yes | yes | yes | no | `point_of_sale` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `point_of_sale.view_form_pos_close_session_wizard` | form |  | `message`, `account_readonly`, `amount_to_balance`, `account_id` | `Close Session`, `Cancel` |  | `point_of_sale` |

Machine-readable definition: `../../../schemas/data/entities/pos.close.session.wizard.json`; views: `../../../schemas/interfaces/views/pos.close.session.wizard.json`.
