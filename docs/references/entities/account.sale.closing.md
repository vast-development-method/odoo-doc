# Sale Closing (`account.sale.closing`)

**Transport name:** `account.sale.closing`  
**Storage name:** `account_sale_closing`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_fr_pos_cert`

Description: Sale Closing

## Identity and behavior

- Default ordering: `date_closing_stop desc, sequence_number desc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (11)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; Help: Frequency and unique sequence number |
| `company_id` | Company | many to one | `res.company` | required; read only |
| `date_closing_stop` | Closing Date | date and time |  | required; read only; Help: Date to which the values are computed |
| `date_closing_start` | Starting Date | date and time |  | required; read only; Help: Date from which the total interval is computed |
| `frequency` | Closing Type | selection |  | required; read only |
| `total_interval` | Period Total | monetary |  | required; read only; Help: Total in receivable accounts during the interval, excluding overlapping periods |
| `cumulative_total` | Cumulative Grand Total | monetary |  | required; read only; Help: Total in receivable accounts since the beginnig of times |
| `sequence_number` | Sequence # | integer |  | required; read only |
| `last_order_id` | Last Pos Order | many to one | `pos.order` | read only; Help: Last Pos order included in the grand total |
| `last_order_hash` | Last Order entry's inalteralbility hash | single line text |  | read only |
| `currency_id` | Currency | many to one | `res.currency` | read only; related through path `company_id.currency_id` and stored; Help: The company's currency |

## Selection values

### `frequency` (Closing Type)

| Value | Label |
|---|---|
| `daily` | Daily |
| `monthly` | Monthly |
| `annually` | Annual |

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_query_for_aml` | internal rule | self, company, first_move_sequence_number, date_start | `l10n_fr_pos_cert` |  |  |
| `_compute_amounts` | computation | self, frequency, company | `l10n_fr_pos_cert` |  | Method used to compute all the business data of the new object. It will search for previous closings of the same frequency to infer the move from which account move lines should be fetched. @param {string} frequency: a valid value of the selection field on the object (daily, monthly, annually)     frequencies are literal (daily means 24 hours and so on) @param {recordset} company: the company for which the closing is done @return {dict} containing {field: value} for each business field of the object |
| `_interval_dates` | internal rule | self, frequency, company | `l10n_fr_pos_cert` |  | Method used to compute the theoretical date from which account move lines should be fetched @param {string} frequency: a valid value of the selection field on the object (daily, monthly, annually)     frequencies are literal (daily means 24 hours and so on) @param {recordset} company: the company for which the closing is done @return {dict} the theoretical date from which account move lines are fetched.     date_stop date to which the move lines are fetched, always now()     the dates are in their the system Database string representation |
| `write` | lifecycle override | self, vals | `l10n_fr_pos_cert` |  |  |
| `_unlink_never` | internal rule | self | `l10n_fr_pos_cert` | ondelete |  |
| `_automated_closing` | internal rule | self, frequency | `l10n_fr_pos_cert` | model | To be executed by the CRON to create an object of the given frequency for each company that needs it @param {string} frequency: a valid value of the selection field on the object (daily, monthly, annually)     frequencies are literal (daily means 24 hours and so on) @return {recordset} all the objects created for the given frequency |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `write` | UserError | Sale Closings are not meant to be written or deleted under any circumstances. | `l10n_fr_pos_cert` |
| `_unlink_never` | UserError | Sale Closings are not meant to be written or deleted under any circumstances. | `l10n_fr_pos_cert` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `l10n_fr_pos_cert` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Sale Closing multi-company | global (all users) | `[('company_id', 'in', company_ids)]` | True | True | True | True |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_fr_pos_cert.list_view_account_sale_closing` | list |  | `date_closing_start`, `date_closing_stop`, `company_id`, `currency_id`, `frequency`, `sequence_number`, `total_interval`, `cumulative_total` |  |  | `l10n_fr_pos_cert` |
| `l10n_fr_pos_cert.form_view_account_sale_closing` | form |  | `name`, `date_closing_start`, `date_closing_stop`, `frequency`, `sequence_number`, `total_interval`, `cumulative_total`, `last_order_id`, `last_order_hash`, `company_id`, `currency_id` |  |  | `l10n_fr_pos_cert` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `l10n_fr_pos_cert.action_list_view_account_sale_closing` | Sales Closings | list,form |  |  |  | `l10n_fr_pos_cert` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `l10n_fr_pos_cert.menu_account_closing` |  | `pos_fr_statements_menu` | `l10n_fr_pos_cert.action_list_view_account_sale_closing` | 80 |  |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `l10n_fr_pos_cert.account_sale_closing_daily` | Generate Daily Sales Closing | 1 days | `_automated_closing` |  |
| `l10n_fr_pos_cert.account_sale_closing_monthly` | Generate Monthly Sales Closing | 1 months | `_automated_closing` |  |
| `l10n_fr_pos_cert.account_sale_closing_annually` | Generate Annual Sales Closing | 12 months | `_automated_closing` |  |

Machine-readable definition: `../../../schemas/data/entities/account.sale.closing.json`; views: `../../../schemas/interfaces/views/account.sale.closing.json`.
