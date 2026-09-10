# Autopost Bills Wizard (`account.autopost.bills.wizard`)

**Transport name:** `account.autopost.bills.wizard`  
**Storage name:** `account_autopost_bills_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `account`

Description: Autopost Bills Wizard

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `partner_id` | Partner | many to one | `res.partner` |  |
| `partner_name` | Partner Name | single line text |  | related through path `partner_id.name` |
| `nb_unmodified_bills` | Number of bills previously unmodified from this partner | integer |  |  |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `action_automate_partner` | user action | self | `account` |  |  |
| `action_ask_later` | user action | self | `account` |  |  |
| `action_never_automate_partner` | user action | self | `account` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | no | `account` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.autopost_bills_wizard` | form |  | `nb_unmodified_bills`, `partner_name` | `action_automate_partner`, `action_ask_later`, `action_never_automate_partner` |  | `account` |

Machine-readable definition: `../../../schemas/data/entities/account.autopost.bills.wizard.json`; views: `../../../schemas/interfaces/views/account.autopost.bills.wizard.json`.
