# Snailmail Letter (`snailmail.letter`)

**Transport name:** `snailmail.letter`  
**Storage name:** `snailmail_letter`  
**Kind:** persistent entity (one table)  
**Defined by package:** `snailmail`

Description: Snailmail Letter

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (24)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `user_id` | Sent by | many to one | `res.users` |  |
| `model` | Model | single line text |  | required |
| `res_id` | Document identifier | integer |  | required |
| `partner_id` | Recipient | many to one | `res.partner` | required |
| `company_id` | Company | many to one | `res.company` | required; read only; default computed dynamically (lambda self: self.env.company.id) |
| `report_template` | Optional report to print and attach | many to one | `ir.actions.report` |  |
| `attachment_id` | Attachment | many to one | `ir.attachment` | indexed (btree_not_null); on delete of the target: cascade |
| `attachment_datas` | Document | binary |  | related through path `attachment_id.datas` |
| `attachment_fname` | Attachment Filename | single line text |  | related through path `attachment_id.name` |
| `color` | Color | boolean |  | default computed dynamically (lambda self: self.env.company.snailmail_color) |
| `cover` | Cover Page | boolean |  | default computed dynamically (lambda self: self.env.company.snailmail_cover) |
| `duplex` | Both side | boolean |  | default computed dynamically (lambda self: self.env.company.snailmail_duplex) |
| `state` | Status | selection |  | required; read only; default `pending`; not copied on duplication; Help: When a letter is created, the status is 'Pending'. If the letter is correctly sent, the status goes in 'Sent', If not, it will got in state 'Error' and the error message will be displayed in the field 'Error Message'. |
| `error_code` | Error | selection |  |  |
| `info_msg` | Information | rich text |  |  |
| `reference` | Related Record | single line text |  | read only; computed by rule `_compute_reference` (not stored) |
| `message_id` | Snailmail Status Message | many to one | `mail.message` | indexed (btree_not_null) |
| `notification_ids` | Notifications | one to many | `mail.notification` | inverse field `letter_id` |
| `street` | Street | single line text |  |  |
| `street2` | Street2 | single line text |  |  |
| `zip` | Zip | single line text |  |  |
| `city` | City | single line text |  |  |
| `state_id` | State | many to one | `res.country.state` |  |
| `country_id` | Country | many to one | `res.country` |  |

## Selection values

### `state` (Status)

| Value | Label |
|---|---|
| `pending` | In Queue |
| `sent` | Sent |
| `error` | Error |
| `canceled` | Cancelled |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Operations (21)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `snailmail` | depends: `attachment_id`, `partner_id` |  |
| `_compute_reference` | computation | self | `snailmail` | depends: `model`, `res_id` |  |
| `create` | lifecycle override | self, vals_list | `snailmail` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `snailmail` |  |  |
| `_onchange_attachment_id` | on change | self | `snailmail` | onchange: `attachment_id` |  |
| `_generate_report_pdf` | internal rule | self, report | `snailmail` |  |  |
| `_fetch_attachment` | internal rule | self | `snailmail` |  | This method will check if we have any existent attachement matching the model and res_ids and create them if not found. |
| `_count_pages_pdf` | internal rule | self, bin_pdf | `snailmail` |  | Count the number of pages of the given pdf file. :param bin_pdf : binary content of the pdf file |
| `_snailmail_create` | internal rule | self, route | `snailmail` |  | Create a dictionnary object to send to snailmail server.  :return: Dict in the form: {     account_token: string,    //IAP Account token of the user     documents: [{         pages: int,         pdf_bin: pdf file         res_id: int (client-side res_id),         res_model: char (client-side res_model),         address: {             name: char,             street: char,             street2: char (OPTIONAL),             zip: int,             city: char,             state: char (state code (OPTIONAL)),             country_code: char (country code)         }         return_address: {              |
| `_get_error_message` | preparation rule | self, error | `snailmail` |  |  |
| `_get_failure_type` | preparation rule | self, error | `snailmail` |  |  |
| `_snailmail_print` | internal rule | self, immediate | `snailmail` |  |  |
| `_snailmail_print_invalid_address` | internal rule | self | `snailmail` |  |  |
| `_snailmail_print_valid_address` | internal rule | self | `snailmail` |  | get response {     'request_code': RESPONSE_OK, # because we receive 200 if good or fail     'total_cost': total_cost,     'credit_error': credit_error,     'request': {         'documents': documents,         'options': options         }     } } |
| `snailmail_print` | operation | self | `snailmail` |  |  |
| `cancel` | operation | self | `snailmail` |  |  |
| `_snailmail_cron` | internal rule | self, autocommit | `snailmail` | model |  |
| `_is_valid_address` | internal rule | self, record | `snailmail` | model |  |
| `_get_cover_address_split` | preparation rule | self | `snailmail` |  |  |
| `_append_cover_page` | internal rule | self, invoice_bin | `snailmail` |  |  |
| `_overwrite_margins` | internal rule | self, invoice_bin | `snailmail` |  | Fill the margins with white for validation purposes. |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_fetch_attachment` | UserError | Please use an A4 Paper format. | `snailmail` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | no | `snailmail` |
| `base.group_system` | yes | yes | yes | yes | `snailmail` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `snailmail.snailmail_letter_list` | list |  | `attachment_id`, `partner_id`, `user_id`, `state`, `info_msg`, `company_id` |  |  | `snailmail` |
| `snailmail.snailmail_letter_form` | form |  | `state`, `display_name`, `reference`, `attachment_datas`, `attachment_fname`, `partner_id`, `user_id`, `info_msg`, `model`, `res_id`, `color`, `duplex` | `Send Now`, `Cancel` |  | `snailmail` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `snailmail.action_mail_letters` | Snailmail Letters | form,list | `[('state', '!=', 'draft')]` |  |  | `snailmail` |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `snailmail.snailmail_print` | Snailmail: process letters queue | 24 hours | `_snailmail_cron` |  |

Machine-readable definition: `../../../schemas/data/entities/snailmail.letter.json`; views: `../../../schemas/interfaces/views/snailmail.letter.json`.
