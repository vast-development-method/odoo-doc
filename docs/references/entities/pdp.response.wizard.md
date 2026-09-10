# PDP Response wizard (`pdp.response.wizard`)

**Transport name:** `pdp.response.wizard`  
**Storage name:** `pdp_response_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `l10n_fr_pdp`

Description: PDP Response wizard

## Fields (10)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `move_ids` | Move | many to many | `account.move` | required |
| `status` | Status | selection |  |  |
| `available_statuses` | Available Statuses | single line text |  | computed by rule `_compute_available_statuses` (not stored); Help: Technical field to enable dynamic selection of status. |
| `reason_code` | Reason Code | selection |  |  |
| `show_reason_code` | Show Reason Code | boolean |  | computed by rule `_compute_show_reason_code` (not stored); Help: Technical field to hide / show the 'Reason Code' in the view. |
| `note` | Additional note | multi line text |  |  |
| `move_count` | Move Count | integer |  | computed by rule `_compute_move_count` and stored |
| `fully_paid` | Fully paid | boolean |  | computed by rule `_compute_paid_amount` and stored |
| `paid_amount` | Payment Amount | monetary |  | computed by rule `_compute_paid_amount` and stored; currency taken from `currency_id` |
| `currency_id` | Currency | many to one | `res.currency` | computed by rule `_compute_currency_id` and stored; precomputed before insertion; Help: The payment's currency. |

## Selection values

### `status` (Status)

| Value | Label |
|---|---|
| `PD` | Paid |
| `cancelled` | Cancelled |
| `suspended` | Suspended |
| `refused` | Refused |
| `AP` | Approved |
| `in_hand` | In Hand |
| `completed` | Completed |

### `reason_code` (Reason Code)

| Value | Label |
|---|---|
| `TX_TVA_ERR` | Incorrect VAT rate |
| `MONTANTTOTAL_ERR` | Incorrect Total Amount |
| `CALCUL_ERR` | Billing calculation error |
| `NON_CONFORME` | Legal information missing |
| `DEST_ERR` | Wrong recipient |
| `TRANSAC_INC` | Unknown transaction |
| `EMMET_INC` | Unknown sender |
| `CONTRAT_TERM` | Contract completed |
| `DOUBLE_FACT` | Duplicate Invoice |
| `CMD_ERR` | Order number is incorrect or missing |
| `ADR_ERR` | Incorrect electronic billing address |
| `REF_CT_ABSENT` | Contract reference required to process the missing invoice |
| `JUSTIF_ABS` | Missing or Insufficient Supporting Documentation |
| `COORD_BANC_ERR` | Bank Account Information Error |
| `SIRET_ERR` | Incorrect or missing SIRET number |
| `CODE_ROUTAGE_ERR` | Missing or incorrect CODE_ROUTAGE |
| `REF_ERR` | Incorrect reference |

## State fields

State machine fields of this entity: `status`. Transitions are specified in the domain documents.

## Operations (14)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `fields_get` | lifecycle override | self, allfields, attributes | `l10n_fr_pdp` | model |  |
| `_compute_move_count` | computation | self | `l10n_fr_pdp` | depends: `move_ids` |  |
| `_compute_currency_id` | computation | self | `l10n_fr_pdp` | depends: `move_ids` |  |
| `_compute_paid_amount` | computation | self | `l10n_fr_pdp` | depends: `move_ids` |  |
| `_compute_show_reason_code` | computation | self | `l10n_fr_pdp` | depends: `status` |  |
| `_compute_available_statuses` | computation | self | `l10n_fr_pdp` | depends: `move_ids` |  |
| `_get_base_lines` | preparation rule | self, move | `l10n_fr_pdp` | model |  |
| `_get_tax_details` | preparation rule | self, base_lines | `l10n_fr_pdp` | model |  |
| `_get_early_payment_discount_tax_details` | preparation rule | self, base_lines | `l10n_fr_pdp` | model |  |
| `_get_payments_data_fully_paid` | preparation rule | self, move | `l10n_fr_pdp` | model |  |
| `_get_payments_data` | preparation rule | self, move, forced_amount | `l10n_fr_pdp` | model |  |
| `_round_format_number_2` | internal rule | self, number | `l10n_fr_pdp` | model |  |
| `_is_fully_paid` | internal rule | self, move | `l10n_fr_pdp` | model |  |
| `button_send` | user action | self | `l10n_fr_pdp` |  |  |

## Validation and error messages (13)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_compute_currency_id` | UserError | The EUR currency is missing. | `l10n_fr_pdp` |
| `_compute_available_statuses` | UserError | All journal entries must either be purchase or sale documents. | `l10n_fr_pdp` |
| `button_send` | UserError | Please select a Status. | `l10n_fr_pdp` |
| `button_send` | UserError | Some of the journal entries were not sent to the Approved Platform yet: %s | `l10n_fr_pdp` |
| `button_send` | UserError | To refuse an invoice please select a Reason Code. | `l10n_fr_pdp` |
| `button_send` | UserError | To refuse an invoice please enter a Note. | `l10n_fr_pdp` |
| `button_send` | UserError | To suspend an invoice please select a Reason Code. | `l10n_fr_pdp` |
| `button_send` | UserError | To suspend an invoice please enter a Note. | `l10n_fr_pdp` |
| `button_send` | UserError | Some of the journal entries are not cancelled: %s | `l10n_fr_pdp` |
| `button_send` | UserError | Some of the journal entries are not posted: %s | `l10n_fr_pdp` |
| `button_send` | UserError | Only journal entries in currency EUR are supported. | `l10n_fr_pdp` |
| `button_send` | UserError | Some of the journal entries are without tax: %s | `l10n_fr_pdp` |
| `button_send` | UserError | Some of the journal entries have no payments to send: %s | `l10n_fr_pdp` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | yes | `l10n_fr_pdp` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_fr_pdp.pdp_response_wizard_form` | form |  | `available_statuses`, `status`, `reason_code`, `currency_id`, `fully_paid`, `paid_amount`, `note` | `Send`, `Discard` |  | `l10n_fr_pdp` |

Machine-readable definition: `../../../schemas/data/entities/pdp.response.wizard.json`; views: `../../../schemas/interfaces/views/pdp.response.wizard.json`.
