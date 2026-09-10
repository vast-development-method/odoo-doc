# Merge Partner Wizard (`base.partner.merge.automatic.wizard`)

**Transport name:** `base.partner.merge.automatic.wizard`  
**Storage name:** `base_partner_merge_automatic_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `base`  
**Extended by packages:** `mail`, `account`, `account`, `website`, `website_slides`, `loyalty`

Description: Merge Partner Wizard

## Fields (14)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `group_by_email` | Email | boolean |  |  |
| `group_by_name` | Name | boolean |  |  |
| `group_by_is_company` | Is Company | boolean |  |  |
| `group_by_vat` | value-added tax | boolean |  |  |
| `group_by_parent_id` | Parent Company | boolean |  |  |
| `state` | State | selection |  | required; read only; default `option` |
| `number_group` | Group of Contacts | integer |  | read only |
| `current_line_id` | Current Line | many to one | `base.partner.merge.line` |  |
| `line_ids` | Lines | one to many | `base.partner.merge.line` | inverse field `wizard_id` |
| `partner_ids` | Contacts | many to many | `res.partner` |  |
| `dst_partner_id` | Destination Contact | many to one | `res.partner` |  |
| `exclude_contact` | A user associated to the contact | boolean |  |  |
| `exclude_journal_item` | Journal Items associated to the contact | boolean |  |  |
| `maximum_group` | Maximum of Group of Contacts | integer |  |  |

## Selection values

### `state` (State)

| Value | Label |
|---|---|
| `option` | Option |
| `selection` | Selection |
| `finished` | Finished |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Operations (26)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `base` | model |  |
| `_get_fk_on` | preparation rule | self, table | `base` |  | return a list of many2one relation with the given table. :param table : the name of the sql table to return relations :returns a list of tuple 'table name', 'column name'. |
| `_has_check_or_unique_constraint` | internal rule | self, table, column | `base` |  |  |
| `_update_foreign_keys_generic` | internal rule | self, model, src_records, dst_record | `base` | model | Update all foreign key from the src_records to dst_record for any model. :param model: model name as a string :param src_records: merge source recordset (does not include destination one) :param dst_record: record of destination |
| `_update_reference_fields_generic` | internal rule | self, referenced_model, src_records, dst_record, additional_update_records | `base` | model | Update all reference fields from the src_records to dst_record for any model. :param referenced_model: model name as a string :param src_records: merge source recordset (does not include destination one) :param dst_record: record of destination :param additional_update_records: list of tuples (model, field_model, field_id) |
| `_update_foreign_keys` | internal rule | self, src_partners, dst_partner | `base`, `loyalty`, `website_slides`, `website` | model | Update all foreign key from the src_partner to dst_partner. All many2one fields will be updated. :param src_partners : merge source res.partner recordset (does not include destination one) :param dst_partner : record of destination res.partner |
| `_update_reference_fields` | internal rule | self, src_partners, dst_partner | `account`, `base` | model | Update all reference fields from the src_partner to dst_partner. :param src_partners : merge source res.partner recordset (does not include destination one) :param dst_partner : record of destination res.partner |
| `_get_summable_fields` | preparation rule | self | `account`, `base` |  | Returns the list of fields that should be summed when merging partners |
| `_update_values` | internal rule | self, src_partners, dst_partner | `base` | model | Update values of dst_partner with the ones from the src_partners. :param src_partners : recordset of source res.partner :param dst_partner : record of destination res.partner |
| `_merge_bank_accounts` | internal rule | self, src_partners, dst_partner | `base` | model | Merge bank accounts of src_partners into dst_partner. :param src_partners: merge source res.partner recordset (does not include destination one) :param dst_partner: record of destination res.partner |
| `_merge` | internal rule | self, partner_ids, dst_partner, extra_checks | `base` |  | private implementation of merge partner :param partner_ids : ids of partner to merge :param dst_partner : record of destination res.partner :param extra_checks: pass False to bypass extra sanity check (e.g. email address) |
| `_log_merge_operation` | internal rule | self, src_partners, dst_partner | `base`, `mail` |  |  |
| `_generate_query` | internal rule | self, fields, maximum_group | `base` | model | Build the SQL query on res.partner table to group them according to given criteria :param fields : list of column names to group by the partners :param maximum_group : limit of the query |
| `_compute_selected_groupby` | computation | self | `base` | model | Returns the list of field names the partner can be grouped (as merge criteria) according to the option checked on the wizard |
| `_partner_use_in` | internal rule | self, aggr_ids, models | `base` | model | Check if there is no occurence of this group of partner in the selected model :param aggr_ids : stringified list of partner ids separated with a comma (sql array_agg) :param models : dict mapping a model name with its foreign key with res_partner table |
| `_get_ordered_partner` | preparation rule | self, partner_ids | `base` | model | Helper : returns a `res.partner` recordset ordered by create_date/active fields :param partner_ids : list of partner ids to sort |
| `_compute_models` | computation | self | `base` |  | Compute the different models needed by the system if you want to exclude some partners. |
| `action_skip` | user action | self | `base` |  | Skip this wizard line. Don't compute any thing, and simply redirect to the new step. |
| `_action_next_screen` | internal rule | self | `base` |  | return the action of the next screen ; this means the wizard is set to treat the next wizard line. Each line is a subset of partner that can be merged together. If no line left, the end screen will be displayed (but an action is still returned). |
| `_process_query` | background operation | self, query | `base` |  | Execute the select request and write the result in this wizard :param query : the SQL query used to fill the wizard line |
| `action_start_manual_process` | user action | self | `base` |  | Start the process 'Merge with Manual Check'. Fill the wizard according to the group_by and exclude options, and redirect to the first step (treatment of first wizard line). After, for each subset of partner to merge, the wizard will be actualized.      - Compute the selected groups (with duplication)     - If the user has selected the ``exclude_xxx`` fields, avoid the partners |
| `action_start_automatic_process` | user action | self | `base` |  | Start the process 'Merge Automatically'. This will fill the wizard with the same mechanism as 'Merge with Manual Check', but instead of refreshing wizard with the current line, it will automatically process all lines by merging partner grouped according to the checked options. |
| `parent_migration_process_cb` | operation | self | `base` |  |  |
| `action_update_all_process` | user action | self | `base` |  |  |
| `action_merge` | user action | self | `base` |  | Merge Contact button. Merge the selected partners, and redirect to the end screen (since there is no other wizard line to process. |
| `_merge_loyalty_cards` | internal rule | self, src_partners, dst_partner | `loyalty` |  | Merge nominative loyalty cards.  :param src_partners: recordset of source res.partner records to merge :param dst_partner: destination res.partner record |

## Validation and error messages (6)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_merge` | UserError | For safety reasons, you cannot merge more than 3 contacts together. You can re-open the wizard several times if needed. | `base` |
| `_merge` | UserError | You cannot merge a contact with one of his parent. | `base` |
| `_merge` | UserError | You cannot merge contacts linked to more than one user even if only one is active. | `base` |
| `_merge` | UserError | All contacts must have the same email. Only the Administrator can merge contacts with different emails. | `base` |
| `_compute_selected_groupby` | UserError | You have to specify a filter for your selection. | `base` |
| `_update_foreign_keys` | UserError | You cannot merge these contacts because multiple contacts are enrolled in the same courses: %s | `website_slides` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_partner_manager` | yes | yes | yes | no | `base` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.base_partner_merge_automatic_wizard_form` | form |  | `number_group`, `group_by_email`, `group_by_name`, `group_by_is_company`, `group_by_vat`, `group_by_parent_id`, `exclude_contact`, `exclude_journal_item`, `maximum_group`, `dst_partner_id`, `partner_ids`, `id`, `display_name`, `email`, `is_company`, `vat`, `country_id` | `Deduplicate the other Contacts`, `Merge Contacts`, `Skip these contacts`, `Merge with Manual Check`, `Merge Automatically`, `Merge Automatically all process`, `Cancel`, `Close` |  | `base` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.action_partner_deduplicate` | Deduplicate Contacts | form |  | `{'active_test': False}` | new | `base` |
| `base.action_partner_merge` | Merge | form |  |  | new | `base` |

Machine-readable definition: `../../../schemas/data/entities/base.partner.merge.automatic.wizard.json`; views: `../../../schemas/interfaces/views/base.partner.merge.automatic.wizard.json`.
