# Physical data catalogue

How every entity, field and relation of the domain model becomes storage: the table of each entity, the physical type of each column, the association tables of the many-to-many relations, the indexes, the foreign keys, the constraints, the number generators, and the derivation procedure that builds and maintains the schema. The catalogue of section 12 lists every entity with its table, its column count, its association tables, its indexes and its constraints. Where this document says "table" it means the relation that stores an entity, which for the thirty-two entities of section 1.1 is a stored query rather than a table. The complete column-level catalogue, one structured document per entity with every field attribute, is in [`../../schemas/data/`](../../schemas/data/): `entities/<transport name>.json` for the field-by-field description, `physical-tables.json` for tables and columns, `association-tables.json` for association tables, `indexes.json` for indexes, `foreign-keys.json` for foreign keys, `table-constraints.json` for the constraints the store reports and `database-sequences.json` for the number generators.

Everything counted in this document is observed on one installation carrying every capability package: 1,240 relations holding 13,518 columns, 2,933 indexes and 4,575 foreign keys. 1,217 of the relations are tables and 23 are stored queries materialized as views.

The storage shapes described here are abstract. A replacement must reproduce the value domain, the precision, the null semantics, the uniqueness and the deletion behaviour of each column. The spelling of a type in a particular storage engine is not part of the specification; the mapping tables name the shape (exact decimal, unbounded text, structured document) and state the behaviour that the shape must provide. Where a type name is reproduced in code font it is the name the observed schema reports, because a rebuild that must import an existing database needs it.

The identity, value, rounding, translation, archiving and deletion rules that these shapes serve are in [`persistence-identity-and-values.md`](persistence-identity-and-values.md); the entity map is in [`domain-model.md`](domain-model.md); how records reach these tables at installation and at import is in [`data-loading-and-exchange.md`](data-loading-and-exchange.md); the records a fresh installation must contain are in [`reference-data.md`](reference-data.md).

## 1. The storage model

### 1.1 One table per entity

1. Every persistent entity owns one table. One row is one record. 600 entities are persistent.
2. Every interactive assistant entity also owns one table, with the same rules. Its rows are removed by the periodic cleanup, and its foreign key defaults differ (section 9.2). 222 entities are interactive assistants.
3. A shared behaviour entity has no table. Its fields are merged into every entity that carries it and become columns of those entities' tables. 161 entities are shared behaviours.
4. 32 persistent entities have no table of their own: they are read from a stored query over other tables. They accept reads and filters, they refuse writes, and the schema builder creates no column, no index, no constraint and no foreign key for them. 22 of them materialize the query as a view, which the schema therefore does carry; the other 10 build the query at read time and the schema carries no relation for them at all. They are marked in the catalogue of section 12.
5. 2 further entities classified persistent carry no relation in the observed installation: one because its capability package is not part of that installation, and one because it is a shared behaviour that the registry reports as persistent. They are marked in the catalogue of section 12.
6. 437 of the 1,240 relations back no entity: they are the association tables of section 8.

### 1.2 The columns every table has

| Column | Full name | Physical type | Rule |
|---|---|---|---|
| `id` | Identifier | `integer`, thirty-two bit integer, fed by the table's own generator, primary key | Assigned by the store, never by the caller, never writable, never reused after a deletion. |
| `create_date` | Created on | `timestamp without time zone`, in coordinated universal time | Written at creation with the timestamp of the transaction. |
| `create_uid` | Created by | `integer`, thirty-two bit integer, foreign key to `res_users`, cleared when the user row is deleted | Written at creation with the acting user. |
| `write_date` | Last updated on | `timestamp without time zone`, in coordinated universal time | Written at creation and at every write, including a write performed only to store a recomputed value. |
| `write_uid` | Last updated by | `integer`, thirty-two bit integer, foreign key to `res_users`, cleared when the user row is deleted | Written at creation and at every write. |

The four audit columns exist only when access logging is enabled for the entity. Access logging is enabled by default for every entity that has an automatic table, and it is mandatory for interactive assistant entities because the cleanup procedure reads `write_date`. 10 entities switch it off; they are marked "no audit columns" in the catalogue of section 12, and their tables carry the primary key and the value columns only.

One further system column exists on the eleven entities that maintain a materialized ancestor path: `parent_path`, variable length text, always indexed. Its content and its maintenance are specified in [`persistence-identity-and-values.md`](persistence-identity-and-values.md), section 14.

### 1.3 Column order inside a table

When the schema builder creates a table it emits the primary key first and then the value columns sorted by a fixed rank of their physical type, so that fixed-width columns are grouped and storage padding is minimized. The rank is:

| Rank | Physical type | Rank | Physical type |
|---|---|---|---|
| 1 | `integer`, thirty-two bit integer | 6 | `numeric`, exact decimal number |
| 2 | `character varying`, variable length text | 7 | `boolean` |
| 3 | `date` | 8 | `timestamp without time zone` |
| 4 | `jsonb`, structured document | 9 | `double precision` |
| 5 | `text`, unbounded text | 16 | every other type |

Within one rank the order is the declaration order of the fields. Column order has no effect on behaviour; a replacement may ignore it.

### 1.4 The schema derivation procedure

The schema is derived from the entity definitions, once per installation or upgrade of a capability package, in this order. Steps 1 to 6 run per entity; steps 7 to 10 run once for all entities, after every entity has been processed, because they depend on tables that may not exist yet when the entity is processed.

1. **Validate the entity name.** A name that is not composed of lowercase letters, digits, underscores and dots is rejected and the entity fails to load.
2. **Create the table if it does not exist**, with the primary key column and every stored column that has a physical type, each column carrying the label of its field as a comment. A column whose field is required is created with a not-null marker at this point.
3. **Create the ancestor path column** when the entity declares a materialized hierarchy and the column is absent; remember that the paths must be computed at the end (step 10).
4. **Release stale not-null markers.** Every column of the table that no longer corresponds to a field is inspected: the column is left in place, it is never dropped automatically, and if it carries a not-null marker the marker is removed, so that a package that removed a required field does not block insertion.
5. **Create or adjust one column per stored field**, in the rank order of section 1.3:
   - When the column is absent, create it. A boolean column is created with the default value false.
   - When the column exists with a different physical type, convert it in place. Converting to or from a per-language document rebuilds the values: converting to a per-language document wraps the existing value under the base language code; converting away from it takes the value stored under the base language code. Converting a column drops the indexes that depend on it, and drops any stored query that depends on it when the conversion cannot proceed otherwise.
   - When the column is new and the field has a default, fill the existing rows with the default value, except for an optional boolean, where an empty value and false are equivalent and the rows are left untouched. This is a single bulk update, not a per-row write.
   - When the field became required and the column has no not-null marker, first compute the missing values (for a derived field, the records whose column is empty are queued for recomputation), flush them, then set the marker. When the field stopped being required, drop the marker.
   - When the field is a stored copy of a value reached through a single link, and the column has just been created, fill it with one bulk update that joins the target table, instead of computing it record by record.
6. **Queue recomputation** of the derived stored fields whose column was just created, for every existing row.
7. **Create the association table** of every stored many-to-many field that does not have one yet (section 8).
8. **Create or repair the foreign keys** (section 9). Foreign keys are created after all tables exist.
9. **Create, drop and recreate the indexes** (section 10) and add the declared table constraints (section 11). A constraint that fails to apply because existing rows violate it is reported with the message of the constraint and the installation of the package is refused.
10. **Compute the ancestor paths** for every entity whose path column was created in step 3.

Two rules govern repeated runs: creating an object that already exists is skipped, and a declared object whose definition changed is dropped and recreated. The definition of a constraint and of an index is stored as a comment on the object, so that the builder can compare the stored definition with the declared one and detect a change.

### 1.5 What the procedure never does

1. It never drops a column, even when the field disappears. Obsolete columns stay until they are removed deliberately.
2. It never drops a table.
3. It never creates a foreign key on a polymorphic link (section 9.3), on a per-company column, on a table read from a stored query, or towards such a table.
4. It never adds an index automatically on a link column; an index exists only where the field declares one (section 10).

## 2. Naming rules

Every name below is contractual: a rebuild that must import an existing database, or serve an existing integration, reproduces it character for character.

| Object | Rule | Example |
|---|---|---|
| Table of an entity | The transport name of the entity with every dot replaced by an underscore. | The entity Journal Entry (`account.move`) is stored in `account_move`. |
| Column of a field | The field name, unchanged. A link to one record conventionally ends in `_id`, a list of records in `_ids`; the suffix is part of the name, not a rule the store applies. | The link from a journal item to its journal entry is the column `move_id`. |
| Association table | `<first table>_<second table>_rel`, where the two tables are sorted alphabetically, unless the relation declares its own name. | `account_account_account_tag_rel` links `account_account` and `account_account_tag`. |
| Association column | `<table>_id`, one per side, unless the relation declares its own names. | `account_account_id`, `account_account_tag_id` |
| Column index | `<table>__<column>_index` | `account_move__name_index` |
| Declared table object (constraint or index) | `<table><object name>`, where the declared object name conventionally starts with an underscore. | `account_move_line_check_credit_debit` |
| Foreign key | Named by the store with its own convention, `<table>_<column>_fkey`; the specification identifies a foreign key by its table, its column and its target, not by its name. | `account_move_journal_id_fkey` |
| Identifier generator | `<table>_id_seq`, one per table, owned by the primary key column. | `account_move_id_seq` |
| Document numbering generator | `ir_sequence_<sequence record identifier on three digits>`, and `ir_sequence_<sequence record identifier>_<date range record identifier>` for a date-range subsequence (section 7.2). | `ir_sequence_042`, `ir_sequence_042_007` |

**Length limit.** Every generated name is limited to 63 characters. A longer name is truncated to its first 54 characters, followed by an underscore and the eight hexadecimal digits of a cyclic redundancy checksum of the full name, which keeps generated names distinct. A replacement that has no such limit may use the full name; a replacement that has a different limit must apply the same truncate-and-checksum rule so that names stay stable and unique.

**Reserved names.** `id`, `create_date`, `create_uid`, `write_date`, `write_uid` and `parent_path` are reserved: a field may not use them, and values supplied for them in a create or a write are silently removed before the operation runs.

## 3. Field type to column type

The complete mapping. "No column" means the value is not stored in the entity's own table; the behaviour column says where it lives instead. The last column counts the stored columns of that shape across every entity table of the observed installation.

| Field type | Storage shape | Physical column type | Value domain and behaviour | Stored columns |
|---|---|---|---|---|
| `Boolean` | boolean | boolean | Two values. The column is created with the default false. An empty column and false are indistinguishable on read: an empty column reads as false. | 1,261 |
| `Integer` | integer | `integer`, thirty-two bit integer | Whole numbers. An empty column reads as zero, and writing zero writes zero, not an empty value. A value outside the signed thirty-two bit range is refused by the store. | 591 |
| `Float` with a declared number of decimal places | decimal with a scale | `numeric`, exact decimal number | The value is rounded to the declared number of decimal places before it is written, and written as a decimal string so that no binary floating point error is introduced. An empty column reads as zero. | counted with the row above |
| `Float` without a declared number of decimal places | decimal without a scale | `double precision`, binary floating point number | Every significant digit is written and nothing is rounded. Used for rates, ratios, factors and geographic coordinates, where a fixed scale would lose information. | 412 |
| `Monetary` | monetary | `numeric`, exact decimal number | Rounded with the rounding factor of the currency named by the field currency field before writing (section 4.4). An empty column reads as zero. | 139 |
| `Char` | single-line text | `character varying`, variable length text, carrying the declared maximum length when the field declares one | A value longer than the declared maximum is truncated on write, not refused. Leading and trailing whitespace is removed by the client and by the import service unless the field switches trimming off. | 1,841 |
| `Text` | long text | `text`, unbounded text | No length limit, no truncation, no trimming. | 187 |
| `Html` | rich text | `text`, unbounded text | Sanitized on write (section 4.5). | 135 |
| `Date` | calendar date | `date`, calendar date | A date with no time and no zone. | 228 |
| `Datetime` | moment | `timestamp without time zone`, date and time stored in coordinated universal time | Written and read in coordinated universal time, with no sub-second part. Converted to the reader zone for display only. | 258 |
| `Binary` stored as an attachment (the default) | binary content | no column | The bytes live in an Attachment record (`ir.attachment`) naming the entity, the record identifier and the field. | 11 |
| `Binary` stored in the table | binary content | `bytea`, binary string | Only for the twelve fields that switch attachment storage off; the columns are listed in section 4.3. | counted with the row above |
| `Image` | image | same as `Binary` | An image field is a binary field with maximum dimensions; it resizes its content on write. | 1 |
| `Selection` | closed value list | `character varying`, variable length text | The stored value is the technical value of the chosen entry, never its label. The closed list belongs to the field definition. | 925 |
| `Reference` | polymorphic reference in one column | `character varying`, variable length text | The value is the transport name of the entity, a comma and the record identifier, for example `account.move,42`. No foreign key. | 10 |
| `Many2one` | link to one record | `integer`, thirty-two bit integer | The identifier of the target record, with a foreign key (section 9). An empty column means no link. | 2,278 |
| `Many2oneReference` | polymorphic link in two columns | `integer`, thirty-two bit integer, plus a separate text column naming the entity | The pair is written and read together; the entity name lives in the companion field the relation names. No foreign key. | 12 |
| `One2many` | list of records | no column | The relation is the mirror of a link column on the target entity. | counted with the row above |
| `Many2many` | list of records on both sides | no column | The relation lives in an association table (section 8). | counted with the row above |
| `Json` | structured document | `jsonb`, structured document | A nested document of keys, values, lists and numbers, written and read as one value, queryable by key. | 24 |
| `Properties` | user-defined field values | `jsonb`, structured document | The values of the user-defined fields of the record, keyed by the internal name of each definition (section 4.6). | 14 |
| `PropertiesDefinition` | user-defined field definitions | `jsonb`, structured document | The list of user-defined field definitions carried by a parent record (section 4.6). | 15 |
| Any text type declared translatable | per-language text | `jsonb`, structured document keyed by language code | See section 4.1. 319 stored columns take this shape. | counted with the row above |
| Any type declared company-dependent | per-company value | `jsonb`, structured document keyed by company identifier | See section 4.2. 52 stored columns take this shape. | counted with the row above |

The same columns counted by the type the storage engine reports, across all 1,240 relations, including the association tables and the views:

| Storage type | Meaning | Columns |
|---|---|---|
| `integer` | thirty-two bit integer | 6,141 |
| `character varying` | variable length text | 2,654 |
| `timestamp without time zone` | date and time without zone offset, stored in coordinated universal time | 1,809 |
| `boolean` | boolean | 1,316 |
| `jsonb` | structured document | 514 |
| `numeric` | exact decimal number | 307 |
| `double precision` | binary floating point number | 264 |
| `text` | unbounded text | 250 |
| `date` | calendar date | 231 |
| `bigint` | sixty-four bit integer | 20 |
| `bytea` | binary string | 12 |

Of the 2,654 `character varying` columns, 2,602 are declared without a maximum length and 52 carry one. A maximum length is a property of the column that a replacement must create with the column, because a value longer than the maximum is truncated on write rather than refused, and a replacement whose column is unbounded would keep text that the original silently shortened. The 52 bounded columns, counted by the maximum they declare:

| Maximum length in characters | Columns |
|---|---|
| 1024 | 1 |
| 256 | 1 |
| 128 | 1 |
| 64 | 1 |
| 50 | 1 |
| 45 | 1 |
| 43 | 1 |
| 40 | 1 |
| 32 | 2 |
| 20 | 10 |
| 16 | 3 |
| 14 | 2 |
| 13 | 2 |
| 11 | 3 |
| 10 | 2 |
| 9 | 1 |
| 8 | 4 |
| 7 | 1 |
| 5 | 4 |
| 4 | 2 |
| 3 | 4 |
| 2 | 3 |
| 1 | 1 |
| **Total** | **52** |

Each of those 52 columns, so that a replacement can declare the same bound on the same column:

| Maximum length in characters | Table | Column | Entity | Transport name |
|---|---|---|---|---|
| 1024 | `ir_attachment` | `url` | Attachment | `ir.attachment` |
| 256 | `l10n_fr_fec_export_wizard` | `filename` | Fichier Echange Informatise | `l10n_fr.fec.export.wizard` |
| 128 | `hr_applicant` | `email_from` | Applicant | `hr.applicant` |
| 64 | `fleet_vehicle_log_contract` | `ins_ref` | Vehicle Contract | `fleet.vehicle.log.contract` |
| 50 | `discuss_channel` | `uuid` | Discussion Channel | `discuss.channel` |
| 45 | `pos_config` | `proxy_ip` | Point of Sale Configuration | `pos.config` |
| 43 | `iap_account` | `account_token` | in-app purchase Account | `iap.account` |
| 40 | `ir_attachment` | `checksum` | Attachment | `ir.attachment` |
| 32 | `hr_applicant` | `partner_phone` | Applicant | `hr.applicant` |
| 32 | `ir_module_module` | `license` | Module | `ir.module.module` |
| 20 | `account_journal` | `l10n_hr_business_premises_label` | Journal | `account.journal` |
| 20 | `account_journal` | `l10n_hr_business_premises_label_refund` | Journal | `account.journal` |
| 20 | `l10n_it_ddt` | `name` | Transport Document | `l10n_it.ddt` |
| 20 | `res_company` | `l10n_it_eco_index_number` | Companies | `res.company` |
| 20 | `stock_picking` | `l10n_ro_edi_stock_trailer_1_number` | Transfer | `stock.picking` |
| 20 | `stock_picking` | `l10n_ro_edi_stock_trailer_2_number` | Transfer | `stock.picking` |
| 20 | `stock_picking` | `l10n_ro_edi_stock_vehicle_number` | Transfer | `stock.picking` |
| 20 | `stock_picking_batch` | `l10n_ro_edi_stock_trailer_1_number` | Batch Transfer | `stock.picking.batch` |
| 20 | `stock_picking_batch` | `l10n_ro_edi_stock_trailer_2_number` | Batch Transfer | `stock.picking.batch` |
| 20 | `stock_picking_batch` | `l10n_ro_edi_stock_vehicle_number` | Batch Transfer | `stock.picking.batch` |
| 16 | `ir_module_module` | `state` | Module | `ir.module.module` |
| 16 | `res_company` | `l10n_it_codice_fiscale` | Companies | `res.company` |
| 16 | `res_partner` | `l10n_it_codice_fiscale` | Contact | `res.partner` |
| 14 | `res_partner` | `l10n_es_edi_facturae_ac_logical_operational_point` | Contact | `res.partner` |
| 14 | `res_partner` | `l10n_es_edi_facturae_ac_physical_gln` | Contact | `res.partner` |
| 13 | `res_partner` | `l10n_hu_group_vat` | Contact | `res.partner` |
| 13 | `res_partner` | `l10n_rs_edi_registration_number` | Contact | `res.partner` |
| 11 | `res_country` | `l10n_ar_legal_entity_vat` | Country | `res.country` |
| 11 | `res_country` | `l10n_ar_natural_vat` | Country | `res.country` |
| 11 | `res_country` | `l10n_ar_other_vat` | Country | `res.country` |
| 10 | `res_bank` | `l10n_cl_sbif_code` | Bank | `res.bank` |
| 10 | `res_partner` | `l10n_es_edi_facturae_ac_center_code` | Contact | `res.partner` |
| 9 | `res_partner` | `l10n_no_bronnoysund_number` | Contact | `res.partner` |
| 8 | `account_analytic_line` | `code` | Analytic Line | `account.analytic.line` |
| 8 | `account_move` | `fapiao` | Journal Entry | `account.move` |
| 8 | `auth_totp_device` | `index` | Authentication Device | `auth_totp.device` |
| 8 | `res_users_apikeys` | `index` | Users application programming interface Keys | `res.users.apikeys` |
| 7 | `res_partner` | `l10n_it_pa_index` | Contact | `res.partner` |
| 5 | `account_journal` | `code` | Journal | `account.journal` |
| 5 | `res_partner` | `l10n_rs_edi_public_funds` | Contact | `res.partner` |
| 5 | `res_partner` | `l10n_tr_nilvera_edispatch_customs_zip` | Contact | `res.partner` |
| 5 | `stock_warehouse` | `code` | Warehouse | `stock.warehouse` |
| 4 | `account_tax` | `l10n_de_datev_code` | Tax | `account.tax` |
| 4 | `res_currency` | `l10n_ar_afip_code` | Currency | `res.currency` |
| 3 | `account_incoterms` | `code` | Incoterms | `account.incoterms` |
| 3 | `account_journal` | `l10n_ec_emission` | Journal | `account.journal` |
| 3 | `account_journal` | `l10n_ec_entity` | Journal | `account.journal` |
| 3 | `res_country` | `l10n_ar_afip_code` | Country | `res.country` |
| 2 | `account_payment_term_line` | `days_next_month` | Payment Terms Line | `account.payment.term.line` |
| 2 | `res_country` | `code` | Country | `res.country` |
| 2 | `res_country_state` | `l10n_in_tin` | Country state | `res.country.state` |
| 1 | `ir_model_constraint` | `type` | Model Constraint | `ir.model.constraint` |

## 4. Special storage forms

### 4.1 Per-language text

A single-line text, long text or rich text field declared translatable is not stored as plain text. Its column is a structured document whose keys are language codes and whose values are the text in that language: the entry `en_US` holding "Customer Invoice", the entry `fr_FR` holding "Facture client" and the entry `nl_NL` holding "Klantfactuur" in one document.

Rules:

1. The base language code is `en_US`. Every stored document holds a value under the base language code; that value is the source text from which translations are derived.
2. Creating a record writes the value twice: once under the base language code and once under the language of the writer, unless they are the same.
3. Reading resolves the language of the reader first and falls back to the base language code when the reader's language is absent. For the two-step language variants used while a translation is being prepared, the fallback chain is: the pending value of the reader's language, then the confirmed value of the reader's language, then the pending base value, then the confirmed base value.
4. Filtering and sorting on a translatable column resolve the same chain: the condition is evaluated on the first non-empty value of the chain.
5. A translatable column can only carry a text-similarity index, and the index is built on the concatenation of all the language values of the document, so that a search matches text in any language (section 10.3).
6. A rich text field that is translatable is translated term by term instead of as a whole: the markup structure is kept and each text term inside it is translated separately, so that a change of markup does not invalidate the translations.
7. Converting a plain column into a per-language document wraps the existing value under the base language code; converting back takes the base language value and discards the others.

319 stored columns carry this shape.

### 4.2 Per-company values

A field declared company-dependent holds one value per company in a single column, a structured document keyed by the company identifier written as text: the entry `1` holding 30 and the entry `3` holding 60 in one document.

Rules:

1. Reading returns the value stored for the active company. When the active company has no entry, the value falls back to the default configured for the entity and the field (a User Default record, `ir.default`), and when there is none, to the empty value of the field type.
2. Writing writes only the entry of the active company; the entries of the other companies are untouched.
3. Filtering and sorting use the entry of the active company, with the same fallback: the condition is evaluated on the value for the active company, and when absent on the configured default, cast to the underlying type of the field.
4. A company-dependent column never carries a foreign key, even when the field is a link, because the identifiers live inside the document. Consistency between the company of the linked record and the company whose entry is written is checked by the company consistency rule, not by the store.
5. A company-dependent field cannot be required and cannot be translatable. It can be of these types only: boolean, integer, decimal, monetary, single-line text, long text, rich text, date, date and time, closed value list, link to one record.
6. An index on a company-dependent column is a partial index on the rows where the document is not empty.

52 stored columns carry this shape.

### 4.3 Binary content

A binary or image field stores its bytes in an Attachment record (`ir.attachment`, table `ir_attachment`) by default, not in a column of the entity's table. The attachment names the entity, the record identifier and the field name, and holds the content either in the file store, keyed by the checksum of the bytes, or inline. Consequences a replacement must reproduce:

1. Reading a binary field reads the attachment; the value is presented as the encoded content of the file.
2. Deleting a record deletes the attachments attached to it, including the ones that hold its binary fields.
3. Duplicating a record duplicates the attachments of the binary fields that are copied.
4. A read that asks for sizes instead of content returns a human-readable size, for example `12.30 Kb`, instead of the bytes, per field.
5. Twelve fields switch attachment storage off and keep their bytes in a `bytea` binary string column of their own table, for values that are small and always read with the record. They are the one-time password enrolment picture (`auth_totp.wizard`.`qrcode`), the uploaded package archive (`base.import.module`.`module_file`), the exported and imported translation files (`base.language.export`.`data` and `base.language.import`.`data`), the uploaded import file (`base_import.import`.`file`), the campaign card preview (`card.campaign`.`image_preview`), the editor conversion test value (`html_editor.converter.test`.`binary`), the stored client-action parameters (`ir.actions.client`.`params_store`), the inline attachment content (`ir.attachment`.`db_datas`), the outgoing mail server certificate and private key (`ir.mail_server`.`smtp_ssl_certificate` and `ir.mail_server`.`smtp_ssl_private_key`) and the resized company logo (`res.company`.`logo_web`).
6. Uploading a scalable vector image is refused for a user who is not an administrator, with the message "Only admins can upload SVG files." The check inspects the first byte of the value and the detected content type of the decoded value.

### 4.4 Decimal scale and monetary rounding

1. A decimal field with a fixed scale rounds to that scale, half away from zero, before writing, and the value is written as a decimal string.
2. A decimal field whose scale comes from a named decimal precision reads the number of decimal places of the named precision record at write time. The pair used by the write is sixteen significant digits and the configured number of decimal places. A named precision that does not exist yields two decimal places.
3. A monetary field rounds with the rounding factor of the currency named by its currency field, half away from zero, before writing, and writes the result as a decimal string with the number of decimal places of the currency.
4. The currency of a monetary field is resolved, at write time, from the values being written if the currency field is among them, then from the value already stored on the record, and it is read with elevated rights so that a user who may not read the currency record can still write the amount.
5. When no currency can be resolved, the amount is stored unrounded.
6. Writing several records at once with different currencies through a single value map is a definition error: the amount is rounded with one currency for the whole set.

The complete rounding, comparison and precision rules are in [`persistence-identity-and-values.md`](persistence-identity-and-values.md), section 5.

### 4.5 Rich text

A rich text value is sanitized before it is written: a fixed list of allowed elements and attributes is kept and everything else is removed. The exact options per field and the override group are specified in [`persistence-identity-and-values.md`](persistence-identity-and-values.md), section 8. The stored value is the sanitized markup; the sanitization is not repeated on read.

### 4.6 User-defined fields

A `Properties` column holds the values of the user-defined fields of the record, as a structured document keyed by the internal name of each definition: the key `3adf37f3258cfe40` holding "red" and the key `aa34746a6851ee4e` holding 1337 in one document.

The definitions — name, type, label, default, target entity, closed value list — live in a `PropertiesDefinition` column on the parent record named by the field, itself a structured document holding a list of definitions. Reading a user-defined field merges the definition from the parent with the value from the child, so that a client receives a complete field description. The allowed definition types are: boolean, integer, decimal, single-line text, long text, rich text, date, date and time, monetary, link to one record, link to many records, closed value list, tags, and separator, a display-only entry with no value. The internal name of a definition is a generated alphanumeric key of at most 512 characters; values whose definition no longer exists are removed when the record is next cleaned. 14 value columns and 15 definition columns exist.

### 4.7 Structured data

A `Json` column holds a document of keys, values, lists and numbers. Numbers are written with the shortest representation that reads back as the same number, so that a value written and read compares equal. No schema is enforced by the storage layer; the validation rules of the entity constrain the content. 24 columns carry this shape.

## 5. Empty values and null semantics

| Field type | Empty value as seen by the application | Stored as | Read back as |
|---|---|---|---|
| `Boolean` | false | false, the column default, or empty | false |
| `Integer` | 0 | 0 | 0 |
| `Float`, `Monetary` | 0 | 0 | 0 |
| `Char`, `Text`, `Html` | the empty text | empty | the empty text |
| `Date`, `Datetime` | no value | empty | no value |
| `Selection` | no value | empty | no value |
| `Many2one`, polymorphic link | no link | empty | no link |
| `One2many`, `Many2many` | the empty list | no row in the association table | the empty list |
| `Binary`, `Image` | no content | no attachment record | no content |
| `Json`, `Properties` | no document | empty | no document |

Two consequences that a replacement must reproduce exactly:

1. Writing an empty text into a text column stores an empty value, not a zero-length string, and a filter for equality with the empty text matches rows whose column is empty.
2. An empty boolean column and a false boolean column are indistinguishable to the application, which is why adding a boolean column to a populated table does not need a bulk update unless the field is required.

## 6. Value conversion on write and on read

Every write passes the application value through the conversion of the field type, and every read passes the stored value back. The conversions that change the value, rather than merely change its representation, are:

| Field type | On write | On read |
|---|---|---|
| `Integer` | An empty value becomes 0. A value carrying a record identifier becomes that identifier. | 0 when empty. |
| `Float` with a scale | Rounded to the scale, half away from zero, then written as a decimal string. | Rounded to the scale when placed in the working set, so that a value written and read in the same transaction compares equal. |
| `Monetary` | Rounded with the rounding factor of the currency, then written with the number of decimal places of the currency. | The stored value, unchanged. |
| `Char` | Truncated to the declared maximum length when the field declares one. For a translatable field, wrapped into a per-language document. | The value of the reader's language, with the fallback chain of section 4.1. |
| `Html` | Sanitized, unless the field switches sanitization off or the writer belongs to the sanitization override group and the stored value already contains content that sanitization would remove. | The stored markup. |
| `Date` | A date and time value is truncated to its date part. Text is parsed as year, month and day. | A date value. |
| `Datetime` | Text is parsed as year, month, day, hour, minute and second; a value carrying a zone offset is refused; sub-second parts are dropped. | A value in coordinated universal time, converted to the reader's zone for display only. |
| `Selection` | The value must belong to the closed list, otherwise the write is refused. | The stored technical value. |
| `Binary` | Content is decoded when needed, the content type is detected, a scalable vector image is refused for a non-administrator, and the bytes are stored in the attachment. | The encoded content, or its size when the read asks for sizes. |
| `Many2one` | A record set becomes its identifier; a pair of identifier and label becomes the identifier; a value map becomes a newly created record. An empty value becomes an empty column. | A record set of the target entity holding zero or one record. |
| `Many2many`, `One2many` | A list of commands: create, update, delete, unlink, link, clear, set. The commands are applied in order and the resulting association rows are written as a set difference against the current rows. | An ordered record set, in the default order of the target entity. |
| `Properties` | Merged with the definitions of the parent record; values whose definition is missing are dropped. | The merged definition and value list. |

## 7. Number generators

The observed installation carries 825 generators: 790 identifier generators, one per table, and 35 document numbering generators.

### 7.1 Identifier generators

Every table has one generator that feeds its primary key, named `<table>_id_seq`. It is created with the table, starts at 1, increments by 1, and is never reset. Identifiers are unique per entity, never reused after a deletion, and gaps are normal: a transaction that consumes an identifier and then rolls back leaves a gap. No business meaning may be attached to the value or to the ordering of identifiers, except that a larger identifier means a later creation within one table, which is used as the last tie-breaker of the default ordering.

### 7.2 Document numbering generators

Document numbers are produced by Sequence records (`ir.sequence`), not by table generators. A sequence record carries a code, a prefix, a suffix, a padding, a step, a next number, an optional company and an implementation choice:

| Implementation | Stored value | Storage | Behaviour |
|---|---|---|---|
| Standard | `standard` | A database generator named `ir_sequence_` followed by the identifier of the sequence record padded to three digits | Fast; concurrent callers never wait; numbers may be skipped when a transaction rolls back. |
| Without gaps | `no_gap` | The next number is the `number_next` column of the sequence record | The row is locked for update, the number is read and incremented in the same statement, and the lock is held until the transaction ends, which makes concurrent callers wait. Numbers are consecutive, except when a numbered record is deleted. |

When the sequence uses date ranges, one subsequence record exists per range, each with its own generator named `ir_sequence_<sequence record identifier>_<range record identifier>`, and the range is created on demand for the calendar year of the requested date, bounded by the neighbouring ranges.

The complete numbering rules, including the format placeholders, the year and month resets and the numbering of accounting documents, are in [`persistence-identity-and-values.md`](persistence-identity-and-values.md), sections 3 and 4.

## 8. Association tables

### 8.1 Rule

Every stored many-to-many relation is materialized by an association table with exactly two columns, both integers, both required, both foreign keys:

| Element of the association table | Rule |
|---|---|
| Table name | `<first table>_<second table>_rel`, the two entity tables sorted alphabetically, unless the relation declares its own name |
| First column | `<table of the declaring entity>_id`, thirty-two bit integer, required |
| Second column | `<table of the target entity>_id`, thirty-two bit integer, required |
| Primary key | The pair of the two columns |
| Additional index | The pair of the two columns in reverse order |
| Foreign key of the first column | To the table of the declaring entity, deleting the association row when the record is deleted |
| Foreign key of the second column | To the table of the target entity, with the deletion behaviour declared by the relation |
| Other columns | None. An association table never carries a payload, a sequence number or audit columns. A relation that needs attributes of its own is modelled as a separate entity with two links instead. |

1. The primary key is the pair of columns, which makes a pair unique: linking the same two records twice is a no-op, not a duplicate row.
2. A second index exists on the pair in reverse order, so that both directions of the relation are served by an index.
3. The foreign key on the first column always deletes the association row when the record it points at is deleted.
4. The foreign key on the second column deletes the association row when the relation declares the cascade behaviour, which is the default, and refuses the deletion of the target record while an association row exists when the relation declares the restrict behaviour. No other behaviour is allowed on a many-to-many relation.
5. The two sides of one relation share one table. When both entities declare a field for the same relation, the two fields are inverses of each other: writing one changes what the other reads.
6. Two different relations may never share the same table and column names. A definition that would collide is refused, except when the two sides belong to the same pair of entities and both declare the table and the columns explicitly, or when one of the two entities is read from a stored query.
7. A relation from an entity to itself must declare its table and its two column names explicitly, because the generated name would be ambiguous.

### 8.2 Catalogue

All 422 association tables, with the two entities they link, the two columns, the fields that declare them and the deletion behaviour on the second side. When a table is declared by two fields, both are listed.

| Association table | First entity | Transport name | First column | Second entity | Transport name | Second column | Declared by | Deletion of the second side |
|---|---|---|---|---|---|---|---|---|
| `Products` | Stock Package Destination | `stock.package.destination` | `stock_package_destination_id` | Product Moves (Stock Move Line) | `stock.move.line` | `stock_move_line_id` | Stock Package Destination (`stock.package.destination`).`move_line_ids` | `cascade` |
| `account_account_account_merge_wizard_rel` | Account merge wizard | `account.merge.wizard` | `account_merge_wizard_id` | Account | `account.account` | `account_account_id` | no declaring field in the observed registry | `cascade` |
| `account_account_account_tag` | Account | `account.account` | `account_account_id` | Account Tag | `account.account.tag` | `account_account_tag_id` | Account (`account.account`).`tag_ids` | `restrict` |
| `account_account_res_company_rel` | Account | `account.account` | `account_account_id` | Companies | `res.company` | `res_company_id` | no declaring field in the observed registry | `cascade` |
| `account_account_tag_account_move_line_rel` | Journal Item | `account.move.line` | `account_move_line_id` | Account Tag | `account.account.tag` | `account_account_tag_id` | no declaring field in the observed registry | `restrict` |
| `account_account_tag_account_tax_repartition_line_rel` | Tax Repartition Line | `account.tax.repartition.line` | `account_tax_repartition_line_id` | Account Tag | `account.account.tag` | `account_account_tag_id` | no declaring field in the observed registry | `restrict` |
| `account_account_tag_product_template_rel` | Product | `product.template` | `product_template_id` | Account Tag | `account.account.tag` | `account_account_tag_id` | no declaring field in the observed registry | `cascade` |
| `account_account_tax_default_rel` | Account | `account.account` | `account_id` | Tax | `account.tax` | `tax_id` | Account (`account.account`).`tax_ids` | `cascade` |
| `account_analytic_account_mrp_bom_rel` | Analytic Account | `account.analytic.account` | `account_analytic_account_id` | Bill of Material | `mrp.bom` | `mrp_bom_id` | no declaring field in the observed registry | `cascade` |
| `account_analytic_account_mrp_production_rel` | Analytic Account | `account.analytic.account` | `account_analytic_account_id` | Manufacturing Order | `mrp.production` | `mrp_production_id` | no declaring field in the observed registry | `cascade` |
| `account_analytic_account_mrp_workcenter_rel` | Analytic Account | `account.analytic.account` | `account_analytic_account_id` | Work Center | `mrp.workcenter` | `mrp_workcenter_id` | no declaring field in the observed registry | `cascade` |
| `account_analytic_line_stock_move_rel` | Stock Move | `stock.move` | `stock_move_id` | Analytic Line | `account.analytic.line` | `account_analytic_line_id` | no declaring field in the observed registry | `cascade` |
| `account_automatic_entry_wizard_account_move_line_rel` | Create Automatic Entries | `account.automatic.entry.wizard` | `account_automatic_entry_wizard_id` | Journal Item | `account.move.line` | `account_move_line_id` | no declaring field in the observed registry | `cascade` |
| `account_bank_statement_ir_attachment_rel` | Bank Statement | `account.bank.statement` | `account_bank_statement_id` | Attachment | `ir.attachment` | `ir_attachment_id` | no declaring field in the observed registry | `cascade` |
| `account_edi_format_account_journal_rel` | Journal | `account.journal` | `account_journal_id` | electronic data interchange format | `account.edi.format` | `account_edi_format_id` | no declaring field in the observed registry | `cascade` |
| `account_fiscal_position_account_tax_rel` | Fiscal Position | `account.fiscal.position` | `account_fiscal_position_id` | Tax | `account.tax` | `account_tax_id` | Fiscal Position (`account.fiscal.position`).`tax_ids`; Tax (`account.tax`).`fiscal_position_ids` | `cascade` |
| `account_fiscal_position_pos_config_rel` | Point of Sale Configuration | `pos.config` | `pos_config_id` | Fiscal Position | `account.fiscal.position` | `account_fiscal_position_id` | no declaring field in the observed registry | `cascade` |
| `account_fiscal_position_res_config_settings_rel` | Config Settings | `res.config.settings` | `res_config_settings_id` | Fiscal Position | `account.fiscal.position` | `account_fiscal_position_id` | no declaring field in the observed registry | `cascade` |
| `account_fiscal_position_res_country_state_rel` | Fiscal Position | `account.fiscal.position` | `account_fiscal_position_id` | Country state | `res.country.state` | `res_country_state_id` | no declaring field in the observed registry | `cascade` |
| `account_invoice_transaction_rel` | Journal Entry | `account.move` | `invoice_id` | Payment Transaction | `payment.transaction` | `transaction_id` | Journal Entry (`account.move`).`transaction_ids`; Payment Transaction (`payment.transaction`).`invoice_ids` | `cascade` |
| `account_journal_account_journal_group_rel` | Account Journal Group | `account.journal.group` | `account_journal_group_id` | Journal | `account.journal` | `account_journal_id` | no declaring field in the observed registry | `cascade` |
| `account_journal_account_reconcile_model_rel` | Preset to create journal entries during a invoices and payments matching | `account.reconcile.model` | `account_reconcile_model_id` | Journal | `account.journal` | `account_journal_id` | no declaring field in the observed registry | `cascade` |
| `account_journal_l10n_fr_fec_export_wizard_rel` | Fichier Echange Informatise | `l10n_fr.fec.export.wizard` | `l10n_fr_fec_export_wizard_id` | Journal | `account.journal` | `account_journal_id` | no declaring field in the observed registry | `cascade` |
| `account_move__account_payment` | Journal Entry | `account.move` | `invoice_id` | Payments | `account.payment` | `payment_id` | Journal Entry (`account.move`).`matched_payment_ids`; Payments (`account.payment`).`invoice_ids` | `cascade` |
| `account_move_account_move_send_batch_wizard_rel` | Account Move Send Batch Wizard | `account.move.send.batch.wizard` | `account_move_send_batch_wizard_id` | Journal Entry | `account.move` | `account_move_id` | no declaring field in the observed registry | `cascade` |
| `account_move_account_peppol_rejection_wizard_rel` | Peppol Rejection wizard | `account.peppol.rejection.wizard` | `account_peppol_rejection_wizard_id` | Journal Entry | `account.move` | `account_move_id` | no declaring field in the observed registry | `cascade` |
| `account_move_account_resequence_wizard_rel` | Remake the sequence of Journal Entries. | `account.resequence.wizard` | `account_resequence_wizard_id` | Journal Entry | `account.move` | `account_move_id` | no declaring field in the observed registry | `cascade` |
| `account_move_debit_move` | Add Debit Note wizard | `account.debit.note` | `debit_id` | Journal Entry | `account.move` | `move_id` | Add Debit Note wizard (`account.debit.note`).`move_ids` | `cascade` |
| `account_move_l10n_id_qris_transaction_rel` | Journal Entry | `account.move` | `account_move_id` | Record of QRIS transactions | `l10n_id.qris.transaction` | `l10n_id_qris_transaction_id` | no declaring field in the observed registry | `cascade` |
| `account_move_l10n_ph_2307_wizard_rel` | Exports 2307 data to a XLS file. | `l10n_ph_2307.wizard` | `l10n_ph_2307_wizard_id` | Journal Entry | `account.move` | `account_move_id` | no declaring field in the observed registry | `cascade` |
| `account_move_line_account_tax_rel` | Tax | `account.tax` | `account_tax_id` | Journal Item | `account.move.line` | `account_move_line_id` | Journal Item (`account.move.line`).`tax_ids`; Tax (`account.tax`).`account_move_line_ids` | `cascade` |
| `account_move_nemhandel_rejection_wizard_rel` | Nemhandel Rejection wizard | `nemhandel.rejection.wizard` | `nemhandel_rejection_wizard_id` | Journal Entry | `account.move` | `account_move_id` | no declaring field in the observed registry | `cascade` |
| `account_move_pdp_response_wizard_rel` | Approved Dematerialization Platform Response wizard | `pdp.response.wizard` | `pdp_response_wizard_id` | Journal Entry | `account.move` | `account_move_id` | no declaring field in the observed registry | `cascade` |
| `account_move_purchase_order_rel` | Purchase Order | `purchase.order` | `purchase_order_id` | Journal Entry | `account.move` | `account_move_id` | no declaring field in the observed registry | `cascade` |
| `account_move_reversal_move` | Account Move Reversal | `account.move.reversal` | `reversal_id` | Journal Entry | `account.move` | `move_id` | Account Move Reversal (`account.move.reversal`).`move_ids` | `cascade` |
| `account_move_reversal_new_move` | Account Move Reversal | `account.move.reversal` | `reversal_id` | Journal Entry | `account.move` | `new_move_id` | Account Move Reversal (`account.move.reversal`).`new_move_ids` | `cascade` |
| `account_move_send_wizard_res_partner_rel` | Account Move Send Wizard | `account.move.send.wizard` | `account_move_send_wizard_id` | Contact | `res.partner` | `res_partner_id` | no declaring field in the observed registry | `cascade` |
| `account_move_tbai_reversed_moves` | Journal Entry | `account.move` | `refund_id` | Journal Entry | `account.move` | `reversed_move_id` | Journal Entry (`account.move`).`l10n_es_tbai_reversed_ids` | `cascade` |
| `account_move_validate_account_move_rel` | Validate Account Move | `validate.account.move` | `validate_account_move_id` | Journal Entry | `account.move` | `account_move_id` | no declaring field in the observed registry | `cascade` |
| `account_payment_account_bank_statement_line_rel` | Bank Statement Line | `account.bank.statement.line` | `account_bank_statement_line_id` | Payments | `account.payment` | `account_payment_id` | Bank Statement Line (`account.bank.statement.line`).`payment_ids` | `cascade` |
| `account_payment_method_line_res_company_rel` | Companies | `res.company` | `res_company_id` | Payment Methods | `account.payment.method.line` | `account_payment_method_line_id` | no declaring field in the observed registry | `cascade` |
| `account_payment_register_l10n_latam_check_rel` | Pay | `account.payment.register` | `account_payment_register_id` | Account payment check | `l10n_latam.check` | `l10n_latam_check_id` | no declaring field in the observed registry | `cascade` |
| `account_payment_register_move_line_rel` | Pay | `account.payment.register` | `wizard_id` | Journal Item | `account.move.line` | `line_id` | Pay (`account.payment.register`).`line_ids` | `cascade` |
| `account_peppol_rejection_action_rel` | Peppol Rejection wizard | `account.peppol.rejection.wizard` | `account_peppol_rejection_wizard_id` | Peppol clarifications used for rejection | `account.peppol.clarification` | `account_peppol_clarification_id` | Peppol Rejection wizard (`account.peppol.rejection.wizard`).`action_ids` | `cascade` |
| `account_peppol_rejection_reason_rel` | Peppol Rejection wizard | `account.peppol.rejection.wizard` | `account_peppol_rejection_wizard_id` | Peppol clarifications used for rejection | `account.peppol.clarification` | `account_peppol_clarification_id` | Peppol Rejection wizard (`account.peppol.rejection.wizard`).`reason_ids` | `cascade` |
| `account_reconcile_model_line_account_tax_rel` | Tax | `account.tax` | `account_tax_id` | Rules for the reconciliation model | `account.reconcile.model.line` | `account_reconcile_model_line_id` | Rules for the reconciliation model (`account.reconcile.model.line`).`tax_ids`; Tax (`account.tax`).`account_reconcile_model_line_ids` | `cascade` |
| `account_reconcile_model_res_partner_rel` | Preset to create journal entries during a invoices and payments matching | `account.reconcile.model` | `account_reconcile_model_id` | Contact | `res.partner` | `res_partner_id` | no declaring field in the observed registry | `cascade` |
| `account_report_section_rel` | Accounting Report | `account.report` | `main_report_id` | Accounting Report | `account.report` | `sub_report_id` | Accounting Report (`account.report`).`section_main_report_ids`; Accounting Report (`account.report`).`section_report_ids` | `cascade` |
| `account_tax_alternatives` | Tax | `account.tax` | `dest_tax_id` | Tax | `account.tax` | `src_tax_id` | Tax (`account.tax`).`original_tax_ids`; Tax (`account.tax`).`replacing_tax_ids` | `cascade` |
| `account_tax_filiation_rel` | Tax | `account.tax` | `parent_tax` | Tax | `account.tax` | `child_tax` | Tax (`account.tax`).`children_tax_ids` | `cascade` |
| `account_tax_hr_expense_split_rel` | Expense Split | `hr.expense.split` | `hr_expense_split_id` | Tax | `account.tax` | `account_tax_id` | no declaring field in the observed registry | `cascade` |
| `account_tax_pos_order_line_rel` | Tax | `account.tax` | `account_tax_id` | Point of Sale Order Lines | `pos.order.line` | `pos_order_line_id` | Tax (`account.tax`).`pos_order_line_ids`; Point of Sale Order Lines (`pos.order.line`).`tax_ids` | `cascade` |
| `account_tax_purchase_order_line_rel` | Tax | `account.tax` | `account_tax_id` | Purchase Order Line | `purchase.order.line` | `purchase_order_line_id` | Tax (`account.tax`).`purchase_order_line_ids`; Purchase Order Line (`purchase.order.line`).`tax_ids` | `cascade` |
| `account_tax_sale_order_line_rel` | Sales Order Line | `sale.order.line` | `sale_order_line_id` | Tax | `account.tax` | `account_tax_id` | no declaring field in the observed registry | `cascade` |
| `account_tax_stock_move_rel` | Stock Move | `stock.move` | `stock_move_id` | Tax | `account.tax` | `account_tax_id` | no declaring field in the observed registry | `cascade` |
| `activity_attachment_rel` | Activity | `mail.activity` | `activity_id` | Attachment | `ir.attachment` | `attachment_id` | Activity (`mail.activity`).`attachment_ids` | `cascade` |
| `adjusting_entries__account_move` | Journal Entry | `account.move` | `move_id` | Journal Entry | `account.move` | `adjusting_entry_move_id` | Journal Entry (`account.move`).`adjusting_entries_move_ids`; Journal Entry (`account.move`).`adjusting_entry_origin_move_ids` | `cascade` |
| `applicant_get_refuse_reason_duplicate_applicants_rel` | Get Refuse Reason | `applicant.get.refuse.reason` | `applicant_get_refuse_reason_id` | Applicant | `hr.applicant` | `hr_applicant_id` | Get Refuse Reason (`applicant.get.refuse.reason`).`duplicate_applicant_ids` | `cascade` |
| `applicant_get_refuse_reason_hr_applicant_rel` | Get Refuse Reason | `applicant.get.refuse.reason` | `applicant_get_refuse_reason_id` | Applicant | `hr.applicant` | `hr_applicant_id` | no declaring field in the observed registry | `cascade` |
| `applicant_get_refuse_reason_ir_attachment_rel` | Get Refuse Reason | `applicant.get.refuse.reason` | `applicant_get_refuse_reason_id` | Attachment | `ir.attachment` | `ir_attachment_id` | no declaring field in the observed registry | `cascade` |
| `applicant_send_mail_hr_applicant_rel` | Send mails to applicants | `applicant.send.mail` | `applicant_send_mail_id` | Applicant | `hr.applicant` | `hr_applicant_id` | no declaring field in the observed registry | `cascade` |
| `applicant_send_mail_ir_attachment_rel` | Send mails to applicants | `applicant.send.mail` | `applicant_send_mail_id` | Attachment | `ir.attachment` | `ir_attachment_id` | no declaring field in the observed registry | `cascade` |
| `badge_unlocked_definition_rel` | Gamification Badge | `gamification.badge` | `gamification_badge_id` | Gamification Goal Definition | `gamification.goal.definition` | `gamification_goal_definition_id` | Gamification Badge (`gamification.badge`).`goal_definition_ids` | `cascade` |
| `base_automation_ir_model_fields_rel` | Automation Rule | `base.automation` | `base_automation_id` | Fields | `ir.model.fields` | `ir_model_fields_id` | no declaring field in the observed registry | `cascade` |
| `base_automation_onchange_fields_rel` | Automation Rule | `base.automation` | `base_automation_id` | Fields | `ir.model.fields` | `ir_model_fields_id` | Automation Rule (`base.automation`).`on_change_field_ids` | `cascade` |
| `base_language_install_website_rel` | Install Language | `base.language.install` | `base_language_install_id` | Website | `website` | `website_id` | no declaring field in the observed registry | `cascade` |
| `base_module_uninstall_ir_module_module_rel` | Module Uninstall | `base.module.uninstall` | `base_module_uninstall_id` | Module | `ir.module.module` | `ir_module_module_id` | no declaring field in the observed registry | `cascade` |
| `base_partner_merge_automatic_wizard_res_partner_rel` | Merge Partner Wizard | `base.partner.merge.automatic.wizard` | `base_partner_merge_automatic_wizard_id` | Contact | `res.partner` | `res_partner_id` | no declaring field in the observed registry | `cascade` |
| `blog_post_blog_tag_rel` | Blog Tag | `blog.tag` | `blog_tag_id` | Blog Post | `blog.post` | `blog_post_id` | no declaring field in the observed registry | `cascade` |
| `calendar_alarm_calendar_event_rel` | Calendar Event | `calendar.event` | `calendar_event_id` | Event Alarm | `calendar.alarm` | `calendar_alarm_id` | Calendar Event (`calendar.event`).`alarm_ids` | `restrict` |
| `calendar_event_res_partner_rel` | Contact | `res.partner` | `res_partner_id` | Calendar Event | `calendar.event` | `calendar_event_id` | Calendar Event (`calendar.event`).`partner_ids`; Contact (`res.partner`).`meeting_ids` | `cascade` |
| `card_campaign_card_campaign_tag_rel` | Marketing Card Campaign | `card.campaign` | `card_campaign_id` | Marketing Card Campaign Tag | `card.campaign.tag` | `card_campaign_tag_id` | no declaring field in the observed registry | `cascade` |
| `chatbot_script_answer_chatbot_script_step_rel` | Chatbot Script Step | `chatbot.script.step` | `chatbot_script_step_id` | Chatbot Script Answer | `chatbot.script.answer` | `chatbot_script_answer_id` | no declaring field in the observed registry | `cascade` |
| `chatbot_script_step_im_livechat_expertise_rel` | Chatbot Script Step | `chatbot.script.step` | `chatbot_script_step_id` | Live Chat Expertise | `im_livechat.expertise` | `im_livechat_expertise_id` | no declaring field in the observed registry | `cascade` |
| `crm_convert_lead_mass_lead_rel` | Convert Lead to Opportunity (in mass) | `crm.lead2opportunity.partner.mass` | `crm_lead2opportunity_partner_mass_id` | Lead | `crm.lead` | `crm_lead_id` | Convert Lead to Opportunity (in mass) (`crm.lead2opportunity.partner.mass`).`lead_tomerge_ids` | `cascade` |
| `crm_iap_lead_industry_crm_iap_lead_mining_request_rel` | customer relationship management Lead Mining Request | `crm.iap.lead.mining.request` | `crm_iap_lead_mining_request_id` | customer relationship management in-app purchase Lead Industry | `crm.iap.lead.industry` | `crm_iap_lead_industry_id` | no declaring field in the observed registry | `cascade` |
| `crm_iap_lead_industry_crm_reveal_rule_rel` | customer relationship management Lead Generation Rules | `crm.reveal.rule` | `crm_reveal_rule_id` | customer relationship management in-app purchase Lead Industry | `crm.iap.lead.industry` | `crm_iap_lead_industry_id` | no declaring field in the observed registry | `cascade` |
| `crm_iap_lead_mining_request_crm_iap_lead_role_rel` | customer relationship management Lead Mining Request | `crm.iap.lead.mining.request` | `crm_iap_lead_mining_request_id` | People Role | `crm.iap.lead.role` | `crm_iap_lead_role_id` | no declaring field in the observed registry | `cascade` |
| `crm_iap_lead_mining_request_crm_tag_rel` | customer relationship management Lead Mining Request | `crm.iap.lead.mining.request` | `crm_iap_lead_mining_request_id` | customer relationship management Tag | `crm.tag` | `crm_tag_id` | no declaring field in the observed registry | `cascade` |
| `crm_iap_lead_mining_request_res_country_rel` | customer relationship management Lead Mining Request | `crm.iap.lead.mining.request` | `crm_iap_lead_mining_request_id` | Country | `res.country` | `res_country_id` | no declaring field in the observed registry | `cascade` |
| `crm_iap_lead_mining_request_res_country_state_rel` | customer relationship management Lead Mining Request | `crm.iap.lead.mining.request` | `crm_iap_lead_mining_request_id` | Country state | `res.country.state` | `res_country_state_id` | no declaring field in the observed registry | `cascade` |
| `crm_iap_lead_role_crm_reveal_rule_rel` | customer relationship management Lead Generation Rules | `crm.reveal.rule` | `crm_reveal_rule_id` | People Role | `crm.iap.lead.role` | `crm_iap_lead_role_id` | no declaring field in the observed registry | `cascade` |
| `crm_lead2opportunity_partner_mass_res_users_rel` | Convert Lead to Opportunity (in mass) | `crm.lead2opportunity.partner.mass` | `crm_lead2opportunity_partner_mass_id` | User | `res.users` | `res_users_id` | no declaring field in the observed registry | `cascade` |
| `crm_lead_crm_lead2opportunity_partner_mass_rel` | Convert Lead to Opportunity (in mass) | `crm.lead2opportunity.partner.mass` | `crm_lead2opportunity_partner_mass_id` | Lead | `crm.lead` | `crm_lead_id` | no declaring field in the observed registry | `cascade` |
| `crm_lead_crm_lead2opportunity_partner_rel` | Convert Lead to Opportunity (not in mass) | `crm.lead2opportunity.partner` | `crm_lead2opportunity_partner_id` | Lead | `crm.lead` | `crm_lead_id` | no declaring field in the observed registry | `cascade` |
| `crm_lead_crm_lead_lost_rel` | Get Lost Reason | `crm.lead.lost` | `crm_lead_lost_id` | Lead | `crm.lead` | `crm_lead_id` | no declaring field in the observed registry | `cascade` |
| `crm_lead_declined_partner` | Lead | `crm.lead` | `lead_id` | Contact | `res.partner` | `partner_id` | Lead (`crm.lead`).`partner_declined_ids` | `cascade` |
| `crm_lead_event_registration_rel` | Lead | `crm.lead` | `crm_lead_id` | Event Registration | `event.registration` | `event_registration_id` | no declaring field in the observed registry | `cascade` |
| `crm_lead_pls_update_crm_lead_scoring_frequency_field_rel` | Update the probabilities | `crm.lead.pls.update` | `crm_lead_pls_update_id` | Fields that can be used for predictive lead scoring computation | `crm.lead.scoring.frequency.field` | `crm_lead_scoring_frequency_field_id` | no declaring field in the observed registry | `cascade` |
| `crm_lead_website_visitor_rel` | Lead | `crm.lead` | `crm_lead_id` | Website Visitor | `website.visitor` | `website_visitor_id` | no declaring field in the observed registry | `cascade` |
| `crm_reveal_rule_crm_tag_rel` | customer relationship management Lead Generation Rules | `crm.reveal.rule` | `crm_reveal_rule_id` | customer relationship management Tag | `crm.tag` | `crm_tag_id` | no declaring field in the observed registry | `cascade` |
| `crm_reveal_rule_res_country_rel` | customer relationship management Lead Generation Rules | `crm.reveal.rule` | `crm_reveal_rule_id` | Country | `res.country` | `res_country_id` | no declaring field in the observed registry | `cascade` |
| `crm_reveal_rule_res_country_state_rel` | customer relationship management Lead Generation Rules | `crm.reveal.rule` | `crm_reveal_rule_id` | Country state | `res.country.state` | `res_country_state_id` | no declaring field in the observed registry | `cascade` |
| `crm_stage_crm_team_rel` | customer relationship management Stages | `crm.stage` | `crm_stage_id` | Sales Team | `crm.team` | `crm_team_id` | no declaring field in the observed registry | `restrict` |
| `crm_tag_event_lead_rule_rel` | Event Lead Rules | `event.lead.rule` | `event_lead_rule_id` | customer relationship management Tag | `crm.tag` | `crm_tag_id` | no declaring field in the observed registry | `cascade` |
| `crm_tag_rel` | Lead | `crm.lead` | `lead_id` | customer relationship management Tag | `crm.tag` | `tag_id` | Lead (`crm.lead`).`tag_ids` | `cascade` |
| `data_recycle_model_res_users_rel` | Recycling Model | `data_recycle.model` | `data_recycle_model_id` | User | `res.users` | `res_users_id` | no declaring field in the observed registry | `cascade` |
| `delivery_carrier_country_rel` | Shipping Methods | `delivery.carrier` | `carrier_id` | Country | `res.country` | `country_id` | Shipping Methods (`delivery.carrier`).`country_ids` | `cascade` |
| `delivery_carrier_state_rel` | Shipping Methods | `delivery.carrier` | `carrier_id` | Country state | `res.country.state` | `state_id` | Shipping Methods (`delivery.carrier`).`state_ids` | `cascade` |
| `delivery_carrier_stock_warehouse_rel` | Shipping Methods | `delivery.carrier` | `delivery_carrier_id` | Warehouse | `stock.warehouse` | `stock_warehouse_id` | no declaring field in the observed registry | `cascade` |
| `delivery_zip_prefix_rel` | Shipping Methods | `delivery.carrier` | `carrier_id` | Delivery Zip Prefix | `delivery.zip.prefix` | `zip_prefix_id` | Shipping Methods (`delivery.carrier`).`zip_prefix_ids` | `cascade` |
| `digest_digest_res_users_rel` | Digest | `digest.digest` | `digest_digest_id` | User | `res.users` | `res_users_id` | no declaring field in the observed registry | `cascade` |
| `digest_tip_res_users_rel` | Digest Tips | `digest.tip` | `digest_tip_id` | User | `res.users` | `res_users_id` | no declaring field in the observed registry | `cascade` |
| `discuss_call_history_im_livechat_channel_member_history_rel` | Keep the call history | `discuss.call.history` | `discuss_call_history_id` | Keep the channel member history | `im_livechat.channel.member.history` | `im_livechat_channel_member_history_id` | no declaring field in the observed registry | `cascade` |
| `discuss_channel_hr_department_rel` | Discussion Channel | `discuss.channel` | `discuss_channel_id` | Department | `hr.department` | `hr_department_id` | no declaring field in the observed registry | `cascade` |
| `discuss_channel_im_livechat_expertise_rel` | Discussion Channel | `discuss.channel` | `discuss_channel_id` | Live Chat Expertise | `im_livechat.expertise` | `im_livechat_expertise_id` | Discussion Channel (`discuss.channel`).`livechat_expertise_ids` | `cascade` |
| `discuss_channel_res_groups_rel` | Discussion Channel | `discuss.channel` | `discuss_channel_id` | Access Groups | `res.groups` | `res_groups_id` | no declaring field in the observed registry | `cascade` |
| `dock_location_stock_picking_type_rel` | Picking Type | `stock.picking.type` | `stock_picking_type_id` | Inventory Locations | `stock.location` | `stock_location_id` | Picking Type (`stock.picking.type`).`dock_ids` | `cascade` |
| `email_template_attachment_rel` | Email Templates | `mail.template` | `email_template_id` | Attachment | `ir.attachment` | `attachment_id` | Email Templates (`mail.template`).`attachment_ids` | `cascade` |
| `employee_bank_account_rel` | Employee | `hr.employee` | `employee_id` | Bank Accounts | `res.partner.bank` | `bank_account_id` | Employee (`hr.employee`).`bank_account_ids` | `cascade` |
| `employee_category_rel` | Employee | `hr.employee` | `employee_id` | Employee Category | `hr.employee.category` | `category_id` | Employee (`hr.employee`).`category_ids`; Employee Category (`hr.employee.category`).`employee_ids` | `cascade` |
| `event_allowed_track_tags_rel` | Event | `event.event` | `event_event_id` | Event Track Tag | `event.track.tag` | `event_track_tag_id` | Event (`event.event`).`allowed_track_tag_ids` | `cascade` |
| `event_booth_event_booth_configurator_rel` | Event Booth Configurator | `event.booth.configurator` | `event_booth_configurator_id` | Event Booth | `event.booth` | `event_booth_id` | no declaring field in the observed registry | `cascade` |
| `event_event_event_question_rel` | Event | `event.event` | `event_event_id` | Event Question | `event.question` | `event_question_id` | Event (`event.event`).`general_question_ids`; Event (`event.event`).`question_ids`; Event (`event.event`).`specific_question_ids` | `cascade` |
| `event_event_event_tag_rel` | Event | `event.event` | `event_event_id` | Event Tag | `event.tag` | `event_tag_id` | no declaring field in the observed registry | `cascade` |
| `event_lead_request_event_lead_rule_rel` | Event Lead Request | `event.lead.request` | `event_lead_request_id` | Event Lead Rules | `event.lead.rule` | `event_lead_rule_id` | no declaring field in the observed registry | `cascade` |
| `event_lead_rule_event_type_rel` | Event Lead Rules | `event.lead.rule` | `event_lead_rule_id` | Event Template | `event.type` | `event_type_id` | no declaring field in the observed registry | `cascade` |
| `event_question_event_type_rel` | Event Template | `event.type` | `event_type_id` | Event Question | `event.question` | `event_question_id` | no declaring field in the observed registry | `cascade` |
| `event_tag_event_type_rel` | Event Template | `event.type` | `event_type_id` | Event Tag | `event.tag` | `event_tag_id` | no declaring field in the observed registry | `cascade` |
| `event_track_event_track_tag_rel` | Event Track | `event.track` | `event_track_id` | Event Track Tag | `event.track.tag` | `event_track_tag_id` | no declaring field in the observed registry | `cascade` |
| `event_track_tags_rel` | Event | `event.event` | `event_event_id` | Event Track Tag | `event.track.tag` | `event_track_tag_id` | Event (`event.event`).`tracks_tag_ids` | `cascade` |
| `expense_tax` | Tax | `account.tax` | `tax_id` | Expense | `hr.expense` | `expense_id` | Tax (`account.tax`).`hr_expense_ids`; Expense (`hr.expense`).`tax_ids` | `cascade` |
| `expiry_picking_confirmation_mrp_production_rel` | Confirm Expiry | `expiry.picking.confirmation` | `expiry_picking_confirmation_id` | Manufacturing Order | `mrp.production` | `mrp_production_id` | no declaring field in the observed registry | `cascade` |
| `expiry_picking_confirmation_stock_lot_rel` | Confirm Expiry | `expiry.picking.confirmation` | `expiry_picking_confirmation_id` | Lot/Serial | `stock.lot` | `stock_lot_id` | no declaring field in the observed registry | `cascade` |
| `expiry_picking_confirmation_stock_picking_rel` | Confirm Expiry | `expiry.picking.confirmation` | `expiry_picking_confirmation_id` | Transfer | `stock.picking` | `stock_picking_id` | no declaring field in the observed registry | `cascade` |
| `fleet_service_type_fleet_vehicle_log_contract_rel` | Vehicle Contract | `fleet.vehicle.log.contract` | `fleet_vehicle_log_contract_id` | Fleet Service Type | `fleet.service.type` | `fleet_service_type_id` | no declaring field in the observed registry | `cascade` |
| `fleet_vehicle_fleet_vehicle_send_mail_rel` | Send mails to Drivers | `fleet.vehicle.send.mail` | `fleet_vehicle_send_mail_id` | Vehicle | `fleet.vehicle` | `fleet_vehicle_id` | no declaring field in the observed registry | `cascade` |
| `fleet_vehicle_mail_compose_message_ir_attachments_rel` | Send mails to Drivers | `fleet.vehicle.send.mail` | `wizard_id` | Attachment | `ir.attachment` | `attachment_id` | Send mails to Drivers (`fleet.vehicle.send.mail`).`attachment_ids` | `cascade` |
| `fleet_vehicle_model_vendors` | Model of a vehicle | `fleet.vehicle.model` | `model_id` | Contact | `res.partner` | `partner_id` | Model of a vehicle (`fleet.vehicle.model`).`vendors` | `cascade` |
| `fleet_vehicle_vehicle_tag_rel` | Vehicle | `fleet.vehicle` | `vehicle_tag_id` | Vehicle Tag | `fleet.vehicle.tag` | `tag_id` | Vehicle (`fleet.vehicle`).`tag_ids` | `cascade` |
| `forum_post_res_users_rel` | Forum Post | `forum.post` | `forum_post_id` | User | `res.users` | `res_users_id` | no declaring field in the observed registry | `cascade` |
| `forum_tag_rel` | Forum Post | `forum.post` | `forum_post_id` | Forum Tag | `forum.tag` | `forum_tag_id` | Forum Post (`forum.post`).`tag_ids`; Forum Tag (`forum.tag`).`post_ids` | `cascade` |
| `gamification_badge_rule_badge_rel` | Gamification Badge | `gamification.badge` | `badge1_id` | Gamification Badge | `gamification.badge` | `badge2_id` | Gamification Badge (`gamification.badge`).`rule_auth_badge_ids` | `cascade` |
| `gamification_challenge_users_rel` | Gamification Challenge | `gamification.challenge` | `gamification_challenge_id` | User | `res.users` | `res_users_id` | Gamification Challenge (`gamification.challenge`).`user_ids` | `cascade` |
| `gamification_invited_user_ids_rel` | Gamification Challenge | `gamification.challenge` | `gamification_challenge_id` | User | `res.users` | `res_users_id` | Gamification Challenge (`gamification.challenge`).`invited_user_ids` | `cascade` |
| `header_footer_quotation_template_rel` | Quotation's Headers & Footers | `quotation.document` | `quotation_document_id` | Quotation Template | `sale.order.template` | `sale_order_template_id` | Quotation's Headers & Footers (`quotation.document`).`quotation_template_ids`; Quotation Template (`sale.order.template`).`quotation_document_ids` | `cascade` |
| `hr_applicant_category_hr_talent_pool_rel` | Talent Pool | `hr.talent.pool` | `hr_talent_pool_id` | Category of applicant | `hr.applicant.category` | `hr_applicant_category_id` | no declaring field in the observed registry | `cascade` |
| `hr_applicant_category_talent_pool_add_applicants_rel` | Add applicants to talent pool | `talent.pool.add.applicants` | `talent_pool_add_applicants_id` | Category of applicant | `hr.applicant.category` | `hr_applicant_category_id` | no declaring field in the observed registry | `cascade` |
| `hr_applicant_hr_applicant_category_rel` | Applicant | `hr.applicant` | `hr_applicant_id` | Category of applicant | `hr.applicant.category` | `hr_applicant_category_id` | no declaring field in the observed registry | `cascade` |
| `hr_applicant_hr_skill_rel` | Applicant | `hr.applicant` | `hr_applicant_id` | Skill | `hr.skill` | `hr_skill_id` | no declaring field in the observed registry | `cascade` |
| `hr_applicant_hr_talent_pool_rel` | Applicant | `hr.applicant` | `hr_applicant_id` | Talent Pool | `hr.talent.pool` | `hr_talent_pool_id` | no declaring field in the observed registry | `cascade` |
| `hr_applicant_job_add_applicants_rel` | Add applicants to a job | `job.add.applicants` | `job_add_applicants_id` | Applicant | `hr.applicant` | `hr_applicant_id` | no declaring field in the observed registry | `cascade` |
| `hr_applicant_res_users_interviewers_rel` | Applicant | `hr.applicant` | `hr_applicant_id` | User | `res.users` | `res_users_id` | Applicant (`hr.applicant`).`interviewer_ids` | `cascade` |
| `hr_applicant_talent_pool_add_applicants_rel` | Add applicants to talent pool | `talent.pool.add.applicants` | `talent_pool_add_applicants_id` | Applicant | `hr.applicant` | `hr_applicant_id` | no declaring field in the observed registry | `cascade` |
| `hr_attendance_overtime_line_hr_attendance_overtime_rule_rel` | Attendance Overtime Line | `hr.attendance.overtime.line` | `hr_attendance_overtime_line_id` | Overtime Rule | `hr.attendance.overtime.rule` | `hr_attendance_overtime_rule_id` | no declaring field in the observed registry | `cascade` |
| `hr_department_hr_leave_mandatory_day_rel` | Mandatory Day | `hr.leave.mandatory.day` | `hr_leave_mandatory_day_id` | Department | `hr.department` | `hr_department_id` | no declaring field in the observed registry | `cascade` |
| `hr_departure_wizard_hr_employee_rel` | Departure Wizard | `hr.departure.wizard` | `hr_departure_wizard_id` | Employee | `hr.employee` | `hr_employee_id` | no declaring field in the observed registry | `cascade` |
| `hr_employee_hr_employee_cv_wizard_rel` | Print Resume | `hr.employee.cv.wizard` | `hr_employee_cv_wizard_id` | Employee | `hr.employee` | `hr_employee_id` | no declaring field in the observed registry | `cascade` |
| `hr_employee_hr_employee_delete_wizard_rel` | Employee Delete Wizard | `hr.employee.delete.wizard` | `hr_employee_delete_wizard_id` | Employee | `hr.employee` | `hr_employee_id` | no declaring field in the observed registry | `cascade` |
| `hr_employee_hr_leave_allocation_generate_multi_wizard_rel` | Generate time off allocations for multiple employees | `hr.leave.allocation.generate.multi.wizard` | `hr_leave_allocation_generate_multi_wizard_id` | Employee | `hr.employee` | `hr_employee_id` | no declaring field in the observed registry | `cascade` |
| `hr_employee_hr_leave_generate_multi_wizard_rel` | Generate time off for multiple employees | `hr.leave.generate.multi.wizard` | `hr_leave_generate_multi_wizard_id` | Employee | `hr.employee` | `hr_employee_id` | no declaring field in the observed registry | `cascade` |
| `hr_employee_hr_skill_rel` | Employee | `hr.employee` | `hr_employee_id` | Skill | `hr.skill` | `hr_skill_id` | no declaring field in the observed registry | `cascade` |
| `hr_employee_hr_work_entry_regeneration_wizard_rel` | Regenerate Employee Work Entries | `hr.work.entry.regeneration.wizard` | `hr_work_entry_regeneration_wizard_id` | Employee | `hr.employee` | `hr_employee_id` | no declaring field in the observed registry | `cascade` |
| `hr_expense_hr_expense_approve_duplicate_rel` | Expense Approve Duplicate | `hr.expense.approve.duplicate` | `hr_expense_approve_duplicate_id` | Expense | `hr.expense` | `hr_expense_id` | no declaring field in the observed registry | `cascade` |
| `hr_expense_hr_expense_refuse_wizard_rel` | Expense Refuse Reason Wizard | `hr.expense.refuse.wizard` | `hr_expense_refuse_wizard_id` | Expense | `hr.expense` | `hr_expense_id` | no declaring field in the observed registry | `cascade` |
| `hr_job_extended_interviewer_res_users` | Job Position | `hr.job` | `hr_job_id` | User | `res.users` | `res_users_id` | Job Position (`hr.job`).`extended_interviewer_ids` | `cascade` |
| `hr_job_hr_leave_mandatory_day_rel` | Mandatory Day | `hr.leave.mandatory.day` | `hr_leave_mandatory_day_id` | Job Position | `hr.job` | `hr_job_id` | no declaring field in the observed registry | `cascade` |
| `hr_job_hr_recruitment_stage_rel` | Recruitment Stages | `hr.recruitment.stage` | `hr_recruitment_stage_id` | Job Position | `hr.job` | `hr_job_id` | no declaring field in the observed registry | `cascade` |
| `hr_job_hr_skill_rel` | Job Position | `hr.job` | `hr_job_id` | Skill | `hr.skill` | `hr_skill_id` | no declaring field in the observed registry | `cascade` |
| `hr_job_job_add_applicants_rel` | Add applicants to a job | `job.add.applicants` | `job_add_applicants_id` | Job Position | `hr.job` | `hr_job_id` | no declaring field in the observed registry | `cascade` |
| `hr_job_res_users_rel` | Job Position | `hr.job` | `hr_job_id` | User | `res.users` | `res_users_id` | no declaring field in the observed registry | `cascade` |
| `hr_leave_type_res_users_rel` | Time Off Type | `hr.leave.type` | `hr_leave_type_id` | User | `res.users` | `res_users_id` | Time Off Type (`hr.leave.type`).`responsible_ids` | `cascade` |
| `hr_talent_pool_talent_pool_add_applicants_rel` | Add applicants to talent pool | `talent.pool.add.applicants` | `talent_pool_add_applicants_id` | Talent Pool | `hr.talent.pool` | `hr_talent_pool_id` | no declaring field in the observed registry | `cascade` |
| `iap_account_res_company_rel` | in-app purchase Account | `iap.account` | `iap_account_id` | Companies | `res.company` | `res_company_id` | no declaring field in the observed registry | `cascade` |
| `iap_account_res_users_rel` | in-app purchase Account | `iap.account` | `iap_account_id` | User | `res.users` | `res_users_id` | no declaring field in the observed registry | `cascade` |
| `im_livechat_channel_country_rel` | Livechat Channel Rules | `im_livechat.channel.rule` | `channel_id` | Country | `res.country` | `country_id` | Livechat Channel Rules (`im_livechat.channel.rule`).`country_ids` | `cascade` |
| `im_livechat_channel_im_user` | User | `res.users` | `user_id` | Livechat Channel | `im_livechat.channel` | `channel_id` | Livechat Channel (`im_livechat.channel`).`user_ids`; User (`res.users`).`livechat_channel_ids` | `cascade` |
| `im_livechat_channel_member_history_discuss_channel_agent_rel` | Discussion Channel | `discuss.channel` | `discuss_channel_id` | Contact | `res.partner` | `res_partner_id` | Discussion Channel (`discuss.channel`).`livechat_agent_partner_ids` | `cascade` |
| `im_livechat_channel_member_history_discuss_channel_bot_rel` | Discussion Channel | `discuss.channel` | `discuss_channel_id` | Contact | `res.partner` | `res_partner_id` | Discussion Channel (`discuss.channel`).`livechat_bot_partner_ids` | `cascade` |
| `im_livechat_channel_member_history_discuss_channel_customer_rel` | Discussion Channel | `discuss.channel` | `discuss_channel_id` | Contact | `res.partner` | `res_partner_id` | Discussion Channel (`discuss.channel`).`livechat_customer_partner_ids` | `cascade` |
| `im_livechat_channel_member_history_im_livechat_expertise_rel` | Keep the channel member history | `im_livechat.channel.member.history` | `im_livechat_channel_member_history_id` | Live Chat Expertise | `im_livechat.expertise` | `im_livechat_expertise_id` | no declaring field in the observed registry | `cascade` |
| `im_livechat_expertise_res_users_settings_rel` | User Settings | `res.users.settings` | `res_users_settings_id` | Live Chat Expertise | `im_livechat.expertise` | `im_livechat_expertise_id` | no declaring field in the observed registry | `cascade` |
| `ir_act_server_group_rel` | Server Actions | `ir.actions.server` | `act_id` | Access Groups | `res.groups` | `gid` | Server Actions (`ir.actions.server`).`group_ids` | `cascade` |
| `ir_act_server_res_partner_rel` | Server Actions | `ir.actions.server` | `ir_act_server_id` | Contact | `res.partner` | `res_partner_id` | no declaring field in the observed registry | `cascade` |
| `ir_act_server_webhook_field_rel` | Server Actions | `ir.actions.server` | `server_id` | Fields | `ir.model.fields` | `field_id` | Server Actions (`ir.actions.server`).`webhook_field_ids` | `cascade` |
| `ir_act_window_group_rel` | Action Window | `ir.actions.act_window` | `act_id` | Access Groups | `res.groups` | `gid` | Action Window (`ir.actions.act_window`).`group_ids` | `cascade` |
| `ir_attachment_pos_config_rel` | Point of Sale Configuration | `pos.config` | `pos_config_id` | Attachment | `ir.attachment` | `ir_attachment_id` | no declaring field in the observed registry | `cascade` |
| `ir_attachment_slide_channel_invite_rel` | Channel Invitation Wizard | `slide.channel.invite` | `slide_channel_invite_id` | Attachment | `ir.attachment` | `ir_attachment_id` | no declaring field in the observed registry | `cascade` |
| `ir_embedded_actions_res_groups_rel` | Embedded Actions | `ir.embedded.actions` | `ir_embedded_actions_id` | Access Groups | `res.groups` | `res_groups_id` | no declaring field in the observed registry | `cascade` |
| `ir_filters_res_users_rel` | Filters | `ir.filters` | `ir_filters_id` | User | `res.users` | `res_users_id` | no declaring field in the observed registry | `cascade` |
| `ir_model_fields_group_rel` | Fields | `ir.model.fields` | `field_id` | Access Groups | `res.groups` | `group_id` | Fields (`ir.model.fields`).`groups` | `cascade` |
| `ir_model_spreadsheet_dashboard_rel` | Spreadsheet Dashboard | `spreadsheet.dashboard` | `spreadsheet_dashboard_id` | Models | `ir.model` | `ir_model_id` | no declaring field in the observed registry | `cascade` |
| `ir_ui_menu_group_rel` | Menu | `ir.ui.menu` | `menu_id` | Access Groups | `res.groups` | `gid` | Menu (`ir.ui.menu`).`group_ids`; Access Groups (`res.groups`).`menu_access` | `cascade` |
| `ir_ui_view_group_rel` | View | `ir.ui.view` | `view_id` | Access Groups | `res.groups` | `group_id` | View (`ir.ui.view`).`group_ids`; Access Groups (`res.groups`).`view_access` | `cascade` |
| `job_favorite_user_rel` | Job Position | `hr.job` | `job_id` | User | `res.users` | `user_id` | Job Position (`hr.job`).`favorite_user_ids` | `cascade` |
| `l10n_ar_afip_reponsibility_type_fiscal_pos_rel` | Fiscal Position | `account.fiscal.position` | `account_fiscal_position_id` | ARCA Responsibility Type | `l10n_ar.afip.responsibility.type` | `l10n_ar_afip_responsibility_type_id` | Fiscal Position (`account.fiscal.position`).`l10n_ar_afip_responsibility_type_ids` | `cascade` |
| `l10n_es_edi_facturae_ac_role_type_res_partner_rel` | Contact | `res.partner` | `res_partner_id` | Administrative Center Role Type | `l10n_es_edi_facturae.ac_role_type` | `l10n_es_edi_facturae_ac_role_type_id` | no declaring field in the observed registry | `cascade` |
| `l10n_id_qris_transaction_pos_order_rel` | Point of Sale Orders | `pos.order` | `pos_order_id` | Record of QRIS transactions | `l10n_id.qris.transaction` | `l10n_id_qris_transaction_id` | no declaring field in the observed registry | `cascade` |
| `l10n_latam_check_account_payment_rel` | Payments | `account.payment` | `payment_id` | Account payment check | `l10n_latam.check` | `check_id` | Payments (`account.payment`).`l10n_latam_move_check_ids`; Account payment check (`l10n_latam.check`).`operation_ids` | `cascade` |
| `l10n_tr_nilvera_delivery_vehicle_rel` | Transfer | `stock.picking` | `stock_picking_id` | GİB Plate numbers | `l10n_tr.nilvera.trailer.plate` | `l10n_tr_nilvera_trailer_plate_id` | Transfer (`stock.picking`).`l10n_tr_nilvera_trailer_plate_ids` | `cascade` |
| `latam_tranfer_check_reltransfer_id` | Checks Mass Transfers | `l10n_latam.payment.mass.transfer` | `check_id` | Account payment check | `l10n_latam.check` | `l10n_latam_check_id` | Checks Mass Transfers (`l10n_latam.payment.mass.transfer`).`check_ids` | `cascade` |
| `livechat_conversation_tag_rel` | Discussion Channel | `discuss.channel` | `discuss_channel_id` | Live Chat Conversation Tags | `im_livechat.conversation.tag` | `im_livechat_conversation_tag_id` | Discussion Channel (`discuss.channel`).`livechat_conversation_tag_ids`; Live Chat Conversation Tags (`im_livechat.conversation.tag`).`conversation_ids` | `cascade` |
| `lot_label_layout_stock_move_line_rel` | Choose the sheet layout to print lot labels | `lot.label.layout` | `lot_label_layout_id` | Product Moves (Stock Move Line) | `stock.move.line` | `stock_move_line_id` | no declaring field in the observed registry | `cascade` |
| `loyalty_card_sale_order_rel` | Sales Order | `sale.order` | `sale_order_id` | Loyalty Coupon | `loyalty.card` | `loyalty_card_id` | no declaring field in the observed registry | `cascade` |
| `loyalty_generate_wizard_res_partner_category_rel` | Generate Coupons | `loyalty.generate.wizard` | `loyalty_generate_wizard_id` | Partner Tags | `res.partner.category` | `res_partner_category_id` | no declaring field in the observed registry | `cascade` |
| `loyalty_generate_wizard_res_partner_rel` | Generate Coupons | `loyalty.generate.wizard` | `loyalty_generate_wizard_id` | Contact | `res.partner` | `res_partner_id` | no declaring field in the observed registry | `cascade` |
| `loyalty_program_pos_config_rel` | Loyalty Program | `loyalty.program` | `loyalty_program_id` | Point of Sale Configuration | `pos.config` | `pos_config_id` | no declaring field in the observed registry | `cascade` |
| `loyalty_program_product_pricelist_rel` | Loyalty Program | `loyalty.program` | `loyalty_program_id` | Pricelist | `product.pricelist` | `product_pricelist_id` | no declaring field in the observed registry | `cascade` |
| `loyalty_reward_product_product_rel` | Loyalty Reward | `loyalty.reward` | `loyalty_reward_id` | Product Variant | `product.product` | `product_product_id` | no declaring field in the observed registry | `cascade` |
| `loyalty_rule_product_product_rel` | Loyalty Rule | `loyalty.rule` | `loyalty_rule_id` | Product Variant | `product.product` | `product_product_id` | no declaring field in the observed registry | `cascade` |
| `loyalty_rule_sale_order_rel` | Sales Order | `sale.order` | `sale_order_id` | Loyalty Rule | `loyalty.rule` | `loyalty_rule_id` | no declaring field in the observed registry | `cascade` |
| `lunch_alert_lunch_location_rel` | Lunch Alert | `lunch.alert` | `lunch_alert_id` | Lunch Locations | `lunch.location` | `lunch_location_id` | no declaring field in the observed registry | `cascade` |
| `lunch_location_lunch_supplier_rel` | Lunch Supplier | `lunch.supplier` | `lunch_supplier_id` | Lunch Locations | `lunch.location` | `lunch_location_id` | no declaring field in the observed registry | `cascade` |
| `lunch_order_topping` | Lunch Order | `lunch.order` | `order_id` | Lunch Extras | `lunch.topping` | `topping_id` | Lunch Order (`lunch.order`).`topping_ids_1`; Lunch Order (`lunch.order`).`topping_ids_2`; Lunch Order (`lunch.order`).`topping_ids_3` | `cascade` |
| `lunch_product_favorite_user_rel` | Lunch Product | `lunch.product` | `product_id` | User | `res.users` | `user_id` | Lunch Product (`lunch.product`).`favorite_user_ids`; User (`res.users`).`favorite_lunch_product_ids` | `cascade` |
| `mail_activity_plan_mail_activity_schedule_rel` | Activity schedule plan Wizard | `mail.activity.schedule` | `mail_activity_schedule_id` | Activity Plan | `mail.activity.plan` | `mail_activity_plan_id` | no declaring field in the observed registry | `cascade` |
| `mail_activity_plan_template_mail_activity_type_rel` | Activity plan template | `mail.activity.plan.template` | `mail_activity_plan_template_id` | Activity Type | `mail.activity.type` | `mail_activity_type_id` | no declaring field in the observed registry | `cascade` |
| `mail_activity_rel` | Activity Type | `mail.activity.type` | `activity_id` | Activity Type | `mail.activity.type` | `recommended_id` | Activity Type (`mail.activity.type`).`previous_type_ids`; Activity Type (`mail.activity.type`).`suggested_next_type_ids` | `cascade` |
| `mail_activity_type_mail_template_rel` | Activity Type | `mail.activity.type` | `mail_activity_type_id` | Email Templates | `mail.template` | `mail_template_id` | no declaring field in the observed registry | `cascade` |
| `mail_canned_response_res_groups_rel` | Canned Response | `mail.canned.response` | `mail_canned_response_id` | Access Groups | `res.groups` | `res_groups_id` | no declaring field in the observed registry | `cascade` |
| `mail_compose_message_ir_attachments_rel` | Email composition wizard | `mail.compose.message` | `wizard_id` | Attachment | `ir.attachment` | `attachment_id` | Email composition wizard (`mail.compose.message`).`attachment_ids` | `cascade` |
| `mail_compose_message_mailing_list_rel` | Email composition wizard | `mail.compose.message` | `mail_compose_message_id` | Mailing List | `mailing.list` | `mailing_list_id` | no declaring field in the observed registry | `cascade` |
| `mail_compose_message_res_partner_rel` | Email composition wizard | `mail.compose.message` | `wizard_id` | Contact | `res.partner` | `partner_id` | Email composition wizard (`mail.compose.message`).`partner_ids` | `cascade` |
| `mail_followers_edit_res_partner_rel` | Followers edit wizard | `mail.followers.edit` | `mail_followers_edit_id` | Contact | `res.partner` | `res_partner_id` | no declaring field in the observed registry | `cascade` |
| `mail_followers_mail_message_subtype_rel` | Document Followers | `mail.followers` | `mail_followers_id` | Message subtypes | `mail.message.subtype` | `mail_message_subtype_id` | no declaring field in the observed registry | `cascade` |
| `mail_group_moderator_rel` | Mail Group | `mail.group` | `mail_group_id` | User | `res.users` | `res_users_id` | Mail Group (`mail.group`).`moderator_ids` | `cascade` |
| `mail_mail_res_partner_rel` | Outgoing Mails | `mail.mail` | `mail_mail_id` | Contact | `res.partner` | `res_partner_id` | no declaring field in the observed registry | `cascade` |
| `mail_mass_mailing_list_rel` | Mailing List | `mailing.list` | `mailing_list_id` | Mass Mailing | `mailing.mailing` | `mailing_mailing_id` | Mailing List (`mailing.list`).`mailing_ids`; Mass Mailing (`mailing.mailing`).`contact_list_ids` | `cascade` |
| `mail_message_res_partner_rel` | Message | `mail.message` | `mail_message_id` | Contact | `res.partner` | `res_partner_id` | no declaring field in the observed registry | `cascade` |
| `mail_message_res_partner_starred_rel` | Message | `mail.message` | `mail_message_id` | Contact | `res.partner` | `res_partner_id` | Message (`mail.message`).`starred_partner_ids` | `cascade` |
| `mail_scheduled_message_res_partner_rel` | Scheduled Message | `mail.scheduled.message` | `mail_scheduled_message_id` | Contact | `res.partner` | `res_partner_id` | no declaring field in the observed registry | `cascade` |
| `mail_template_ir_actions_report_rel` | Email Templates | `mail.template` | `mail_template_id` | Report Action | `ir.actions.report` | `ir_actions_report_id` | Email Templates (`mail.template`).`report_template_ids` | `cascade` |
| `mail_template_mail_template_reset_rel` | Mail Template Reset | `mail.template.reset` | `mail_template_reset_id` | Email Templates | `mail.template` | `mail_template_id` | no declaring field in the observed registry | `cascade` |
| `mailing_contact_import_mailing_list_rel` | Mailing Contact Import | `mailing.contact.import` | `mailing_contact_import_id` | Mailing List | `mailing.list` | `mailing_list_id` | no declaring field in the observed registry | `cascade` |
| `mailing_contact_mailing_contact_to_list_rel` | Add Contacts to Mailing List | `mailing.contact.to.list` | `mailing_contact_to_list_id` | Mailing Contact | `mailing.contact` | `mailing_contact_id` | no declaring field in the observed registry | `cascade` |
| `mailing_contact_res_partner_category_rel` | Mailing Contact | `mailing.contact` | `mailing_contact_id` | Partner Tags | `res.partner.category` | `res_partner_category_id` | no declaring field in the observed registry | `cascade` |
| `mailing_list_mailing_list_merge_rel` | Merge Mass Mailing List | `mailing.list.merge` | `mailing_list_merge_id` | Mailing List | `mailing.list` | `mailing_list_id` | no declaring field in the observed registry | `cascade` |
| `maintenance_team_users_rel` | Maintenance Teams | `maintenance.team` | `maintenance_team_id` | User | `res.users` | `res_users_id` | Maintenance Teams (`maintenance.team`).`member_ids` | `cascade` |
| `mass_mailing_ir_attachments_rel` | Mass Mailing | `mailing.mailing` | `mass_mailing_id` | Attachment | `ir.attachment` | `attachment_id` | Mass Mailing (`mailing.mailing`).`attachment_ids` | `cascade` |
| `meeting_category_rel` | Calendar Event | `calendar.event` | `event_id` | Event Meeting Type | `calendar.event.type` | `type_id` | Calendar Event (`calendar.event`).`categ_ids` | `cascade` |
| `merge_opportunity_rel` | Merge Opportunities | `crm.merge.opportunity` | `merge_id` | Lead | `crm.lead` | `opportunity_id` | Merge Opportunities (`crm.merge.opportunity`).`opportunity_ids` | `cascade` |
| `message_attachment_rel` | Message | `mail.message` | `message_id` | Attachment | `ir.attachment` | `attachment_id` | Message (`mail.message`).`attachment_ids` | `cascade` |
| `module_country` | Module | `ir.module.module` | `module_id` | Country | `res.country` | `country_id` | Module (`ir.module.module`).`country_ids` | `cascade` |
| `mrp_account_wip_accounting_mrp_production_rel` | Wizard to post Manufacturing work in progress account move | `mrp.account.wip.accounting` | `mrp_account_wip_accounting_id` | Manufacturing Order | `mrp.production` | `mrp_production_id` | no declaring field in the observed registry | `cascade` |
| `mrp_bom_byproduct_product_template_attribute_value_rel` | Byproduct | `mrp.bom.byproduct` | `mrp_bom_byproduct_id` | Product Template Attribute Value | `product.template.attribute.value` | `product_template_attribute_value_id` | no declaring field in the observed registry | `restrict` |
| `mrp_bom_line_product_template_attribute_value_rel` | Bill of Material Line | `mrp.bom.line` | `mrp_bom_line_id` | Product Template Attribute Value | `product.template.attribute.value` | `product_template_attribute_value_id` | no declaring field in the observed registry | `restrict` |
| `mrp_bom_stock_replenishment_info_rel` | Stock supplier replenishment information | `stock.replenishment.info` | `stock_replenishment_info_id` | Bill of Material | `mrp.bom` | `mrp_bom_id` | no declaring field in the observed registry | `cascade` |
| `mrp_bom_subcontractor` | Bill of Material | `mrp.bom` | `mrp_bom_id` | Contact | `res.partner` | `res_partner_id` | Bill of Material (`mrp.bom`).`subcontractor_ids` | `cascade` |
| `mrp_consumption_warning_mrp_production_rel` | Wizard in case of consumption in warning/strict and more component has been used for a manufacturing order (related to the bill of materials) | `mrp.consumption.warning` | `mrp_consumption_warning_id` | Manufacturing Order | `mrp.production` | `mrp_production_id` | no declaring field in the observed registry | `cascade` |
| `mrp_production_group_rel` | Production Group | `mrp.production.group` | `parent_group_id` | Production Group | `mrp.production.group` | `child_group_id` | Production Group (`mrp.production.group`).`child_ids`; Production Group (`mrp.production.group`).`parent_ids` | `cascade` |
| `mrp_production_mrp_production_backorder_rel` | Wizard to mark as done or create back order | `mrp.production.backorder` | `mrp_production_backorder_id` | Manufacturing Order | `mrp.production` | `mrp_production_id` | no declaring field in the observed registry | `cascade` |
| `mrp_production_picking_label_type_rel` | Choose whether to print product or lot/sn labels | `picking.label.type` | `picking_label_type_id` | Manufacturing Order | `mrp.production` | `mrp_production_id` | no declaring field in the observed registry | `cascade` |
| `mrp_production_stock_landed_cost_rel` | Stock Landed Cost | `stock.landed.cost` | `stock_landed_cost_id` | Manufacturing Order | `mrp.production` | `mrp_production_id` | no declaring field in the observed registry | `cascade` |
| `mrp_production_stock_lot_rel` | Manufacturing Order | `mrp.production` | `mrp_production_id` | Lot/Serial | `stock.lot` | `stock_lot_id` | no declaring field in the observed registry | `cascade` |
| `mrp_routing_workcenter_dependencies_rel` | Work Center Usage | `mrp.routing.workcenter` | `operation_id` | Work Center Usage | `mrp.routing.workcenter` | `blocked_by_id` | Work Center Usage (`mrp.routing.workcenter`).`blocked_by_operation_ids`; Work Center Usage (`mrp.routing.workcenter`).`needed_by_operation_ids` | `cascade` |
| `mrp_routing_workcenter_product_template_attribute_value_rel` | Work Center Usage | `mrp.routing.workcenter` | `mrp_routing_workcenter_id` | Product Template Attribute Value | `product.template.attribute.value` | `product_template_attribute_value_id` | no declaring field in the observed registry | `restrict` |
| `mrp_workcenter_alternative_rel` | Work Center | `mrp.workcenter` | `workcenter_id` | Work Center | `mrp.workcenter` | `alternative_workcenter_id` | Work Center (`mrp.workcenter`).`alternative_workcenter_ids` | `cascade` |
| `mrp_workcenter_mrp_workcenter_tag_rel` | Work Center | `mrp.workcenter` | `mrp_workcenter_id` | Add tag for the workcenter | `mrp.workcenter.tag` | `mrp_workcenter_tag_id` | no declaring field in the observed registry | `cascade` |
| `mrp_workorder_dependencies_rel` | Work Order | `mrp.workorder` | `workorder_id` | Work Order | `mrp.workorder` | `blocked_by_id` | Work Order (`mrp.workorder`).`blocked_by_workorder_ids`; Work Order (`mrp.workorder`).`needed_by_workorder_ids` | `cascade` |
| `mrp_workorder_mo_analytic_rel` | Work Order | `mrp.workorder` | `mrp_workorder_id` | Analytic Line | `account.analytic.line` | `account_analytic_line_id` | Work Order (`mrp.workorder`).`mo_analytic_account_line_ids` | `cascade` |
| `mrp_workorder_wc_analytic_rel` | Work Order | `mrp.workorder` | `mrp_workorder_id` | Analytic Line | `account.analytic.line` | `account_analytic_line_id` | Work Order (`mrp.workorder`).`wc_analytic_account_line_ids` | `cascade` |
| `myinvois_document_invoice_rel` | Journal Entry | `account.move` | `invoice_id` | MyInvois Document | `myinvois.document` | `document_id` | Journal Entry (`account.move`).`MyInvois Documents`; MyInvois Document (`myinvois.document`).`Invoices` | `cascade` |
| `myinvois_document_pos_order_rel` | MyInvois Document | `myinvois.document` | `document_id` | Point of Sale Orders | `pos.order` | `order_id` | MyInvois Document (`myinvois.document`).`Orders`; Point of Sale Orders (`pos.order`).`Consolidated Invoices` | `cascade` |
| `onboarding_onboarding_onboarding_onboarding_step_rel` | Onboarding | `onboarding.onboarding` | `onboarding_onboarding_id` | Onboarding Step | `onboarding.onboarding.step` | `onboarding_onboarding_step_id` | no declaring field in the observed registry | `cascade` |
| `onboarding_progress_onboarding_progress_step_rel` | Onboarding Progress Tracker | `onboarding.progress` | `onboarding_progress_id` | Onboarding Progress Step Tracker | `onboarding.progress.step` | `onboarding_progress_step_id` | no declaring field in the observed registry | `cascade` |
| `payment_capture_wizard_payment_transaction_rel` | Payment Capture Wizard | `payment.capture.wizard` | `payment_capture_wizard_id` | Payment Transaction | `payment.transaction` | `payment_transaction_id` | no declaring field in the observed registry | `cascade` |
| `payment_country_rel` | Payment Provider | `payment.provider` | `payment_id` | Country | `res.country` | `country_id` | Payment Provider (`payment.provider`).`available_country_ids` | `cascade` |
| `payment_currency_rel` | Payment Provider | `payment.provider` | `payment_provider_id` | Currency | `res.currency` | `currency_id` | Payment Provider (`payment.provider`).`available_currency_ids` | `cascade` |
| `payment_method_payment_provider_rel` | Payment Method | `payment.method` | `payment_method_id` | Payment Provider | `payment.provider` | `payment_provider_id` | no declaring field in the observed registry | `cascade` |
| `payment_method_res_country_rel` | Payment Method | `payment.method` | `payment_method_id` | Country | `res.country` | `res_country_id` | no declaring field in the observed registry | `cascade` |
| `payment_method_res_currency_rel` | Payment Method | `payment.method` | `payment_method_id` | Currency | `res.currency` | `res_currency_id` | no declaring field in the observed registry | `cascade` |
| `payment_provider_pos_payment_method_rel` | Point of Sale Payment Methods | `pos.payment.method` | `pos_payment_method_id` | Payment Provider | `payment.provider` | `payment_provider_id` | no declaring field in the observed registry | `cascade` |
| `picking_label_type_stock_picking_rel` | Choose whether to print product or lot/sn labels | `picking.label.type` | `picking_label_type_id` | Transfer | `stock.picking` | `stock_picking_id` | no declaring field in the observed registry | `cascade` |
| `picking_type_favorite_user_rel` | Picking Type | `stock.picking.type` | `picking_type_id` | User | `res.users` | `user_id` | Picking Type (`stock.picking.type`).`favorite_user_ids` | `cascade` |
| `portal_share_res_partner_rel` | Portal Sharing | `portal.share` | `portal_share_id` | Contact | `res.partner` | `res_partner_id` | no declaring field in the observed registry | `cascade` |
| `portal_wizard_res_partner_rel` | Grant Portal Access | `portal.wizard` | `portal_wizard_id` | Contact | `res.partner` | `res_partner_id` | no declaring field in the observed registry | `cascade` |
| `pos_bill_pos_config_rel` | Point of Sale Configuration | `pos.config` | `pos_config_id` | Coins/Bills | `pos.bill` | `pos_bill_id` | no declaring field in the observed registry | `cascade` |
| `pos_category_pos_config_rel` | Point of Sale Configuration | `pos.config` | `pos_config_id` | Point of Sale Category | `pos.category` | `pos_category_id` | no declaring field in the observed registry | `cascade` |
| `pos_category_product_template_rel` | Product | `product.template` | `product_template_id` | Point of Sale Category | `pos.category` | `pos_category_id` | no declaring field in the observed registry | `cascade` |
| `pos_category_res_config_settings_rel` | Config Settings | `res.config.settings` | `res_config_settings_id` | Point of Sale Category | `pos.category` | `pos_category_id` | no declaring field in the observed registry | `cascade` |
| `pos_config_pos_note_rel` | Point of Sale Configuration | `pos.config` | `pos_config_id` | Point of Sale Note | `pos.note` | `pos_note_id` | no declaring field in the observed registry | `cascade` |
| `pos_config_pos_payment_method_rel` | Point of Sale Configuration | `pos.config` | `pos_config_id` | Point of Sale Payment Methods | `pos.payment.method` | `pos_payment_method_id` | no declaring field in the observed registry | `cascade` |
| `pos_config_pos_preset_rel` | Point of Sale Configuration | `pos.config` | `pos_config_id` | Easily load a set of configuration options | `pos.preset` | `pos_preset_id` | no declaring field in the observed registry | `cascade` |
| `pos_config_pos_self_order_custom_link_rel` | Custom links that the restaurant can configure to be displayed on the self order screen | `pos_self_order.custom_link` | `pos_self_order_custom_link_id` | Point of Sale Configuration | `pos.config` | `pos_config_id` | no declaring field in the observed registry | `cascade` |
| `pos_config_printer_rel` | Point of Sale Printer | `pos.printer` | `printer_id` | Point of Sale Configuration | `pos.config` | `config_id` | Point of Sale Configuration (`pos.config`).`printer_ids`; Point of Sale Printer (`pos.printer`).`pos_config_ids` | `cascade` |
| `pos_config_product_pricelist_rel` | Point of Sale Configuration | `pos.config` | `pos_config_id` | Pricelist | `product.pricelist` | `product_pricelist_id` | no declaring field in the observed registry | `cascade` |
| `pos_config_res_lang_rel` | Point of Sale Configuration | `pos.config` | `pos_config_id` | Languages | `res.lang` | `res_lang_id` | no declaring field in the observed registry | `cascade` |
| `pos_config_restaurant_floor_rel` | Point of Sale Configuration | `pos.config` | `pos_config_id` | Restaurant Floor | `restaurant.floor` | `restaurant_floor_id` | no declaring field in the observed registry | `cascade` |
| `pos_config_trust_relation` | Point of Sale Configuration | `pos.config` | `is_trusting` | Point of Sale Configuration | `pos.config` | `is_trusted` | Point of Sale Configuration (`pos.config`).`trusted_config_ids` | `cascade` |
| `pos_detail_configs` | Point of Sale Details Report | `pos.details.wizard` | `pos_details_wizard_id` | Point of Sale Configuration | `pos.config` | `pos_config_id` | Point of Sale Details Report (`pos.details.wizard`).`pos_config_ids` | `cascade` |
| `pos_hr_advanced_employee_hr_employee` | Point of Sale Configuration | `pos.config` | `pos_config_id` | Employee | `hr.employee` | `hr_employee_id` | Point of Sale Configuration (`pos.config`).`advanced_employee_ids` | `cascade` |
| `pos_hr_basic_employee_hr_employee` | Point of Sale Configuration | `pos.config` | `pos_config_id` | Employee | `hr.employee` | `hr_employee_id` | Point of Sale Configuration (`pos.config`).`basic_employee_ids` | `cascade` |
| `pos_hr_minimal_employee_hr_employee` | Point of Sale Configuration | `pos.config` | `pos_config_id` | Employee | `hr.employee` | `hr_employee_id` | Point of Sale Configuration (`pos.config`).`minimal_employee_ids` | `cascade` |
| `pos_order_line_product_template_attribute_value_rel` | Point of Sale Order Lines | `pos.order.line` | `pos_order_line_id` | Product Template Attribute Value | `product.template.attribute.value` | `product_template_attribute_value_id` | no declaring field in the observed registry | `cascade` |
| `pos_payment_method_config_fast_validation_relation` | Point of Sale Configuration | `pos.config` | `pos_config_id` | Point of Sale Payment Methods | `pos.payment.method` | `pos_payment_method_id` | Point of Sale Configuration (`pos.config`).`fast_payment_method_ids` | `cascade` |
| `pos_product_optional_rel` | Product | `product.template` | `src_id` | Product | `product.template` | `dest_id` | Product (`product.template`).`pos_optional_product_ids` | `cascade` |
| `pos_self_order_background_rels` | Point of Sale Configuration | `pos.config` | `pos_config_id` | Attachment | `ir.attachment` | `ir_attachment_id` | Point of Sale Configuration (`pos.config`).`self_ordering_image_background_ids` | `cascade` |
| `printer_category_rel` | Point of Sale Printer | `pos.printer` | `printer_id` | Point of Sale Category | `pos.category` | `category_id` | Point of Sale Printer (`pos.printer`).`product_categories_ids` | `cascade` |
| `product_accessory_rel` | Product | `product.template` | `src_id` | Product Variant | `product.product` | `dest_id` | Product (`product.template`).`accessory_product_ids` | `cascade` |
| `product_alternative_rel` | Product | `product.template` | `src_id` | Product | `product.template` | `dest_id` | Product (`product.template`).`alternative_product_ids` | `cascade` |
| `product_attr_exclusion_value_ids_rel` | Product Template Attribute Exclusion | `product.template.attribute.exclusion` | `product_template_attribute_exclusion_id` | Product Template Attribute Value | `product.template.attribute.value` | `product_template_attribute_value_id` | Product Template Attribute Exclusion (`product.template.attribute.exclusion`).`value_ids` | `cascade` |
| `product_attribute_product_template_rel` | Product Attribute | `product.attribute` | `product_attribute_id` | Product | `product.template` | `product_template_id` | no declaring field in the observed registry | `cascade` |
| `product_attribute_value_product_template_attribute_line_rel` | Attribute Value | `product.attribute.value` | `product_attribute_value_id` | Product Template Attribute Line | `product.template.attribute.line` | `product_template_attribute_line_id` | Attribute Value (`product.attribute.value`).`pav_attribute_line_ids`; Product Template Attribute Line (`product.template.attribute.line`).`value_ids` | `cascade` |
| `product_category_stock_picking_type_rel` | Picking Type | `stock.picking.type` | `stock_picking_type_id` | Product Category | `product.category` | `product_category_id` | no declaring field in the observed registry | `cascade` |
| `product_combo_product_template_rel` | Product | `product.template` | `product_template_id` | Product Combo | `product.combo` | `product_combo_id` | no declaring field in the observed registry | `cascade` |
| `product_document_sale_pdf_form_field_rel` | Product Document | `product.document` | `product_document_id` | Form fields of inside quotation documents. | `sale.pdf.form.field` | `sale_pdf_form_field_id` | no declaring field in the observed registry | `cascade` |
| `product_feed_product_public_category_rel` | Product Feed | `product.feed` | `product_feed_id` | Website Product Category | `product.public.category` | `product_public_category_id` | no declaring field in the observed registry | `cascade` |
| `product_label_layout_product_product_rel` | Choose the sheet layout to print the labels | `product.label.layout` | `product_label_layout_id` | Product Variant | `product.product` | `product_product_id` | no declaring field in the observed registry | `cascade` |
| `product_label_layout_product_template_rel` | Choose the sheet layout to print the labels | `product.label.layout` | `product_label_layout_id` | Product | `product.template` | `product_template_id` | no declaring field in the observed registry | `cascade` |
| `product_label_layout_stock_move_rel` | Choose the sheet layout to print the labels | `product.label.layout` | `product_label_layout_id` | Stock Move | `stock.move` | `stock_move_id` | no declaring field in the observed registry | `cascade` |
| `product_optional_rel` | Product | `product.template` | `src_id` | Product | `product.template` | `dest_id` | Product (`product.template`).`optional_product_ids` | `cascade` |
| `product_pricelist_res_config_settings_rel` | Config Settings | `res.config.settings` | `res_config_settings_id` | Pricelist | `product.pricelist` | `product_pricelist_id` | no declaring field in the observed registry | `cascade` |
| `product_public_category_product_template_rel` | Website Product Category | `product.public.category` | `product_public_category_id` | Product | `product.template` | `product_template_id` | Website Product Category (`product.public.category`).`product_tmpl_ids`; Product (`product.template`).`public_categ_ids` | `cascade` |
| `product_supplier_taxes_rel` | Product | `product.template` | `prod_id` | Tax | `account.tax` | `tax_id` | Product (`product.template`).`supplier_taxes_id` | `cascade` |
| `product_supplierinfo_stock_replenishment_info_rel` | Stock supplier replenishment information | `stock.replenishment.info` | `stock_replenishment_info_id` | Supplier Pricelist | `product.supplierinfo` | `product_supplierinfo_id` | no declaring field in the observed registry | `cascade` |
| `product_tag_delivery_carrier_excluded_rel` | Shipping Methods | `delivery.carrier` | `delivery_carrier_id` | Product Tag | `product.tag` | `product_tag_id` | Shipping Methods (`delivery.carrier`).`excluded_tag_ids` | `cascade` |
| `product_tag_delivery_carrier_must_have_rel` | Shipping Methods | `delivery.carrier` | `delivery_carrier_id` | Product Tag | `product.tag` | `product_tag_id` | Shipping Methods (`delivery.carrier`).`must_have_tag_ids` | `cascade` |
| `product_tag_product_product_rel` | Product Variant | `product.product` | `product_product_id` | Product Tag | `product.tag` | `product_tag_id` | Product Variant (`product.product`).`additional_product_tag_ids`; Product Tag (`product.tag`).`product_product_ids` | `cascade` |
| `product_tag_product_template_rel` | Product | `product.template` | `product_template_id` | Product Tag | `product.tag` | `product_tag_id` | Product Tag (`product.tag`).`product_template_ids`; Product (`product.template`).`product_tag_ids` | `cascade` |
| `product_taxes_rel` | Product | `product.template` | `prod_id` | Tax | `account.tax` | `tax_id` | Product (`product.template`).`taxes_id` | `cascade` |
| `product_template_attribute_value_purchase_order_line_rel` | Purchase Order Line | `purchase.order.line` | `purchase_order_line_id` | Product Template Attribute Value | `product.template.attribute.value` | `product_template_attribute_value_id` | no declaring field in the observed registry | `restrict` |
| `product_template_attribute_value_sale_order_line_rel` | Sales Order Line | `sale.order.line` | `sale_order_line_id` | Product Template Attribute Value | `product.template.attribute.value` | `product_template_attribute_value_id` | no declaring field in the observed registry | `restrict` |
| `product_template_uom_uom_rel` | Product | `product.template` | `product_template_id` | Product Unit of Measure | `uom.uom` | `uom_uom_id` | no declaring field in the observed registry | `cascade` |
| `product_variant_combination` | Product Variant | `product.product` | `product_product_id` | Product Template Attribute Value | `product.template.attribute.value` | `product_template_attribute_value_id` | Product Variant (`product.product`).`product_template_attribute_value_ids`; Product Variant (`product.product`).`product_template_variant_value_ids`; Product Template Attribute Value (`product.template.attribute.value`).`ptav_product_variant_ids` | `restrict` |
| `project_favorite_user_rel` | Project | `project.project` | `project_id` | User | `res.users` | `user_id` | Project (`project.project`).`favorite_user_ids`; User (`res.users`).`favorite_project_ids` | `cascade` |
| `project_project_project_tags_rel` | Project | `project.project` | `project_project_id` | Project Tags | `project.tags` | `project_tags_id` | Project (`project.project`).`tag_ids`; Project Tags (`project.tags`).`project_ids` | `cascade` |
| `project_project_project_task_type_delete_wizard_rel` | Project Task Stage Delete Wizard | `project.task.type.delete.wizard` | `project_task_type_delete_wizard_id` | Project | `project.project` | `project_project_id` | no declaring field in the observed registry | `cascade` |
| `project_project_stage_project_project_stage_delete_wizard_rel` | Project Stage Delete Wizard | `project.project.stage.delete.wizard` | `project_project_stage_delete_wizard_id` | Project Stage | `project.project.stage` | `project_project_stage_id` | no declaring field in the observed registry | `cascade` |
| `project_role_project_task_rel` | Task | `project.task` | `project_task_id` | Project Role | `project.role` | `project_role_id` | no declaring field in the observed registry | `cascade` |
| `project_share_wizard_res_partner_rel` | Project Sharing | `project.share.wizard` | `project_share_wizard_id` | Contact | `res.partner` | `res_partner_id` | no declaring field in the observed registry | `cascade` |
| `project_tags_project_task_rel` | Task | `project.task` | `project_task_id` | Project Tags | `project.tags` | `project_tags_id` | Burndown Chart (`project.task.burndown.chart.report`).`tag_ids`; Tasks Analysis (`report.project.task.user`).`tag_ids` | `cascade` |
| `project_task_type_project_task_type_delete_wizard_rel` | Project Task Stage Delete Wizard | `project.task.type.delete.wizard` | `project_task_type_delete_wizard_id` | Task Stage | `project.task.type` | `project_task_type_id` | no declaring field in the observed registry | `cascade` |
| `project_task_type_rel` | Project | `project.project` | `project_id` | Task Stage | `project.task.type` | `type_id` | Project (`project.project`).`type_ids`; Task Stage (`project.task.type`).`project_ids` | `cascade` |
| `project_template_role_to_users_map_res_users_rel` | Project role to users mapping | `project.template.role.to.users.map` | `project_template_role_to_users_map_id` | User | `res.users` | `res_users_id` | no declaring field in the observed registry | `cascade` |
| `purchase_order_stock_picking_rel` | Purchase Order | `purchase.order` | `purchase_order_id` | Transfer | `stock.picking` | `stock_picking_id` | no declaring field in the observed registry | `cascade` |
| `purchase_requisition_create_alternative_res_partner_rel` | Wizard to preset values for alternative purchase order | `purchase.requisition.create.alternative` | `purchase_requisition_create_alternative_id` | Contact | `res.partner` | `res_partner_id` | no declaring field in the observed registry | `cascade` |
| `quotation_document_sale_order_rel` | Sales Order | `sale.order` | `sale_order_id` | Quotation's Headers & Footers | `quotation.document` | `quotation_document_id` | no declaring field in the observed registry | `cascade` |
| `quotation_document_sale_pdf_form_field_rel` | Quotation's Headers & Footers | `quotation.document` | `quotation_document_id` | Form fields of inside quotation documents. | `sale.pdf.form.field` | `sale_pdf_form_field_id` | no declaring field in the observed registry | `cascade` |
| `refunded_invoices` | Journal Entry | `account.move` | `refund_account_move` | Journal Entry | `account.move` | `original_account_move` | Journal Entry (`account.move`).`pos_refunded_invoice_ids` | `cascade` |
| `rel_badge_auth_users` | Gamification Badge | `gamification.badge` | `gamification_badge_id` | User | `res.users` | `res_users_id` | Gamification Badge (`gamification.badge`).`rule_auth_user_ids` | `cascade` |
| `rel_modules_langexport` | Language Export | `base.language.export` | `wiz_id` | Module | `ir.module.module` | `module_id` | Language Export (`base.language.export`).`modules` | `cascade` |
| `rel_slide_tag` | Slides | `slide.slide` | `slide_id` | Slide Tag | `slide.tag` | `tag_id` | Slides (`slide.slide`).`tag_ids` | `cascade` |
| `rel_upload_groups` | Course | `slide.channel` | `channel_id` | Access Groups | `res.groups` | `group_id` | Course (`slide.channel`).`upload_group_ids` | `cascade` |
| `repair_order_repair_tags_rel` | Repair Order | `repair.order` | `repair_order_id` | Repair Tags | `repair.tags` | `repair_tags_id` | no declaring field in the observed registry | `cascade` |
| `res_company_spreadsheet_dashboard_rel` | Spreadsheet Dashboard | `spreadsheet.dashboard` | `spreadsheet_dashboard_id` | Companies | `res.company` | `res_company_id` | no declaring field in the observed registry | `cascade` |
| `res_company_users_rel` | Companies | `res.company` | `cid` | User | `res.users` | `user_id` | Companies (`res.company`).`user_ids`; User (`res.users`).`company_ids` | `cascade` |
| `res_country_group_pricelist_rel` | Pricelist | `product.pricelist` | `pricelist_id` | Country Group | `res.country.group` | `res_country_group_id` | Pricelist (`product.pricelist`).`country_group_ids`; Country Group (`res.country.group`).`pricelist_ids` | `cascade` |
| `res_country_group_res_country_state_rel` | Country Group | `res.country.group` | `res_country_group_id` | Country state | `res.country.state` | `res_country_state_id` | no declaring field in the observed registry | `cascade` |
| `res_country_res_country_group_rel` | Country | `res.country` | `res_country_id` | Country Group | `res.country.group` | `res_country_group_id` | Country (`res.country`).`country_group_ids`; Country Group (`res.country.group`).`country_ids` | `cascade` |
| `res_groups_implied_rel` | Access Groups | `res.groups` | `gid` | Access Groups | `res.groups` | `hid` | Access Groups (`res.groups`).`implied_by_ids`; Access Groups (`res.groups`).`implied_ids` | `cascade` |
| `res_groups_report_rel` | Report Action | `ir.actions.report` | `uid` | Access Groups | `res.groups` | `gid` | Report Action (`ir.actions.report`).`group_ids` | `cascade` |
| `res_groups_slide_channel_rel` | Course | `slide.channel` | `slide_channel_id` | Access Groups | `res.groups` | `res_groups_id` | no declaring field in the observed registry | `cascade` |
| `res_groups_spreadsheet_dashboard_rel` | Spreadsheet Dashboard | `spreadsheet.dashboard` | `spreadsheet_dashboard_id` | Access Groups | `res.groups` | `res_groups_id` | no declaring field in the observed registry | `cascade` |
| `res_groups_users_rel` | Access Groups | `res.groups` | `gid` | User | `res.users` | `uid` | Access Groups (`res.groups`).`user_ids`; User (`res.users`).`group_ids` | `cascade` |
| `res_groups_website_menu_rel` | Website Menu | `website.menu` | `website_menu_id` | Access Groups | `res.groups` | `res_groups_id` | no declaring field in the observed registry | `cascade` |
| `res_lang_install_rel` | Install Language | `base.language.install` | `language_wizard_id` | Languages | `res.lang` | `lang_id` | Install Language (`base.language.install`).`lang_ids` | `cascade` |
| `res_lang_res_users_settings_rel` | User Settings | `res.users.settings` | `res_users_settings_id` | Languages | `res.lang` | `res_lang_id` | no declaring field in the observed registry | `cascade` |
| `res_lang_survey_survey_rel` | Survey | `survey.survey` | `survey_survey_id` | Languages | `res.lang` | `res_lang_id` | no declaring field in the observed registry | `cascade` |
| `res_partner_res_partner_category_rel` | Partner Tags | `res.partner.category` | `category_id` | Contact | `res.partner` | `partner_id` | no declaring field in the observed registry | `cascade` |
| `res_partner_res_partner_tag_rel` | Contact | `res.partner` | `partner_id` | Partner Tags - These tags can be used on website to find customers by sector, or ... | `res.partner.tag` | `tag_id` | Contact (`res.partner`).`website_tag_ids`; Partner Tags - These tags can be used on website to find customers by sector, or ... (`res.partner.tag`).`partner_ids` | `cascade` |
| `res_partner_slide_channel_invite_rel` | Channel Invitation Wizard | `slide.channel.invite` | `slide_channel_invite_id` | Contact | `res.partner` | `res_partner_id` | no declaring field in the observed registry | `cascade` |
| `res_partner_stock_picking_rel` | Transfer | `stock.picking` | `stock_picking_id` | Contact | `res.partner` | `res_partner_id` | no declaring field in the observed registry | `cascade` |
| `res_partner_task_share_wizard_rel` | Task Sharing | `task.share.wizard` | `task_share_wizard_id` | Contact | `res.partner` | `res_partner_id` | no declaring field in the observed registry | `cascade` |
| `res_role_res_users_rel` | Represents a role in the system used to categorize users. Each role has a unique name and can be associated with multiple users. Roles can be mentioned in messages to notify all associated users. | `res.role` | `res_role_id` | User | `res.users` | `res_users_id` | Represents a role in the system used to categorize users. Each role has a unique name and can be associated with multiple users. Roles can be mentioned in messages to notify all associated users. (`res.role`).`user_ids`; User (`res.users`).`role_ids` | `cascade` |
| `res_users_spreadsheet_dashboard_rel` | Spreadsheet Dashboard | `spreadsheet.dashboard` | `spreadsheet_dashboard_id` | User | `res.users` | `res_users_id` | no declaring field in the observed registry | `cascade` |
| `res_users_survey_survey_rel` | Survey | `survey.survey` | `survey_survey_id` | User | `res.users` | `res_users_id` | no declaring field in the observed registry | `cascade` |
| `res_users_web_tour_tour_rel` | Tours | `web_tour.tour` | `web_tour_tour_id` | User | `res.users` | `res_users_id` | no declaring field in the observed registry | `cascade` |
| `rule_group_rel` | Record Rule | `ir.rule` | `rule_group_id` | Access Groups | `res.groups` | `group_id` | Record Rule (`ir.rule`).`groups`; Access Groups (`res.groups`).`rule_groups` | `cascade` |
| `sale_advance_payment_inv_sale_order_rel` | Sales Advance Payment Invoice | `sale.advance.payment.inv` | `sale_advance_payment_inv_id` | Sales Order | `sale.order` | `sale_order_id` | no declaring field in the observed registry | `cascade` |
| `sale_order_disabled_auto_rewards_rel` | Sales Order | `sale.order` | `sale_order_id` | Loyalty Reward | `loyalty.reward` | `loyalty_reward_id` | Sales Order (`sale.order`).`disabled_auto_rewards` | `cascade` |
| `sale_order_line_invoice_rel` | Journal Item | `account.move.line` | `invoice_line_id` | Sales Order Line | `sale.order.line` | `order_line_id` | Journal Item (`account.move.line`).`sale_line_ids`; Sales Order Line (`sale.order.line`).`invoice_lines` | `cascade` |
| `sale_order_line_product_document_rel` | Sales Order Line | `sale.order.line` | `sale_order_line_id` | Product Document | `product.document` | `product_document_id` | Sales Order Line (`sale.order.line`).`product_document_ids` | `cascade` |
| `sale_order_line_stock_route_rel` | Sales Order Line | `sale.order.line` | `sale_order_line_id` | Inventory Routes | `stock.route` | `stock_route_id` | no declaring field in the observed registry | `restrict` |
| `sale_order_mass_cancel_wizard_rel` | Cancel multiple quotations | `sale.mass.cancel.orders` | `sale_mass_cancel_orders_id` | Sales Order | `sale.order` | `sale_order_id` | Cancel multiple quotations (`sale.mass.cancel.orders`).`sale_order_ids` | `cascade` |
| `sale_order_tag_rel` | Sales Order | `sale.order` | `order_id` | customer relationship management Tag | `crm.tag` | `tag_id` | Sales Order (`sale.order`).`tag_ids` | `cascade` |
| `sale_order_transaction_rel` | Payment Transaction | `payment.transaction` | `transaction_id` | Sales Order | `sale.order` | `sale_order_id` | Payment Transaction (`payment.transaction`).`sale_order_ids`; Sales Order (`sale.order`).`transaction_ids` | `cascade` |
| `scheduled_message_attachment_rel` | Scheduled Message | `mail.scheduled.message` | `scheduled_message_id` | Attachment | `ir.attachment` | `attachment_id` | Scheduled Message (`mail.scheduled.message`).`attachment_ids` | `cascade` |
| `sent_account_move__pdp_flow` | French Approved Dematerialization Platform Flow | `l10n.fr.pdp.reports.flow` | `flow_id` | Journal Entry | `account.move` | `move_id` | Journal Entry (`account.move`).`l10n_fr_pdp_sent_in_flow_ids`; French Approved Dematerialization Platform Flow (`l10n.fr.pdp.reports.flow`).`sent_move_ids` | `cascade` |
| `slide_channel_prerequisite_slide_channel_rel` | Course | `slide.channel` | `channel_id` | Course | `slide.channel` | `prerequisite_channel_id` | Course (`slide.channel`).`prerequisite_channel_ids`; Course (`slide.channel`).`prerequisite_of_channel_ids` | `cascade` |
| `slide_channel_tag_rel` | Course | `slide.channel` | `channel_id` | Channel/Course Tag | `slide.channel.tag` | `tag_id` | Course (`slide.channel`).`tag_ids`; Channel/Course Tag (`slide.channel.tag`).`channel_ids` | `cascade` |
| `sms_template_sms_template_reset_rel` | text message Template Reset | `sms.template.reset` | `sms_template_reset_id` | text message Templates | `sms.template` | `sms_template_id` | no declaring field in the observed registry | `cascade` |
| `stock_add_to_wave_stock_move_line_rel` | Wave Transfer Lines | `stock.add.to.wave` | `stock_add_to_wave_id` | Product Moves (Stock Move Line) | `stock.move.line` | `stock_move_line_id` | no declaring field in the observed registry | `cascade` |
| `stock_add_to_wave_stock_picking_rel` | Wave Transfer Lines | `stock.add.to.wave` | `stock_add_to_wave_id` | Transfer | `stock.picking` | `stock_picking_id` | no declaring field in the observed registry | `cascade` |
| `stock_conflict_quant_rel` | Conflict in Inventory | `stock.inventory.conflict` | `stock_inventory_conflict_id` | Quants | `stock.quant` | `stock_quant_id` | Conflict in Inventory (`stock.inventory.conflict`).`quant_ids` | `cascade` |
| `stock_inventory_adjustment_name_stock_quant_rel` | Inventory Adjustment Reference / Reason | `stock.inventory.adjustment.name` | `stock_inventory_adjustment_name_id` | Quants | `stock.quant` | `stock_quant_id` | no declaring field in the observed registry | `cascade` |
| `stock_inventory_conflict_stock_quant_rel` | Conflict in Inventory | `stock.inventory.conflict` | `stock_inventory_conflict_id` | Quants | `stock.quant` | `stock_quant_id` | no declaring field in the observed registry | `cascade` |
| `stock_inventory_warning_stock_quant_rel` | Inventory Adjustment Warning | `stock.inventory.warning` | `stock_inventory_warning_id` | Quants | `stock.quant` | `stock_quant_id` | no declaring field in the observed registry | `cascade` |
| `stock_landed_cost_stock_picking_rel` | Stock Landed Cost | `stock.landed.cost` | `stock_landed_cost_id` | Transfer | `stock.picking` | `stock_picking_id` | no declaring field in the observed registry | `cascade` |
| `stock_location_stock_picking_type_rel` | Picking Type | `stock.picking.type` | `stock_picking_type_id` | Inventory Locations | `stock.location` | `stock_location_id` | no declaring field in the observed registry | `cascade` |
| `stock_move_created_purchase_line_rel` | Purchase Order Line | `purchase.order.line` | `created_purchase_line_id` | Stock Move | `stock.move` | `move_id` | Purchase Order Line (`purchase.order.line`).`move_dest_ids`; Stock Move (`stock.move`).`created_purchase_line_ids` | `cascade` |
| `stock_move_line_consume_rel` | Product Moves (Stock Move Line) | `stock.move.line` | `consume_line_id` | Product Moves (Stock Move Line) | `stock.move.line` | `produce_line_id` | Product Moves (Stock Move Line) (`stock.move.line`).`consume_line_ids`; Product Moves (Stock Move Line) (`stock.move.line`).`produce_line_ids` | `cascade` |
| `stock_move_line_stock_put_in_pack_rel` | Put In Pack Wizard | `stock.put.in.pack` | `stock_put_in_pack_id` | Product Moves (Stock Move Line) | `stock.move.line` | `stock_move_line_id` | no declaring field in the observed registry | `cascade` |
| `stock_move_move_rel` | Stock Move | `stock.move` | `move_orig_id` | Stock Move | `stock.move` | `move_dest_id` | Stock Move (`stock.move`).`move_dest_ids`; Stock Move (`stock.move`).`move_orig_ids` | `cascade` |
| `stock_notification_product_partner_rel` | Product Variant | `product.product` | `product_product_id` | Contact | `res.partner` | `res_partner_id` | Product Variant (`product.product`).`stock_notification_partner_ids` | `cascade` |
| `stock_orderpoint_snooze_stock_warehouse_orderpoint_rel` | Snooze Orderpoint | `stock.orderpoint.snooze` | `stock_orderpoint_snooze_id` | Minimum Inventory Rule | `stock.warehouse.orderpoint` | `stock_warehouse_orderpoint_id` | no declaring field in the observed registry | `cascade` |
| `stock_package_history_stock_picking_rel` | Transfer | `stock.picking` | `stock_picking_id` | Stock Package History | `stock.package.history` | `stock_package_history_id` | no declaring field in the observed registry | `cascade` |
| `stock_package_stock_put_in_pack_rel` | Put In Pack Wizard | `stock.put.in.pack` | `stock_put_in_pack_id` | Package | `stock.package` | `stock_package_id` | no declaring field in the observed registry | `cascade` |
| `stock_package_type_stock_putaway_rule_rel` | Putaway Rule | `stock.putaway.rule` | `stock_putaway_rule_id` | Stock package type | `stock.package.type` | `stock_package_type_id` | no declaring field in the observed registry | `cascade` |
| `stock_package_type_stock_route_rel` | Stock package type | `stock.package.type` | `stock_package_type_id` | Inventory Routes | `stock.route` | `stock_route_id` | no declaring field in the observed registry | `cascade` |
| `stock_picking_backorder_rel` | Backorder Confirmation | `stock.backorder.confirmation` | `stock_backorder_confirmation_id` | Transfer | `stock.picking` | `stock_picking_id` | Backorder Confirmation (`stock.backorder.confirmation`).`pick_ids` | `cascade` |
| `stock_picking_sms_rel` | Confirm Stock text message | `confirm.stock.sms` | `confirm_stock_sms_id` | Transfer | `stock.picking` | `stock_picking_id` | Confirm Stock text message (`confirm.stock.sms`).`pick_ids` | `cascade` |
| `stock_quant_stock_quant_relocate_rel` | Stock Quantity Relocation | `stock.quant.relocate` | `stock_quant_relocate_id` | Quants | `stock.quant` | `stock_quant_id` | no declaring field in the observed registry | `cascade` |
| `stock_quant_stock_request_count_rel` | Stock Request an Inventory Count | `stock.request.count` | `stock_request_count_id` | Quants | `stock.quant` | `stock_quant_id` | no declaring field in the observed registry | `cascade` |
| `stock_reference_move_rel` | Stock Move | `stock.move` | `move_id` | Reference between stock documents | `stock.reference` | `reference_id` | Stock Move (`stock.move`).`reference_ids`; Reference between stock documents (`stock.reference`).`move_ids` | `cascade` |
| `stock_reference_pos_order_rel` | Point of Sale Orders | `pos.order` | `pos_order_id` | Reference between stock documents | `stock.reference` | `reference_id` | Point of Sale Orders (`pos.order`).`stock_reference_ids`; Reference between stock documents (`stock.reference`).`pos_order_ids` | `cascade` |
| `stock_reference_production_rel` | Manufacturing Order | `mrp.production` | `production_id` | Reference between stock documents | `stock.reference` | `reference_id` | Manufacturing Order (`mrp.production`).`reference_ids`; Reference between stock documents (`stock.reference`).`production_ids` | `cascade` |
| `stock_reference_purchase_rel` | Purchase Order | `purchase.order` | `purchase_id` | Reference between stock documents | `stock.reference` | `reference_id` | Purchase Order (`purchase.order`).`reference_ids`; Reference between stock documents (`stock.reference`).`purchase_ids` | `cascade` |
| `stock_reference_repair_rel` | Repair Order | `repair.order` | `repair_id` | Reference between stock documents | `stock.reference` | `reference_id` | Repair Order (`repair.order`).`reference_ids` | `cascade` |
| `stock_reference_sale_rel` | Sales Order | `sale.order` | `sale_id` | Reference between stock documents | `stock.reference` | `reference_id` | Sales Order (`sale.order`).`stock_reference_ids`; Reference between stock documents (`stock.reference`).`sale_ids` | `cascade` |
| `stock_route_categ` | Inventory Routes | `stock.route` | `route_id` | Product Category | `product.category` | `categ_id` | Product Category (`product.category`).`route_ids`; Inventory Routes (`stock.route`).`categ_ids` | `cascade` |
| `stock_route_move` | Stock Move | `stock.move` | `move_id` | Inventory Routes | `stock.route` | `route_id` | Stock Move (`stock.move`).`route_ids` | `cascade` |
| `stock_route_product` | Inventory Routes | `stock.route` | `route_id` | Product | `product.template` | `product_id` | Product (`product.template`).`route_ids`; Inventory Routes (`stock.route`).`product_ids` | `cascade` |
| `stock_route_shipping` | Shipping Methods | `delivery.carrier` | `shipping_id` | Inventory Routes | `stock.route` | `route_id` | Shipping Methods (`delivery.carrier`).`route_ids` | `cascade` |
| `stock_route_stock_rules_report_rel` | Stock Rules report | `stock.rules.report` | `stock_rules_report_id` | Inventory Routes | `stock.route` | `stock_route_id` | no declaring field in the observed registry | `cascade` |
| `stock_route_warehouse` | Inventory Routes | `stock.route` | `route_id` | Warehouse | `stock.warehouse` | `warehouse_id` | Inventory Routes (`stock.route`).`warehouse_ids`; Warehouse (`stock.warehouse`).`route_ids` | `cascade` |
| `stock_rules_report_stock_warehouse_rel` | Stock Rules report | `stock.rules.report` | `stock_rules_report_id` | Warehouse | `stock.warehouse` | `stock_warehouse_id` | no declaring field in the observed registry | `cascade` |
| `stock_scrap_stock_scrap_reason_tag_rel` | Scrap | `stock.scrap` | `stock_scrap_id` | Scrap Reason Tag | `stock.scrap.reason.tag` | `stock_scrap_reason_tag_id` | no declaring field in the observed registry | `cascade` |
| `stock_wh_resupply_table` | Warehouse | `stock.warehouse` | `supplied_wh_id` | Warehouse | `stock.warehouse` | `supplier_wh_id` | Warehouse (`stock.warehouse`).`resupply_wh_ids` | `cascade` |
| `summary_emp_rel` | human resources Time Off Summary Report By Employee | `hr.holidays.summary.employee` | `sum_id` | Employee | `hr.employee` | `emp_id` | human resources Time Off Summary Report By Employee (`hr.holidays.summary.employee`).`emp` | `cascade` |
| `survey_invite_partner_ids` | Survey Invitation Wizard | `survey.invite` | `invite_id` | Contact | `res.partner` | `partner_id` | Survey Invitation Wizard (`survey.invite`).`partner_ids` | `cascade` |
| `survey_mail_compose_message_ir_attachments_rel` | Survey Invitation Wizard | `survey.invite` | `wizard_id` | Attachment | `ir.attachment` | `attachment_id` | Survey Invitation Wizard (`survey.invite`).`attachment_ids` | `cascade` |
| `survey_question_survey_question_answer_rel` | Survey Question | `survey.question` | `survey_question_id` | Survey Label | `survey.question.answer` | `survey_question_answer_id` | no declaring field in the observed registry | `cascade` |
| `survey_question_survey_user_input_rel` | Survey User Input | `survey.user_input` | `survey_user_input_id` | Survey Question | `survey.question` | `survey_question_id` | no declaring field in the observed registry | `cascade` |
| `task_dependencies_rel` | Task | `project.task` | `task_id` | Task | `project.task` | `depends_on_id` | Task (`project.task`).`depend_on_ids`; Task (`project.task`).`dependent_ids`; Tasks Analysis (`report.project.task.user`).`dependent_ids` | `cascade` |
| `team_favorite_user_rel` | Sales Team | `crm.team` | `team_id` | User | `res.users` | `user_id` | Sales Team (`crm.team`).`favorite_user_ids` | `cascade` |
| `template_attribute_value_mrp_production_rel` | Manufacturing Order | `mrp.production` | `production_id` | Product Template Attribute Value | `product.template.attribute.value` | `template_attribute_value_id` | Manufacturing Order (`mrp.production`).`never_product_template_attribute_value_ids` | `cascade` |
| `template_attribute_value_stock_move_rel` | Stock Move | `stock.move` | `move_id` | Product Template Attribute Value | `product.template.attribute.value` | `template_attribute_value_id` | Stock Move (`stock.move`).`never_product_template_attribute_value_ids` | `cascade` |
| `utm_tag_rel` | campaign tracking parameter Campaign | `utm.campaign` | `tag_id` | campaign tracking parameter Tag | `utm.tag` | `campaign_id` | campaign tracking parameter Campaign (`utm.campaign`).`tag_ids` | `cascade` |
| `warning_purchase_order_alternative_rel` | Wizard in case purchase order still has open alternative requests for quotation | `purchase.requisition.alternative.warning` | `purchase_requisition_alternative_warning_id` | Purchase Order | `purchase.order` | `purchase_order_id` | Wizard in case purchase order still has open alternative requests for quotation (`purchase.requisition.alternative.warning`).`alternative_po_ids` | `cascade` |
| `warning_purchase_order_rel` | Wizard in case purchase order still has open alternative requests for quotation | `purchase.requisition.alternative.warning` | `purchase_requisition_alternative_warning_id` | Purchase Order | `purchase.order` | `purchase_order_id` | Wizard in case purchase order still has open alternative requests for quotation (`purchase.requisition.alternative.warning`).`po_ids` | `cascade` |
| `website_lang_rel` | Website | `website` | `website_id` | Languages | `res.lang` | `lang_id` | Website (`website`).`language_ids` | `cascade` |
| `wip_move_production_rel` | Manufacturing Order | `mrp.production` | `production_id` | Journal Entry | `account.move` | `move_id` | Journal Entry (`account.move`).`wip_production_ids`; Manufacturing Order (`mrp.production`).`wip_move_ids` | `cascade` |

## 9. Foreign keys

The observed installation carries 4,575 foreign keys.

### 9.1 Where a foreign key exists

A foreign key is created on the column of every stored link to one record, from the column to the primary key of the target table, and on both columns of every association table. No other column carries a foreign key.

### 9.2 Deletion behaviour

The behaviour applied when the target record is deleted is declared by the relation. When the relation does not declare one, the default depends on the kind of the two entities:

| Source entity | Target entity | Link required | Default behaviour |
|---|---|---|---|
| Persistent | Persistent | yes | `restrict`: the deletion of the target is refused while a source row points at it. |
| Persistent | Persistent | no | `set null`: the column of the source row is cleared and the source row survives. |
| Interactive assistant | Persistent | yes | `cascade`: the assistant row is deleted with the target, so that a leftover assistant record never blocks a business deletion. |
| Interactive assistant | Persistent | no | `set null` |
| Association table, first column | Persistent | always | `cascade` |
| Association table, second column | Persistent | always | `cascade`, unless the relation declares `restrict`. |

Three rules constrain the declaration:

1. A required link may not declare `set null`. The definition is refused with the message "The m2o field" followed by the field name, "of model" followed by the entity name, "is required but declares its ondelete policy as being 'set null'. Only 'restrict' and 'cascade' make sense." A replacement must refuse the same combination.
2. A link towards one of the framework registry entities — the entity registry (`ir.model`), the field registry (`ir.model.fields`), the value registry (`ir.model.fields.selection`), the external identifier registry (`ir.model.data`) and the access rule registry (`ir.model.access`) — may not declare `restrict`, because those rows are removed when a capability package is uninstalled.
3. A link from a persistent entity to an interactive assistant entity is forbidden.

### 9.3 Where a foreign key is deliberately absent

| Case | Reason |
|---|---|
| Per-company columns | The identifiers live inside a structured document keyed by company, so the store cannot reference them. |
| Polymorphic links, whether the pair of an entity-name column and an identifier column or the single reference column | The target table varies per row. |
| Entities read from a stored query, as source or as target | There is no table to attach the key to. |
| The action registry and its specialized tables | The action tables share a common parent table, which cannot carry the key. |
| Columns of a relation that the definition marks as not stored | There is no column. |

For every case in this table, referential integrity is the responsibility of the application: deleting a target record leaves dangling references, and every read of such a link must tolerate a missing target — the value reads as no link, and an operation that requires the target raises the missing-record error.

### 9.4 Repairing foreign keys

When the schema is rebuilt and a foreign key exists with a different target or a different deletion behaviour, the existing key is dropped and the correct one is created. Keys on the same column that duplicate each other are dropped, keeping one. A key that cannot be created because existing rows violate it stops the installation of the package and the transaction is rolled back.

## 10. Indexes

The observed installation carries 2,933 indexes. 1,000 of them are column indexes created from a field declaration and are listed in section 10.2; the others are primary keys, association-table indexes and the objects declared by the entities and listed in section 11.2.

### 10.1 Kinds

| Kind | Created when | Physical form |
|---|---|---|
| Primary key | Always, on `id` | Unique index on the primary key column |
| Plain | The field declares an index | Ordered index on the column |
| Partial | The field declares an index restricted to non-empty values | Ordered index on the column, restricted to the rows where the column is not empty. For a per-company column the indexed expression is the test that the column is not empty, and the same restriction applies. |
| Text similarity | The field declares a similarity index | Inverted index for substring and case-insensitive matching. For a per-language column the indexed expression is the concatenation of all the language values of the document. When the accent-insensitive function is available and immutable, the indexed expression is wrapped in it, so that accent-insensitive searches use the index. |
| Unique index, declared | The entity declares a unique index, optionally with a condition | Unique index on the listed columns, restricted by the condition when there is one |
| Index, declared | The entity declares an index, optionally with a condition or an expression | Index on the listed columns or expression |
| Association table index | Always, on every association table | Index on the pair of columns in reverse order, in addition to the primary key on the pair |
| Ancestor path index | Always, on the eleven entities that maintain one | Ordered index on the ancestor path column, which serves prefix matching for subtree queries |

Rules:

1. An index is never created automatically on a link column. Link columns that need one declare it.
2. A per-language column can only carry a text-similarity index. A declaration of any other kind on such a column is ignored and a warning is logged.
3. When an index exists with a different access method than the declared kind — for example an ordered index where a similarity index is now declared — it is dropped and recreated.
4. Index creation is attempted inside a savepoint; a failure is logged and does not abort the installation, except for unique indexes, whose failure means the data violates the declared uniqueness and the installation is refused.
5. The definition of a declared index is stored as a comment on the index, so that a later run can detect a changed definition. An index that exists without a comment is left untouched, which allows an operator to tune an index without the builder undoing it.

### 10.2 Indexed columns

Every column that carries a column-level index, with the kind of index: 603 plain, 360 partial and 37 text similarity, 1,000 in all.

| Entity | Transport name | Table | Column | Index kind |
|---|---|---|---|---|
| Account | `account.account` | `account_account` | `account_type` | plain |
| Account | `account.account` | `account_account` | `code_store` | partial |
| Account | `account.account` | `account_account` | `name` | text similarity |
| Analytic Account | `account.analytic.account` | `account_analytic_account` | `code` | plain |
| Analytic Account | `account.analytic.account` | `account_analytic_account` | `name` | text similarity |
| Analytic Account | `account.analytic.account` | `account_analytic_account` | `partner_id` | partial |
| Analytic Account | `account.analytic.account` | `account_analytic_account` | `plan_id` | plain |
| Analytic Plan's Applicabilities | `account.analytic.applicability` | `account_analytic_applicability` | `analytic_plan_id` | partial |
| Analytic Line | `account.analytic.line` | `account_analytic_line` | `account_id` | plain |
| Analytic Line | `account.analytic.line` | `account_analytic_line` | `date` | plain |
| Analytic Line | `account.analytic.line` | `account_analytic_line` | `employee_id` | plain |
| Analytic Line | `account.analytic.line` | `account_analytic_line` | `general_account_id` | partial |
| Analytic Line | `account.analytic.line` | `account_analytic_line` | `global_leave_id` | partial |
| Analytic Line | `account.analytic.line` | `account_analytic_line` | `holiday_id` | partial |
| Analytic Line | `account.analytic.line` | `account_analytic_line` | `move_line_id` | plain |
| Analytic Line | `account.analytic.line` | `account_analytic_line` | `order_id` | plain |
| Analytic Line | `account.analytic.line` | `account_analytic_line` | `parent_task_id` | partial |
| Analytic Line | `account.analytic.line` | `account_analytic_line` | `product_id` | partial |
| Analytic Line | `account.analytic.line` | `account_analytic_line` | `project_id` | plain |
| Analytic Line | `account.analytic.line` | `account_analytic_line` | `so_line` | partial |
| Analytic Line | `account.analytic.line` | `account_analytic_line` | `task_id` | partial |
| Analytic Line | `account.analytic.line` | `account_analytic_line` | `timesheet_invoice_id` | partial |
| Analytic Line | `account.analytic.line` | `account_analytic_line` | `user_id` | plain |
| Analytic Plans | `account.analytic.plan` | `account_analytic_plan` | `default_applicability` | partial |
| Analytic Plans | `account.analytic.plan` | `account_analytic_plan` | `parent_id` | partial |
| Analytic Plans | `account.analytic.plan` | `account_analytic_plan` | `parent_path` | plain |
| Bank Statement | `account.bank.statement` | `account_bank_statement` | `date` | plain |
| Bank Statement Line | `account.bank.statement.line` | `account_bank_statement_line` | `move_id` | plain |
| Bank Statement Line | `account.bank.statement.line` | `account_bank_statement_line` | `partner_name` | partial |
| Bank Statement Line | `account.bank.statement.line` | `account_bank_statement_line` | `payment_ref` | text similarity |
| Bank Statement Line | `account.bank.statement.line` | `account_bank_statement_line` | `pos_session_id` | partial |
| Bank Statement Line | `account.bank.statement.line` | `account_bank_statement_line` | `statement_id` | plain |
| Account Cash Rounding | `account.cash.rounding` | `account_cash_rounding` | `loss_account_id` | partial |
| Account Cash Rounding | `account.cash.rounding` | `account_cash_rounding` | `profit_account_id` | partial |
| Electronic Document for an account.move | `account.edi.document` | `account_edi_document` | `move_id` | plain |
| Account electronic data interchange proxy user | `account_edi_proxy_client.user` | `account_edi_proxy_client_user` | `company_id` | plain |
| Fiscal Position | `account.fiscal.position` | `account_fiscal_position` | `company_id` | plain |
| Account Group | `account.group` | `account_group` | `parent_id` | plain |
| Journal | `account.journal` | `account_journal` | `bank_account_id` | partial |
| Journal | `account.journal` | `account_journal` | `company_id` | plain |
| Journal Entry | `account.move` | `account_move` | `auto_post_origin_id` | partial |
| Journal Entry | `account.move` | `account_move` | `campaign_id` | partial |
| Journal Entry | `account.move` | `account_move` | `company_id` | plain |
| Journal Entry | `account.move` | `account_move` | `date` | plain |
| Journal Entry | `account.move` | `account_move` | `debit_origin_id` | partial |
| Journal Entry | `account.move` | `account_move` | `inalterable_hash` | partial |
| Journal Entry | `account.move` | `account_move` | `invoice_date` | plain |
| Journal Entry | `account.move` | `account_move` | `invoice_date_due` | plain |
| Journal Entry | `account.move` | `account_move` | `l10n_es_edi_verifactu_substituted_entry_id` | partial |
| Journal Entry | `account.move` | `account_move` | `l10n_hu_edi_state` | partial |
| Journal Entry | `account.move` | `account_move` | `l10n_hu_edi_transaction_code` | text similarity |
| Journal Entry | `account.move` | `account_move` | `l10n_in_withholding_ref_move_id` | partial |
| Journal Entry | `account.move` | `account_move` | `l10n_in_withholding_ref_payment_id` | partial |
| Journal Entry | `account.move` | `account_move` | `l10n_latam_document_type_id` | partial |
| Journal Entry | `account.move` | `account_move` | `l10n_pl_edi_number` | plain |
| Journal Entry | `account.move` | `account_move` | `medium_id` | partial |
| Journal Entry | `account.move` | `account_move` | `message_main_attachment_id` | partial |
| Journal Entry | `account.move` | `account_move` | `move_type` | plain |
| Journal Entry | `account.move` | `account_move` | `name` | text similarity |
| Journal Entry | `account.move` | `account_move` | `origin_payment_id` | partial |
| Journal Entry | `account.move` | `account_move` | `partner_bank_id` | partial |
| Journal Entry | `account.move` | `account_move` | `partner_id` | plain |
| Journal Entry | `account.move` | `account_move` | `payment_reference` | text similarity |
| Journal Entry | `account.move` | `account_move` | `peppol_message_uuid` | partial |
| Journal Entry | `account.move` | `account_move` | `ref` | text similarity |
| Journal Entry | `account.move` | `account_move` | `reversed_entry_id` | partial |
| Journal Entry | `account.move` | `account_move` | `reversed_pos_order_id` | partial |
| Journal Entry | `account.move` | `account_move` | `secure_sequence_number` | plain |
| Journal Entry | `account.move` | `account_move` | `source_id` | partial |
| Journal Entry | `account.move` | `account_move` | `statement_line_id` | partial |
| Journal Entry | `account.move` | `account_move` | `tax_cash_basis_origin_move_id` | partial |
| Journal Entry | `account.move` | `account_move` | `tax_cash_basis_rec_id` | partial |
| Journal Item | `account.move.line` | `account_move_line` | `cogs_origin_id` | partial |
| Journal Item | `account.move.line` | `account_move_line` | `company_id` | plain |
| Journal Item | `account.move.line` | `account_move_line` | `date_maturity` | plain |
| Journal Item | `account.move.line` | `account_move_line` | `expense_id` | partial |
| Journal Item | `account.move.line` | `account_move_line` | `full_reconcile_id` | partial |
| Journal Item | `account.move.line` | `account_move_line` | `group_tax_id` | partial |
| Journal Item | `account.move.line` | `account_move_line` | `journal_id` | plain |
| Journal Item | `account.move.line` | `account_move_line` | `l10n_in_gstr_section` | partial |
| Journal Item | `account.move.line` | `account_move_line` | `l10n_latam_document_type_id` | partial |
| Journal Item | `account.move.line` | `account_move_line` | `matching_number` | plain |
| Journal Item | `account.move.line` | `account_move_line` | `move_id` | plain |
| Journal Item | `account.move.line` | `account_move_line` | `move_name` | plain |
| Journal Item | `account.move.line` | `account_move_line` | `payment_id` | partial |
| Journal Item | `account.move.line` | `account_move_line` | `product_id` | plain |
| Journal Item | `account.move.line` | `account_move_line` | `purchase_line_id` | partial |
| Journal Item | `account.move.line` | `account_move_line` | `ref` | text similarity |
| Journal Item | `account.move.line` | `account_move_line` | `statement_id` | partial |
| Journal Item | `account.move.line` | `account_move_line` | `statement_line_id` | partial |
| Journal Item | `account.move.line` | `account_move_line` | `vehicle_id` | partial |
| Partial Reconcile | `account.partial.reconcile` | `account_partial_reconcile` | `credit_move_id` | plain |
| Partial Reconcile | `account.partial.reconcile` | `account_partial_reconcile` | `debit_move_id` | plain |
| Partial Reconcile | `account.partial.reconcile` | `account_partial_reconcile` | `exchange_move_id` | partial |
| Partial Reconcile | `account.partial.reconcile` | `account_partial_reconcile` | `full_reconcile_id` | partial |
| Payments | `account.payment` | `account_payment` | `destination_account_id` | partial |
| Payments | `account.payment` | `account_payment` | `force_outstanding_account_id` | partial |
| Payments | `account.payment` | `account_payment` | `message_main_attachment_id` | partial |
| Payments | `account.payment` | `account_payment` | `move_id` | plain |
| Payments | `account.payment` | `account_payment` | `outstanding_account_id` | partial |
| Payments | `account.payment` | `account_payment` | `paired_internal_transfer_payment_id` | partial |
| Payments | `account.payment` | `account_payment` | `payment_method_line_id` | plain |
| Payments | `account.payment` | `account_payment` | `pos_session_id` | partial |
| Payments | `account.payment` | `account_payment` | `source_payment_id` | partial |
| Payment Methods | `account.payment.method.line` | `account_payment_method_line` | `journal_id` | partial |
| Payment Terms Line | `account.payment.term.line` | `account_payment_term_line` | `payment_id` | plain |
| Business Level Responses for Peppol | `account.peppol.response` | `account_peppol_response` | `move_id` | partial |
| Business Level Responses for Peppol | `account.peppol.response` | `account_peppol_response` | `peppol_message_uuid` | partial |
| Rules for the reconciliation model | `account.reconcile.model.line` | `account_reconcile_model_line` | `model_id` | partial |
| Accounting Report | `account.report` | `account_report` | `root_report_id` | partial |
| Accounting Report Column | `account.report.column` | `account_report_column` | `report_id` | partial |
| Accounting Report Expression | `account.report.expression` | `account_report_expression` | `report_line_id` | plain |
| Accounting Report Line | `account.report.line` | `account_report_line` | `parent_id` | partial |
| Accounting Report Line | `account.report.line` | `account_report_line` | `report_id` | plain |
| Tax Group | `account.tax.group` | `account_tax_group` | `l10n_ar_tribute_afip_code` | plain |
| Tax Group | `account.tax.group` | `account_tax_group` | `l10n_ar_vat_afip_code` | plain |
| Tax Repartition Line | `account.tax.repartition.line` | `account_tax_repartition_line` | `tax_id` | partial |
| Passkey | `auth.passkey.key` | `auth_passkey_key` | `create_uid` | plain |
| Barcode Rule | `barcode.rule` | `barcode_rule` | `barcode_nomenclature_id` | partial |
| Base Import Mapping | `base_import.mapping` | `base_import_mapping` | `res_model` | plain |
| Blog | `blog.blog` | `blog_blog` | `website_id` | plain |
| Blog Post | `blog.post` | `blog_post` | `author_id` | partial |
| Blog Post | `blog.post` | `blog_post` | `blog_id` | plain |
| Blog Post | `blog.post` | `blog_post` | `is_published` | plain |
| Blog Post | `blog.post` | `blog_post` | `website_id` | plain |
| Blog Tag | `blog.tag` | `blog_tag` | `category_id` | plain |
| Communication Bus | `bus.bus` | `bus_bus` | `create_date` | plain |
| Calendar Attendee Information | `calendar.attendee` | `calendar_attendee` | `event_id` | plain |
| Calendar Event | `calendar.event` | `calendar_event` | `access_token` | plain |
| Calendar Event | `calendar.event` | `calendar_event` | `applicant_id` | partial |
| Calendar Event | `calendar.event` | `calendar_event` | `google_id` | partial |
| Calendar Event | `calendar.event` | `calendar_event` | `microsoft_id` | plain |
| Calendar Event | `calendar.event` | `calendar_event` | `ms_universal_event_id` | plain |
| Calendar Event | `calendar.event` | `calendar_event` | `opportunity_id` | plain |
| Calendar Event | `calendar.event` | `calendar_event` | `recurrence_id` | partial |
| Calendar Event | `calendar.event` | `calendar_event` | `start` | plain |
| Calendar Event | `calendar.event` | `calendar_event` | `user_id` | partial |
| Calendar Event | `calendar.event` | `calendar_event` | `videocall_channel_id` | partial |
| Calendar Filters | `calendar.filters` | `calendar_filters` | `partner_id` | plain |
| Calendar Filters | `calendar.filters` | `calendar_filters` | `user_id` | plain |
| Event Recurrence Rule | `calendar.recurrence` | `calendar_recurrence` | `google_id` | partial |
| Event Recurrence Rule | `calendar.recurrence` | `calendar_recurrence` | `microsoft_id` | plain |
| Event Recurrence Rule | `calendar.recurrence` | `calendar_recurrence` | `ms_universal_event_id` | plain |
| Marketing Card | `card.card` | `card_card` | `campaign_id` | plain |
| Chatbot Message | `chatbot.message` | `chatbot_message` | `discuss_channel_id` | plain |
| Chatbot Message | `chatbot.message` | `chatbot_message` | `script_step_id` | partial |
| Chatbot Script | `chatbot.script` | `chatbot_script` | `operator_partner_id` | plain |
| Chatbot Script Answer | `chatbot.script.answer` | `chatbot_script_answer` | `script_step_id` | plain |
| Chatbot Script Step | `chatbot.script.step` | `chatbot_script_step` | `chatbot_script_id` | plain |
| Chatbot Script Step | `chatbot.script.step` | `chatbot_script_step` | `crm_team_id` | partial |
| Lead | `crm.lead` | `crm_lead` | `campaign_id` | partial |
| Lead | `crm.lead` | `crm_lead` | `company_id` | plain |
| Lead | `crm.lead` | `crm_lead` | `contact_name` | text similarity |
| Lead | `crm.lead` | `crm_lead` | `date_last_stage_update` | plain |
| Lead | `crm.lead` | `crm_lead` | `email_domain_criterion` | partial |
| Lead | `crm.lead` | `crm_lead` | `email_from` | text similarity |
| Lead | `crm.lead` | `crm_lead` | `email_normalized` | text similarity |
| Lead | `crm.lead` | `crm_lead` | `event_id` | partial |
| Lead | `crm.lead` | `crm_lead` | `event_lead_rule_id` | partial |
| Lead | `crm.lead` | `crm_lead` | `lead_mining_request_id` | partial |
| Lead | `crm.lead` | `crm_lead` | `lost_reason_id` | plain |
| Lead | `crm.lead` | `crm_lead` | `medium_id` | partial |
| Lead | `crm.lead` | `crm_lead` | `name` | text similarity |
| Lead | `crm.lead` | `crm_lead` | `origin_channel_id` | partial |
| Lead | `crm.lead` | `crm_lead` | `origin_survey_id` | partial |
| Lead | `crm.lead` | `crm_lead` | `partner_assigned_id` | partial |
| Lead | `crm.lead` | `crm_lead` | `partner_id` | plain |
| Lead | `crm.lead` | `crm_lead` | `partner_name` | text similarity |
| Lead | `crm.lead` | `crm_lead` | `phone_sanitized` | partial |
| Lead | `crm.lead` | `crm_lead` | `priority` | plain |
| Lead | `crm.lead` | `crm_lead` | `reveal_id` | partial |
| Lead | `crm.lead` | `crm_lead` | `reveal_rule_id` | partial |
| Lead | `crm.lead` | `crm_lead` | `source_id` | partial |
| Lead | `crm.lead` | `crm_lead` | `stage_id` | plain |
| Lead | `crm.lead` | `crm_lead` | `team_id` | plain |
| Lead | `crm.lead` | `crm_lead` | `type` | plain |
| Lead | `crm.lead` | `crm_lead` | `user_id` | plain |
| Lead Scoring Frequency | `crm.lead.scoring.frequency` | `crm_lead_scoring_frequency` | `variable` | plain |
| customer relationship management Reveal View | `crm.reveal.view` | `crm_reveal_view` | `create_date` | plain |
| customer relationship management Reveal View | `crm.reveal.view` | `crm_reveal_view` | `reveal_rule_id` | partial |
| customer relationship management Reveal View | `crm.reveal.view` | `crm_reveal_view` | `reveal_state` | plain |
| Sales Team | `crm.team` | `crm_team` | `company_id` | plain |
| Sales Team Member | `crm.team.member` | `crm_team_member` | `crm_team_id` | plain |
| Sales Team Member | `crm.team.member` | `crm_team_member` | `user_id` | plain |
| Recycling Record | `data_recycle.record` | `data_recycle_record` | `recycle_model_id` | partial |
| Recycling Record | `data_recycle.record` | `data_recycle_record` | `res_id` | plain |
| Shipping Methods | `delivery.carrier` | `delivery_carrier` | `is_published` | plain |
| Shipping Methods | `delivery.carrier` | `delivery_carrier` | `website_id` | plain |
| Delivery Price Rules | `delivery.price.rule` | `delivery_price_rule` | `carrier_id` | plain |
| Keep the call history | `discuss.call.history` | `discuss_call_history` | `channel_id` | plain |
| Keep the call history | `discuss.call.history` | `discuss_call_history` | `start_call_message_id` | plain |
| Keep the call history | `discuss.call.history` | `discuss_call_history` | `start_dt` | plain |
| Discussion Channel | `discuss.channel` | `discuss_channel` | `last_interest_dt` | plain |
| Discussion Channel | `discuss.channel` | `discuss_channel` | `livechat_channel_id` | partial |
| Discussion Channel | `discuss.channel` | `discuss_channel` | `livechat_operator_id` | partial |
| Discussion Channel | `discuss.channel` | `discuss_channel` | `livechat_visitor_id` | partial |
| Discussion Channel | `discuss.channel` | `discuss_channel` | `parent_channel_id` | plain |
| Channel Member | `discuss.channel.member` | `discuss_channel_member` | `fetched_message_id` | partial |
| Channel Member | `discuss.channel.member` | `discuss_channel_member` | `guest_id` | plain |
| Channel Member | `discuss.channel.member` | `discuss_channel_member` | `last_interest_dt` | plain |
| Channel Member | `discuss.channel.member` | `discuss_channel_member` | `partner_id` | plain |
| Channel Member | `discuss.channel.member` | `discuss_channel_member` | `seen_message_id` | partial |
| Channel Member | `discuss.channel.member` | `discuss_channel_member` | `unpin_dt` | plain |
| Mail RTC session | `discuss.channel.rtc.session` | `discuss_channel_rtc_session` | `channel_id` | partial |
| Mail RTC session | `discuss.channel.rtc.session` | `discuss_channel_rtc_session` | `partner_id` | plain |
| Mail RTC session | `discuss.channel.rtc.session` | `discuss_channel_rtc_session` | `write_date` | plain |
| Metadata for voice attachments | `discuss.voice.metadata` | `discuss_voice_metadata` | `attachment_id` | plain |
| Event Booth | `event.booth` | `event_booth` | `booth_category_id` | plain |
| Event Booth | `event.booth` | `event_booth` | `event_id` | plain |
| Event Booth | `event.booth` | `event_booth` | `event_type_id` | plain |
| Event Booth | `event.booth` | `event_booth` | `sale_order_id` | partial |
| Event Booth | `event.booth` | `event_booth` | `sale_order_line_id` | partial |
| Event Booth Registration | `event.booth.registration` | `event_booth_registration` | `event_booth_id` | plain |
| Event Booth Registration | `event.booth.registration` | `event_booth_registration` | `sale_order_line_id` | plain |
| Event | `event.event` | `event_event` | `is_published` | plain |
| Event | `event.event` | `event_event` | `website_id` | plain |
| Event Ticket | `event.event.ticket` | `event_event_ticket` | `event_id` | plain |
| Event Ticket | `event.event.ticket` | `event_event_ticket` | `product_id` | plain |
| Event Automated Mailing | `event.mail` | `event_mail` | `event_id` | plain |
| Registration Mail Scheduler | `event.mail.registration` | `event_mail_registration` | `registration_id` | plain |
| Registration Mail Scheduler | `event.mail.registration` | `event_mail_registration` | `scheduler_id` | plain |
| Slot Mail Scheduler | `event.mail.slot` | `event_mail_slot` | `scheduler_id` | plain |
| Event Question Answer | `event.question.answer` | `event_question_answer` | `question_id` | plain |
| Quiz | `event.quiz` | `event_quiz` | `event_track_id` | partial |
| Question's Answer | `event.quiz.answer` | `event_quiz_answer` | `question_id` | plain |
| Content Quiz Question | `event.quiz.question` | `event_quiz_question` | `quiz_id` | plain |
| Event Registration | `event.registration` | `event_registration` | `event_id` | plain |
| Event Registration | `event.registration` | `event_registration` | `event_slot_id` | partial |
| Event Registration | `event.registration` | `event_registration` | `event_ticket_id` | partial |
| Event Registration | `event.registration` | `event_registration` | `name` | text similarity |
| Event Registration | `event.registration` | `event_registration` | `partner_id` | partial |
| Event Registration | `event.registration` | `event_registration` | `pos_order_line_id` | partial |
| Event Registration | `event.registration` | `event_registration` | `sale_order_line_id` | partial |
| Event Registration | `event.registration` | `event_registration` | `utm_campaign_id` | plain |
| Event Registration | `event.registration` | `event_registration` | `utm_medium_id` | plain |
| Event Registration | `event.registration` | `event_registration` | `utm_source_id` | plain |
| Event Registration | `event.registration` | `event_registration` | `visitor_id` | partial |
| Event Registration Answer | `event.registration.answer` | `event_registration_answer` | `registration_id` | plain |
| Event Slot | `event.slot` | `event_slot` | `event_id` | plain |
| Event Sponsor | `event.sponsor` | `event_sponsor` | `event_id` | plain |
| Event Sponsor | `event.sponsor` | `event_sponsor` | `is_published` | plain |
| Event Tag | `event.tag` | `event_tag` | `category_id` | plain |
| Event Tag | `event.tag` | `event_tag` | `is_published` | plain |
| Event Tag | `event.tag` | `event_tag` | `website_id` | plain |
| Event Tag Category | `event.tag.category` | `event_tag_category` | `is_published` | plain |
| Event Tag Category | `event.tag.category` | `event_tag_category` | `website_id` | plain |
| Event Track | `event.track` | `event_track` | `event_id` | plain |
| Event Track | `event.track` | `event_track` | `is_published` | plain |
| Event Track | `event.track` | `event_track` | `stage_id` | plain |
| Event Track Tag | `event.track.tag` | `event_track_tag` | `category_id` | partial |
| Track / Visitor Link | `event.track.visitor` | `event_track_visitor` | `partner_id` | plain |
| Track / Visitor Link | `event.track.visitor` | `event_track_visitor` | `track_id` | plain |
| Track / Visitor Link | `event.track.visitor` | `event_track_visitor` | `visitor_id` | plain |
| Event Booth Template | `event.type.booth` | `event_type_booth` | `booth_category_id` | plain |
| Event Booth Template | `event.type.booth` | `event_type_booth` | `event_type_id` | plain |
| Event Template Ticket | `event.type.ticket` | `event_type_ticket` | `product_id` | plain |
| Incoming Mail Server | `fetchmail.server` | `fetchmail_server` | `server_type` | plain |
| Incoming Mail Server | `fetchmail.server` | `fetchmail_server` | `state` | plain |
| Vehicle | `fleet.vehicle` | `fleet_vehicle` | `driver_employee_id` | partial |
| Drivers history on a vehicle | `fleet.vehicle.assignation.log` | `fleet_vehicle_assignation_log` | `vehicle_id` | plain |
| Vehicle Contract | `fleet.vehicle.log.contract` | `fleet_vehicle_log_contract` | `user_id` | plain |
| Vehicle Contract | `fleet.vehicle.log.contract` | `fleet_vehicle_log_contract` | `vehicle_id` | plain |
| Services for vehicles | `fleet.vehicle.log.services` | `fleet_vehicle_log_services` | `account_move_line_id` | partial |
| Services for vehicles | `fleet.vehicle.log.services` | `fleet_vehicle_log_services` | `vehicle_id` | plain |
| Model of a vehicle | `fleet.vehicle.model` | `fleet_vehicle_model` | `brand_id` | partial |
| Forum | `forum.forum` | `forum_forum` | `website_id` | plain |
| Forum Post | `forum.post` | `forum_post` | `create_date` | plain |
| Forum Post | `forum.post` | `forum_post` | `create_uid` | plain |
| Forum Post | `forum.post` | `forum_post` | `forum_id` | plain |
| Forum Post | `forum.post` | `forum_post` | `parent_id` | plain |
| Forum Post | `forum.post` | `forum_post` | `write_date` | plain |
| Forum Post | `forum.post` | `forum_post` | `write_uid` | plain |
| Post Vote | `forum.post.vote` | `forum_post_vote` | `create_date` | plain |
| Post Vote | `forum.post.vote` | `forum_post_vote` | `forum_id` | partial |
| Post Vote | `forum.post.vote` | `forum_post_vote` | `post_id` | plain |
| Forum Tag | `forum.tag` | `forum_tag` | `forum_id` | plain |
| Gamification Badge | `gamification.badge` | `gamification_badge` | `is_published` | plain |
| Gamification User Badge | `gamification.badge.user` | `gamification_badge_user` | `badge_id` | plain |
| Gamification User Badge | `gamification.badge.user` | `gamification_badge_user` | `employee_id` | plain |
| Gamification User Badge | `gamification.badge.user` | `gamification_badge_user` | `user_id` | plain |
| Gamification Challenge | `gamification.challenge` | `gamification_challenge` | `reward_id` | partial |
| Gamification generic goal for challenge | `gamification.challenge.line` | `gamification_challenge_line` | `challenge_id` | plain |
| Gamification Goal | `gamification.goal` | `gamification_goal` | `challenge_id` | plain |
| Gamification Goal | `gamification.goal` | `gamification_goal` | `user_id` | plain |
| Track Karma Changes | `gamification.karma.tracking` | `gamification_karma_tracking` | `tracking_date` | plain |
| Track Karma Changes | `gamification.karma.tracking` | `gamification_karma_tracking` | `user_id` | plain |
| Applicant | `hr.applicant` | `hr_applicant` | `active` | plain |
| Applicant | `hr.applicant` | `hr_applicant` | `campaign_id` | partial |
| Applicant | `hr.applicant` | `hr_applicant` | `date_last_stage_update` | plain |
| Applicant | `hr.applicant` | `hr_applicant` | `email_from` | text similarity |
| Applicant | `hr.applicant` | `hr_applicant` | `email_normalized` | text similarity |
| Applicant | `hr.applicant` | `hr_applicant` | `employee_id` | partial |
| Applicant | `hr.applicant` | `hr_applicant` | `job_id` | plain |
| Applicant | `hr.applicant` | `hr_applicant` | `linkedin_profile` | partial |
| Applicant | `hr.applicant` | `hr_applicant` | `medium_id` | partial |
| Applicant | `hr.applicant` | `hr_applicant` | `message_main_attachment_id` | partial |
| Applicant | `hr.applicant` | `hr_applicant` | `partner_id` | partial |
| Applicant | `hr.applicant` | `hr_applicant` | `partner_phone` | partial |
| Applicant | `hr.applicant` | `hr_applicant` | `partner_phone_sanitized` | partial |
| Applicant | `hr.applicant` | `hr_applicant` | `pool_applicant_id` | partial |
| Applicant | `hr.applicant` | `hr_applicant` | `sequence` | plain |
| Applicant | `hr.applicant` | `hr_applicant` | `source_id` | partial |
| Applicant | `hr.applicant` | `hr_applicant` | `stage_id` | plain |
| Skill level for an applicant | `hr.applicant.skill` | `hr_applicant_skill` | `applicant_id` | plain |
| Attendance | `hr.attendance` | `hr_attendance` | `check_in` | plain |
| Attendance | `hr.attendance` | `hr_attendance` | `date` | plain |
| Attendance | `hr.attendance` | `hr_attendance` | `employee_id` | plain |
| Attendance Overtime Line | `hr.attendance.overtime.line` | `hr_attendance_overtime_line` | `date` | plain |
| Attendance Overtime Line | `hr.attendance.overtime.line` | `hr_attendance_overtime_line` | `employee_id` | plain |
| Overtime Rule | `hr.attendance.overtime.rule` | `hr_attendance_overtime_rule` | `ruleset_id` | plain |
| Department | `hr.department` | `hr_department` | `company_id` | plain |
| Department | `hr.department` | `hr_department` | `parent_id` | plain |
| Department | `hr.department` | `hr_department` | `parent_path` | plain |
| Employee | `hr.employee` | `hr_employee` | `company_id` | plain |
| Employee | `hr.employee` | `hr_employee` | `message_main_attachment_id` | partial |
| Employee | `hr.employee` | `hr_employee` | `parent_id` | plain |
| Employee | `hr.employee` | `hr_employee` | `resource_id` | plain |
| Employee | `hr.employee` | `hr_employee` | `user_id` | partial |
| Employee | `hr.employee` | `hr_employee` | `work_contact_id` | partial |
| Skill level for employee | `hr.employee.skill` | `hr_employee_skill` | `employee_id` | plain |
| Expense | `hr.expense` | `hr_expense` | `account_move_id` | partial |
| Expense | `hr.expense` | `hr_expense` | `employee_id` | plain |
| Expense | `hr.expense` | `hr_expense` | `message_main_attachment_id` | partial |
| Expense | `hr.expense` | `hr_expense` | `sale_order_id` | partial |
| Expense | `hr.expense` | `hr_expense` | `sale_order_line_id` | partial |
| Expense | `hr.expense` | `hr_expense` | `state` | plain |
| Job Position | `hr.job` | `hr_job` | `department_id` | partial |
| Job Position | `hr.job` | `hr_job` | `is_published` | plain |
| Job Position | `hr.job` | `hr_job` | `name` | text similarity |
| Job Position | `hr.job` | `hr_job` | `survey_id` | partial |
| Job Position | `hr.job` | `hr_job` | `website_id` | plain |
| Skills for job positions | `hr.job.skill` | `hr_job_skill` | `job_id` | plain |
| Time Off | `hr.leave` | `hr_leave` | `date_from` | plain |
| Time Off | `hr.leave` | `hr_leave` | `employee_id` | plain |
| Time Off | `hr.leave` | `hr_leave` | `message_main_attachment_id` | partial |
| Time Off | `hr.leave` | `hr_leave` | `user_id` | plain |
| Accrual Plan Level | `hr.leave.accrual.level` | `hr_leave_accrual_level` | `accrual_plan_id` | plain |
| Accrual Plan | `hr.leave.accrual.plan` | `hr_leave_accrual_plan` | `time_off_type_id` | partial |
| Time Off Allocation | `hr.leave.allocation` | `hr_leave_allocation` | `accrual_plan_id` | partial |
| Time Off Allocation | `hr.leave.allocation` | `hr_leave_allocation` | `date_from` | plain |
| Time Off Allocation | `hr.leave.allocation` | `hr_leave_allocation` | `employee_id` | plain |
| Time Off Type | `hr.leave.type` | `hr_leave_type` | `work_entry_type_id` | partial |
| Source of Applicants | `hr.recruitment.source` | `hr_recruitment_source` | `job_id` | plain |
| Resume line of an employee | `hr.resume.line` | `hr_resume_line` | `channel_id` | partial |
| Resume line of an employee | `hr.resume.line` | `hr_resume_line` | `employee_id` | plain |
| Resume line of an employee | `hr.resume.line` | `hr_resume_line` | `event_id` | partial |
| Skill | `hr.skill` | `hr_skill` | `skill_type_id` | plain |
| Skill Level | `hr.skill.level` | `hr_skill_level` | `skill_type_id` | partial |
| Version | `hr.version` | `hr_version` | `department_id` | plain |
| Version | `hr.version` | `hr_version` | `employee_id` | plain |
| Version | `hr.version` | `hr_version` | `job_id` | plain |
| human resources Work Entry | `hr.work.entry` | `hr_work_entry` | `employee_id` | plain |
| human resources Work Entry | `hr.work.entry` | `hr_work_entry` | `version_id` | plain |
| human resources Work Entry | `hr.work.entry` | `hr_work_entry` | `work_entry_type_id` | plain |
| Keep the channel member history | `im_livechat.channel.member.history` | `im_livechat_channel_member_history` | `channel_id` | plain |
| Keep the channel member history | `im_livechat.channel.member.history` | `im_livechat_channel_member_history` | `chatbot_script_id` | partial |
| Keep the channel member history | `im_livechat.channel.member.history` | `im_livechat_channel_member_history` | `guest_id` | partial |
| Keep the channel member history | `im_livechat.channel.member.history` | `im_livechat_channel_member_history` | `member_id` | partial |
| Keep the channel member history | `im_livechat.channel.member.history` | `im_livechat_channel_member_history` | `partner_id` | partial |
| Livechat Channel Rules | `im_livechat.channel.rule` | `im_livechat_channel_rule` | `channel_id` | partial |
| Report Action | `ir.actions.report` | `ir_act_report_xml` | `paperformat_id` | partial |
| Server Actions | `ir.actions.server` | `ir_act_server` | `base_automation_id` | partial |
| Server Actions | `ir.actions.server` | `ir_act_server` | `model_id` | plain |
| Server Actions | `ir.actions.server` | `ir_act_server` | `parent_id` | plain |
| Action Window View | `ir.actions.act_window.view` | `ir_act_window_view` | `act_window_id` | partial |
| Configuration Wizards | `ir.actions.todo` | `ir_actions_todo` | `action_id` | plain |
| Asset | `ir.asset` | `ir_asset` | `theme_template_id` | partial |
| Attachment | `ir.attachment` | `ir_attachment` | `original_id` | partial |
| Attachment | `ir.attachment` | `ir_attachment` | `store_fname` | plain |
| Attachment | `ir.attachment` | `ir_attachment` | `theme_template_id` | partial |
| Attachment | `ir.attachment` | `ir_attachment` | `url` | partial |
| Scheduled Actions | `ir.cron` | `ir_cron` | `ir_actions_server_id` | plain |
| Progress of Scheduled Actions | `ir.cron.progress` | `ir_cron_progress` | `cron_id` | plain |
| Triggered actions | `ir.cron.trigger` | `ir_cron_trigger` | `call_at` | plain |
| Triggered actions | `ir.cron.trigger` | `ir_cron_trigger` | `cron_id` | plain |
| Default Values | `ir.default` | `ir_default` | `company_id` | plain |
| Default Values | `ir.default` | `ir_default` | `field_id` | plain |
| Default Values | `ir.default` | `ir_default` | `user_id` | plain |
| Exports | `ir.exports` | `ir_exports` | `resource` | plain |
| Exports Line | `ir.exports.line` | `ir_exports_line` | `export_id` | plain |
| Filters | `ir.filters` | `ir_filters` | `embedded_action_id` | partial |
| Logging | `ir.logging` | `ir_logging` | `dbname` | plain |
| Logging | `ir.logging` | `ir_logging` | `level` | plain |
| Logging | `ir.logging` | `ir_logging` | `type` | plain |
| Mail Server | `ir.mail_server` | `ir_mail_server` | `name` | plain |
| Model Access | `ir.model.access` | `ir_model_access` | `group_id` | plain |
| Model Access | `ir.model.access` | `ir_model_access` | `model_id` | plain |
| Model Access | `ir.model.access` | `ir_model_access` | `name` | plain |
| Model Constraint | `ir.model.constraint` | `ir_model_constraint` | `model` | plain |
| Model Constraint | `ir.model.constraint` | `ir_model_constraint` | `module` | plain |
| Model Constraint | `ir.model.constraint` | `ir_model_constraint` | `name` | plain |
| Fields | `ir.model.fields` | `ir_model_fields` | `model` | plain |
| Fields | `ir.model.fields` | `ir_model_fields` | `model_id` | plain |
| Fields | `ir.model.fields` | `ir_model_fields` | `name` | plain |
| Fields | `ir.model.fields` | `ir_model_fields` | `state` | plain |
| Fields | `ir.model.fields` | `ir_model_fields` | `website_form_blacklisted` | plain |
| Fields Selection | `ir.model.fields.selection` | `ir_model_fields_selection` | `field_id` | plain |
| Relation Model | `ir.model.relation` | `ir_model_relation` | `model` | plain |
| Relation Model | `ir.model.relation` | `ir_model_relation` | `module` | plain |
| Relation Model | `ir.model.relation` | `ir_model_relation` | `name` | plain |
| Application | `ir.module.category` | `ir_module_category` | `parent_id` | plain |
| Module | `ir.module.module` | `ir_module_module` | `category_id` | plain |
| Module | `ir.module.module` | `ir_module_module` | `state` | plain |
| Module dependency | `ir.module.module.dependency` | `ir_module_module_dependency` | `name` | plain |
| Module exclusion | `ir.module.module.exclusion` | `ir_module_module_exclusion` | `name` | plain |
| Profiling results | `ir.profile` | `ir_profile` | `session` | plain |
| Record Rule | `ir.rule` | `ir_rule` | `model_id` | plain |
| Menu | `ir.ui.menu` | `ir_ui_menu` | `parent_id` | plain |
| Menu | `ir.ui.menu` | `ir_ui_menu` | `parent_path` | plain |
| View | `ir.ui.view` | `ir_ui_view` | `inherit_id` | plain |
| View | `ir.ui.view` | `ir_ui_view` | `key` | partial |
| View | `ir.ui.view` | `ir_ui_view` | `model` | plain |
| View | `ir.ui.view` | `ir_ui_view` | `theme_template_id` | partial |
| Custom View | `ir.ui.view.custom` | `ir_ui_view_custom` | `ref_id` | plain |
| Custom View | `ir.ui.view.custom` | `ir_ui_view_custom` | `user_id` | plain |
| ARCA Responsibility Type | `l10n_ar.afip.responsibility.type` | `l10n_ar_afip_responsibility_type` | `code` | plain |
| ARCA Responsibility Type | `l10n_ar.afip.responsibility.type` | `l10n_ar_afip_responsibility_type` | `name` | text similarity |
| electronic data interchange and fiscalization information for Croatian electronic invoicing | `l10n_hr_edi.addendum` | `l10n_hr_edi_addendum` | `move_id` | plain |
| Declaration of Intent | `l10n_it_edi_doi.declaration_of_intent` | `l10n_it_edi_doi_declaration_of_intent` | `company_id` | plain |
| Declaration of Intent | `l10n_it_edi_doi.declaration_of_intent` | `l10n_it_edi_doi_declaration_of_intent` | `partner_id` | plain |
| Latam Document Type | `l10n_latam.document.type` | `l10n_latam_document_type` | `country_id` | plain |
| SInvoice symbol | `l10n_vn_edi_viettel.sinvoice.symbol` | `l10n_vn_edi_viettel_sinvoice_symbol` | `invoice_template_id` | plain |
| Link Tracker | `link.tracker` | `link_tracker` | `campaign_id` | partial |
| Link Tracker | `link.tracker` | `link_tracker` | `medium_id` | partial |
| Link Tracker | `link.tracker` | `link_tracker` | `source_id` | partial |
| Link Tracker Click | `link.tracker.click` | `link_tracker_click` | `campaign_id` | partial |
| Link Tracker Click | `link.tracker.click` | `link_tracker_click` | `link_id` | plain |
| Link Tracker Click | `link.tracker.click` | `link_tracker_click` | `mailing_trace_id` | partial |
| Link Tracker Click | `link.tracker.click` | `link_tracker_click` | `mass_mailing_id` | partial |
| Link Tracker Code | `link.tracker.code` | `link_tracker_code` | `link_id` | plain |
| Loyalty Coupon | `loyalty.card` | `loyalty_card` | `partner_id` | plain |
| Loyalty Coupon | `loyalty.card` | `loyalty_card` | `program_id` | partial |
| History for Loyalty cards and Electronic Wallets | `loyalty.history` | `loyalty_history` | `card_id` | plain |
| Loyalty Communication | `loyalty.mail` | `loyalty_mail` | `program_id` | plain |
| Loyalty Program | `loyalty.program` | `loyalty_program` | `website_id` | plain |
| Loyalty Reward | `loyalty.reward` | `loyalty_reward` | `program_id` | plain |
| Loyalty Rule | `loyalty.rule` | `loyalty_rule` | `program_id` | plain |
| Lunch Order | `lunch.order` | `lunch_order` | `state` | plain |
| Lunch Order | `lunch.order` | `lunch_order` | `supplier_id` | plain |
| Lunch Extras | `lunch.topping` | `lunch_topping` | `supplier_id` | partial |
| Activity | `mail.activity` | `mail_activity` | `calendar_event_id` | partial |
| Activity | `mail.activity` | `mail_activity` | `date_deadline` | plain |
| Activity | `mail.activity` | `mail_activity` | `res_id` | plain |
| Activity | `mail.activity` | `mail_activity` | `res_model` | plain |
| Activity | `mail.activity` | `mail_activity` | `res_model_id` | plain |
| Activity | `mail.activity` | `mail_activity` | `user_id` | plain |
| Activity Plan | `mail.activity.plan` | `mail_activity_plan` | `department_id` | partial |
| Activity plan template | `mail.activity.plan.template` | `mail_activity_plan_template` | `plan_id` | plain |
| Create activity and to-do at the same time | `mail.activity.todo.create` | `mail_activity_todo_create` | `date_deadline` | plain |
| Activity Type | `mail.activity.type` | `mail_activity_type` | `create_uid` | plain |
| Email Aliases | `mail.alias` | `mail_alias` | `alias_full_name` | partial |
| Mail Blacklist | `mail.blacklist` | `mail_blacklist` | `email` | text similarity |
| Canned Response | `mail.canned.response` | `mail_canned_response` | `source` | text similarity |
| Document Followers | `mail.followers` | `mail_followers` | `partner_id` | plain |
| Document Followers | `mail.followers` | `mail_followers` | `res_id` | plain |
| Document Followers | `mail.followers` | `mail_followers` | `res_model` | plain |
| Mail Gateway Allowed | `mail.gateway.allowed` | `mail_gateway_allowed` | `email_normalized` | plain |
| Mailing List Member | `mail.group.member` | `mail_group_member` | `email_normalized` | plain |
| Mailing List Member | `mail.group.member` | `mail_group_member` | `mail_group_id` | plain |
| Mailing List Message | `mail.group.message` | `mail_group_message` | `group_message_parent_id` | plain |
| Mailing List Message | `mail.group.message` | `mail_group_message` | `mail_group_id` | plain |
| Mailing List Message | `mail.group.message` | `mail_group_message` | `mail_message_id` | plain |
| Mailing List Message | `mail.group.message` | `mail_group_message` | `moderation_status` | plain |
| Mailing List black/white list | `mail.group.moderation` | `mail_group_moderation` | `mail_group_id` | plain |
| Store link preview data | `mail.link.preview` | `mail_link_preview` | `create_date` | plain |
| Outgoing Mails | `mail.mail` | `mail_mail` | `fetchmail_server_id` | partial |
| Outgoing Mails | `mail.mail` | `mail_mail` | `mail_message_id` | plain |
| Message | `mail.message` | `mail_message` | `author_id` | plain |
| Message | `mail.message` | `mail_message` | `mail_activity_type_id` | partial |
| Message | `mail.message` | `mail_message` | `message_id` | plain |
| Message | `mail.message` | `mail_message` | `parent_id` | partial |
| Message | `mail.message` | `mail_message` | `subtype_id` | plain |
| Link between link previews and messages | `mail.message.link.preview` | `mail_message_link_preview` | `link_preview_id` | plain |
| Link between link previews and messages | `mail.message.link.preview` | `mail_message_link_preview` | `message_id` | plain |
| Message Reaction | `mail.message.reaction` | `mail_message_reaction` | `message_id` | plain |
| Message Translation | `mail.message.translation` | `mail_message_translation` | `create_date` | plain |
| Message Notifications | `mail.notification` | `mail_notification` | `is_read` | plain |
| Message Notifications | `mail.notification` | `mail_notification` | `letter_id` | partial |
| Message Notifications | `mail.notification` | `mail_notification` | `mail_mail_id` | plain |
| Message Notifications | `mail.notification` | `mail_notification` | `mail_message_id` | plain |
| Message Notifications | `mail.notification` | `mail_notification` | `notification_status` | plain |
| Message Notifications | `mail.notification` | `mail_notification` | `notification_type` | plain |
| Message Notifications | `mail.notification` | `mail_notification` | `res_partner_id` | plain |
| Message Notifications | `mail.notification` | `mail_notification` | `sms_id_int` | partial |
| Push Notification Device | `mail.push.device` | `mail_push_device` | `partner_id` | plain |
| Email Templates | `mail.template` | `mail_template` | `mail_server_id` | partial |
| Email Templates | `mail.template` | `mail_template` | `model` | plain |
| Mail Tracking Value | `mail.tracking.value` | `mail_tracking_value` | `field_id` | plain |
| Mail Tracking Value | `mail.tracking.value` | `mail_tracking_value` | `mail_message_id` | plain |
| Mailing Favorite Filters | `mailing.filter` | `mailing_filter` | `create_uid` | plain |
| Mass Mailing | `mailing.mailing` | `mailing_mailing` | `campaign_id` | plain |
| Mass Mailing | `mailing.mailing` | `mailing_mailing` | `card_campaign_id` | partial |
| Mass Mailing | `mailing.mailing` | `mailing_mailing` | `mail_server_id` | partial |
| Mailing List Subscription | `mailing.subscription` | `mailing_subscription` | `list_id` | plain |
| Mailing Statistics | `mailing.trace` | `mailing_trace` | `campaign_id` | partial |
| Mailing Statistics | `mailing.trace` | `mailing_trace` | `mail_mail_id` | partial |
| Mailing Statistics | `mailing.trace` | `mailing_trace` | `mail_mail_id_int` | partial |
| Mailing Statistics | `mailing.trace` | `mailing_trace` | `mass_mailing_id` | plain |
| Mailing Statistics | `mailing.trace` | `mailing_trace` | `sms_id_int` | partial |
| Maintenance Equipment | `maintenance.equipment` | `maintenance_equipment` | `category_id` | partial |
| Maintenance Equipment | `maintenance.equipment` | `maintenance_equipment` | `employee_id` | partial |
| Maintenance Equipment | `maintenance.equipment` | `maintenance_equipment` | `maintenance_team_id` | partial |
| Maintenance Equipment | `maintenance.equipment` | `maintenance_equipment` | `owner_user_id` | partial |
| Maintenance Request | `maintenance.request` | `maintenance_request` | `category_id` | partial |
| Maintenance Request | `maintenance.request` | `maintenance_request` | `equipment_id` | plain |
| Maintenance Request | `maintenance.request` | `maintenance_request` | `maintenance_team_id` | plain |
| Bill of Material | `mrp.bom` | `mrp_bom` | `company_id` | plain |
| Bill of Material | `mrp.bom` | `mrp_bom` | `product_id` | plain |
| Bill of Material | `mrp.bom` | `mrp_bom` | `product_tmpl_id` | plain |
| Byproduct | `mrp.bom.byproduct` | `mrp_bom_byproduct` | `bom_id` | plain |
| Byproduct | `mrp.bom.byproduct` | `mrp_bom_byproduct` | `company_id` | plain |
| Bill of Material Line | `mrp.bom.line` | `mrp_bom_line` | `bom_id` | plain |
| Bill of Material Line | `mrp.bom.line` | `mrp_bom_line` | `company_id` | plain |
| Bill of Material Line | `mrp.bom.line` | `mrp_bom_line` | `product_id` | plain |
| Bill of Material Line | `mrp.bom.line` | `mrp_bom_line` | `product_tmpl_id` | plain |
| Manufacturing Order | `mrp.production` | `mrp_production` | `company_id` | plain |
| Manufacturing Order | `mrp.production` | `mrp_production` | `date_start` | plain |
| Manufacturing Order | `mrp.production` | `mrp_production` | `orderpoint_id` | partial |
| Manufacturing Order | `mrp.production` | `mrp_production` | `picking_type_id` | plain |
| Manufacturing Order | `mrp.production` | `mrp_production` | `production_group_id` | plain |
| Manufacturing Order | `mrp.production` | `mrp_production` | `reservation_state` | plain |
| Manufacturing Order | `mrp.production` | `mrp_production` | `state` | plain |
| Production Group | `mrp.production.group` | `mrp_production_group` | `name` | plain |
| Work Center Usage | `mrp.routing.workcenter` | `mrp_routing_workcenter` | `bom_id` | plain |
| Work Center Usage | `mrp.routing.workcenter` | `mrp_routing_workcenter` | `workcenter_id` | plain |
| Unbuild Order | `mrp.unbuild` | `mrp_unbuild` | `company_id` | plain |
| Unbuild Order | `mrp.unbuild` | `mrp_unbuild` | `mo_id` | partial |
| Work Center | `mrp.workcenter` | `mrp_workcenter` | `company_id` | plain |
| Work Center | `mrp.workcenter` | `mrp_workcenter` | `resource_calendar_id` | plain |
| Work Center | `mrp.workcenter` | `mrp_workcenter` | `resource_id` | plain |
| Work Center Capacity | `mrp.workcenter.capacity` | `mrp_workcenter_capacity` | `workcenter_id` | plain |
| Workcenter Productivity Log | `mrp.workcenter.productivity` | `mrp_workcenter_productivity` | `company_id` | plain |
| Workcenter Productivity Log | `mrp.workcenter.productivity` | `mrp_workcenter_productivity` | `workcenter_id` | plain |
| Workcenter Productivity Log | `mrp.workcenter.productivity` | `mrp_workcenter_productivity` | `workorder_id` | plain |
| Work Order | `mrp.workorder` | `mrp_workorder` | `operation_id` | partial |
| Work Order | `mrp.workorder` | `mrp_workorder` | `production_id` | plain |
| Work Order | `mrp.workorder` | `mrp_workorder` | `state` | plain |
| Work Order | `mrp.workorder` | `mrp_workorder` | `workcenter_id` | plain |
| MyInvois Document | `myinvois.document` | `myinvois_document` | `myinvois_external_uuid` | plain |
| MyInvois Document | `myinvois.document` | `myinvois_document` | `name` | text similarity |
| Business Level Responses for Nemhandel | `nemhandel.response` | `nemhandel_response` | `move_id` | partial |
| Onboarding Progress Tracker | `onboarding.progress` | `onboarding_progress` | `onboarding_id` | plain |
| Onboarding Progress Step Tracker | `onboarding.progress.step` | `onboarding_progress_step` | `step_id` | plain |
| Payment Method | `payment.method` | `payment_method` | `primary_payment_method_id` | partial |
| Payment Provider | `payment.provider` | `payment_provider` | `company_id` | plain |
| Payment Token | `payment.token` | `payment_token` | `company_id` | plain |
| Payment Token | `payment.token` | `payment_token` | `partner_id` | plain |
| Payment Transaction | `payment.transaction` | `payment_transaction` | `company_id` | plain |
| Payment Transaction | `payment.transaction` | `payment_transaction` | `operation` | plain |
| Payment Transaction | `payment.transaction` | `payment_transaction` | `source_transaction_id` | partial |
| Payment Transaction | `payment.transaction` | `payment_transaction` | `state` | plain |
| Payment Transaction | `payment.transaction` | `payment_transaction` | `token_id` | partial |
| Point of Sale Category | `pos.category` | `pos_category` | `parent_id` | plain |
| Point of Sale Configuration | `pos.config` | `pos_config` | `crm_team_id` | partial |
| Point of Sale Orders | `pos.order` | `pos_order` | `account_move` | partial |
| Point of Sale Orders | `pos.order` | `pos_order` | `company_id` | plain |
| Point of Sale Orders | `pos.order` | `pos_order` | `date_order` | plain |
| Point of Sale Orders | `pos.order` | `pos_order` | `partner_id` | partial |
| Point of Sale Orders | `pos.order` | `pos_order` | `pos_reference` | plain |
| Point of Sale Orders | `pos.order` | `pos_order` | `session_id` | plain |
| Point of Sale Orders | `pos.order` | `pos_order` | `state` | plain |
| Point of Sale Orders | `pos.order` | `pos_order` | `table_id` | partial |
| Point of Sale Order Lines | `pos.order.line` | `pos_order_line` | `combo_parent_id` | partial |
| Point of Sale Order Lines | `pos.order.line` | `pos_order_line` | `coupon_id` | partial |
| Point of Sale Order Lines | `pos.order.line` | `pos_order_line` | `course_id` | partial |
| Point of Sale Order Lines | `pos.order.line` | `pos_order_line` | `order_id` | plain |
| Point of Sale Order Lines | `pos.order.line` | `pos_order_line` | `refunded_orderline_id` | partial |
| Point of Sale Order Lines | `pos.order.line` | `pos_order_line` | `reward_id` | partial |
| Point of Sale Order Lines | `pos.order.line` | `pos_order_line` | `sale_order_line_id` | partial |
| Point of Sale Order Lines | `pos.order.line` | `pos_order_line` | `sale_order_origin_id` | partial |
| Specify product lot/serial number in point of sale order line | `pos.pack.operation.lot` | `pos_pack_operation_lot` | `pos_order_line_id` | partial |
| Point of Sale Payments | `pos.payment` | `pos_payment` | `account_move_id` | partial |
| Point of Sale Payments | `pos.payment` | `pos_payment` | `employee_id` | plain |
| Point of Sale Payments | `pos.payment` | `pos_payment` | `pos_order_id` | plain |
| Point of Sale Payments | `pos.payment` | `pos_payment` | `session_id` | plain |
| Point of Sale Payment Methods | `pos.payment.method` | `pos_payment_method` | `journal_id` | partial |
| Point of Sale Session | `pos.session` | `pos_session` | `config_id` | plain |
| Point of Sale Session | `pos.session` | `pos_session` | `move_id` | plain |
| Point of Sale Session | `pos.session` | `pos_session` | `state` | plain |
| Point of Sale Session | `pos.session` | `pos_session` | `user_id` | plain |
| Product Attribute | `product.attribute` | `product_attribute` | `category_id` | plain |
| Product Attribute | `product.attribute` | `product_attribute` | `sequence` | plain |
| Product Attribute Category | `product.attribute.category` | `product_attribute_category` | `sequence` | plain |
| Product Attribute Custom Value | `product.attribute.custom.value` | `product_attribute_custom_value` | `pos_order_line_id` | partial |
| Product Attribute Custom Value | `product.attribute.custom.value` | `product_attribute_custom_value` | `sale_order_line_id` | partial |
| Attribute Value | `product.attribute.value` | `product_attribute_value` | `attribute_id` | plain |
| Attribute Value | `product.attribute.value` | `product_attribute_value` | `sequence` | plain |
| Product Category | `product.category` | `product_category` | `name` | text similarity |
| Product Category | `product.category` | `product_category` | `parent_id` | plain |
| Product Category | `product.category` | `product_category` | `parent_path` | plain |
| Product Category | `product.category` | `product_category` | `property_account_expense_categ_id` | partial |
| Product Category | `product.category` | `product_category` | `property_account_income_categ_id` | partial |
| Product Category | `product.category` | `product_category` | `property_cost_method` | partial |
| Product Category | `product.category` | `product_category` | `property_price_difference_account_id` | partial |
| Product Category | `product.category` | `product_category` | `property_stock_journal` | partial |
| Product Category | `product.category` | `product_category` | `property_stock_valuation_account_id` | partial |
| Product Category | `product.category` | `product_category` | `property_valuation` | partial |
| Product Combo | `product.combo` | `product_combo` | `company_id` | plain |
| Product Combo Item | `product.combo.item` | `product_combo_item` | `combo_id` | plain |
| Product Image | `product.image` | `product_image` | `product_tmpl_id` | plain |
| Product Image | `product.image` | `product_image` | `product_variant_id` | plain |
| Pricelist Rule | `product.pricelist.item` | `product_pricelist_item` | `compute_price` | plain |
| Pricelist Rule | `product.pricelist.item` | `product_pricelist_item` | `pricelist_id` | plain |
| Pricelist Rule | `product.pricelist.item` | `product_pricelist_item` | `product_id` | partial |
| Pricelist Rule | `product.pricelist.item` | `product_pricelist_item` | `product_tmpl_id` | partial |
| Product Variant | `product.product` | `product_product` | `barcode` | partial |
| Product Variant | `product.product` | `product_product` | `combination_indices` | plain |
| Product Variant | `product.product` | `product_product` | `default_code` | plain |
| Product Variant | `product.product` | `product_product` | `l10n_tr_ctsp_number` | partial |
| Product Variant | `product.product` | `product_product` | `product_tmpl_id` | plain |
| Product Variant | `product.product` | `product_product` | `standard_price` | partial |
| Website Product Category | `product.public.category` | `product_public_category` | `parent_id` | plain |
| Website Product Category | `product.public.category` | `product_public_category` | `parent_path` | plain |
| Website Product Category | `product.public.category` | `product_public_category` | `sequence` | plain |
| Website Product Category | `product.public.category` | `product_public_category` | `website_id` | plain |
| Supplier Pricelist | `product.supplierinfo` | `product_supplierinfo` | `company_id` | plain |
| Supplier Pricelist | `product.supplierinfo` | `product_supplierinfo` | `product_tmpl_id` | plain |
| Supplier Pricelist | `product.supplierinfo` | `product_supplierinfo` | `purchase_requisition_line_id` | partial |
| Product Tag | `product.tag` | `product_tag` | `website_id` | plain |
| Product | `product.template` | `product_template` | `company_id` | plain |
| Product | `product.template` | `product_template` | `description` | text similarity |
| Product | `product.template` | `product_template` | `description_ecommerce` | text similarity |
| Product | `product.template` | `product_template` | `description_sale` | text similarity |
| Product | `product.template` | `product_template` | `is_published` | plain |
| Product | `product.template` | `product_template` | `l10n_tr_default_sales_return_account_id` | partial |
| Product | `product.template` | `product_template` | `name` | text similarity |
| Product | `product.template` | `product_template` | `project_id` | partial |
| Product | `product.template` | `product_template` | `project_template_id` | partial |
| Product | `product.template` | `product_template` | `property_account_expense_id` | partial |
| Product | `product.template` | `product_template` | `property_account_income_id` | partial |
| Product | `product.template` | `product_template` | `property_price_difference_account_id` | partial |
| Product | `product.template` | `product_template` | `property_stock_inventory` | partial |
| Product | `product.template` | `product_template` | `property_stock_production` | partial |
| Product | `product.template` | `product_template` | `responsible_id` | partial |
| Product | `product.template` | `product_template` | `service_to_purchase` | partial |
| Product | `product.template` | `product_template` | `task_template_id` | partial |
| Product | `product.template` | `product_template` | `variants_default_code` | text similarity |
| Product | `product.template` | `product_template` | `website_description` | text similarity |
| Product | `product.template` | `product_template` | `website_id` | plain |
| Product | `product.template` | `product_template` | `website_sequence` | plain |
| Product Template Attribute Exclusion | `product.template.attribute.exclusion` | `product_template_attribute_exclusion` | `product_tmpl_id` | plain |
| Product Template Attribute Line | `product.template.attribute.line` | `product_template_attribute_line` | `attribute_id` | plain |
| Product Template Attribute Line | `product.template.attribute.line` | `product_template_attribute_line` | `product_tmpl_id` | plain |
| Product Template Attribute Value | `product.template.attribute.value` | `product_template_attribute_value` | `attribute_id` | plain |
| Product Template Attribute Value | `product.template.attribute.value` | `product_template_attribute_value` | `attribute_line_id` | plain |
| Product Template Attribute Value | `product.template.attribute.value` | `product_template_attribute_value` | `product_tmpl_id` | plain |
| Link between products and their Units of Measure | `product.uom` | `product_uom` | `barcode` | partial |
| Link between products and their Units of Measure | `product.uom` | `product_uom` | `product_id` | plain |
| Link between products and their Units of Measure | `product.uom` | `product_uom` | `uom_id` | plain |
| Product Value | `product.value` | `product_value` | `move_id` | partial |
| Product Value | `product.value` | `product_value` | `product_id` | plain |
| Product Wishlist | `product.wishlist` | `product_wishlist` | `partner_id` | partial |
| Project Milestone | `project.milestone` | `project_milestone` | `project_id` | plain |
| Project Milestone | `project.milestone` | `project_milestone` | `sale_line_id` | partial |
| Project | `project.project` | `project_project` | `account_id` | plain |
| Project | `project.project` | `project_project` | `date` | plain |
| Project | `project.project` | `project_project` | `name` | text similarity |
| Project | `project.project` | `project_project` | `partner_id` | partial |
| Project | `project.project` | `project_project` | `reinvoiced_sale_order_id` | partial |
| Project | `project.project` | `project_project` | `sale_line_id` | partial |
| Project | `project.project` | `project_project` | `stage_id` | plain |
| Project Sales line, employee mapping | `project.sale.line.employee.map` | `project_sale_line_employee_map` | `project_id` | plain |
| Task | `project.task` | `project_task` | `create_date` | plain |
| Task | `project.task` | `project_task` | `date_deadline` | plain |
| Task | `project.task` | `project_task` | `date_end` | plain |
| Task | `project.task` | `project_task` | `date_last_stage_update` | plain |
| Task | `project.task` | `project_task` | `milestone_id` | partial |
| Task | `project.task` | `project_task` | `name` | text similarity |
| Task | `project.task` | `project_task` | `parent_id` | plain |
| Task | `project.task` | `project_task` | `partner_id` | partial |
| Task | `project.task` | `project_task` | `priority` | plain |
| Task | `project.task` | `project_task` | `project_id` | plain |
| Task | `project.task` | `project_task` | `recurrence_id` | partial |
| Task | `project.task` | `project_task` | `sale_line_id` | partial |
| Task | `project.task` | `project_task` | `stage_id` | plain |
| Task | `project.task` | `project_task` | `state` | plain |
| Task Stage | `project.task.type` | `project_task_type` | `user_id` | plain |
| Personal Task Stage | `project.task.stage.personal` | `project_task_user_rel` | `task_id` | plain |
| Personal Task Stage | `project.task.stage.personal` | `project_task_user_rel` | `user_id` | plain |
| Project Update | `project.update` | `project_update` | `project_id` | plain |
| Purchase Order | `purchase.order` | `purchase_order` | `company_id` | plain |
| Purchase Order | `purchase.order` | `purchase_order` | `date_approve` | plain |
| Purchase Order | `purchase.order` | `purchase_order` | `date_order` | plain |
| Purchase Order | `purchase.order` | `purchase_order` | `date_planned` | plain |
| Purchase Order | `purchase.order` | `purchase_order` | `name` | text similarity |
| Purchase Order | `purchase.order` | `purchase_order` | `partner_id` | plain |
| Purchase Order | `purchase.order` | `purchase_order` | `priority` | plain |
| Purchase Order | `purchase.order` | `purchase_order` | `purchase_group_id` | partial |
| Purchase Order | `purchase.order` | `purchase_order` | `requisition_id` | partial |
| Purchase Order | `purchase.order` | `purchase_order` | `state` | plain |
| Purchase Order | `purchase.order` | `purchase_order` | `user_id` | plain |
| Purchase Order Line | `purchase.order.line` | `purchase_order_line` | `date_planned` | plain |
| Purchase Order Line | `purchase.order.line` | `purchase_order_line` | `order_id` | plain |
| Purchase Order Line | `purchase.order.line` | `purchase_order_line` | `orderpoint_id` | partial |
| Purchase Order Line | `purchase.order.line` | `purchase_order_line` | `partner_id` | partial |
| Purchase Order Line | `purchase.order.line` | `purchase_order_line` | `product_id` | partial |
| Purchase Order Line | `purchase.order.line` | `purchase_order_line` | `sale_line_id` | partial |
| Purchase Requisition Line | `purchase.requisition.line` | `purchase_requisition_line` | `move_dest_id` | partial |
| Purchase Requisition Line | `purchase.requisition.line` | `purchase_requisition_line` | `requisition_id` | plain |
| Rating | `rating.rating` | `rating_rating` | `message_id` | plain |
| Rating | `rating.rating` | `rating_rating` | `parent_res_id` | plain |
| Rating | `rating.rating` | `rating_rating` | `parent_res_model` | plain |
| Rating | `rating.rating` | `rating_rating` | `parent_res_model_id` | plain |
| Rating | `rating.rating` | `rating_rating` | `publisher_id` | partial |
| Rating | `rating.rating` | `rating_rating` | `res_id` | plain |
| Rating | `rating.rating` | `rating_rating` | `res_model` | plain |
| Rating | `rating.rating` | `rating_rating` | `res_model_id` | plain |
| Repair Order | `repair.order` | `repair_order` | `company_id` | plain |
| Repair Order | `repair.order` | `repair_order` | `location_dest_id` | plain |
| Repair Order | `repair.order` | `repair_order` | `location_id` | plain |
| Repair Order | `repair.order` | `repair_order` | `name` | text similarity |
| Repair Order | `repair.order` | `repair_order` | `partner_id` | plain |
| Repair Order | `repair.order` | `repair_order` | `parts_location_id` | plain |
| Repair Order | `repair.order` | `repair_order` | `picking_id` | partial |
| Repair Order | `repair.order` | `repair_order` | `picking_type_id` | plain |
| Repair Order | `repair.order` | `repair_order` | `product_location_dest_id` | plain |
| Repair Order | `repair.order` | `repair_order` | `product_location_src_id` | plain |
| Repair Order | `repair.order` | `repair_order` | `recycle_location_id` | plain |
| Repair Order | `repair.order` | `repair_order` | `sale_order_id` | partial |
| Repair Order | `repair.order` | `repair_order` | `schedule_date` | plain |
| Repair Order | `repair.order` | `repair_order` | `state` | plain |
| Bank | `res.bank` | `res_bank` | `bic` | plain |
| Companies | `res.company` | `res_company` | `alias_domain_id` | partial |
| Companies | `res.company` | `res_company` | `parent_id` | plain |
| Companies | `res.company` | `res_company` | `parent_path` | plain |
| Companies | `res.company` | `res_company` | `partner_id` | plain |
| Country state | `res.country.state` | `res_country_state` | `country_id` | plain |
| Currency Rate | `res.currency.rate` | `res_currency_rate` | `currency_id` | plain |
| Currency Rate | `res.currency.rate` | `res_currency_rate` | `name` | plain |
| Device Log | `res.device.log` | `res_device_log` | `last_activity` | plain |
| Device Log | `res.device.log` | `res_device_log` | `session_identifier` | plain |
| Device Log | `res.device.log` | `res_device_log` | `user_id` | plain |
| Access Groups | `res.groups` | `res_groups` | `privilege_id` | plain |
| Privileges | `res.groups.privilege` | `res_groups_privilege` | `category_id` | plain |
| Contact | `res.partner` | `res_partner` | `activation` | partial |
| Contact | `res.partner` | `res_partner` | `assigned_partner_id` | partial |
| Contact | `res.partner` | `res_partner` | `barcode` | partial |
| Contact | `res.partner` | `res_partner` | `commercial_partner_id` | plain |
| Contact | `res.partner` | `res_partner` | `company_id` | plain |
| Contact | `res.partner` | `res_partner` | `company_registry` | partial |
| Contact | `res.partner` | `res_partner` | `complete_name` | plain |
| Contact | `res.partner` | `res_partner` | `credit_limit` | partial |
| Contact | `res.partner` | `res_partner` | `ignore_abnormal_invoice_amount` | partial |
| Contact | `res.partner` | `res_partner` | `ignore_abnormal_invoice_date` | partial |
| Contact | `res.partner` | `res_partner` | `invoice_edi_format_store` | partial |
| Contact | `res.partner` | `res_partner` | `invoice_sending_method` | partial |
| Contact | `res.partner` | `res_partner` | `is_published` | plain |
| Contact | `res.partner` | `res_partner` | `l10n_ar_afip_responsibility_type_id` | partial |
| Contact | `res.partner` | `res_partner` | `l10n_cl_sii_taxpayer_type` | partial |
| Contact | `res.partner` | `res_partner` | `l10n_hu_group_vat` | plain |
| Contact | `res.partner` | `res_partner` | `l10n_latam_identification_type_id` | partial |
| Contact | `res.partner` | `res_partner` | `l10n_vn_edi_symbol` | partial |
| Contact | `res.partner` | `res_partner` | `name` | plain |
| Contact | `res.partner` | `res_partner` | `nemhandel_verification_state` | partial |
| Contact | `res.partner` | `res_partner` | `parent_id` | plain |
| Contact | `res.partner` | `res_partner` | `peppol_verification_state` | partial |
| Contact | `res.partner` | `res_partner` | `property_account_payable_id` | partial |
| Contact | `res.partner` | `res_partner` | `property_account_position_id` | partial |
| Contact | `res.partner` | `res_partner` | `property_account_receivable_id` | partial |
| Contact | `res.partner` | `res_partner` | `property_delivery_carrier_id` | partial |
| Contact | `res.partner` | `res_partner` | `property_inbound_payment_method_line_id` | partial |
| Contact | `res.partner` | `res_partner` | `property_outbound_payment_method_line_id` | partial |
| Contact | `res.partner` | `res_partner` | `property_payment_term_id` | partial |
| Contact | `res.partner` | `res_partner` | `property_purchase_currency_id` | partial |
| Contact | `res.partner` | `res_partner` | `property_stock_customer` | partial |
| Contact | `res.partner` | `res_partner` | `property_stock_subcontractor` | partial |
| Contact | `res.partner` | `res_partner` | `property_stock_supplier` | partial |
| Contact | `res.partner` | `res_partner` | `property_supplier_payment_term_id` | partial |
| Contact | `res.partner` | `res_partner` | `receipt_reminder_email` | partial |
| Contact | `res.partner` | `res_partner` | `ref` | plain |
| Contact | `res.partner` | `res_partner` | `reminder_date_before_receipt` | partial |
| Contact | `res.partner` | `res_partner` | `specific_property_product_pricelist` | partial |
| Contact | `res.partner` | `res_partner` | `trust` | partial |
| Contact | `res.partner` | `res_partner` | `vat` | plain |
| Contact | `res.partner` | `res_partner` | `website_id` | plain |
| Bank Accounts | `res.partner.bank` | `res_partner_bank` | `partner_id` | plain |
| Partner Tags | `res.partner.category` | `res_partner_category` | `parent_id` | plain |
| Partner Tags | `res.partner.category` | `res_partner_category` | `parent_path` | plain |
| Partner Grade | `res.partner.grade` | `res_partner_grade` | `is_published` | plain |
| Partner Tags - These tags can be used on website to find customers by sector, or ... | `res.partner.tag` | `res_partner_tag` | `is_published` | plain |
| User | `res.users` | `res_users` | `create_date` | plain |
| User | `res.users` | `res_users` | `partner_id` | plain |
| User | `res.users` | `res_users` | `property_warehouse_id` | partial |
| User | `res.users` | `res_users` | `rank_id` | partial |
| Users Log | `res.users.log` | `res_users_log` | `create_uid` | plain |
| User Settings for Embedded Actions | `res.users.settings.embedded.action` | `res_users_settings_embedded_action` | `user_setting_id` | partial |
| User Settings Volumes | `res.users.settings.volumes` | `res_users_settings_volumes` | `guest_id` | plain |
| User Settings Volumes | `res.users.settings.volumes` | `res_users_settings_volumes` | `partner_id` | plain |
| User Settings Volumes | `res.users.settings.volumes` | `res_users_settings_volumes` | `user_setting_id` | plain |
| Resource Working Time | `resource.calendar` | `resource_calendar` | `company_id` | partial |
| Work Detail | `resource.calendar.attendance` | `resource_calendar_attendance` | `calendar_id` | plain |
| Work Detail | `resource.calendar.attendance` | `resource_calendar_attendance` | `dayofweek` | plain |
| Work Detail | `resource.calendar.attendance` | `resource_calendar_attendance` | `hour_from` | plain |
| Resource Time Off Detail | `resource.calendar.leaves` | `resource_calendar_leaves` | `calendar_id` | plain |
| Resource Time Off Detail | `resource.calendar.leaves` | `resource_calendar_leaves` | `resource_id` | plain |
| Resources | `resource.resource` | `resource_resource` | `user_id` | partial |
| point of sale Restaurant Order Course | `restaurant.order.course` | `restaurant_order_course` | `order_id` | plain |
| Restaurant Table | `restaurant.table` | `restaurant_table` | `floor_id` | partial |
| Sales Order | `sale.order` | `sale_order` | `campaign_id` | partial |
| Sales Order | `sale.order` | `sale_order` | `company_id` | plain |
| Sales Order | `sale.order` | `sale_order` | `create_date` | plain |
| Sales Order | `sale.order` | `sale_order` | `medium_id` | partial |
| Sales Order | `sale.order` | `sale_order` | `name` | text similarity |
| Sales Order | `sale.order` | `sale_order` | `opportunity_id` | partial |
| Sales Order | `sale.order` | `sale_order` | `partner_id` | plain |
| Sales Order | `sale.order` | `sale_order` | `partner_invoice_id` | partial |
| Sales Order | `sale.order` | `sale_order` | `partner_shipping_id` | partial |
| Sales Order | `sale.order` | `sale_order` | `project_id` | partial |
| Sales Order | `sale.order` | `sale_order` | `source_id` | partial |
| Sales Order | `sale.order` | `sale_order` | `state` | plain |
| Sales Order | `sale.order` | `sale_order` | `team_id` | plain |
| Sales Order | `sale.order` | `sale_order` | `user_id` | plain |
| Sale Order Coupon Points - Keeps track of how a sale order impacts a coupon | `sale.order.coupon.points` | `sale_order_coupon_points` | `order_id` | plain |
| Sales Order Line | `sale.order.line` | `sale_order_line` | `company_id` | plain |
| Sales Order Line | `sale.order.line` | `sale_order_line` | `event_id` | partial |
| Sales Order Line | `sale.order.line` | `sale_order_line` | `linked_line_id` | plain |
| Sales Order Line | `sale.order.line` | `sale_order_line` | `order_id` | plain |
| Sales Order Line | `sale.order.line` | `sale_order_line` | `order_partner_id` | plain |
| Sales Order Line | `sale.order.line` | `sale_order_line` | `product_id` | partial |
| Sales Order Line | `sale.order.line` | `sale_order_line` | `project_id` | plain |
| Sales Order Line | `sale.order.line` | `sale_order_line` | `task_id` | plain |
| Quotation Template | `sale.order.template` | `sale_order_template` | `journal_id` | partial |
| Quotation Template Line | `sale.order.template.line` | `sale_order_template_line` | `company_id` | plain |
| Quotation Template Line | `sale.order.template.line` | `sale_order_template_line` | `sale_order_template_id` | plain |
| Slide Question's Answer | `slide.answer` | `slide_answer` | `question_id` | plain |
| Course | `slide.channel` | `slide_channel` | `forum_id` | partial |
| Course | `slide.channel` | `slide_channel` | `is_published` | plain |
| Course | `slide.channel` | `slide_channel` | `product_id` | partial |
| Course | `slide.channel` | `slide_channel` | `website_id` | plain |
| Channel / Partners (Members) | `slide.channel.partner` | `slide_channel_partner` | `channel_id` | plain |
| Channel / Partners (Members) | `slide.channel.partner` | `slide_channel_partner` | `partner_id` | plain |
| Channel/Course Tag | `slide.channel.tag` | `slide_channel_tag` | `group_id` | plain |
| Channel/Course Tag | `slide.channel.tag` | `slide_channel_tag` | `group_sequence` | plain |
| Channel/Course Tag | `slide.channel.tag` | `slide_channel_tag` | `sequence` | plain |
| Channel/Course Groups | `slide.channel.tag.group` | `slide_channel_tag_group` | `is_published` | plain |
| Channel/Course Groups | `slide.channel.tag.group` | `slide_channel_tag_group` | `sequence` | plain |
| Embedded Slides View Counter | `slide.embed` | `slide_embed` | `slide_id` | plain |
| Content Quiz Question | `slide.question` | `slide_question` | `slide_id` | plain |
| Slides | `slide.slide` | `slide_slide` | `category_id` | partial |
| Slides | `slide.slide` | `slide_slide` | `channel_id` | plain |
| Slides | `slide.slide` | `slide_slide` | `is_published` | plain |
| Slides | `slide.slide` | `slide_slide` | `survey_id` | partial |
| Slide / Partner decorated m2m | `slide.slide.partner` | `slide_slide_partner` | `channel_id` | plain |
| Slide / Partner decorated m2m | `slide.slide.partner` | `slide_slide_partner` | `partner_id` | plain |
| Slide / Partner decorated m2m | `slide.slide.partner` | `slide_slide_partner` | `slide_id` | plain |
| Additional resource for a particular slide | `slide.slide.resource` | `slide_slide_resource` | `slide_id` | plain |
| Outgoing text message | `sms.sms` | `sms_sms` | `mail_message_id` | plain |
| text message Templates | `sms.template` | `sms_template` | `model` | plain |
| Link text message to mailing/text message tracking models | `sms.tracker` | `sms_tracker` | `mail_notification_id` | partial |
| Link text message to mailing/text message tracking models | `sms.tracker` | `sms_tracker` | `mailing_trace_id` | partial |
| Twilio Number | `sms.twilio.number` | `sms_twilio_number` | `company_id` | plain |
| Snailmail Letter | `snailmail.letter` | `snailmail_letter` | `attachment_id` | partial |
| Snailmail Letter | `snailmail.letter` | `snailmail_letter` | `message_id` | partial |
| Spreadsheet Dashboard | `spreadsheet.dashboard` | `spreadsheet_dashboard` | `dashboard_group_id` | plain |
| Stock Landed Cost | `stock.landed.cost` | `stock_landed_cost` | `account_move_id` | partial |
| Stock Landed Cost | `stock.landed.cost` | `stock_landed_cost` | `vendor_bill_id` | partial |
| Stock Landed Cost Line | `stock.landed.cost.lines` | `stock_landed_cost_lines` | `cost_id` | plain |
| Inventory Locations | `stock.location` | `stock_location` | `company_id` | plain |
| Inventory Locations | `stock.location` | `stock_location` | `location_id` | plain |
| Inventory Locations | `stock.location` | `stock_location` | `parent_path` | plain |
| Inventory Locations | `stock.location` | `stock_location` | `storage_category_id` | partial |
| Inventory Locations | `stock.location` | `stock_location` | `usage` | plain |
| Lot/Serial | `stock.lot` | `stock_lot` | `company_id` | plain |
| Lot/Serial | `stock.lot` | `stock_lot` | `name` | text similarity |
| Lot/Serial | `stock.lot` | `stock_lot` | `product_id` | plain |
| Lot/Serial | `stock.lot` | `stock_lot` | `standard_price` | partial |
| Stock Move | `stock.move` | `stock_move` | `account_move_id` | partial |
| Stock Move | `stock.move` | `stock_move` | `company_id` | plain |
| Stock Move | `stock.move` | `stock_move` | `consume_unbuild_id` | partial |
| Stock Move | `stock.move` | `stock_move` | `created_production_id` | plain |
| Stock Move | `stock.move` | `stock_move` | `date` | plain |
| Stock Move | `stock.move` | `stock_move` | `location_dest_id` | plain |
| Stock Move | `stock.move` | `stock_move` | `location_final_id` | plain |
| Stock Move | `stock.move` | `stock_move` | `location_id` | plain |
| Stock Move | `stock.move` | `stock_move` | `orderpoint_id` | plain |
| Stock Move | `stock.move` | `stock_move` | `origin_returned_move_id` | plain |
| Stock Move | `stock.move` | `stock_move` | `partner_id` | partial |
| Stock Move | `stock.move` | `stock_move` | `picking_id` | plain |
| Stock Move | `stock.move` | `stock_move` | `product_id` | plain |
| Stock Move | `stock.move` | `stock_move` | `production_id` | partial |
| Stock Move | `stock.move` | `stock_move` | `purchase_line_id` | partial |
| Stock Move | `stock.move` | `stock_move` | `raw_material_production_id` | partial |
| Stock Move | `stock.move` | `stock_move` | `repair_id` | partial |
| Stock Move | `stock.move` | `stock_move` | `repair_line_type` | plain |
| Stock Move | `stock.move` | `stock_move` | `restrict_partner_id` | partial |
| Stock Move | `stock.move` | `stock_move` | `sale_line_id` | partial |
| Stock Move | `stock.move` | `stock_move` | `scrap_id` | partial |
| Stock Move | `stock.move` | `stock_move` | `state` | plain |
| Stock Move | `stock.move` | `stock_move` | `unbuild_id` | partial |
| Stock Move | `stock.move` | `stock_move` | `workorder_id` | partial |
| Product Moves (Stock Move Line) | `stock.move.line` | `stock_move_line` | `company_id` | plain |
| Product Moves (Stock Move Line) | `stock.move.line` | `stock_move_line` | `location_dest_id` | plain |
| Product Moves (Stock Move Line) | `stock.move.line` | `stock_move_line` | `location_id` | plain |
| Product Moves (Stock Move Line) | `stock.move.line` | `stock_move_line` | `lot_id` | plain |
| Product Moves (Stock Move Line) | `stock.move.line` | `stock_move_line` | `move_id` | plain |
| Product Moves (Stock Move Line) | `stock.move.line` | `stock_move_line` | `owner_id` | partial |
| Product Moves (Stock Move Line) | `stock.move.line` | `stock_move_line` | `package_history_id` | partial |
| Product Moves (Stock Move Line) | `stock.move.line` | `stock_move_line` | `picking_id` | plain |
| Product Moves (Stock Move Line) | `stock.move.line` | `stock_move_line` | `product_id` | plain |
| Product Moves (Stock Move Line) | `stock.move.line` | `stock_move_line` | `workorder_id` | partial |
| Package | `stock.package` | `stock_package` | `company_id` | plain |
| Package | `stock.package` | `stock_package` | `location_id` | plain |
| Package | `stock.package` | `stock_package` | `name` | text similarity |
| Package | `stock.package` | `stock_package` | `package_dest_id` | partial |
| Package | `stock.package` | `stock_package` | `package_type_id` | plain |
| Package | `stock.package` | `stock_package` | `parent_package_id` | partial |
| Package | `stock.package` | `stock_package` | `parent_path` | plain |
| Stock package type | `stock.package.type` | `stock_package_type` | `company_id` | plain |
| Transfer | `stock.picking` | `stock_picking` | `backorder_id` | partial |
| Transfer | `stock.picking` | `stock_picking` | `batch_id` | plain |
| Transfer | `stock.picking` | `stock_picking` | `company_id` | plain |
| Transfer | `stock.picking` | `stock_picking` | `name` | text similarity |
| Transfer | `stock.picking` | `stock_picking` | `origin` | text similarity |
| Transfer | `stock.picking` | `stock_picking` | `owner_id` | partial |
| Transfer | `stock.picking` | `stock_picking` | `partner_id` | partial |
| Transfer | `stock.picking` | `stock_picking` | `picking_type_id` | plain |
| Transfer | `stock.picking` | `stock_picking` | `pos_order_id` | plain |
| Transfer | `stock.picking` | `stock_picking` | `pos_session_id` | plain |
| Transfer | `stock.picking` | `stock_picking` | `return_id` | partial |
| Transfer | `stock.picking` | `stock_picking` | `sale_id` | partial |
| Transfer | `stock.picking` | `stock_picking` | `scheduled_date` | plain |
| Transfer | `stock.picking` | `stock_picking` | `state` | plain |
| Batch Transfer | `stock.picking.batch` | `stock_picking_batch` | `company_id` | plain |
| Batch Transfer | `stock.picking.batch` | `stock_picking_batch` | `picking_type_id` | plain |
| Batch Transfer | `stock.picking.batch` | `stock_picking_batch` | `state` | plain |
| Picking Type | `stock.picking.type` | `stock_picking_type` | `company_id` | plain |
| Picking Type | `stock.picking.type` | `stock_picking_type` | `return_picking_type_id` | partial |
| Putaway Rule | `stock.putaway.rule` | `stock_putaway_rule` | `category_id` | partial |
| Putaway Rule | `stock.putaway.rule` | `stock_putaway_rule` | `company_id` | plain |
| Putaway Rule | `stock.putaway.rule` | `stock_putaway_rule` | `location_in_id` | plain |
| Putaway Rule | `stock.putaway.rule` | `stock_putaway_rule` | `product_id` | partial |
| Quants | `stock.quant` | `stock_quant` | `location_id` | plain |
| Quants | `stock.quant` | `stock_quant` | `lot_id` | plain |
| Quants | `stock.quant` | `stock_quant` | `owner_id` | partial |
| Quants | `stock.quant` | `stock_quant` | `package_id` | plain |
| Quants | `stock.quant` | `stock_quant` | `product_id` | plain |
| Inventory Routes | `stock.route` | `stock_route` | `company_id` | plain |
| Inventory Routes | `stock.route` | `stock_route` | `supplied_wh_id` | partial |
| Stock Rule | `stock.rule` | `stock_rule` | `action` | plain |
| Stock Rule | `stock.rule` | `stock_rule` | `company_id` | plain |
| Stock Rule | `stock.rule` | `stock_rule` | `location_dest_id` | plain |
| Stock Rule | `stock.rule` | `stock_rule` | `location_src_id` | plain |
| Stock Rule | `stock.rule` | `stock_rule` | `route_id` | plain |
| Stock Rule | `stock.rule` | `stock_rule` | `warehouse_id` | plain |
| Scrap | `stock.scrap` | `stock_scrap` | `production_id` | partial |
| Scrap | `stock.scrap` | `stock_scrap` | `workorder_id` | partial |
| Storage Category Capacity | `stock.storage.category.capacity` | `stock_storage_category_capacity` | `package_type_id` | partial |
| Storage Category Capacity | `stock.storage.category.capacity` | `stock_storage_category_capacity` | `product_id` | partial |
| Storage Category Capacity | `stock.storage.category.capacity` | `stock_storage_category_capacity` | `storage_category_id` | plain |
| Valuation Adjustment Lines | `stock.valuation.adjustment.lines` | `stock_valuation_adjustment_lines` | `cost_id` | plain |
| Warehouse | `stock.warehouse` | `stock_warehouse` | `view_location_id` | plain |
| Minimum Inventory Rule | `stock.warehouse.orderpoint` | `stock_warehouse_orderpoint` | `company_id` | plain |
| Minimum Inventory Rule | `stock.warehouse.orderpoint` | `stock_warehouse_orderpoint` | `location_id` | plain |
| Minimum Inventory Rule | `stock.warehouse.orderpoint` | `stock_warehouse_orderpoint` | `product_id` | plain |
| Minimum Inventory Rule | `stock.warehouse.orderpoint` | `stock_warehouse_orderpoint` | `warehouse_id` | plain |
| Survey Invitation Wizard | `survey.invite` | `survey_invite` | `author_id` | plain |
| Survey Question | `survey.question` | `survey_question` | `survey_id` | partial |
| Survey Label | `survey.question.answer` | `survey_question_answer` | `matrix_question_id` | partial |
| Survey Label | `survey.question.answer` | `survey_question_answer` | `question_id` | partial |
| Survey | `survey.survey` | `survey_survey` | `certification_badge_id` | partial |
| Survey | `survey.survey` | `survey_survey` | `team_id` | partial |
| Survey User Input | `survey.user_input` | `survey_user_input` | `applicant_id` | partial |
| Survey User Input | `survey.user_input` | `survey_user_input` | `partner_id` | partial |
| Survey User Input | `survey.user_input` | `survey_user_input` | `slide_partner_id` | partial |
| Survey User Input | `survey.user_input` | `survey_user_input` | `survey_id` | plain |
| Survey User Input Line | `survey.user_input.line` | `survey_user_input_line` | `question_id` | plain |
| Survey User Input Line | `survey.user_input.line` | `survey_user_input_line` | `user_input_id` | plain |
| Website Theme Menu | `theme.website.menu` | `theme_website_menu` | `page_id` | partial |
| Website Theme Menu | `theme.website.menu` | `theme_website_menu` | `parent_id` | plain |
| Website Theme Page | `theme.website.page` | `theme_website_page` | `view_id` | plain |
| Product Unit of Measure | `uom.uom` | `uom_uom` | `parent_path` | plain |
| Product Unit of Measure | `uom.uom` | `uom_uom` | `relative_uom_id` | partial |
| Tour's step | `web_tour.tour.step` | `web_tour_tour_step` | `tour_id` | plain |
| Website | `website` | `website` | `salesteam_id` | partial |
| Website Checkout Step | `website.checkout.step` | `website_checkout_step` | `is_published` | plain |
| Website Checkout Step | `website.checkout.step` | `website_checkout_step` | `website_id` | plain |
| Model Page | `website.controller.page` | `website_controller_page` | `is_published` | plain |
| Model Page | `website.controller.page` | `website_controller_page` | `view_id` | plain |
| Model Page | `website.controller.page` | `website_controller_page` | `website_id` | plain |
| Website Event Menu | `website.event.menu` | `website_event_menu` | `event_id` | partial |
| Website Menu | `website.menu` | `website_menu` | `controller_page_id` | partial |
| Website Menu | `website.menu` | `website_menu` | `page_id` | partial |
| Website Menu | `website.menu` | `website_menu` | `parent_id` | plain |
| Website Menu | `website.menu` | `website_menu` | `parent_path` | plain |
| Website Menu | `website.menu` | `website_menu` | `theme_template_id` | partial |
| Page | `website.page` | `website_page` | `is_published` | plain |
| Page | `website.page` | `website_page` | `theme_template_id` | partial |
| Page | `website.page` | `website_page` | `view_id` | plain |
| Page | `website.page` | `website_page` | `website_id` | plain |
| Website rewrite | `website.rewrite` | `website_rewrite` | `url_from` | plain |
| Website rewrite | `website.rewrite` | `website_rewrite` | `website_id` | plain |
| Electronic Commerce Extra Info Shown on product page | `website.sale.extra.field` | `website_sale_extra_field` | `website_id` | partial |
| Website Snippet Filter | `website.snippet.filter` | `website_snippet_filter` | `is_published` | plain |
| Website Snippet Filter | `website.snippet.filter` | `website_snippet_filter` | `website_id` | plain |
| Visited Pages | `website.track` | `website_track` | `page_id` | plain |
| Visited Pages | `website.track` | `website_track` | `product_id` | partial |
| Visited Pages | `website.track` | `website_track` | `url` | plain |
| Visited Pages | `website.track` | `website_track` | `visitor_id` | plain |
| Website Visitor | `website.visitor` | `website_visitor` | `livechat_operator_id` | partial |
| Website Visitor | `website.visitor` | `website_visitor` | `partner_id` | partial |

### 10.3 Text-similarity indexes

The 37 columns indexed for text similarity are the ones that carry user-visible names and references used in contains searches. Three behaviours depend on them:

1. A contains or case-insensitive contains search on such a column is rewritten so that the index can be used, by pre-filtering with a similarity condition on the pattern before applying the exact condition.
2. On a per-language column, the pre-filter runs against the concatenation of all the language values, then the exact condition runs against the value of the reader's language; the pre-filter can therefore match a row whose translation in another language contains the pattern, and the exact condition then discards it.
3. A pattern shorter than three characters cannot use a similarity index; the search falls back to a sequential scan, which a replacement should expect for short patterns.

## 11. Constraints

### 11.1 Not-null constraints

A column carries a not-null constraint exactly when its field is required. The constraint is added after the existing rows have been filled with the default value, or recomputed for a derived field, and it is dropped as soon as the field stops being required. A required field whose value is missing at write time is refused by the entity-level validation before the store is reached, with the message "The following fields are invalid:" followed by the list of field labels.

### 11.2 Declared table constraints and indexes

An entity may declare table objects of three kinds:

| Kind | Meaning | Violation |
|---|---|---|
| Table constraint | A uniqueness statement over one or more columns, or a check statement over an expression, applied to every row | The declared message is shown to the user; when no message is declared, a generic message naming the constraint is shown. |
| Unique index | A uniqueness statement over one or more columns or expressions, optionally restricted by a condition, so that uniqueness applies only to the rows that satisfy the condition | The declared message is shown to the user. |
| Index | A non-unique index, optionally restricted by a condition, declared for query performance | Not applicable. |

A declared object is applied after the columns exist, inside the installation transaction. When existing rows violate it, the installation fails and the message of the constraint is reported.

The entities declare 291 table constraints, 34 unique indexes and 65 indexes. The complete catalogue follows. Every value of the last column that is enclosed in quotation marks is the message reproduced exactly as the system shows it.

| Entity | Transport name | Table | Object | Kind | Definition | Message shown when violated |
|---|---|---|---|---|---|---|
| ARCA Responsibility Type | `l10n_ar.afip.responsibility.type` | `l10n_ar_afip_responsibility_type` | `_code_uniq` | table constraint | `unique(code)` | "Code must be unique!" |
| ARCA Responsibility Type | `l10n_ar.afip.responsibility.type` | `l10n_ar_afip_responsibility_type` | `_name_uniq` | table constraint | `unique(name)` | "Name must be unique!" |
| Access Groups | `res.groups` | `res_groups` | `_check_api_key_duration` | table constraint | `CHECK(api_key_duration >= 0)` | "The api key duration cannot be a negative value." |
| Access Groups | `res.groups` | `res_groups` | `_name_uniq` | table constraint | `UNIQUE (privilege_id, name)` | "The name of the group must be unique within a group privilege!" |
| Account Group | `account.group` | `account_group` | `_check_length_prefix` | table constraint | `CHECK(char_length(COALESCE(code_prefix_start, '')) = char_length(COALESCE(code_prefix_end, '')))` | "The length of the starting and the ending code prefix must be the same" |
| Account Journal Group | `account.journal.group` | `account_journal_group` | `_uniq_name` | table constraint | `unique(company_id, name)` | "A Ledger group name must be unique per company." |
| Account Lock Exception | `account.lock_exception` | `account_lock_exception` | `_company_id_end_datetime_idx` | index | `(company_id, user_id, end_datetime) WHERE active IS TRUE` | the default message of the constraint kind |
| Account Tag | `account.account.tag` | `account_account_tag` | `_name_uniq` | table constraint | `unique(name, applicability, country_id)` | "A tag with the same name and applicability already exists in this country." |
| Account electronic data interchange proxy user | `account_edi_proxy_client.user` | `account_edi_proxy_client_user` | `_peppol_proxy_types_conflict` | table constraint | `EXCLUDE (                 company_id WITH =,                 edi_mode WITH =             )             WHERE (active IS TRUE AND proxy_type IN ('peppol', 'pdp'))` | "You can not have both a Peppol and a PDP proxy user" |
| Account electronic data interchange proxy user | `account_edi_proxy_client.user` | `account_edi_proxy_client_user` | `_unique_active_company_proxy` | unique index | `(company_id, proxy_type, edi_mode) WHERE (active IS TRUE)` | "This company has an active user already created for this EDI type" |
| Account electronic data interchange proxy user | `account_edi_proxy_client.user` | `account_edi_proxy_client_user` | `_unique_id_client` | table constraint | `unique(id_client)` | "This id_client is already used on another user." |
| Account electronic data interchange proxy user | `account_edi_proxy_client.user` | `account_edi_proxy_client_user` | `_unique_identification_l10n_gr_edi` | unique index | `(edi_identification, edi_mode) WHERE (active IS TRUE AND proxy_type = 'l10n_gr_edi')` | "This EDI identification is already assigned to an active user." |
| Account electronic data interchange proxy user | `account_edi_proxy_client.user` | `account_edi_proxy_client_user` | `_unique_identification_l10n_it_edi` | unique index | `(edi_identification, proxy_type, edi_mode) WHERE (active AND proxy_type = 'l10n_it_edi')` | "This edi identification is already assigned to an active user" |
| Account electronic data interchange proxy user | `account_edi_proxy_client.user` | `account_edi_proxy_client_user` | `_unique_identification_l10n_my_edi` | unique index | `(edi_identification, proxy_type, edi_mode) WHERE (active AND proxy_type = 'l10n_my_edi')` | "This edi identification is already assigned to an active user" |
| Account move line to be created when posting work in progress account move | `mrp.account.wip.accounting.line` | `mrp_account_wip_accounting_line` | `_check_debit_credit` | table constraint | `CHECK ( debit = 0 OR credit = 0 )` | "A single line cannot be both credit and debit." |
| Account payment check | `l10n_latam.check` | `l10n_latam_check` | `_unique` | unique index | `(name, payment_method_line_id) WHERE outstanding_line_id IS NOT NULL` | the default message of the constraint kind |
| Accounting Report Expression | `account.report.expression` | `account_report_expression` | `_domain_engine_subformula_required` | table constraint | `CHECK(engine != 'domain' OR subformula IS NOT NULL)` | "Expressions using 'domain' engine should all have a subformula." |
| Accounting Report Expression | `account.report.expression` | `account_report_expression` | `_line_label_uniq` | table constraint | `UNIQUE(report_line_id,label)` | "The expression label must be unique per report line." |
| Accounting Report Line | `account.report.line` | `account_report_line` | `_code_uniq` | table constraint | `unique (report_id, code)` | "A report line with the same code already exists." |
| Accounts Mapping of Fiscal Position | `account.fiscal.position.account` | `account_fiscal_position_account` | `_account_src_dest_uniq` | table constraint | `unique (position_id,account_src_id,account_dest_id)` | "An account fiscal position could be defined only one time on same accounts." |
| Accrual Plan Level | `hr.leave.accrual.level` | `hr_leave_accrual_level` | `_added_value_greater_than_zero` | table constraint | `CHECK(added_value > 0)` | "You must give a rate greater than 0 in accrual plan levels." |
| Accrual Plan Level | `hr.leave.accrual.level` | `hr_leave_accrual_level` | `_start_count_check` | table constraint | `CHECK((start_count > 0 AND milestone_date = 'after') OR (start_count = 0 AND milestone_date = 'creation'))` | "You can not start an accrual in the past." |
| Accrual Plan Level | `hr.leave.accrual.level` | `hr_leave_accrual_level` | `_valid_accrual_validity_value` | table constraint | `CHECK(accrual_validity IS NOT TRUE OR COALESCE(accrual_validity_count, 0) > 0)` | "You cannot have an accrual validity time set to 0." |
| Accrual Plan Level | `hr.leave.accrual.level` | `hr_leave_accrual_level` | `_valid_postpone_max_days_value` | table constraint | `CHECK(action_with_unused_accruals <> 'all' OR carryover_options <> 'limited' OR COALESCE(postpone_max_days, 0) > 0)` | "You cannot have a maximum quantity to carryover set to 0." |
| Accrual Plan Level | `hr.leave.accrual.level` | `hr_leave_accrual_level` | `_valid_yearly_cap_value` | table constraint | `CHECK(cap_accrued_time_yearly IS NOT TRUE OR COALESCE(maximum_leave_yearly, 0) > 0)` | "You cannot have a cap on yearly accrued time without setting a maximum amount." |
| Action Window View | `ir.actions.act_window.view` | `ir_act_window_view` | `_unique_mode_per_action` | unique index | `(act_window_id, view_mode)` | the default message of the constraint kind |
| Actions | `ir.actions.actions` | `ir_actions` | `_path_unique` | table constraint | `unique(path)` | "Path to show in the URL must be unique! Please choose another one." |
| Activity | `mail.activity` | `mail_activity` | `_check_res_id_is_set_if_model` | table constraint | `CHECK(             (COALESCE(res_model, '') <> '' AND (res_id IS NOT NULL AND res_id != 0)) OR             (COALESCE(res_model, '') = '' AND (res_id IS NULL OR res_id = 0))         )` | "Activities have to be linked to records with a not null res_id." |
| Activity | `mail.activity` | `mail_activity` | `_check_user_id_is_set_if_model` | table constraint | `CHECK(             (COALESCE(res_model, '') <> '' OR user_id IS NOT NULL)         )` | "Activities must be assigned if not attached to a document." |
| Add tag for the workcenter | `mrp.workcenter.tag` | `mrp_workcenter_tag` | `_tag_name_unique` | table constraint | `unique(name)` | "The tag name must be unique." |
| Additional resource for a particular slide | `slide.slide.resource` | `slide_slide_resource` | `_check_file_type` | table constraint | `CHECK (resource_type != 'file' OR link IS NULL)` | "A resource of type file cannot contain a link." |
| Additional resource for a particular slide | `slide.slide.resource` | `slide_slide_resource` | `_check_url` | table constraint | `CHECK (resource_type != 'url' OR link IS NOT NULL)` | "A resource of type url must contain a link." |
| Analytic Line | `account.analytic.line` | `account_analytic_line` | `_timeoff_timesheet_idx` | index | `(task_id) WHERE (global_leave_id IS NOT NULL OR holiday_id IS NOT NULL) AND project_id IS NOT NULL` | the default message of the constraint kind |
| Applicant | `hr.applicant` | `hr_applicant` | `_job_id_stage_id_idx` | index | `(job_id, stage_id) WHERE active IS TRUE` | the default message of the constraint kind |
| Applicant Degree | `hr.recruitment.degree` | `hr_recruitment_degree` | `_name_uniq` | table constraint | `unique (name)` | "The name of the Degree of Recruitment must be unique!" |
| Applicant Degree | `hr.recruitment.degree` | `hr_recruitment_degree` | `_score_range` | table constraint | `check(score >= 0 and score <= 1)` | "Score should be between 0 and 100%" |
| Attachment | `ir.attachment` | `ir_attachment` | `_res_idx` | index | `(res_model, res_id)` | the default message of the constraint kind |
| Attendance Overtime Line | `hr.attendance.overtime.line` | `hr_attendance_overtime_line` | `_overtime_start_before_end` | table constraint | `CHECK (time_stop > time_start)` | "Starting time should be before end time." |
| Bank Accounts | `res.partner.bank` | `res_partner_bank` | `_unique_number` | table constraint | `unique(sanitized_acc_number, partner_id)` | "The combination Account Number/Partner must be unique." |
| Bank Statement | `account.bank.statement` | `account_bank_statement` | `_first_line_index_idx` | index | `(journal_id, first_line_index)` | the default message of the constraint kind |
| Bank Statement | `account.bank.statement` | `account_bank_statement` | `_journal_id_date_desc_id_desc_idx` | index | `(journal_id, date DESC, id DESC)` | the default message of the constraint kind |
| Bank Statement Line | `account.bank.statement.line` | `account_bank_statement_line` | `_main_idx` | index | `(journal_id, company_id, internal_index)` | the default message of the constraint kind |
| Bank Statement Line | `account.bank.statement.line` | `account_bank_statement_line` | `_orphan_idx` | index | `(journal_id, company_id, internal_index) WHERE statement_id IS NULL` | the default message of the constraint kind |
| Bank Statement Line | `account.bank.statement.line` | `account_bank_statement_line` | `_unreconciled_idx` | index | `(journal_id, company_id, internal_index) WHERE is_reconciled IS NOT TRUE` | the default message of the constraint kind |
| Bill of Material | `mrp.bom` | `mrp_bom` | `_qty_positive` | table constraint | `check (product_qty > 0)` | "The quantity to produce must be positive!" |
| Bill of Material Line | `mrp.bom.line` | `mrp_bom_line` | `_bom_qty_zero` | table constraint | `CHECK (product_qty>=0)` | "All product quantities must be greater or equal to 0. Lines with 0 quantities can be used as optional lines.  You should install the mrp_byproduct module if you want to manage extra products on BoMs!" |
| Blog Tag | `blog.tag` | `blog_tag` | `_name_uniq` | table constraint | `unique (name)` | "Tag name already exists!" |
| Blog Tag Category | `blog.tag.category` | `blog_tag_category` | `_name_uniq` | table constraint | `unique (name)` | "Tag category already exists!" |
| Brazilian city zip range | `l10n_br.zip.range` | `l10n_br_zip_range` | `_uniq_end` | table constraint | `unique("end")` | "The "to" zip must be unique." |
| Brazilian city zip range | `l10n_br.zip.range` | `l10n_br_zip_range` | `_uniq_start` | table constraint | `unique(start)` | "The "from" zip must be unique" |
| CPV Code | `l10n_ro.cpv.code` | `l10n_ro_cpv_code` | `_code_uniq` | table constraint | `unique (code)` | "Code must be unique!" |
| Calendar Filters | `calendar.filters` | `calendar_filters` | `_user_id_partner_id_unique` | table constraint | `UNIQUE(user_id, partner_id)` | "A user cannot have the same contact twice." |
| Category of applicant | `hr.applicant.category` | `hr_applicant_category` | `_name_uniq` | table constraint | `unique (name)` | "Tag name already exists!" |
| Category of the model | `fleet.vehicle.model.category` | `fleet_vehicle_model_category` | `_name_uniq` | table constraint | `UNIQUE (name)` | "Category name must be unique" |
| Channel / Partners (Members) | `slide.channel.partner` | `slide_channel_partner` | `_channel_partner_uniq` | table constraint | `unique(channel_id, partner_id)` | "A partner membership to a channel must be unique!" |
| Channel / Partners (Members) | `slide.channel.partner` | `slide_channel_partner` | `_check_completion` | table constraint | `check(completion >= 0 and completion <= 100)` | "The completion of a channel is a percentage and should be between 0% and 100." |
| Channel Member | `discuss.channel.member` | `discuss_channel_member` | `_guest_unique` | unique index | `(channel_id, guest_id) WHERE guest_id IS NOT NULL` | the default message of the constraint kind |
| Channel Member | `discuss.channel.member` | `discuss_channel_member` | `_partner_or_guest_exists` | table constraint | `CHECK((partner_id IS NOT NULL AND guest_id IS NULL) OR (partner_id IS NULL AND guest_id IS NOT NULL))` | "A channel member must be a partner or a guest." |
| Channel Member | `discuss.channel.member` | `discuss_channel_member` | `_partner_unique` | unique index | `(channel_id, partner_id) WHERE partner_id IS NOT NULL` | the default message of the constraint kind |
| Channel Member | `discuss.channel.member` | `discuss_channel_member` | `_seen_message_id_idx` | index | `(channel_id, partner_id, seen_message_id)` | the default message of the constraint kind |
| Chatbot Message | `chatbot.message` | `chatbot_message` | `_channel_id_user_raw_script_answer_id_idx` | index | `(discuss_channel_id, user_raw_script_answer_id) WHERE user_raw_script_answer_id IS NOT NULL` | the default message of the constraint kind |
| Chatbot Message | `chatbot.message` | `chatbot_message` | `_unique_mail_message_id` | table constraint | `unique (mail_message_id)` | "A mail.message can only be linked to a single chatbot message" |
| Collaborators in project shared | `project.collaborator` | `project_collaborator` | `_unique_collaborator` | table constraint | `UNIQUE(project_id, partner_id)` | "A collaborator cannot be selected more than once in the project sharing access. Please remove duplicate(s) and try again." |
| Companies | `res.company` | `res_company` | `_check_quotation_validity_days` | table constraint | `CHECK(quotation_validity_days >= 0)` | "You cannot set a negative number for the default quotation validity. Leave empty (or 0) to disable the automatic expiration of quotations." |
| Companies | `res.company` | `res_company` | `_name_uniq` | table constraint | `unique (name)` | "The company name must be unique!" |
| Contact | `res.partner` | `res_partner` | `_check_name` | table constraint | `CHECK( (type='contact' AND name IS NOT NULL) or (type!='contact') )` | "Contacts require a name" |
| Contact | `res.partner` | `res_partner` | `_l10n_it_codice_fiscale` | table constraint | `CHECK(l10n_it_codice_fiscale IS NULL OR l10n_it_codice_fiscale = '' OR LENGTH(l10n_it_codice_fiscale) >= 11)` | "Codice fiscale must have between 11 and 16 characters." |
| Contact | `res.partner` | `res_partner` | `_l10n_it_pa_index` | table constraint | `CHECK(l10n_it_pa_index IS NULL OR l10n_it_pa_index = '' OR LENGTH(l10n_it_pa_index) >= 6)` | "Destination Code (SDI) must have between 6 and 7 characters." |
| Country | `res.country` | `res_country` | `_code_uniq` | table constraint | `unique (code)` | "The code of the country must be unique!" |
| Country | `res.country` | `res_country` | `_name_uniq` | table constraint | `unique (name)` | "The name of the country must be unique!" |
| Country Group | `res.country.group` | `res_country_group` | `_check_code_uniq` | table constraint | `unique(code)` | "The country group code must be unique!" |
| Country state | `res.country.state` | `res_country_state` | `_name_code_uniq` | table constraint | `unique(country_id, code)` | "The code of the state must be unique by country!" |
| Course | `slide.channel` | `slide_channel` | `_check_enroll` | table constraint | `CHECK(visibility != 'members' OR enroll = 'invite')` | "The Enroll Policy should be set to 'On Invitation' when visibility is set to 'Course Attendees'" |
| Course | `slide.channel` | `slide_channel` | `_forum_uniq` | table constraint | `unique (forum_id)` | "Only one course per forum!" |
| Course | `slide.channel` | `slide_channel` | `_product_id_check` | table constraint | `CHECK( enroll!='payment' OR product_id IS NOT NULL )` | "Product is required for on payment channels." |
| Currency | `res.currency` | `res_currency` | `_rounding_gt_zero` | table constraint | `CHECK (rounding>0)` | "The rounding factor must be greater than 0!" |
| Currency | `res.currency` | `res_currency` | `_unique_name` | table constraint | `unique (name)` | "The currency code must be unique!" |
| Currency Rate | `res.currency.rate` | `res_currency_rate` | `_currency_rate_check` | table constraint | `CHECK (rate>0)` | "The currency rate must be strictly positive." |
| Currency Rate | `res.currency.rate` | `res_currency_rate` | `_unique_name_per_day` | table constraint | `unique (name,currency_id,company_id)` | "Only one currency rate per day allowed!" |
| Custom View | `ir.ui.view.custom` | `ir_ui_view_custom` | `_user_id_ref_id` | index | `(user_id, ref_id)` | the default message of the constraint kind |
| Decimal Precision | `decimal.precision` | `decimal_precision` | `_name_uniq` | table constraint | `unique (name)` | "Only one value can be defined for each given usage!" |
| Declaration of Intent | `l10n_it_edi_doi.declaration_of_intent` | `l10n_it_edi_doi_declaration_of_intent` | `_protocol_number_unique` | table constraint | `unique(protocol_number_part1, protocol_number_part2)` | "The Protocol Number of a Declaration of Intent must be unique! Please choose another one." |
| Declaration of Intent | `l10n_it_edi_doi.declaration_of_intent` | `l10n_it_edi_doi_declaration_of_intent` | `_threshold_positive` | table constraint | `CHECK(threshold > 0)` | "The Threshold of a Declaration of Intent must be positive." |
| Delivery Zip Prefix | `delivery.zip.prefix` | `delivery_zip_prefix` | `_name_uniq` | table constraint | `unique (name)` | "Prefix already exists!" |
| Device Log | `res.device.log` | `res_device_log` | `_composite_idx` | index | `(user_id, session_identifier, platform, browser, last_activity, id) WHERE revoked IS NOT TRUE` | the default message of the constraint kind |
| Device Log | `res.device.log` | `res_device_log` | `_revoked_idx` | index | `(revoked) WHERE revoked IS NOT TRUE` | the default message of the constraint kind |
| Discussion Channel | `discuss.channel` | `discuss_channel` | `_from_message_id_unique` | table constraint | `UNIQUE(from_message_id)` | "Messages can only be linked to one sub-channel" |
| Discussion Channel | `discuss.channel` | `discuss_channel` | `_group_public_id_check` | table constraint | `CHECK (channel_type = 'channel' OR group_public_id IS NULL)` | "Group authorization and group auto-subscription are only supported on channels." |
| Discussion Channel | `discuss.channel` | `discuss_channel` | `_has_crm_lead_index` | index | `(has_crm_lead) WHERE has_crm_lead IS TRUE` | the default message of the constraint kind |
| Discussion Channel | `discuss.channel` | `discuss_channel` | `_livechat_channel_type_create_date_idx` | index | `(channel_type, create_date) WHERE channel_type = 'livechat'` | the default message of the constraint kind |
| Discussion Channel | `discuss.channel` | `discuss_channel` | `_livechat_end_dt_idx` | index | `(livechat_end_dt) WHERE livechat_end_dt IS NULL` | the default message of the constraint kind |
| Discussion Channel | `discuss.channel` | `discuss_channel` | `_livechat_end_dt_status_constraint` | table constraint | `CHECK(livechat_end_dt IS NULL or livechat_status IS NULL)` | "Closed Live Chat session should not have a status." |
| Discussion Channel | `discuss.channel` | `discuss_channel` | `_livechat_failure_idx` | index | `(livechat_failure) WHERE livechat_failure IN ('no_answer', 'no_agent')` | the default message of the constraint kind |
| Discussion Channel | `discuss.channel` | `discuss_channel` | `_livechat_is_escalated_idx` | index | `(livechat_is_escalated) WHERE livechat_is_escalated IS TRUE` | the default message of the constraint kind |
| Discussion Channel | `discuss.channel` | `discuss_channel` | `_livechat_operator_id` | table constraint | `CHECK((channel_type = 'livechat' and livechat_operator_id is not null) or (channel_type != 'livechat'))` | "Livechat Operator ID is required for a channel of type livechat." |
| Discussion Channel | `discuss.channel` | `discuss_channel` | `_uuid_unique` | table constraint | `UNIQUE(uuid)` | "The channel UUID must be unique" |
| Document Followers | `mail.followers` | `mail_followers` | `_mail_followers_res_partner_res_model_id_uniq` | table constraint | `unique(res_model,res_id,partner_id)` | "Error, a partner cannot follow twice the same object." |
| Electronic Document for an account.move | `account.edi.document` | `account_edi_document` | `_unique_edi_document_by_move_by_format` | table constraint | `UNIQUE(edi_format_id, move_id)` | "Only one edi document by move by format" |
| Email Aliases | `mail.alias` | `mail_alias` | `_name_domain_unique` | unique index | `(alias_name, COALESCE(alias_domain_id, 0))` | the default message of the constraint kind |
| Email Domain | `mail.alias.domain` | `mail_alias_domain` | `_bounce_email_uniques` | table constraint | `UNIQUE(bounce_alias, name)` | "Bounce emails should be unique" |
| Email Domain | `mail.alias.domain` | `mail_alias_domain` | `_catchall_email_uniques` | table constraint | `UNIQUE(catchall_alias, name)` | "Catchall emails should be unique" |
| Embedded Actions | `ir.embedded.actions` | `ir_embedded_actions` | `_check_only_one_action_defined` | table constraint | `CHECK(             (action_id IS NOT NULL AND python_method IS NULL)             OR (action_id IS NULL AND python_method IS NOT NULL)         )` | "Constraint to ensure that either an XML action or a python_method is defined, but not both." |
| Embedded Actions | `ir.embedded.actions` | `ir_embedded_actions` | `_check_python_method_requires_name` | table constraint | `CHECK(NOT (python_method IS NOT NULL AND name IS NULL))` | "Constraint to ensure that if a python_method is defined, then the name must also be defined." |
| Employee | `hr.employee` | `hr_employee` | `_barcode_uniq` | table constraint | `unique (barcode)` | "The Badge ID must be unique, this one is already assigned to another employee." |
| Employee | `hr.employee` | `hr_employee` | `_user_uniq` | table constraint | `unique (user_id, company_id)` | "A user cannot be linked to multiple employees in the same company." |
| Employee Category | `hr.employee.category` | `hr_employee_category` | `_name_uniq` | table constraint | `unique (name)` | "Tag name already exists!" |
| Employee Location | `hr.employee.location` | `hr_employee_location` | `_uniq_exceptional_per_day` | table constraint | `unique(employee_id, date)` | "Only one default work location and one exceptional work location per day per employee." |
| Event Booth Registration | `event.booth.registration` | `event_booth_registration` | `_unique_registration` | table constraint | `unique(sale_order_line_id, event_booth_id)` | "There can be only one registration for a booth by sale order line" |
| Event Lead Request | `event.lead.request` | `event_lead_request` | `_uniq_event` | table constraint | `unique(event_id)` | "You can only have one generation request per event at a time." |
| Event Meeting Type | `calendar.event.type` | `calendar_event_type` | `_name_uniq` | table constraint | `unique (name)` | "Tag name already exists!" |
| Event Question | `event.question` | `event_question` | `_check_default_question_is_reusable` | table constraint | `CHECK(is_default IS DISTINCT FROM TRUE OR is_reusable IS TRUE)` | "A default question must be reusable." |
| Event Recurrence Rule | `calendar.recurrence` | `calendar_recurrence` | `_month_day` | table constraint | `{'expression': '"CHECK (\\n        rrule_type != \'monthly\'\\n        OR month_by != \'day\'\\n        OR day >= 1 AND day <= 31\\n        OR weekday IN %s AND byday IN %s)" % (tuple((wd[0] for wd in WEEKDAY_SELECTION)), tuple((bd[0] for bd in BYDAY_SELECTION)))'}` | "The day must be between 1 and 31" |
| Event Registration | `event.registration` | `event_registration` | `_barcode_event_uniq` | table constraint | `unique(barcode)` | "Barcode should be unique" |
| Event Registration Answer | `event.registration.answer` | `event_registration_answer` | `_value_check` | table constraint | `CHECK(value_answer_id IS NOT NULL OR COALESCE(value_text_box, '') <> '')` | "There must be a suggested value or a text value." |
| Event Track Tag | `event.track.tag` | `event_track_tag` | `_name_uniq` | table constraint | `unique (name)` | "Tag name already exists!" |
| Fields | `ir.model.fields` | `ir_model_fields` | `_name_manual_field` | table constraint | `CHECK (state != 'manual' OR name LIKE 'x\_%')` | "Custom fields must have a name that starts with 'x_'!" |
| Fields | `ir.model.fields` | `ir_model_fields` | `_name_unique` | table constraint | `UNIQUE(model, name)` | "Field names must be unique per model." |
| Fields | `ir.model.fields` | `ir_model_fields` | `_size_gt_zero` | table constraint | `CHECK (size>=0)` | "Size of the field cannot be negative." |
| Fields Selection | `ir.model.fields.selection` | `ir_model_fields_selection` | `_selection_field_uniq` | table constraint | `UNIQUE (field_id, value)` | "Selections values must be unique per field" |
| Filters | `ir.filters` | `ir_filters` | `_check_res_id_only_when_embedded_action` | table constraint | `CHECK(NOT (embedded_parent_res_id IS NOT NULL AND embedded_action_id IS NULL))` | "Constraint to ensure that the embedded_parent_res_id is only defined when a top_action_id is defined." |
| Filters | `ir.filters` | `ir_filters` | `_check_sort_json` | table constraint | `CHECK(sort IS NULL OR jsonb_typeof(sort::jsonb) = 'array')` | "Invalid sort definition" |
| Filters | `ir.filters` | `ir_filters` | `_get_filters_index` | index | `(model_id, action_id, embedded_action_id, embedded_parent_res_id)` | the default message of the constraint kind |
| Form fields of inside quotation documents. | `sale.pdf.form.field` | `sale_pdf_form_field` | `_unique_name_per_doc_type` | table constraint | `UNIQUE(name, document_type)` | "Form field name must be unique for a given document type." |
| Forum Tag | `forum.tag` | `forum_tag` | `_name_uniq` | table constraint | `unique (name, forum_id)` | "Tag name already exists!" |
| GİB Plate numbers | `l10n_tr.nilvera.trailer.plate` | `l10n_tr_nilvera_trailer_plate` | `_name_uniq` | table constraint | `unique(name,plate_number_type)` | "A Plate Number with that type already exists." |
| Indian permanent account number Entity | `l10n_in.pan.entity` | `l10n_in_pan_entity` | `_name_uniq` | table constraint | `unique (name)` | "A PAN Entity with same PAN Number already exists." |
| Indian port code | `l10n_in.port.code` | `l10n_in_port_code` | `_code_uniq` | table constraint | `unique (code)` | "The Port Code must be unique!" |
| Inventory Locations | `stock.location` | `stock_location` | `_barcode_company_uniq` | table constraint | `unique (barcode,company_id)` | "The barcode for a location must be unique per company!" |
| Inventory Locations | `stock.location` | `stock_location` | `_inventory_freq_nonneg` | table constraint | `check(cyclic_inventory_frequency >= 0)` | "The inventory frequency (days) for a location must be non-negative" |
| Inventory Locations | `stock.location` | `stock_location` | `_parent_path_id_idx` | index | `(parent_path, id)` | the default message of the constraint kind |
| Job Platforms | `hr.job.platform` | `hr_job_platform` | `_email_uniq` | table constraint | `unique (email)` | "The Email must be unique, this one already corresponds to another Job Platform." |
| Job Position | `hr.job` | `hr_job` | `_name_company_uniq` | table constraint | `unique(name, company_id, department_id)` | "The name of the job position must be unique per department in company!" |
| Job Position | `hr.job` | `hr_job` | `_no_of_recruitment_positive` | table constraint | `CHECK(no_of_recruitment >= 0)` | "The expected number of new employees must be positive." |
| Journal | `account.journal` | `account_journal` | `_code_company_uniq` | table constraint | `unique (company_id, code)` | "Journal codes must be unique per company." |
| Journal Entry | `account.move` | `account_move` | `_account_move_sanitize_payment_ref_idx` | index | `(regexp_replace(COALESCE(payment_reference, ''), '[^a-zA-Z0-9]', '', 'g'))` | the default message of the constraint kind |
| Journal Entry | `account.move` | `account_move` | `_checked_idx` | index | `(journal_id) WHERE (checked IS NOT TRUE)` | the default message of the constraint kind |
| Journal Entry | `account.move` | `account_move` | `_duplicate_bills_idx` | index | `(ref) WHERE (move_type IN ('in_invoice', 'in_refund'))` | the default message of the constraint kind |
| Journal Entry | `account.move` | `account_move` | `_journal_id_company_id_idx` | index | `(journal_id, company_id, date)` | the default message of the constraint kind |
| Journal Entry | `account.move` | `account_move` | `_journal_id_date_idx` | index | `(journal_id, date)` | the default message of the constraint kind |
| Journal Entry | `account.move` | `account_move` | `_l10n_pl_edi_number_company_id_move_type_uniq` | table constraint | `UNIQUE(l10n_pl_edi_number, company_id, move_type)` | "The KSeF number must be unique per company per move_type" |
| Journal Entry | `account.move` | `account_move` | `_made_gaps` | index | `(journal_id, state, payment_state, move_type, date) WHERE (made_sequence_gap IS TRUE)` | the default message of the constraint kind |
| Journal Entry | `account.move` | `account_move` | `_payment_idx` | index | `(journal_id, state, payment_state, move_type, date)` | the default message of the constraint kind |
| Journal Entry | `account.move` | `account_move` | `_unique_name` | unique index | `(name, journal_id) WHERE (state = 'posted'AND name != '/')` | "Another entry with the same name already exists." |
| Journal Entry | `account.move` | `account_move` | `_unique_name` | unique index | `(name, journal_id) WHERE (state = 'posted' AND name != '/' AND (l10n_latam_document_type_id IS NULL OR move_type NOT IN ('in_invoice', 'in_refund', 'in_receipt')))` | "Another entry with the same name already exists." |
| Journal Entry | `account.move` | `account_move` | `_unique_name_latam` | unique index | `(name, commercial_partner_id, l10n_latam_document_type_id, company_id) WHERE (state = 'posted' AND name != '/' AND (l10n_latam_document_type_id IS NOT NULL AND move_type IN ('in_invoice', 'in_refund', 'in_receipt')))` | "Another entry with the same name already exists." |
| Journal Item | `account.move.line` | `account_move_line` | `_account_id_date_idx` | index | `(account_id, date)` | the default message of the constraint kind |
| Journal Item | `account.move.line` | `account_move_line` | `_check_accountable_required_fields` | table constraint | `CHECK(display_type IN ('line_section', 'line_subsection', 'line_note') OR account_id IS NOT NULL)` | "Missing required account on accountable line." |
| Journal Item | `account.move.line` | `account_move_line` | `_check_amount_currency_balance_sign` | table constraint | `CHECK(                 display_type IN ('line_section', 'line_subsection', 'line_note')                 OR (                     (balance <= 0 AND amount_currency <= 0)                     OR                     (balance >= 0 AND amount_currency >= 0)                 )             )` | "The amount expressed in the secondary currency must be positive when account is debited and negative when account is credited. If the currency is the same as the one from the company, this amount must strictly be equal to the balance." |
| Journal Item | `account.move.line` | `account_move_line` | `_check_credit_debit` | table constraint | `CHECK(display_type IN ('line_section', 'line_subsection', 'line_note') OR credit * debit=0)` | "Wrong credit or debit value in accounting entry!" |
| Journal Item | `account.move.line` | `account_move_line` | `_check_non_accountable_fields_null` | table constraint | `CHECK(display_type NOT IN ('line_section', 'line_subsection', 'line_note') OR (amount_currency = 0 AND debit = 0 AND credit = 0 AND account_id IS NULL))` | "Forbidden balance or account on non-accountable line" |
| Journal Item | `account.move.line` | `account_move_line` | `_date_name_id_idx` | index | `(date desc, move_name desc, id)` | the default message of the constraint kind |
| Journal Item | `account.move.line` | `account_move_line` | `_journal_id_neg_amnt_residual_idx` | index | `(journal_id) WHERE amount_residual < 0` | the default message of the constraint kind |
| Journal Item | `account.move.line` | `account_move_line` | `_partner_id_ref_idx` | index | `(partner_id, ref)` | the default message of the constraint kind |
| Journal Item | `account.move.line` | `account_move_line` | `_unreconciled_index` | index | `(account_id, partner_id) WHERE reconciled IS NOT TRUE` | the default message of the constraint kind |
| Keep the call history | `discuss.call.history` | `discuss_call_history` | `_channel_id_end_dt_idx` | index | `(channel_id, end_dt) WHERE end_dt IS NULL` | the default message of the constraint kind |
| Keep the call history | `discuss.call.history` | `discuss_call_history` | `_channel_id_not_null_constraint` | table constraint | `CHECK (channel_id IS NOT NULL)` | "Call history must have a channel" |
| Keep the call history | `discuss.call.history` | `discuss_call_history` | `_message_id_unique_constraint` | table constraint | `UNIQUE (start_call_message_id)` | "Messages can only be linked to one call history" |
| Keep the call history | `discuss.call.history` | `discuss_call_history` | `_start_dt_is_not_null_constraint` | table constraint | `CHECK (start_dt IS NOT NULL)` | "Call history must have a start date" |
| Keep the channel member history | `im_livechat.channel.member.history` | `im_livechat_channel_member_history` | `_channel_id_guest_id_unique` | unique index | `(channel_id, guest_id) WHERE guest_id IS NOT NULL` | "One guest can only be linked to one history on a channel" |
| Keep the channel member history | `im_livechat.channel.member.history` | `im_livechat_channel_member_history` | `_channel_id_partner_id_unique` | unique index | `(channel_id, partner_id) WHERE partner_id IS NOT NULL` | "One partner can only be linked to one history on a channel" |
| Keep the channel member history | `im_livechat.channel.member.history` | `im_livechat_channel_member_history` | `_member_id_unique` | table constraint | `UNIQUE(member_id)` | "Members can only be linked to one history" |
| Keep the channel member history | `im_livechat.channel.member.history` | `im_livechat_channel_member_history` | `_partner_id_or_guest_id_constraint` | table constraint | `CHECK(partner_id IS NULL OR guest_id IS NULL)` | "History should either be linked to a partner or a guest but not both" |
| Languages | `res.lang` | `res_lang` | `_code_uniq` | table constraint | `unique(code)` | "The code of the language must be unique!" |
| Languages | `res.lang` | `res_lang` | `_name_uniq` | table constraint | `unique(name)` | "The name of the language must be unique!" |
| Languages | `res.lang` | `res_lang` | `_url_code_uniq` | table constraint | `unique(url_code)` | "The URL code of the language must be unique!" |
| Lead | `crm.lead` | `crm_lead` | `_check_probability` | table constraint | `check(probability >= 0 and probability <= 100)` | "The probability of closing the deal should be between 0% and 100%!" |
| Lead | `crm.lead` | `crm_lead` | `_create_date_team_id_idx` | index | `(create_date, team_id)` | the default message of the constraint kind |
| Lead | `crm.lead` | `crm_lead` | `_default_order_idx` | index | `(priority DESC, id DESC) WHERE active IS TRUE` | the default message of the constraint kind |
| Lead | `crm.lead` | `crm_lead` | `_user_id_team_id_type_index` | index | `(user_id, team_id, type)` | the default message of the constraint kind |
| Link Tracker Code | `link.tracker.code` | `link_tracker_code` | `_code` | table constraint | `unique( code )` | "Code must be unique." |
| Link between link previews and messages | `mail.message.link.preview` | `mail_message_link_preview` | `_unique_message_link_preview` | unique index | `(message_id, link_preview_id)` | the default message of the constraint kind |
| Link between products and their Units of Measure | `product.uom` | `product_uom` | `_barcode_uniq` | table constraint | `unique(barcode)` | "A barcode can only be assigned to one packaging." |
| Link text message to mailing/text message tracking models | `sms.tracker` | `sms_tracker` | `_sms_uuid_unique` | table constraint | `unique(sms_uuid)` | "A record for this UUID already exists" |
| Live Chat Conversation Tags | `im_livechat.conversation.tag` | `im_livechat_conversation_tag` | `_name_unique` | unique index | `(name)` | the default message of the constraint kind |
| Live Chat Expertise | `im_livechat.expertise` | `im_livechat_expertise` | `_name_unique` | unique index | `(name)` | the default message of the constraint kind |
| Livechat Channel | `im_livechat.channel` | `im_livechat_channel` | `_max_sessions_mode_greater_than_zero` | table constraint | `CHECK(max_sessions > 0)` | "Concurrent session number should be greater than zero." |
| Loyalty Coupon | `loyalty.card` | `loyalty_card` | `_card_code_unique` | table constraint | `UNIQUE(code)` | "A coupon/loyalty card must have a unique code." |
| Loyalty Program | `loyalty.program` | `loyalty_program` | `_check_max_usage` | table constraint | `CHECK (limit_usage = False OR max_usage > 0)` | "Max usage must be strictly positive if a limit is used." |
| Loyalty Reward | `loyalty.reward` | `loyalty_reward` | `_discount_positive` | table constraint | `CHECK (reward_type != 'discount' OR discount > 0)` | "The discount must be strictly positive." |
| Loyalty Reward | `loyalty.reward` | `loyalty_reward` | `_product_qty_positive` | table constraint | `CHECK (reward_type != 'product' OR reward_product_qty > 0)` | "The reward product quantity must be strictly positive." |
| Loyalty Reward | `loyalty.reward` | `loyalty_reward` | `_required_points_positive` | table constraint | `CHECK (required_points > 0)` | "The required points for a reward must be strictly positive." |
| Loyalty Rule | `loyalty.rule` | `loyalty_rule` | `_reward_point_amount_positive` | table constraint | `CHECK (reward_point_amount > 0)` | "Rule points reward must be strictly positive." |
| Lunch Alert | `lunch.alert` | `lunch_alert` | `_notification_time_range` | table constraint | `CHECK(notification_time >= 0 and notification_time <= 12)` | "Notification time must be between 0 and 12" |
| Lunch Order | `lunch.order` | `lunch_order` | `_user_product_date` | index | `(user_id, product_id, date)` | the default message of the constraint kind |
| Lunch Supplier | `lunch.supplier` | `lunch_supplier` | `_automatic_email_time_range` | table constraint | `CHECK(automatic_email_time >= 0 AND automatic_email_time <= 12)` | "Automatic Email Sending Time should be between 0 and 12" |
| Mail Blacklist | `mail.blacklist` | `mail_blacklist` | `_unique_email` | table constraint | `unique (email)` | "Email address already exists!" |
| Mail RTC session | `discuss.channel.rtc.session` | `discuss_channel_rtc_session` | `_channel_member_unique` | table constraint | `UNIQUE(channel_member_id)` | "There can only be one rtc session per channel member" |
| Mail Server | `ir.mail_server` | `ir_mail_server` | `_certificate_requires_tls` | table constraint | `CHECK(smtp_encryption != 'none' OR smtp_authentication != 'certificate')` | "Certificate-based authentication requires a TLS transport" |
| Mail Server | `ir.mail_server` | `ir_mail_server` | `_unique_owner_user_id` | table constraint | `UNIQUE(owner_user_id)` | "owner_user_id must be unique" |
| Mailing List Member | `mail.group.member` | `mail_group_member` | `_unique_partner` | table constraint | `UNIQUE(partner_id, mail_group_id)` | "This partner is already subscribed to the group" |
| Mailing List Subscription | `mailing.subscription` | `mailing_subscription` | `_unique_contact_list` | table constraint | `unique (contact_id, list_id)` | "A mailing contact cannot subscribe to the same mailing list multiple times." |
| Mailing List black/white list | `mail.group.moderation` | `mail_group_moderation` | `_mail_group_email_uniq` | table constraint | `UNIQUE(mail_group_id, email)` | "You can create only one rule for a given email address in a group." |
| Mailing Statistics | `mailing.trace` | `mailing_trace` | `_check_res_id_is_set` | table constraint | `CHECK(res_id IS NOT NULL AND res_id !=0 )` | "Traces have to be linked to records with a not null res_id." |
| Maintenance Equipment | `maintenance.equipment` | `maintenance_equipment` | `_serial_no` | table constraint | `unique(serial_no)` | "Another asset already exists with this serial number!" |
| Mandatory Day | `hr.leave.mandatory.day` | `hr_leave_mandatory_day` | `_date_from_after_day_to` | table constraint | `CHECK(start_date <= end_date)` | "The start date must be anterior than the end date." |
| Manufacturing Order | `mrp.production` | `mrp_production` | `_name_uniq` | table constraint | `unique(name, company_id)` | "Reference must be unique per Company!" |
| Manufacturing Order | `mrp.production` | `mrp_production` | `_qty_positive` | table constraint | `check (product_qty > 0)` | "The quantity to produce must be positive!" |
| Marketing Card | `card.card` | `card_card` | `_campaign_record_unique` | table constraint | `unique(campaign_id, res_id)` | "Each record should be unique for a campaign" |
| Marketing Card Campaign Tag | `card.campaign.tag` | `card_campaign_tag` | `_name_uniq` | table constraint | `unique(name)` | "Tags may not reuse existing names." |
| Mass Mailing | `mailing.mailing` | `mailing_mailing` | `_email_from` | table constraint | `CHECK(email_from IS NOT NULL OR mailing_type != 'mail')` | "email from is required for mailing" |
| Mass Mailing | `mailing.mailing` | `mailing_mailing` | `_percentage_valid` | table constraint | `CHECK(ab_testing_pc >= 0 AND ab_testing_pc <= 100)` | "The A/B Testing Percentage needs to be between 0 and 100%" |
| Message | `mail.message` | `mail_message` | `_date_res_id_id_for_burndown_chart` | index | `(date, res_id, id) WHERE model = 'project.task' AND message_type = 'notification'` | the default message of the constraint kind |
| Message | `mail.message` | `mail_message` | `_model_res_id_id_idx` | index | `(model, res_id, id)` | the default message of the constraint kind |
| Message | `mail.message` | `mail_message` | `_model_res_id_idx` | index | `(model, res_id)` | the default message of the constraint kind |
| Message Notifications | `mail.notification` | `mail_notification` | `_author_id_notification_status_failure` | index | `(author_id, notification_status) WHERE notification_status IN ('bounce', 'exception')` | the default message of the constraint kind |
| Message Notifications | `mail.notification` | `mail_notification` | `_notification_partner_or_email_required` | table constraint | `CHECK(notification_type != 'email' OR failure_type IS NOT NULL OR res_partner_id IS NOT NULL OR COALESCE(mail_email_address, '') != '')` | "Customer or email is required for inbox / email notification" |
| Message Notifications | `mail.notification` | `mail_notification` | `_notification_partner_required` | table constraint | `CHECK(notification_type != 'inbox' OR res_partner_id IS NOT NULL)` | "Customer is required for inbox notification" |
| Message Notifications | `mail.notification` | `mail_notification` | `_res_partner_id_is_read_notification_status_mail_message_id` | index | `(res_partner_id, is_read, notification_status, mail_message_id)` | the default message of the constraint kind |
| Message Notifications | `mail.notification` | `mail_notification` | `_unique_mail_message_id_res_partner_id_` | unique index | `(mail_message_id, res_partner_id) WHERE res_partner_id IS NOT NULL` | the default message of the constraint kind |
| Message Reaction | `mail.message.reaction` | `mail_message_reaction` | `_guest_unique` | unique index | `(message_id, content, guest_id) WHERE guest_id IS NOT NULL` | the default message of the constraint kind |
| Message Reaction | `mail.message.reaction` | `mail_message_reaction` | `_partner_or_guest_exists` | table constraint | `CHECK((partner_id IS NOT NULL AND guest_id IS NULL) OR (partner_id IS NULL AND guest_id IS NOT NULL))` | "A message reaction must be from a partner or from a guest." |
| Message Reaction | `mail.message.reaction` | `mail_message_reaction` | `_partner_unique` | unique index | `(message_id, content, partner_id) WHERE partner_id IS NOT NULL` | the default message of the constraint kind |
| Message Translation | `mail.message.translation` | `mail_message_translation` | `_unique` | unique index | `(message_id, target_lang)` | the default message of the constraint kind |
| Minimum Inventory Rule | `stock.warehouse.orderpoint` | `stock_warehouse_orderpoint` | `_product_location_check` | table constraint | `unique (product_id, location_id, company_id)` | "A replenishment rule already exists for this product on this location." |
| Model Constraint | `ir.model.constraint` | `ir_model_constraint` | `_module_name_uniq` | table constraint | `UNIQUE (name, module)` | "Constraints with the same name are unique per module." |
| Model Data | `ir.model.data` | `ir_model_data` | `_model_res_id_index` | index | `(model, res_id)` | the default message of the constraint kind |
| Model Data | `ir.model.data` | `ir_model_data` | `_module_name_uniq_index` | unique index | `(module, name)` | the default message of the constraint kind |
| Model Data | `ir.model.data` | `ir_model_data` | `_name_nospaces` | table constraint | `CHECK(name NOT LIKE '% %')` | "External IDs cannot contain spaces" |
| Model Inheritance Tree | `ir.model.inherit` | `ir_model_inherit` | `_uniq` | table constraint | `UNIQUE(model_id, parent_id)` | "Models inherits from another only once" |
| Model Page | `website.controller.page` | `website_controller_page` | `_unique_name_slugified` | table constraint | `UNIQUE(name_slugified)` | "url should be unique" |
| Models | `ir.model` | `ir_model` | `_obj_name_uniq` | table constraint | `UNIQUE (model)` | "Each model must have a unique name." |
| Module | `ir.module.module` | `ir_module_module` | `_name_uniq` | table constraint | `UNIQUE (name)` | "The name of the module must be unique!" |
| Onboarding | `onboarding.onboarding` | `onboarding_onboarding` | `_route_name_uniq` | table constraint | `UNIQUE (route_name)` | "Onboarding alias must be unique." |
| Onboarding Progress Step Tracker | `onboarding.progress.step` | `onboarding_progress_step` | `_company_uniq` | unique index | `(step_id, COALESCE(company_id, 0))` | the default message of the constraint kind |
| Onboarding Progress Tracker | `onboarding.progress` | `onboarding_progress` | `_onboarding_company_uniq` | unique index | `(onboarding_id, COALESCE(company_id, 0))` | the default message of the constraint kind |
| Outgoing text message | `sms.sms` | `sms_sms` | `_uuid_unique` | table constraint | `unique(uuid)` | "UUID must be unique" |
| Overtime Rule | `hr.attendance.overtime.rule` | `hr_attendance_overtime_rule` | `_timing_start_is_hour` | table constraint | `CHECK(0 <= timing_start AND timing_start < 24)` | "Timing Start is an hour of the day" |
| Overtime Rule | `hr.attendance.overtime.rule` | `hr_attendance_overtime_rule` | `_timing_stop_is_hour` | table constraint | `CHECK(0 <= timing_stop AND timing_stop <= 24)` | "Timing Stop is an hour of the day" |
| Partner in-app purchase | `res.partner.iap` | `res_partner_iap` | `_unique_partner_id` | table constraint | `UNIQUE(partner_id)` | "Only one partner IAP is allowed for one partner" |
| Passkey | `auth.passkey.key` | `auth_passkey_key` | `_unique_identifier` | table constraint | `UNIQUE(credential_identifier)` | "The credential identifier should be unique." |
| Payment Methods | `account.payment.method` | `account_payment_method` | `_name_code_unique` | table constraint | `unique (code, payment_type)` | "The combination code/payment type already exists!" |
| Payment Provider | `payment.provider` | `payment_provider` | `_custom_providers_setup` | table constraint | `CHECK(custom_mode IS NULL OR (code = 'custom' AND custom_mode IS NOT NULL))` | "Only custom providers should have a custom mode." |
| Payment Transaction | `payment.transaction` | `payment_transaction` | `_reference_uniq` | table constraint | `unique(reference)` | "Reference must be unique!" |
| Payments | `account.payment` | `account_payment` | `_check_amount_not_negative` | table constraint | `CHECK(amount >= 0.0)` | "The payment amount cannot be negative." |
| Payments | `account.payment` | `account_payment` | `_journal_id_company_id_idx` | index | `(journal_id, company_id)` | the default message of the constraint kind |
| Payments | `account.payment` | `account_payment` | `_unmatched_idx` | index | `(journal_id, company_id) WHERE is_matched IS NOT TRUE` | the default message of the constraint kind |
| People Role | `crm.iap.lead.role` | `crm_iap_lead_role` | `_name_uniq` | table constraint | `unique (name)` | "Role name already exists!" |
| People Seniority | `crm.iap.lead.seniority` | `crm_iap_lead_seniority` | `_name_uniq` | table constraint | `unique (name)` | "Name already exists!" |
| Personal Task Stage | `project.task.stage.personal` | `project_task_user_rel` | `_project_personal_stage_unique` | table constraint | `UNIQUE (task_id, user_id)` | "A task can only have a single personal stage per user." |
| Phone Blacklist | `phone.blacklist` | `phone_blacklist` | `_unique_number` | table constraint | `unique (number)` | "Number already exists" |
| Point of Sale Note | `pos.note` | `pos_note` | `_name_unique` | table constraint | `unique (name)` | "A note with this name already exists" |
| Point of Sale Order Lines | `pos.order.line` | `pos_order_line` | `_unique_uuid` | table constraint | `unique (uuid)` | "An order line with this uuid already exists" |
| Point of Sale Orders | `pos.order` | `pos_order` | `_unique_uuid` | table constraint | `unique (uuid)` | "An order with this uuid already exists" |
| Point of Sale Payments | `pos.payment` | `pos_payment` | `_unique_uuid` | table constraint | `unique (uuid)` | "A payment with this uuid already exists" |
| Post Vote | `forum.post.vote` | `forum_post_vote` | `_vote_uniq` | table constraint | `unique (post_id, user_id)` | "Vote already exists!" |
| Product | `product.template` | `product_template` | `_default_code_gist_idx` | index | `{'expression': "lambda registry: 'USING GIST(unaccent(default_code) gist_trgm_ops)' if registry.has_trigram and registry.has_unaccent == FunctionStatus.INDEXABLE else 'USING GIST(default_code gist_trgm_ops)' if registry.has_trigram else ''"}` | the default message of the constraint kind |
| Product | `product.template` | `product_template` | `_description_ecommerce_gist_idx` | index | `{'expression': "lambda registry: get_translated_field_gist_index(registry, 'description_ecommerce')"}` | the default message of the constraint kind |
| Product | `product.template` | `product_template` | `_description_gist_idx` | index | `{'expression': "lambda registry: get_translated_field_gist_index(registry, 'description')"}` | the default message of the constraint kind |
| Product | `product.template` | `product_template` | `_description_sale_gist_idx` | index | `{'expression': "lambda registry: get_translated_field_gist_index(registry, 'description_sale')"}` | the default message of the constraint kind |
| Product | `product.template` | `product_template` | `_is_favorite_index` | index | `(is_favorite) WHERE is_favorite IS TRUE` | the default message of the constraint kind |
| Product | `product.template` | `product_template` | `_name_gist_idx` | index | `{'expression': "lambda registry: get_translated_field_gist_index(registry, 'name')"}` | the default message of the constraint kind |
| Product Attribute | `product.attribute` | `product_attribute` | `_check_multi_checkbox_no_variant` | table constraint | `CHECK(display_type != 'multi' OR create_variant = 'no_variant')` | "Multi-checkbox display type is not compatible with the creation of variants" |
| Product Attribute Custom Value | `product.attribute.custom.value` | `product_attribute_custom_value` | `_sol_custom_value_unique` | table constraint | `unique(custom_product_template_attribute_value_id, sale_order_line_id)` | "Only one Custom Value is allowed per Attribute Value per Sales Order Line." |
| Product Moves (Stock Move Line) | `stock.move.line` | `stock_move_line` | `_free_reservation_index` | index | `(id, company_id, product_id, lot_id, location_id, owner_id, package_id)         WHERE (state IS NULL OR state NOT IN ('cancel', 'done')) AND quantity_product_uom > 0 AND picked IS NOT TRUE` | the default message of the constraint kind |
| Product Tag | `product.tag` | `product_tag` | `_name_uniq` | table constraint | `unique (name)` | "Tag name already exists!" |
| Product Template Attribute Value | `product.template.attribute.value` | `product_template_attribute_value` | `_attribute_value_unique` | table constraint | `unique(attribute_line_id, product_attribute_value_id)` | "Each value should be defined only once per attribute per product." |
| Product Unit of Measure | `uom.uom` | `uom_uom` | `_factor_gt_zero` | table constraint | `CHECK (relative_factor!=0)` | "The conversion ratio for a unit of measure cannot be 0!" |
| Product Variant | `product.product` | `product_product` | `_combination_unique` | unique index | `(product_tmpl_id, combination_indices) WHERE active IS TRUE` | the default message of the constraint kind |
| Product Variant | `product.product` | `product_product` | `_is_favorite_index` | index | `(is_favorite) WHERE is_favorite IS TRUE` | the default message of the constraint kind |
| Product Wishlist | `product.wishlist` | `product_wishlist` | `_product_unique_partner_id` | table constraint | `UNIQUE(product_id, partner_id)` | "Duplicated wishlisted product for this partner." |
| Project | `project.project` | `project_project` | `_project_date_greater` | table constraint | `check(date >= date_start)` | "The project's start date must be before its end date." |
| Project Sales line, employee mapping | `project.sale.line.employee.map` | `project_sale_line_employee_map` | `_uniqueness_employee` | table constraint | `UNIQUE(project_id,employee_id)` | "An employee cannot be selected more than once in the mapping. Please remove duplicate(s) and try again." |
| Project Tags | `project.tags` | `project_tags` | `_name_uniq` | table constraint | `unique (name)` | "A tag with the same name already exists." |
| Properties Base Definition | `properties.base.definition` | `properties_base_definition` | `_unique_properties_field_id` | table constraint | `UNIQUE(properties_field_id)` | "Only one definition per properties field" |
| Purchase Order Line | `purchase.order.line` | `purchase_order_line` | `_accountable_required_fields` | table constraint | `CHECK(display_type IS NOT NULL OR is_downpayment OR (product_id IS NOT NULL AND product_uom_id IS NOT NULL AND date_planned IS NOT NULL))` | "Missing required fields on accountable purchase order line." |
| Purchase Order Line | `purchase.order.line` | `purchase_order_line` | `_non_accountable_null_fields` | table constraint | `CHECK(display_type IS NULL OR (product_id IS NULL AND price_unit = 0 AND product_uom_qty = 0 AND product_uom_id IS NULL AND date_planned is NULL))` | "Forbidden values on non-accountable purchase order line" |
| Push Notification Device | `mail.push.device` | `mail_push_device` | `_endpoint_unique` | table constraint | `unique(endpoint)` | "The endpoint must be unique !" |
| Quotation Template Line | `sale.order.template.line` | `sale_order_template_line` | `_accountable_product_id_required` | table constraint | `CHECK(display_type IS NOT NULL OR (product_id IS NOT NULL AND product_uom_id IS NOT NULL))` | "Missing required product and UoM on accountable sale quote line." |
| Quotation Template Line | `sale.order.template.line` | `sale_order_template_line` | `_non_accountable_fields_null` | table constraint | `CHECK(display_type IS NULL OR (product_id IS NULL AND product_uom_qty = 0 AND product_uom_id IS NULL))` | "Forbidden product, quantity and UoM on non-accountable sale quote line" |
| Rank based on karma | `gamification.karma.rank` | `gamification_karma_rank` | `_karma_min_check` | table constraint | `CHECK( karma_min > 0 )` | "The required karma has to be above 0." |
| Rating | `rating.rating` | `rating_rating` | `_consumed_idx` | index | `(res_model, res_id, write_date) WHERE consumed IS TRUE` | the default message of the constraint kind |
| Rating | `rating.rating` | `rating_rating` | `_parent_consumed_idx` | index | `(parent_res_model, parent_res_id, write_date) WHERE consumed IS TRUE` | the default message of the constraint kind |
| Rating | `rating.rating` | `rating_rating` | `_rating_range` | table constraint | `check(rating >= 0 and rating <= 5)` | "Rating should be between 0 and 5" |
| Record Rule | `ir.rule` | `ir_rule` | `_no_access_rights` | table constraint | `CHECK (perm_read!=False or perm_write!=False or perm_create!=False or perm_unlink!=False)` | "Rule must have at least one checked access right!" |
| Recycling Model | `data_recycle.model` | `data_recycle_model` | `_check_notif_freq` | table constraint | `CHECK(notify_frequency > 0)` | "The notification frequency should be greater than 0" |
| Repair Tags | `repair.tags` | `repair_tags` | `_name_uniq` | table constraint | `unique (name)` | "Tag name already exists!" |
| Represents a role in the system used to categorize users. Each role has a unique name and can be associated with multiple users. Roles can be mentioned in messages to notify all associated users. | `res.role` | `res_role` | `_unique_name` | unique index | `(name)` | "A role with the same name already exists." |
| Resources | `resource.resource` | `resource_resource` | `_check_time_efficiency` | table constraint | `CHECK(time_efficiency>0)` | "Time efficiency must be strictly positive" |
| Resume line of an employee | `hr.resume.line` | `hr_resume_line` | `_date_check` | table constraint | `CHECK ((date_start <= date_end OR date_end IS NULL))` | "The start date must be anterior to the end date." |
| SInvoice symbol | `l10n_vn_edi_viettel.sinvoice.symbol` | `l10n_vn_edi_viettel_sinvoice_symbol` | `_name_template_uniq` | table constraint | `unique (name, invoice_template_id)` | "The combination symbol/template must be unique!" |
| SInvoice template | `l10n_vn_edi_viettel.sinvoice.template` | `l10n_vn_edi_viettel_sinvoice_template` | `_name_uniq` | table constraint | `unique (name)` | "The template code must be unique!" |
| Sale Order Coupon Points - Keeps track of how a sale order impacts a coupon | `sale.order.coupon.points` | `sale_order_coupon_points` | `_order_coupon_unique` | table constraint | `UNIQUE (order_id, coupon_id)` | "The coupon points entry already exists." |
| Sales Order | `sale.order` | `sale_order` | `_date_order_conditional_required` | table constraint | `CHECK((state = 'sale' AND date_order IS NOT NULL) OR state != 'sale')` | "A confirmed sales order requires a confirmation date." |
| Sales Order | `sale.order` | `sale_order` | `_date_order_id_idx` | index | `(date_order desc, id desc)` | the default message of the constraint kind |
| Sales Order Line | `sale.order.line` | `sale_order_line` | `_accountable_required_fields` | table constraint | `CHECK(display_type IS NOT NULL OR is_downpayment OR (product_id IS NOT NULL AND product_uom_id IS NOT NULL))` | "Missing required fields on accountable sale order line." |
| Sales Order Line | `sale.order.line` | `sale_order_line` | `_name_search_services_index` | index | `(order_id DESC, sequence, id) WHERE is_service IS TRUE` | the default message of the constraint kind |
| Sales Order Line | `sale.order.line` | `sale_order_line` | `_non_accountable_null_fields` | table constraint | `CHECK(display_type IS NULL OR (product_id IS NULL AND price_unit = 0 AND product_uom_qty = 0 AND product_uom_id IS NULL AND customer_lead = 0))` | "Forbidden values on non-accountable sale order line" |
| Save favorite Animated Image from Tenor application programming interface | `discuss.gif.favorite` | `discuss_gif_favorite` | `_user_gif_favorite` | table constraint | `unique(create_uid,tenor_gif_id)` | "User should not have duplicated favorite GIF" |
| Scheduled Actions | `ir.cron` | `ir_cron` | `_check_strictly_positive_interval` | table constraint | `CHECK(interval_number > 0)` | "The interval number must be a strictly positive number." |
| Scrap Reason Tag | `stock.scrap.reason.tag` | `stock_scrap_reason_tag` | `_name_uniq` | table constraint | `unique (name)` | "Tag name already exists!" |
| Sequence Date Range | `ir.sequence.date_range` | `ir_sequence_date_range` | `_unique_range_per_sequence` | table constraint | `UNIQUE(sequence_id, date_from, date_to)` | "You cannot create two date ranges for the same sequence with the same date range." |
| Shipping Methods | `delivery.carrier` | `delivery_carrier` | `_margin_not_under_100_percent` | table constraint | `CHECK (margin >= -1)` | "Margin cannot be lower than -100%" |
| Shipping Methods | `delivery.carrier` | `delivery_carrier` | `_shipping_insurance_is_percentage` | table constraint | `CHECK(shipping_insurance >= 0 AND shipping_insurance <= 100)` | "The shipping insurance must be a percentage between 0 and 100." |
| Skill Level | `hr.skill.level` | `hr_skill_level` | `_check_level_progress` | table constraint | `CHECK(level_progress BETWEEN 0 AND 100)` | "Progress should be a number between 0 and 100." |
| Slide / Partner decorated m2m | `slide.slide.partner` | `slide_slide_partner` | `_check_vote` | table constraint | `CHECK(vote IN (-1, 0, 1))` | "The vote must be 1, 0 or -1." |
| Slide / Partner decorated m2m | `slide.slide.partner` | `slide_slide_partner` | `_slide_partner_uniq` | table constraint | `unique(slide_id, partner_id)` | "A partner membership to a slide must be unique!" |
| Slide Tag | `slide.tag` | `slide_tag` | `_slide_tag_unique` | table constraint | `UNIQUE(name)` | "A tag must be unique!" |
| Slides | `slide.slide` | `slide_slide` | `_check_certification_preview` | table constraint | `CHECK(slide_category != 'certification' OR is_preview = False)` | "A slide of type certification cannot be previewed." |
| Slides | `slide.slide` | `slide_slide` | `_check_survey_id` | table constraint | `CHECK(slide_category != 'certification' OR survey_id IS NOT NULL)` | "A slide of type 'certification' requires a certification." |
| Slides | `slide.slide` | `slide_slide` | `_exclusion_html_content_and_url` | table constraint | `CHECK(html_content IS NULL OR url IS NULL)` | "A slide is either filled with a url or HTML content. Not both." |
| Stock Move | `stock.move` | `stock_move` | `_product_location_index` | index | `(product_id, location_id, location_dest_id, company_id, state)` | the default message of the constraint kind |
| Stock package type | `stock.package.type` | `stock_package_type` | `_barcode_uniq` | table constraint | `unique(barcode)` | "A barcode can only be assigned to one package type!" |
| Stock package type | `stock.package.type` | `stock_package_type` | `_positive_height` | table constraint | `CHECK(height>=0.0)` | "Height must be positive" |
| Stock package type | `stock.package.type` | `stock_package_type` | `_positive_length` | table constraint | `CHECK(packaging_length>=0.0)` | "Length must be positive" |
| Stock package type | `stock.package.type` | `stock_package_type` | `_positive_max_weight` | table constraint | `CHECK(max_weight>=0.0)` | "Max Weight must be positive" |
| Stock package type | `stock.package.type` | `stock_package_type` | `_positive_width` | table constraint | `CHECK(width>=0.0)` | "Width must be positive" |
| Storage Category | `stock.storage.category` | `stock_storage_category` | `_positive_max_weight` | table constraint | `CHECK(max_weight >= 0)` | "Max weight should be a positive number." |
| Storage Category Capacity | `stock.storage.category.capacity` | `stock_storage_category_capacity` | `_positive_quantity` | table constraint | `CHECK(quantity > 0)` | "Quantity should be a positive number." |
| Storage Category Capacity | `stock.storage.category.capacity` | `stock_storage_category_capacity` | `_unique_package_type` | table constraint | `UNIQUE(package_type_id, storage_category_id)` | "Multiple capacity rules for one package type." |
| Storage Category Capacity | `stock.storage.category.capacity` | `stock_storage_category_capacity` | `_unique_product` | table constraint | `UNIQUE(product_id, storage_category_id)` | "Multiple capacity rules for one product." |
| Store link preview data | `mail.link.preview` | `mail_link_preview` | `_unique_source_url` | unique index | `(source_url)` | the default message of the constraint kind |
| Survey | `survey.survey` | `survey_survey` | `_access_token_unique` | table constraint | `unique(access_token)` | "Access token should be unique" |
| Survey | `survey.survey` | `survey_survey` | `_attempts_limit_check` | table constraint | `CHECK( (is_attempts_limited=False) OR (attempts_limit is not null AND attempts_limit > 0) )` | "The attempts limit needs to be a positive number if the survey has a limited number of attempts." |
| Survey | `survey.survey` | `survey_survey` | `_badge_uniq` | table constraint | `unique (certification_badge_id)` | "The badge for each survey should be unique!" |
| Survey | `survey.survey` | `survey_survey` | `_certification_check` | table constraint | `CHECK( scoring_type!='no_scoring' OR certification=False )` | "You can only create certifications for surveys that have a scoring mechanism." |
| Survey | `survey.survey` | `survey_survey` | `_scoring_success_min_check` | table constraint | `CHECK( scoring_success_min IS NULL OR (scoring_success_min>=0 AND scoring_success_min<=100) )` | "The percentage of success has to be defined between 0 and 100." |
| Survey | `survey.survey` | `survey_survey` | `_session_code_unique` | table constraint | `unique(session_code)` | "Session code should be unique" |
| Survey | `survey.survey` | `survey_survey` | `_session_speed_rating_has_time_limit` | table constraint | `CHECK (session_speed_rating != TRUE OR session_speed_rating_time_limit IS NOT NULL AND session_speed_rating_time_limit > 0)` | "A positive default time limit is required when the session rewards quick answers." |
| Survey | `survey.survey` | `survey_survey` | `_time_limit_check` | table constraint | `CHECK( (is_time_limited=False) OR (time_limit is not null AND time_limit > 0) )` | "The time limit needs to be a positive number if the survey is time limited." |
| Survey Label | `survey.question.answer` | `survey_question_answer` | `_value_not_empty` | table constraint | `CHECK (value IS NOT NULL OR value_image_filename IS NOT NULL)` | "Suggested answer value must not be empty (a text and/or an image must be provided)." |
| Survey Question | `survey.question` | `survey_question` | `_is_time_limited_have_time_limit` | table constraint | `CHECK (is_time_limited != TRUE OR time_limit IS NOT NULL AND time_limit > 0)` | "All time-limited questions need a positive time limit" |
| Survey Question | `survey.question` | `survey_question` | `_positive_answer_score` | table constraint | `CHECK (answer_score >= 0)` | "An answer score for a non-multiple choice question cannot be negative!" |
| Survey Question | `survey.question` | `survey_question` | `_positive_len_max` | table constraint | `CHECK (validation_length_max >= 0)` | "A length must be positive!" |
| Survey Question | `survey.question` | `survey_question` | `_positive_len_min` | table constraint | `CHECK (validation_length_min >= 0)` | "A length must be positive!" |
| Survey Question | `survey.question` | `survey_question` | `_scale` | table constraint | `CHECK (question_type != 'scale' OR (scale_min >= 0 AND scale_max <= 10 AND scale_min < scale_max))` | "The scale must be a growing non-empty range between 0 and 10 (inclusive)" |
| Survey Question | `survey.question` | `survey_question` | `_scored_date_have_answers` | table constraint | `CHECK (is_scored_question != True OR question_type != 'date' OR answer_date is not null)` | "All "Is a scored question = True" and "Question Type: Date" questions need an answer" |
| Survey Question | `survey.question` | `survey_question` | `_scored_datetime_have_answers` | table constraint | `CHECK (is_scored_question != True OR question_type != 'datetime' OR answer_datetime is not null)` | "All "Is a scored question = True" and "Question Type: Datetime" questions need an answer" |
| Survey Question | `survey.question` | `survey_question` | `_validation_date` | table constraint | `CHECK (validation_min_date <= validation_max_date)` | "Max date cannot be smaller than min date!" |
| Survey Question | `survey.question` | `survey_question` | `_validation_datetime` | table constraint | `CHECK (validation_min_datetime <= validation_max_datetime)` | "Max datetime cannot be smaller than min datetime!" |
| Survey Question | `survey.question` | `survey_question` | `_validation_float` | table constraint | `CHECK (validation_min_float_value <= validation_max_float_value)` | "Max value cannot be smaller than min value!" |
| Survey Question | `survey.question` | `survey_question` | `_validation_length` | table constraint | `CHECK (validation_length_min <= validation_length_max)` | "Max length cannot be smaller than min length!" |
| Survey User Input | `survey.user_input` | `survey_user_input` | `_unique_token` | table constraint | `UNIQUE (access_token)` | "An access token must be unique!" |
| System Parameter | `ir.config_parameter` | `ir_config_parameter` | `_key_uniq` | table constraint | `unique (key)` | "Key must be unique." |
| Task | `project.task` | `project_task` | `_is_template_idx` | index | `(is_template) WHERE is_template IS TRUE` | the default message of the constraint kind |
| Task | `project.task` | `project_task` | `_private_task_has_no_parent` | table constraint | `CHECK (NOT (project_id IS NULL AND parent_id IS NOT NULL))` | "A private task cannot have a parent." |
| Task | `project.task` | `project_task` | `_recurring_task_has_no_parent` | table constraint | `CHECK (NOT (recurring_task IS TRUE AND parent_id IS NOT NULL))` | "You cannot convert this task into a sub-task because it is recurrent." |
| Tax Office in Poland | `l10n_pl.l10n_pl_tax_office` | `l10n_pl_l10n_pl_tax_office` | `_code_company_uniq` | table constraint | `unique (code)` | "The code of the tax office must be unique !" |
| Tax office in Czech Republic | `l10n_cz.tax_office` | `l10n_cz_tax_office` | `_workplace_code_unique` | table constraint | `UNIQUE (workplace_code)` | "The territorial workplace code must be unique" |
| Thumb drive used to sign invoices in Egypt | `l10n_eg_edi.thumb.drive` | `l10n_eg_edi_thumb_drive` | `_user_drive_uniq` | table constraint | `unique (user_id, company_id)` | "You can only have one thumb drive per user per company!" |
| Time Off | `hr.leave` | `hr_leave` | `_date_check2` | table constraint | `CHECK ((date_from <= date_to))` | "The start date must be before or equal to the end date." |
| Time Off | `hr.leave` | `hr_leave` | `_date_check3` | table constraint | `CHECK ((request_date_from <= request_date_to))` | "The request start date must be before or equal to the request end date." |
| Time Off | `hr.leave` | `hr_leave` | `_date_to_date_from_index` | index | `(date_to, date_from)` | the default message of the constraint kind |
| Time Off | `hr.leave` | `hr_leave` | `_duration_check` | table constraint | `CHECK ( number_of_days >= 0 )` | "If you want to change the number of days you should use the 'period' mode" |
| Time Off Allocation | `hr.leave.allocation` | `hr_leave_allocation` | `_duration_check` | table constraint | `CHECK( ( number_of_days > 0 AND allocation_type='regular') or (allocation_type != 'regular'))` | "The duration must be greater than 0." |
| Time Off Type | `hr.leave.type` | `hr_leave_type` | `_check_negative` | table constraint | `CHECK(NOT allows_negative OR max_allowed_negative > 0)` | "The maximum excess amount should be greater than 0. If you want to set 0, disable the negative cap instead." |
| Tours | `web_tour.tour` | `web_tour_tour` | `_uniq_name` | table constraint | `unique(name)` | "A tour already exists with this name . Tour's name must be unique!" |
| Transfer | `stock.picking` | `stock_picking` | `_name_uniq` | table constraint | `unique(name, company_id)` | "Reference must be unique per company!" |
| Unbuild Order | `mrp.unbuild` | `mrp_unbuild` | `_qty_positive` | table constraint | `check (product_qty > 0)` | "The quantity to unbuild must be positive!" |
| User | `res.users` | `res_users` | `_login_key` | table constraint | `UNIQUE (login)` | "You can not have two users with the same login!" |
| User | `res.users` | `res_users` | `_login_key` | table constraint | `unique (login, website_id)` | "You can not have two users with the same login!" |
| User | `res.users` | `res_users` | `_notification_type` | table constraint | `CHECK (notification_type = 'email' OR NOT share)` | "Only internal user can receive notifications in the system" |
| User | `res.users` | `res_users` | `_uniq_users_oauth_provider_oauth_uid` | table constraint | `unique(oauth_provider_id, oauth_uid)` | "OAuth UID must be unique per provider" |
| User Settings | `res.users.settings` | `res_users_settings` | `_unique_user_id` | table constraint | `UNIQUE(user_id)` | "One user should only have one user settings." |
| User Settings Volumes | `res.users.settings.volumes` | `res_users_settings_volumes` | `_guest_unique` | unique index | `(user_setting_id, guest_id) WHERE guest_id IS NOT NULL` | the default message of the constraint kind |
| User Settings Volumes | `res.users.settings.volumes` | `res_users_settings_volumes` | `_partner_or_guest_exists` | table constraint | `CHECK((partner_id IS NOT NULL AND guest_id IS NULL) OR (partner_id IS NULL AND guest_id IS NOT NULL))` | "A volume setting must have a partner or a guest." |
| User Settings Volumes | `res.users.settings.volumes` | `res_users_settings_volumes` | `_partner_unique` | unique index | `(user_setting_id, partner_id) WHERE partner_id IS NOT NULL` | the default message of the constraint kind |
| User Settings for Embedded Actions | `res.users.settings.embedded.action` | `res_users_settings_embedded_action` | `_res_user_settings_embedded_action_unique` | table constraint | `UNIQUE (user_setting_id, action_id, res_id)` | "The user should have one unique embedded action setting per user setting, action and record id." |
| User/Guest Presence | `mail.presence` | `mail_presence` | `_guest_unique` | unique index | `(guest_id) WHERE guest_id IS NOT NULL` | the default message of the constraint kind |
| User/Guest Presence | `mail.presence` | `mail_presence` | `_partner_or_guest_exists` | table constraint | `CHECK((user_id IS NOT NULL AND guest_id IS NULL) OR (user_id IS NULL AND guest_id IS NOT NULL))` | "A mail presence must have a user or a guest." |
| User/Guest Presence | `mail.presence` | `mail_presence` | `_user_unique` | unique index | `(user_id) WHERE user_id IS NOT NULL` | the default message of the constraint kind |
| Vehicle Status | `fleet.vehicle.state` | `fleet_vehicle_state` | `_fleet_state_name_unique` | table constraint | `unique(name)` | "State name already exists" |
| Vehicle Tag | `fleet.vehicle.tag` | `fleet_vehicle_tag` | `_name_uniq` | table constraint | `unique (name)` | "Tag name already exists!" |
| Version | `hr.version` | `hr_version` | `_check_contract_start_date_defined` | table constraint | `CHECK(contract_date_end IS NULL OR contract_date_start IS NOT NULL)` | "The contract must have a start date." |
| Version | `hr.version` | `hr_version` | `_check_unique_date_version` | unique index | `(employee_id, date_version) WHERE active = TRUE AND employee_id IS NOT NULL` | "An employee cannot have multiple active versions sharing the same effective date." |
| View | `ir.ui.view` | `ir_ui_view` | `_inheritance_mode` | table constraint | `CHECK (mode != 'extension' OR inherit_id IS NOT NULL)` | "Invalid inheritance mode: if the mode is 'extension', the view must extend an other view" |
| View | `ir.ui.view` | `ir_ui_view` | `_model_type_inherit_id` | index | `(model, inherit_id)` | the default message of the constraint kind |
| View | `ir.ui.view` | `ir_ui_view` | `_qweb_required_key` | table constraint | `CHECK (type != 'qweb' OR key IS NOT NULL)` | "Invalid key: QWeb view should have a key" |
| Warehouse | `stock.warehouse` | `stock_warehouse` | `_warehouse_code_uniq` | table constraint | `unique(code, company_id)` | "The short name of the warehouse must be unique per company!" |
| Warehouse | `stock.warehouse` | `stock_warehouse` | `_warehouse_name_uniq` | table constraint | `unique(name, company_id)` | "The name of the warehouse must be unique per company!" |
| Website | `website` | `website` | `_domain_unique` | table constraint | `unique(domain)` | "Website Domain should be unique." |
| Website Visitor | `website.visitor` | `website_visitor` | `_access_token_unique` | table constraint | `unique(access_token)` | "Access token should be unique." |
| Work Center Capacity | `mrp.workcenter.capacity` | `mrp_workcenter_capacity` | `_positive_capacity` | table constraint | `CHECK(capacity >= 0)` | "Capacity should be a non-negative number." |
| Work Center Capacity | `mrp.workcenter.capacity` | `mrp_workcenter_capacity` | `_workcenter_product_product_uom_unique` | unique index | `(workcenter_id, COALESCE(product_id, 0), product_uom_id)` | "Product/Unit capacity should be unique for each workcenter." |
| Work Entries Employees | `hr.user.work.entry.employee` | `hr_user_work_entry_employee` | `_user_id_employee_id_unique` | table constraint | `UNIQUE(user_id,employee_id)` | "You cannot have the same employee twice." |
| campaign tracking parameter Campaign | `utm.campaign` | `utm_campaign` | `_unique_name` | table constraint | `UNIQUE(name)` | "The name must be unique" |
| campaign tracking parameter Medium | `utm.medium` | `utm_medium` | `_unique_name` | table constraint | `UNIQUE(name)` | "The name must be unique" |
| campaign tracking parameter Source | `utm.source` | `utm_source` | `_unique_name` | table constraint | `UNIQUE(name)` | "The name must be unique" |
| campaign tracking parameter Tag | `utm.tag` | `utm_tag` | `_name_uniq` | table constraint | `unique (name)` | "Tag name already exists!" |
| customer relationship management Lead Generation Rules | `crm.reveal.rule` | `crm_reveal_rule` | `_limit_extra_contacts` | table constraint | `check(extra_contacts >= 1 and extra_contacts <= 5)` | "Maximum 5 contacts are allowed!" |
| customer relationship management Recurring revenue plans | `crm.recurring.plan` | `crm_recurring_plan` | `_check_number_of_months` | table constraint | `CHECK(number_of_months >= 0)` | "The number of month can't be negative." |
| customer relationship management Reveal View | `crm.reveal.view` | `crm_reveal_view` | `_ip_rule_id` | unique index | `(reveal_rule_id,reveal_ip)` | the default message of the constraint kind |
| customer relationship management Reveal View | `crm.reveal.view` | `crm_reveal_view` | `_state_create_date` | index | `(reveal_state,create_date)` | the default message of the constraint kind |
| customer relationship management Tag | `crm.tag` | `crm_tag` | `_name_uniq` | table constraint | `unique (name)` | "Tag name already exists!" |
| customer relationship management in-app purchase Lead Industry | `crm.iap.lead.industry` | `crm_iap_lead_industry` | `_name_uniq` | table constraint | `unique (name)` | "Industry name already exists!" |
| electronic data interchange format | `account.edi.format` | `account_edi_format` | `_unique_code` | table constraint | `unique (code)` | "This code already exists" |
| human resources Work Entry | `hr.work.entry` | `hr_work_entry` | `_contract_date_start_stop_idx` | index | `(version_id, date) WHERE state IN ('draft', 'validated')` | the default message of the constraint kind |
| in-app purchase Service | `iap.service` | `iap_service` | `_unique_technical_name` | table constraint | `UNIQUE(technical_name)` | "Only one service can exist with a specific technical_name" |
| indian section alert | `l10n_in.section.alert` | `l10n_in_section_alert` | `_aggregate_limit` | table constraint | `CHECK(aggregate_limit >= 0)` | "Aggregate limit must be positive" |
| indian section alert | `l10n_in.section.alert` | `l10n_in_section_alert` | `_per_transaction_limit` | table constraint | `CHECK(per_transaction_limit >= 0)` | "Per transaction limit must be positive" |
| time-based one-time password rate limit logs | `auth.totp.rate.limit.log` | `auth_totp_rate_limit_log` | `_user_id_limit_type_create_date_idx` | index | `(user_id, limit_type, create_date)` | the default message of the constraint kind |

The store itself reports 296 unique and check constraints on the observed schema, which is the same set seen from the other side: the declared table constraints of the catalogue above, minus the ones the store records as indexes, plus the ones the store derives.

## 12. Per-entity physical catalogue

One row per entity. "Value columns" counts the columns of the table other than the primary key and the four audit columns; entities with no table count zero. "Association tables" counts the association tables the entity takes part in, on either side. "Indexed columns" counts the columns of the entity that carry a column-level index. "Table constraints" and "Declared indexes" count the objects the entity declares. "Specified in" names the domain folder that specifies the entity.

### 12.1 Persistent entities

600 entities, 7,623 value columns.

| Entity | Transport name | Table | Specified in | Value columns | Association tables | Indexed columns | Table constraints | Declared indexes | Notes |
|---|---|---|---|---|---|---|---|---|---|
| Access Groups | `res.groups` | `res_groups` | Identity and Access | 10 | 16 | 1 | 2 | 0 |  |
| Account | `account.account` | `account_account` | General Ledger | 15 | 4 | 3 | 0 | 0 |  |
| Account Cash Rounding | `account.cash.rounding` | `account_cash_rounding` | General Ledger | 6 | 0 | 2 | 0 | 0 |  |
| Account codes first 2 digits | `account.root` | `account_root` | General Ledger | 0 | 0 | 0 | 0 | 0 | no table: read from a stored query built at read time |
| Account electronic data interchange proxy user | `account_edi_proxy_client.user` | `account_edi_proxy_client_user` | Electronic Invoicing and Document Exchange | 11 | 0 | 1 | 2 | 4 |  |
| Account Group | `account.group` | `account_group` | General Ledger | 5 | 0 | 1 | 1 | 0 |  |
| Account Journal Group | `account.journal.group` | `account_journal_group` | General Ledger | 3 | 1 | 0 | 1 | 0 |  |
| Account Lock Exception | `account.lock_exception` | `account_lock_exception` | General Ledger | 8 | 0 | 0 | 0 | 1 |  |
| Account payment check | `l10n_latam.check` | `l10n_latam_check` | Payments and Bank Reconciliation | 11 | 3 | 0 | 0 | 1 |  |
| Account Tag | `account.account.tag` | `account_account_tag` | General Ledger | 5 | 4 | 0 | 1 | 0 |  |
| Accounting Assert Test | `accounting.assert.test` | `accounting_assert_test` | Financial Reporting | 5 | 0 | 0 | 0 | 0 |  |
| Accounting Report | `account.report` | `account_report` | General Ledger | 31 | 1 | 1 | 0 | 0 |  |
| Accounting Report Column | `account.report.column` | `account_report_column` | General Ledger | 8 | 0 | 1 | 0 | 0 |  |
| Accounting Report Expression | `account.report.expression` | `account_report_expression` | General Ledger | 11 | 0 | 1 | 2 | 0 |  |
| Accounting Report External Value | `account.report.external.value` | `account_report_external_value` | General Ledger | 8 | 0 | 0 | 0 | 0 |  |
| Accounting Report Line | `account.report.line` | `account_report_line` | General Ledger | 13 | 0 | 2 | 1 | 0 |  |
| Accounts Mapping of Fiscal Position | `account.fiscal.position.account` | `account_fiscal_position_account` | General Ledger | 4 | 0 | 0 | 1 | 0 |  |
| Accrual Plan | `hr.leave.accrual.plan` | `hr_leave_accrual_plan` | Time Off | 12 | 0 | 1 | 0 | 0 |  |
| Accrual Plan Level | `hr.leave.accrual.level` | `hr_leave_accrual_level` | Time Off | 27 | 0 | 1 | 5 | 0 |  |
| Action uniform resource locator | `ir.actions.act_url` | `ir_act_url` | Platform Foundation | 9 | 0 | 0 | 0 | 0 |  |
| Action Window | `ir.actions.act_window` | `ir_act_window` | Platform Foundation | 20 | 1 | 0 | 0 | 0 |  |
| Action Window Close | `ir.actions.act_window_close` | `ir_actions` | Platform Foundation | 7 | 0 | 0 | 0 | 0 |  |
| Action Window View | `ir.actions.act_window.view` | `ir_act_window_view` | Platform Foundation | 5 | 0 | 1 | 0 | 1 |  |
| Actions | `ir.actions.actions` | `ir_actions` | Platform Foundation | 7 | 0 | 0 | 1 | 0 |  |
| Activity | `mail.activity` | `mail_activity` | Messaging and Activities | 18 | 1 | 6 | 2 | 0 |  |
| Activity Plan | `mail.activity.plan` | `mail_activity_plan` | Messaging and Activities | 6 | 1 | 1 | 0 | 0 |  |
| Activity plan template | `mail.activity.plan.template` | `mail_activity_plan_template` | Messaging and Activities | 10 | 1 | 1 | 0 | 0 |  |
| Activity Type | `mail.activity.type` | `mail_activity_type` | Messaging and Activities | 15 | 3 | 1 | 0 | 0 |  |
| Add tag for the workcenter | `mrp.workcenter.tag` | `mrp_workcenter_tag` | Manufacturing | 2 | 1 | 0 | 1 | 0 |  |
| Additional resource for a particular slide | `slide.slide.resource` | `slide_slide_resource` | Learning, Surveys and Gamification | 6 | 0 | 1 | 2 | 0 |  |
| Administrative Center Role Type | `l10n_es_edi_facturae.ac_role_type` | `l10n_es_edi_facturae_ac_role_type` | Fiscal Localizations | 2 | 1 | 0 | 0 | 0 |  |
| All Website Route | `website.route` | `website_route` | Website and Storefront | 1 | 0 | 0 | 0 | 0 |  |
| Analytic Account | `account.analytic.account` | `account_analytic_account` | Analytic Accounting | 7 | 3 | 4 | 0 | 0 |  |
| Analytic Distribution Model | `account.analytic.distribution.model` | `account_analytic_distribution_model` | Analytic Accounting | 8 | 0 | 0 | 0 | 0 |  |
| Analytic Line | `account.analytic.line` | `account_analytic_line` | Analytic Accounting | 30 | 3 | 15 | 0 | 1 |  |
| Analytic Plan's Applicabilities | `account.analytic.applicability` | `account_analytic_applicability` | Analytic Accounting | 6 | 0 | 1 | 0 | 0 |  |
| Analytic Plans | `account.analytic.plan` | `account_analytic_plan` | Analytic Accounting | 8 | 0 | 3 | 0 | 0 |  |
| Applicant | `hr.applicant` | `hr_applicant` | Recruitment | 42 | 9 | 17 | 0 | 1 |  |
| Applicant Degree | `hr.recruitment.degree` | `hr_recruitment_degree` | Recruitment | 3 | 0 | 0 | 2 | 0 |  |
| Application | `ir.module.category` | `ir_module_category` | Platform Foundation | 6 | 0 | 1 | 0 | 0 |  |
| ARCA Responsibility Type | `l10n_ar.afip.responsibility.type` | `l10n_ar_afip_responsibility_type` | Fiscal Localizations | 4 | 1 | 2 | 2 | 0 |  |
| Argentinean Partner Taxes | `l10n_ar.partner.tax` | `l10n_ar_partner_tax` | Fiscal Localizations | 6 | 0 | 0 | 0 | 0 |  |
| Asset | `ir.asset` | `ir_asset` | Platform Foundation | 10 | 0 | 1 | 0 | 0 |  |
| Attachment | `ir.attachment` | `ir_attachment` | Platform Foundation | 20 | 14 | 4 | 0 | 1 |  |
| Attendance | `hr.attendance` | `hr_attendance` | Attendances and Working Time | 21 | 0 | 3 | 0 | 0 |  |
| Attendance and Leave Analysis Report | `hr.leave.attendance.report` | `hr_leave_attendance_report` | Time Off | 7 | 0 | 0 | 0 | 0 | no table: read from a stored query, materialized as a view |
| Attendance Overtime Line | `hr.attendance.overtime.line` | `hr_attendance_overtime_line` | Attendances and Working Time | 9 | 1 | 2 | 1 | 0 |  |
| Attribute Value | `product.attribute.value` | `product_attribute_value` | Products and Catalog | 8 | 1 | 2 | 0 | 0 |  |
| Authentication Device | `auth_totp.device` | `auth_totp_device` | Identity and Access | 6 | 0 | 0 | 0 | 0 |  |
| Automation Rule | `base.automation` | `base_automation` | Platform Foundation | 18 | 2 | 0 | 0 | 0 |  |
| Bank | `res.bank` | `res_bank` | Contacts and Organizations | 15 | 0 | 1 | 0 | 0 |  |
| Bank Accounts | `res.partner.bank` | `res_partner_bank` | Contacts and Organizations | 24 | 1 | 1 | 1 | 0 |  |
| Bank Statement | `account.bank.statement` | `account_bank_statement` | General Ledger | 11 | 1 | 1 | 0 | 2 |  |
| Bank Statement Line | `account.bank.statement.line` | `account_bank_statement_line` | General Ledger | 20 | 1 | 5 | 0 | 3 |  |
| Barcode Nomenclature | `barcode.nomenclature` | `barcode_nomenclature` | Products and Catalog | 4 | 0 | 0 | 0 | 0 |  |
| Barcode Rule | `barcode.rule` | `barcode_rule` | Products and Catalog | 10 | 0 | 1 | 0 | 0 |  |
| Base Import Mapping | `base_import.mapping` | `base_import_mapping` | Automation and Integration | 3 | 0 | 1 | 0 | 0 |  |
| Batch Transfer | `stock.picking.batch` | `stock_picking_batch` | Inventory Operations | 27 | 0 | 3 | 0 | 0 |  |
| Bill of Material | `mrp.bom` | `mrp_bom` | Manufacturing | 18 | 3 | 3 | 1 | 0 |  |
| Bill of Material Line | `mrp.bom.line` | `mrp_bom_line` | Manufacturing | 9 | 1 | 4 | 1 | 0 |  |
| Blog | `blog.blog` | `blog_blog` | Website and Storefront | 13 | 0 | 1 | 0 | 0 |  |
| Blog Post | `blog.post` | `blog_post` | Website and Storefront | 22 | 1 | 4 | 0 | 0 |  |
| Blog Tag | `blog.tag` | `blog_tag` | Website and Storefront | 9 | 1 | 1 | 1 | 0 |  |
| Blog Tag Category | `blog.tag.category` | `blog_tag_category` | Website and Storefront | 1 | 0 | 0 | 1 | 0 |  |
| Brand of the vehicle | `fleet.vehicle.model.brand` | `fleet_vehicle_model_brand` | Fleet | 3 | 0 | 0 | 0 | 0 |  |
| Brazilian city zip range | `l10n_br.zip.range` | `l10n_br_zip_range` | Fiscal Localizations | 3 | 0 | 0 | 2 | 0 |  |
| Business Level Responses for Nemhandel | `nemhandel.response` | `nemhandel_response` | Fiscal Localizations | 4 | 0 | 1 | 0 | 0 |  |
| Business Level Responses for Peppol | `account.peppol.response` | `account_peppol_response` | Electronic Invoicing and Document Exchange | 11 | 0 | 2 | 0 | 0 |  |
| Byproduct | `mrp.bom.byproduct` | `mrp_bom_byproduct` | Manufacturing | 8 | 1 | 2 | 0 | 0 |  |
| Calendar Attendee Information | `calendar.attendee` | `calendar_attendee` | Calendar and Scheduling | 6 | 0 | 1 | 0 | 0 |  |
| Calendar Event | `calendar.event` | `calendar_event` | Calendar and Scheduling | 32 | 3 | 10 | 0 | 0 |  |
| Calendar Filters | `calendar.filters` | `calendar_filters` | Calendar and Scheduling | 4 | 0 | 2 | 1 | 0 |  |
| Campaign Stage | `utm.stage` | `utm_stage` | Customer Relationship Management | 2 | 0 | 0 | 0 | 0 |  |
| campaign tracking parameter Campaign | `utm.campaign` | `utm_campaign` | Customer Relationship Management | 13 | 1 | 0 | 1 | 0 |  |
| campaign tracking parameter Medium | `utm.medium` | `utm_medium` | Customer Relationship Management | 2 | 0 | 0 | 1 | 0 |  |
| campaign tracking parameter Source | `utm.source` | `utm_source` | Customer Relationship Management | 1 | 0 | 0 | 1 | 0 |  |
| campaign tracking parameter Tag | `utm.tag` | `utm_tag` | Customer Relationship Management | 2 | 1 | 0 | 1 | 0 |  |
| Canned Response | `mail.canned.response` | `mail_canned_response` | Messaging and Activities | 4 | 1 | 1 | 0 | 0 |  |
| Cashmoves report | `lunch.cashmove.report` | `lunch_cashmove_report` | Lunch Ordering | 5 | 0 | 0 | 0 | 0 | no table: read from a stored query, materialized as a view |
| Category of applicant | `hr.applicant.category` | `hr_applicant_category` | Recruitment | 2 | 3 | 0 | 1 | 0 |  |
| Category of the model | `fleet.vehicle.model.category` | `fleet_vehicle_model_category` | Fleet | 4 | 0 | 0 | 1 | 0 |  |
| Certificate | `certificate.certificate` | `certificate_certificate` | Electronic Invoicing and Document Exchange | 13 | 0 | 0 | 0 | 0 |  |
| Channel / Partners (Members) | `slide.channel.partner` | `slide_channel_partner` | Learning, Surveys and Gamification | 8 | 0 | 2 | 2 | 0 |  |
| Channel Member | `discuss.channel.member` | `discuss_channel_member` | Messaging and Activities | 13 | 0 | 6 | 1 | 3 |  |
| Channel/Course Groups | `slide.channel.tag.group` | `slide_channel_tag_group` | Learning, Surveys and Gamification | 3 | 0 | 2 | 0 | 0 |  |
| Channel/Course Tag | `slide.channel.tag` | `slide_channel_tag` | Learning, Surveys and Gamification | 5 | 1 | 3 | 0 | 0 |  |
| Chatbot Message | `chatbot.message` | `chatbot_message` | Messaging and Activities | 6 | 0 | 2 | 1 | 1 |  |
| Chatbot Script | `chatbot.script` | `chatbot_script` | Messaging and Activities | 4 | 0 | 1 | 0 | 0 |  |
| Chatbot Script Answer | `chatbot.script.answer` | `chatbot_script_answer` | Messaging and Activities | 4 | 1 | 1 | 0 | 0 |  |
| Chatbot Script Step | `chatbot.script.step` | `chatbot_script_step` | Messaging and Activities | 5 | 2 | 2 | 0 | 0 |  |
| City | `res.city` | `res_city` | Contacts and Organizations | 5 | 0 | 0 | 0 | 0 |  |
| Client Action | `ir.actions.client` | `ir_act_client` | Platform Foundation | 12 | 0 | 0 | 0 | 0 |  |
| Cloud Storage Migration Report | `cloud.storage.migration.report` | `cloud_storage_migration_report` | Automation and Integration | 0 | 0 | 0 | 0 | 0 | no table: read from a stored query built at read time |
| Code Translation | `transifex.code.translation` | `transifex_code_translation` | Automation and Integration | 4 | 0 | 0 | 0 | 0 | no audit columns |
| Coins/Bills | `pos.bill` | `pos_bill` | Point of Sale | 2 | 1 | 0 | 0 | 0 |  |
| Collaborators in project shared | `project.collaborator` | `project_collaborator` | Projects and Tasks | 3 | 0 | 0 | 1 | 0 |  |
| Communication Bus | `bus.bus` | `bus_bus` | Messaging and Activities | 2 | 0 | 1 | 0 | 0 |  |
| Companies | `res.company` | `res_company` | Contacts and Organizations | 334 | 5 | 4 | 2 | 0 |  |
| Company directory access protocol configuration | `res.company.ldap` | `res_company_ldap` | Identity and Access | 0 | 0 | 0 | 0 | 0 | no relation in the observed installation |
| Configuration Wizards | `ir.actions.todo` | `ir_actions_todo` | Platform Foundation | 4 | 0 | 1 | 0 | 0 |  |
| Contact | `res.partner` | `res_partner` | Contacts and Organizations | 175 | 30 | 41 | 3 | 0 |  |
| Content Quiz Question | `event.quiz.question` | `event_quiz_question` | Events | 3 | 0 | 1 | 0 | 0 |  |
| Content Quiz Question | `slide.question` | `slide_question` | Learning, Surveys and Gamification | 3 | 0 | 1 | 0 | 0 |  |
| Contract Type | `hr.contract.type` | `hr_contract_type` | Human Resources Core | 4 | 0 | 0 | 0 | 0 |  |
| Copy of a shared dashboard | `spreadsheet.dashboard.share` | `spreadsheet_dashboard_share` | Spreadsheets and Dashboards | 2 | 0 | 0 | 0 | 0 |  |
| Country | `res.country` | `res_country` | Contacts and Organizations | 18 | 8 | 0 | 2 | 0 |  |
| Country Group | `res.country.group` | `res_country_group` | Contacts and Organizations | 2 | 3 | 0 | 1 | 0 |  |
| Country state | `res.country.state` | `res_country_state` | Contacts and Organizations | 4 | 5 | 1 | 1 | 0 |  |
| Course | `slide.channel` | `slide_channel` | Learning, Surveys and Gamification | 48 | 4 | 4 | 3 | 0 |  |
| CPV Code | `l10n_ro.cpv.code` | `l10n_ro_cpv_code` | Fiscal Localizations | 2 | 0 | 0 | 1 | 0 |  |
| Croatian KPD Category | `l10n_hr.kpd.category` | `l10n_hr_kpd_category` | Fiscal Localizations | 3 | 0 | 0 | 0 | 0 |  |
| Croatian tax expence categories | `l10n.hr.tax.category` | `l10n_hr_tax_category` | Fiscal Localizations | 6 | 0 | 0 | 0 | 0 |  |
| Cryptographic Keys | `certificate.key` | `certificate_key` | Electronic Invoicing and Document Exchange | 6 | 0 | 0 | 0 | 0 |  |
| Currency | `res.currency` | `res_currency` | Multi-Currency | 13 | 2 | 0 | 2 | 0 |  |
| Currency Rate | `res.currency.rate` | `res_currency_rate` | Multi-Currency | 4 | 0 | 2 | 2 | 0 |  |
| Custom links that the restaurant can configure to be displayed on the self order screen | `pos_self_order.custom_link` | `pos_self_order_custom_link` | Point of Sale | 5 | 1 | 0 | 0 | 0 |  |
| Custom View | `ir.ui.view.custom` | `ir_ui_view_custom` | Platform Foundation | 3 | 0 | 2 | 0 | 1 |  |
| Customer Alias on Nilvera | `l10n_tr.nilvera.alias` | `l10n_tr_nilvera_alias` | Fiscal Localizations | 2 | 0 | 0 | 0 | 0 |  |
| customer relationship management Activity Analysis | `crm.activity.report` | `crm_activity_report` | Customer Relationship Management | 19 | 0 | 0 | 0 | 0 | no table: read from a stored query, materialized as a view |
| customer relationship management in-app purchase Lead Industry | `crm.iap.lead.industry` | `crm_iap_lead_industry` | Customer Relationship Management | 4 | 2 | 0 | 1 | 0 |  |
| customer relationship management Lead Generation Rules | `crm.reveal.rule` | `crm_reveal_rule` | Customer Relationship Management | 18 | 5 | 0 | 1 | 0 |  |
| customer relationship management Lead Mining Request | `crm.iap.lead.mining.request` | `crm_iap_lead_mining_request` | Customer Relationship Management | 15 | 5 | 0 | 0 | 0 |  |
| customer relationship management Partnership Analysis | `crm.partner.report.assign` | `crm_partner_report_assign` | Customer Relationship Management | 0 | 0 | 0 | 0 | 0 | no table: read from a stored query built at read time |
| customer relationship management Recurring revenue plans | `crm.recurring.plan` | `crm_recurring_plan` | Customer Relationship Management | 4 | 0 | 0 | 1 | 0 |  |
| customer relationship management Reveal View | `crm.reveal.view` | `crm_reveal_view` | Customer Relationship Management | 3 | 0 | 3 | 0 | 2 |  |
| customer relationship management Stages | `crm.stage` | `crm_stage` | Customer Relationship Management | 7 | 1 | 0 | 0 | 0 |  |
| customer relationship management Tag | `crm.tag` | `crm_tag` | Sales | 2 | 5 | 0 | 1 | 0 |  |
| Decimal Precision | `decimal.precision` | `decimal_precision` | Platform Foundation | 2 | 0 | 0 | 1 | 0 |  |
| Declaration of Intent | `l10n_it_edi_doi.declaration_of_intent` | `l10n_it_edi_doi_declaration_of_intent` | Fiscal Localizations | 14 | 0 | 2 | 2 | 0 |  |
| Default Values | `ir.default` | `ir_default` | Platform Foundation | 5 | 0 | 3 | 0 | 0 |  |
| Delivery Price Rules | `delivery.price.rule` | `delivery_price_rule` | Delivery and Shipping | 8 | 0 | 1 | 0 | 0 |  |
| Delivery Zip Prefix | `delivery.zip.prefix` | `delivery_zip_prefix` | Delivery and Shipping | 1 | 1 | 0 | 1 | 0 |  |
| Department | `hr.department` | `hr_department` | Human Resources Core | 9 | 2 | 3 | 0 | 0 |  |
| Departure Reason | `hr.departure.reason` | `hr_departure_reason` | Human Resources Core | 3 | 0 | 0 | 0 | 0 |  |
| Device Log | `res.device.log` | `res_device_log` | Identity and Access | 11 | 0 | 3 | 0 | 2 |  |
| Devices | `res.device` | `res_device` | Identity and Access | 11 | 0 | 0 | 0 | 0 | no table: read from a stored query, materialized as a view |
| Digest | `digest.digest` | `digest_digest` | Messaging and Activities | 18 | 1 | 0 | 0 | 0 |  |
| Digest Tips | `digest.tip` | `digest_tip` | Messaging and Activities | 4 | 1 | 0 | 0 | 0 |  |
| Discussion Channel | `discuss.channel` | `discuss_channel` | Messaging and Activities | 32 | 7 | 5 | 5 | 5 |  |
| District | `l10n_pe.res.city.district` | `l10n_pe_res_city_district` | Fiscal Localizations | 3 | 0 | 0 | 0 | 0 |  |
| Document Followers | `mail.followers` | `mail_followers` | Messaging and Activities | 3 | 1 | 3 | 1 | 0 | no audit columns |
| Document object for tracking CIUS-RO extensible markup language sent to E-Factura | `l10n_ro_edi.document` | `l10n_ro_edi_document` | Fiscal Localizations | 11 | 0 | 0 | 0 | 0 |  |
| Drivers history on a vehicle | `fleet.vehicle.assignation.log` | `fleet_vehicle_assignation_log` | Fleet | 5 | 0 | 1 | 0 | 0 |  |
| E-Faktur Document | `l10n_id_efaktur_coretax.document` | `l10n_id_efaktur_coretax_document` | Fiscal Localizations | 5 | 0 | 0 | 0 | 0 |  |
| Easily load a set of configuration options | `pos.preset` | `pos_preset` | Point of Sale | 14 | 1 | 0 | 0 | 0 |  |
| Electronic Commerce Extra Info Shown on product page | `website.sale.extra.field` | `website_sale_extra_field` | Website and Storefront | 3 | 0 | 1 | 0 | 0 |  |
| electronic data interchange and fiscalization information for Croatian electronic invoicing | `l10n_hr_edi.addendum` | `l10n_hr_edi_addendum` | Fiscal Localizations | 14 | 0 | 1 | 0 | 0 |  |
| electronic data interchange format | `account.edi.format` | `account_edi_format` | Electronic Invoicing and Document Exchange | 2 | 1 | 0 | 1 | 0 |  |
| Electronic Document for an account.move | `account.edi.document` | `account_edi_document` | Electronic Invoicing and Document Exchange | 6 | 0 | 1 | 1 | 0 |  |
| Electronic Waybill | `l10n.in.ewaybill` | `l10n_in_ewaybill` | Fiscal Localizations | 27 | 0 | 0 | 0 | 0 |  |
| Electronic Waybill Document Type | `l10n.in.ewaybill.type` | `l10n_in_ewaybill_type` | Fiscal Localizations | 6 | 0 | 0 | 0 | 0 |  |
| Email Aliases | `mail.alias` | `mail_alias` | Messaging and Activities | 12 | 0 | 1 | 0 | 1 |  |
| Email Domain | `mail.alias.domain` | `mail_alias_domain` | Messaging and Activities | 5 | 0 | 0 | 2 | 0 |  |
| Email Templates | `mail.template` | `mail_template` | Messaging and Activities | 21 | 4 | 2 | 0 | 0 |  |
| Embedded Actions | `ir.embedded.actions` | `ir_embedded_actions` | Platform Foundation | 11 | 1 | 0 | 2 | 0 |  |
| Embedded Slides View Counter | `slide.embed` | `slide_embed` | Learning, Surveys and Gamification | 3 | 0 | 1 | 0 | 0 |  |
| Employee | `hr.employee` | `hr_employee` | Human Resources Core | 58 | 13 | 6 | 2 | 0 |  |
| Employee Category | `hr.employee.category` | `hr_employee_category` | Human Resources Core | 2 | 1 | 0 | 1 | 0 |  |
| Employee Certification Report | `hr.employee.certification.report` | `hr_employee_certification_report` | Human Resources Core | 8 | 0 | 0 | 0 | 0 | no table: read from a stored query, materialized as a view |
| Employee Location | `hr.employee.location` | `hr_employee_location` | Human Resources Core | 3 | 0 | 0 | 1 | 0 |  |
| Employee Skills Report | `hr.employee.skill.history.report` | `hr_employee_skill_history_report` | Human Resources Core | 5 | 0 | 0 | 0 | 0 | no table: read from a stored query, materialized as a view |
| Employee Skills Report | `hr.employee.skill.report` | `hr_employee_skill_report` | Human Resources Core | 8 | 0 | 0 | 0 | 0 | no table: read from a stored query, materialized as a view |
| Estimated Time of Arrival code for activity type | `l10n_eg_edi.activity.type` | `l10n_eg_edi_activity_type` | Fiscal Localizations | 2 | 0 | 0 | 0 | 0 |  |
| Estimated Time of Arrival code for the unit of measures | `l10n_eg_edi.uom.code` | `l10n_eg_edi_uom_code` | Fiscal Localizations | 2 | 0 | 0 | 0 | 0 |  |
| Event | `event.event` | `event_event` | Events | 45 | 4 | 2 | 0 | 0 |  |
| Event Alarm | `calendar.alarm` | `calendar_alarm` | Calendar and Scheduling | 9 | 1 | 0 | 0 | 0 |  |
| Event Automated Mailing | `event.mail` | `event_mail` | Events | 11 | 0 | 1 | 0 | 0 |  |
| Event Booth | `event.booth` | `event_booth` | Events | 14 | 1 | 5 | 0 | 0 |  |
| Event Booth Category | `event.booth.category` | `event_booth_category` | Events | 9 | 0 | 0 | 0 | 0 |  |
| Event Booth Registration | `event.booth.registration` | `event_booth_registration` | Events | 11 | 0 | 2 | 1 | 0 |  |
| Event Booth Template | `event.type.booth` | `event_type_booth` | Events | 4 | 0 | 2 | 0 | 0 |  |
| Event Lead Request | `event.lead.request` | `event_lead_request` | Customer Relationship Management | 2 | 1 | 0 | 1 | 0 | no audit columns |
| Event Lead Rules | `event.lead.rule` | `event_lead_rule` | Customer Relationship Management | 10 | 3 | 0 | 0 | 0 |  |
| Event Meeting Type | `calendar.event.type` | `calendar_event_type` | Calendar and Scheduling | 2 | 1 | 0 | 1 | 0 |  |
| Event Question | `event.question` | `event_question` | Events | 8 | 2 | 0 | 1 | 0 |  |
| Event Question Answer | `event.question.answer` | `event_question_answer` | Events | 3 | 0 | 1 | 0 | 0 |  |
| Event Recurrence Rule | `calendar.recurrence` | `calendar_recurrence` | Calendar and Scheduling | 27 | 0 | 3 | 1 | 0 |  |
| Event Registration | `event.registration` | `event_registration` | Events | 22 | 1 | 11 | 1 | 0 |  |
| Event Registration Answer | `event.registration.answer` | `event_registration_answer` | Events | 4 | 0 | 1 | 1 | 0 |  |
| Event Sales Report | `event.sale.report` | `event_sale_report` | Events | 25 | 0 | 0 | 0 | 0 | no table: read from a stored query, materialized as a view |
| Event Slot | `event.slot` | `event_slot` | Events | 7 | 0 | 1 | 0 | 0 |  |
| Event Sponsor | `event.sponsor` | `event_sponsor` | Events | 16 | 0 | 2 | 0 | 0 |  |
| Event Sponsor Level | `event.sponsor.type` | `event_sponsor_type` | Events | 3 | 0 | 0 | 0 | 0 |  |
| Event Stage | `event.stage` | `event_stage` | Events | 5 | 0 | 0 | 0 | 0 |  |
| Event Tag | `event.tag` | `event_tag` | Events | 7 | 2 | 3 | 0 | 0 |  |
| Event Tag Category | `event.tag.category` | `event_tag_category` | Events | 4 | 0 | 2 | 0 | 0 |  |
| Event Template | `event.type` | `event_type` | Events | 13 | 3 | 0 | 0 | 0 |  |
| Event Template Ticket | `event.type.ticket` | `event_type_ticket` | Events | 8 | 0 | 1 | 0 | 0 |  |
| Event Ticket | `event.event.ticket` | `event_event_ticket` | Events | 13 | 0 | 2 | 0 | 0 |  |
| Event Track | `event.track` | `event_track` | Events | 38 | 1 | 3 | 0 | 0 |  |
| Event Track Location | `event.track.location` | `event_track_location` | Events | 2 | 0 | 0 | 0 | 0 |  |
| Event Track Stage | `event.track.stage` | `event_track_stage` | Events | 12 | 0 | 0 | 0 | 0 |  |
| Event Track Tag | `event.track.tag` | `event_track_tag` | Events | 4 | 3 | 1 | 1 | 0 |  |
| Event Track Tag Category | `event.track.tag.category` | `event_track_tag_category` | Events | 2 | 0 | 0 | 0 | 0 |  |
| Expense | `hr.expense` | `hr_expense` | Expenses | 32 | 3 | 6 | 0 | 0 |  |
| Exports | `ir.exports` | `ir_exports` | Platform Foundation | 2 | 0 | 1 | 0 | 0 |  |
| Exports Line | `ir.exports.line` | `ir_exports_line` | Platform Foundation | 2 | 0 | 1 | 0 | 0 |  |
| Fields | `ir.model.fields` | `ir_model_fields` | Platform Foundation | 41 | 4 | 5 | 3 | 0 |  |
| Fields Selection | `ir.model.fields.selection` | `ir_model_fields_selection` | Platform Foundation | 4 | 0 | 1 | 1 | 0 |  |
| Fields that can be used for predictive lead scoring computation | `crm.lead.scoring.frequency.field` | `crm_lead_scoring_frequency_field` | Customer Relationship Management | 2 | 1 | 0 | 0 | 0 |  |
| Filters | `ir.filters` | `ir_filters` | Platform Foundation | 10 | 1 | 1 | 2 | 1 |  |
| Fiscal Position | `account.fiscal.position` | `account_fiscal_position` | General Ledger | 14 | 5 | 1 | 0 | 0 |  |
| Fleet Analysis Report | `fleet.vehicle.cost.report` | `fleet_vehicle_cost_report` | Fleet | 9 | 0 | 0 | 0 | 0 | no table: read from a stored query, materialized as a view |
| Fleet Odometer Analysis Report | `fleet.vehicle.odometer.report` | `fleet_vehicle_odometer_report` | Fleet | 4 | 0 | 0 | 0 | 0 | no table: read from a stored query, materialized as a view |
| Fleet Service Type | `fleet.service.type` | `fleet_service_type` | Fleet | 2 | 1 | 0 | 0 | 0 |  |
| Form fields of inside quotation documents. | `sale.pdf.form.field` | `sale_pdf_form_field` | Sales | 3 | 2 | 0 | 1 | 0 |  |
| Forum | `forum.forum` | `forum_forum` | Website and Storefront | 55 | 0 | 1 | 0 | 0 |  |
| Forum Post | `forum.post` | `forum_post` | Website and Storefront | 27 | 2 | 6 | 0 | 0 |  |
| Forum Tag | `forum.tag` | `forum_tag` | Website and Storefront | 10 | 1 | 1 | 1 | 0 |  |
| French Approved Dematerialization Platform Flow | `l10n.fr.pdp.reports.flow` | `l10n_fr_pdp_reports_flow` | Fiscal Localizations | 15 | 1 | 0 | 0 | 0 |  |
| Full Reconcile | `account.full.reconcile` | `account_full_reconcile` | General Ledger | 0 | 0 | 0 | 0 | 0 |  |
| Gamification Badge | `gamification.badge` | `gamification_badge` | Learning, Surveys and Gamification | 9 | 3 | 1 | 0 | 0 |  |
| Gamification Challenge | `gamification.challenge` | `gamification_challenge` | Learning, Surveys and Gamification | 22 | 2 | 1 | 0 | 0 |  |
| Gamification generic goal for challenge | `gamification.challenge.line` | `gamification_challenge_line` | Learning, Surveys and Gamification | 4 | 0 | 1 | 0 | 0 |  |
| Gamification Goal | `gamification.goal` | `gamification_goal` | Learning, Surveys and Gamification | 13 | 0 | 2 | 0 | 0 |  |
| Gamification Goal Definition | `gamification.goal.definition` | `gamification_goal_definition` | Learning, Surveys and Gamification | 17 | 1 | 0 | 0 | 0 |  |
| Gamification User Badge | `gamification.badge.user` | `gamification_badge_user` | Learning, Surveys and Gamification | 7 | 0 | 3 | 0 | 0 |  |
| Geo Provider | `base.geo_provider` | `base_geo_provider` | Platform Foundation | 2 | 0 | 0 | 0 | 0 |  |
| GİB Plate numbers | `l10n_tr.nilvera.trailer.plate` | `l10n_tr_nilvera_trailer_plate` | Fiscal Localizations | 2 | 1 | 0 | 1 | 0 |  |
| Greece document object for tracking all sent extensible markup language to myDATA | `l10n_gr_edi.document` | `l10n_gr_edi_document` | Fiscal Localizations | 15 | 0 | 0 | 0 | 0 |  |
| Group of dashboards | `spreadsheet.dashboard.group` | `spreadsheet_dashboard_group` | Spreadsheets and Dashboards | 2 | 0 | 0 | 0 | 0 |  |
| Guest | `mail.guest` | `mail_guest` | Messaging and Activities | 6 | 0 | 0 | 0 | 0 |  |
| Helper methods for crm_iap_mine modules | `crm.iap.lead.helpers` | `crm_iap_lead_helpers` | Customer Relationship Management | 0 | 0 | 0 | 0 | 0 |  |
| History for Loyalty cards and Electronic Wallets | `loyalty.history` | `loyalty_history` | Loyalty and Promotions | 6 | 0 | 1 | 0 | 0 |  |
| human resources Work Entry | `hr.work.entry` | `hr_work_entry` | Work Entries | 13 | 0 | 3 | 0 | 1 |  |
| human resources Work Entry Type | `hr.work.entry.type` | `hr_work_entry_type` | Work Entries | 11 | 0 | 0 | 0 | 0 |  |
| ICE Server | `mail.ice.server` | `mail_ice_server` | Messaging and Activities | 4 | 0 | 0 | 0 | 0 |  |
| Identification Types | `l10n_latam.identification.type` | `l10n_latam_identification_type` | Fiscal Localizations | 10 | 0 | 0 | 0 | 0 |  |
| in-app purchase Account | `iap.account` | `iap_account` | Automation and Integration | 8 | 2 | 0 | 0 | 0 |  |
| in-app purchase Service | `iap.service` | `iap_service` | Automation and Integration | 5 | 0 | 0 | 1 | 0 |  |
| Incoming Mail Server | `fetchmail.server` | `fetchmail_server` | Messaging and Activities | 24 | 0 | 2 | 0 | 0 |  |
| Incoterms | `account.incoterms` | `account_incoterms` | General Ledger | 3 | 0 | 0 | 0 | 0 |  |
| Indian permanent account number Entity | `l10n_in.pan.entity` | `l10n_in_pan_entity` | Fiscal Localizations | 6 | 0 | 0 | 1 | 0 |  |
| Indian port code | `l10n_in.port.code` | `l10n_in_port_code` | Fiscal Localizations | 3 | 0 | 0 | 1 | 0 |  |
| indian section alert | `l10n_in.section.alert` | `l10n_in_section_alert` | Fiscal Localizations | 9 | 0 | 0 | 2 | 0 |  |
| Industry | `res.partner.industry` | `res_partner_industry` | Contacts and Organizations | 3 | 0 | 0 | 0 | 0 |  |
| Inventory Locations | `stock.location` | `stock_location` | Inventory Operations | 16 | 2 | 5 | 2 | 1 |  |
| Inventory Routes | `stock.route` | `stock_route` | Inventory Operations | 12 | 8 | 2 | 0 | 0 |  |
| Invoices Statistics | `account.invoice.report` | `account_invoice_report` | General Ledger | 0 | 0 | 0 | 0 | 0 | no table: read from a stored query built at read time |
| Italian Document Type | `l10n_it.document.type` | `l10n_it_document_type` | Fiscal Localizations | 3 | 0 | 0 | 0 | 0 |  |
| Job Platforms | `hr.job.platform` | `hr_job_platform` | Recruitment | 3 | 0 | 0 | 1 | 0 |  |
| Job Position | `hr.job` | `hr_job` | Human Resources Core | 31 | 7 | 5 | 2 | 0 |  |
| Journal | `account.journal` | `account_journal` | General Ledger | 64 | 4 | 2 | 1 | 0 |  |
| Journal Entry | `account.move` | `account_move` | General Ledger | 249 | 20 | 32 | 1 | 10 |  |
| Journal Item | `account.move.line` | `account_move_line` | General Ledger | 70 | 5 | 19 | 4 | 5 |  |
| Keep the call history | `discuss.call.history` | `discuss_call_history` | Messaging and Activities | 4 | 1 | 3 | 3 | 1 |  |
| Keep the channel member history | `im_livechat.channel.member.history` | `im_livechat_channel_member_history` | Messaging and Activities | 13 | 2 | 5 | 2 | 2 |  |
| KRA defined codes that justify a given tax rate / exemption | `l10n_ke.item.code` | `l10n_ke_item_code` | Fiscal Localizations | 3 | 0 | 0 | 0 | 0 |  |
| l10n_ar.earnings.scale | `l10n_ar.earnings.scale` | `l10n_ar_earnings_scale` | Fiscal Localizations | 1 | 0 | 0 | 0 | 0 |  |
| l10n_ar.earnings.scale.line | `l10n_ar.earnings.scale.line` | `l10n_ar_earnings_scale_line` | Fiscal Localizations | 5 | 0 | 0 | 0 | 0 |  |
| Languages | `res.lang` | `res_lang` | Contacts and Organizations | 12 | 5 | 0 | 3 | 0 |  |
| Latam Document Type | `l10n_latam.document.type` | `l10n_latam_document_type` | Fiscal Localizations | 12 | 0 | 1 | 0 | 0 |  |
| Lead | `crm.lead` | `crm_lead` | Customer Relationship Management | 69 | 9 | 27 | 1 | 3 |  |
| Lead Scoring Frequency | `crm.lead.scoring.frequency` | `crm_lead_scoring_frequency` | Customer Relationship Management | 5 | 0 | 1 | 0 | 0 |  |
| Link between link previews and messages | `mail.message.link.preview` | `mail_message_link_preview` | Messaging and Activities | 4 | 0 | 2 | 0 | 1 |  |
| Link between products and their Units of Measure | `product.uom` | `product_uom` | Products and Catalog | 4 | 0 | 3 | 1 | 0 |  |
| Link text message to mailing/text message tracking models | `sms.tracker` | `sms_tracker` | Messaging and Activities | 4 | 0 | 2 | 1 | 0 |  |
| Link Tracker | `link.tracker` | `link_tracker` | Marketing and Mass Mailing | 8 | 0 | 3 | 0 | 0 |  |
| Link Tracker Click | `link.tracker.click` | `link_tracker_click` | Marketing and Mass Mailing | 6 | 0 | 4 | 0 | 0 |  |
| Link Tracker Code | `link.tracker.code` | `link_tracker_code` | Marketing and Mass Mailing | 2 | 0 | 1 | 1 | 0 |  |
| Live Chat Conversation Tags | `im_livechat.conversation.tag` | `im_livechat_conversation_tag` | Messaging and Activities | 2 | 1 | 0 | 0 | 1 |  |
| Live Chat Expertise | `im_livechat.expertise` | `im_livechat_expertise` | Messaging and Activities | 1 | 4 | 0 | 0 | 1 |  |
| Livechat Channel | `im_livechat.channel` | `im_livechat_channel` | Messaging and Activities | 11 | 1 | 0 | 1 | 0 |  |
| Livechat Channel Rules | `im_livechat.channel.rule` | `im_livechat_channel_rule` | Messaging and Activities | 7 | 1 | 1 | 0 | 0 |  |
| Livechat Support Channel Report | `im_livechat.report.channel` | `im_livechat_report_channel` | Messaging and Activities | 0 | 0 | 0 | 0 | 0 | no table: read from a stored query built at read time |
| Logging | `ir.logging` | `ir_logging` | Platform Foundation | 8 | 0 | 3 | 0 | 0 |  |
| Lot/Serial | `stock.lot` | `stock_lot` | Inventory Operations | 13 | 2 | 4 | 0 | 0 |  |
| Loyalty Communication | `loyalty.mail` | `loyalty_mail` | Loyalty and Promotions | 6 | 0 | 1 | 0 | 0 |  |
| Loyalty Coupon | `loyalty.card` | `loyalty_card` | Loyalty and Promotions | 9 | 1 | 2 | 1 | 0 |  |
| Loyalty Program | `loyalty.program` | `loyalty_program` | Loyalty and Promotions | 18 | 2 | 1 | 1 | 0 |  |
| Loyalty Reward | `loyalty.reward` | `loyalty_reward` | Loyalty and Promotions | 18 | 2 | 1 | 3 | 0 |  |
| Loyalty Rule | `loyalty.rule` | `loyalty_rule` | Loyalty and Promotions | 16 | 2 | 1 | 1 | 0 |  |
| Lunch Alert | `lunch.alert` | `lunch_alert` | Lunch Ordering | 17 | 1 | 0 | 1 | 0 |  |
| Lunch Cashmove | `lunch.cashmove` | `lunch_cashmove` | Lunch Ordering | 5 | 0 | 0 | 0 | 0 |  |
| Lunch Extras | `lunch.topping` | `lunch_topping` | Lunch Ordering | 5 | 1 | 1 | 0 | 0 |  |
| Lunch Locations | `lunch.location` | `lunch_location` | Lunch Ordering | 3 | 2 | 0 | 0 | 0 |  |
| Lunch Order | `lunch.order` | `lunch_order` | Lunch Ordering | 15 | 1 | 2 | 0 | 1 |  |
| Lunch Product | `lunch.product` | `lunch_product` | Lunch Ordering | 8 | 1 | 0 | 0 | 0 |  |
| Lunch Product Category | `lunch.product.category` | `lunch_product_category` | Lunch Ordering | 3 | 0 | 0 | 0 | 0 |  |
| Lunch Supplier | `lunch.supplier` | `lunch_supplier` | Lunch Ordering | 24 | 1 | 0 | 1 | 0 |  |
| Mail Blacklist | `mail.blacklist` | `mail_blacklist` | Messaging and Activities | 3 | 0 | 1 | 1 | 0 |  |
| Mail Gateway Allowed | `mail.gateway.allowed` | `mail_gateway_allowed` | Messaging and Activities | 2 | 0 | 1 | 0 | 0 |  |
| Mail Group | `mail.group` | `mail_group` | Messaging and Activities | 12 | 1 | 0 | 0 | 0 |  |
| Mail RTC session | `discuss.channel.rtc.session` | `discuss_channel_rtc_session` | Messaging and Activities | 7 | 0 | 3 | 1 | 0 |  |
| Mail Scheduling on Event Category | `event.type.mail` | `event_type_mail` | Events | 5 | 0 | 0 | 0 | 0 |  |
| Mail Server | `ir.mail_server` | `ir_mail_server` | Platform Foundation | 23 | 0 | 1 | 2 | 0 |  |
| Mail Tracking Value | `mail.tracking.value` | `mail_tracking_value` | Messaging and Activities | 14 | 0 | 2 | 0 | 0 |  |
| Mailing Contact | `mailing.contact` | `mailing_contact` | Marketing and Mass Mailing | 11 | 2 | 0 | 0 | 0 |  |
| Mailing Favorite Filters | `mailing.filter` | `mailing_filter` | Marketing and Mass Mailing | 3 | 0 | 1 | 0 | 0 |  |
| Mailing List | `mailing.list` | `mailing_list` | Marketing and Mass Mailing | 3 | 4 | 0 | 0 | 0 |  |
| Mailing List black/white list | `mail.group.moderation` | `mail_group_moderation` | Messaging and Activities | 3 | 0 | 1 | 1 | 0 |  |
| Mailing List Member | `mail.group.member` | `mail_group_member` | Messaging and Activities | 4 | 0 | 2 | 1 | 0 |  |
| Mailing List Message | `mail.group.message` | `mail_group_message` | Messaging and Activities | 6 | 0 | 4 | 0 | 0 |  |
| Mailing List Subscription | `mailing.subscription` | `mailing_subscription` | Marketing and Mass Mailing | 5 | 0 | 1 | 1 | 0 |  |
| Mailing Statistics | `mailing.trace` | `mailing_trace` | Marketing and Mass Mailing | 20 | 0 | 5 | 1 | 0 |  |
| Mailing Subscription Reason | `mailing.subscription.optout` | `mailing_subscription_optout` | Marketing and Mass Mailing | 3 | 0 | 0 | 0 | 0 |  |
| Maintenance Equipment | `maintenance.equipment` | `maintenance_equipment` | Repair and Maintenance | 26 | 0 | 4 | 1 | 0 |  |
| Maintenance Equipment Category | `maintenance.equipment.category` | `maintenance_equipment_category` | Repair and Maintenance | 7 | 0 | 0 | 0 | 0 |  |
| Maintenance Request | `maintenance.request` | `maintenance_request` | Repair and Maintenance | 29 | 0 | 3 | 0 | 0 |  |
| Maintenance Stage | `maintenance.stage` | `maintenance_stage` | Repair and Maintenance | 4 | 0 | 0 | 0 | 0 |  |
| Maintenance Teams | `maintenance.team` | `maintenance_team` | Repair and Maintenance | 5 | 1 | 0 | 0 | 0 |  |
| Malaysian Industry Classification | `l10n_my_edi.industry_classification` | `l10n_my_edi_industry_classification` | Fiscal Localizations | 2 | 0 | 0 | 0 | 0 |  |
| Mandatory Day | `hr.leave.mandatory.day` | `hr_leave_mandatory_day` | Time Off | 6 | 2 | 0 | 1 | 0 |  |
| Manufacturing Order | `mrp.production` | `mrp_production` | Manufacturing | 36 | 11 | 7 | 2 | 0 |  |
| manufacturing Workorder productivity losses | `mrp.workcenter.productivity.loss.type` | `mrp_workcenter_productivity_loss_type` | Manufacturing | 1 | 0 | 0 | 0 | 0 |  |
| Mapping of account codes per company | `account.code.mapping` | `account_code_mapping` | General Ledger | 0 | 0 | 0 | 0 | 0 | no table: read from a stored query built at read time |
| Marketing Card | `card.card` | `card_card` | Marketing and Mass Mailing | 5 | 0 | 1 | 1 | 0 |  |
| Marketing Card Campaign | `card.campaign` | `card_campaign` | Marketing and Mass Mailing | 35 | 1 | 0 | 0 | 0 |  |
| Marketing Card Campaign Tag | `card.campaign.tag` | `card_campaign_tag` | Marketing and Mass Mailing | 2 | 1 | 0 | 1 | 0 |  |
| Marketing Card Template | `card.template` | `card_template` | Marketing and Mass Mailing | 6 | 0 | 0 | 0 | 0 |  |
| Mass Mailing | `mailing.mailing` | `mailing_mailing` | Marketing and Mass Mailing | 36 | 2 | 3 | 2 | 0 |  |
| Mass Mailing Statistics | `mailing.trace.report` | `mailing_trace_report` | Marketing and Mass Mailing | 17 | 0 | 0 | 0 | 0 | no table: read from a stored query, materialized as a view |
| Menu | `ir.ui.menu` | `ir_ui_menu` | Platform Foundation | 7 | 1 | 2 | 0 | 0 |  |
| Message | `mail.message` | `mail_message` | Messaging and Activities | 25 | 3 | 5 | 0 | 3 |  |
| Message Notifications | `mail.notification` | `mail_notification` | Messaging and Activities | 14 | 0 | 8 | 2 | 3 | no audit columns |
| Message Reaction | `mail.message.reaction` | `mail_message_reaction` | Messaging and Activities | 4 | 0 | 1 | 1 | 2 | no audit columns |
| Message subtypes | `mail.message.subtype` | `mail_message_subtype` | Messaging and Activities | 10 | 1 | 0 | 0 | 0 |  |
| Message Translation | `mail.message.translation` | `mail_message_translation` | Messaging and Activities | 4 | 0 | 1 | 0 | 1 |  |
| Metadata for voice attachments | `discuss.voice.metadata` | `discuss_voice_metadata` | Messaging and Activities | 1 | 0 | 1 | 0 | 0 |  |
| Minimum Inventory Rule | `stock.warehouse.orderpoint` | `stock_warehouse_orderpoint` | Inventory Operations | 17 | 1 | 4 | 1 | 0 |  |
| Model Access | `ir.model.access` | `ir_model_access` | Platform Foundation | 8 | 0 | 3 | 0 | 0 |  |
| Model Constraint | `ir.model.constraint` | `ir_model_constraint` | Platform Foundation | 6 | 0 | 3 | 1 | 0 |  |
| Model Data | `ir.model.data` | `ir_model_data` | Platform Foundation | 5 | 0 | 0 | 1 | 2 |  |
| Model Inheritance Tree | `ir.model.inherit` | `ir_model_inherit` | Platform Foundation | 3 | 0 | 0 | 1 | 0 | no audit columns |
| Model of a vehicle | `fleet.vehicle.model` | `fleet_vehicle_model` | Fleet | 23 | 1 | 1 | 0 | 0 |  |
| Model Page | `website.controller.page` | `website_controller_page` | Website and Storefront | 8 | 0 | 3 | 1 | 0 |  |
| Models | `ir.model` | `ir_model` | Platform Foundation | 15 | 1 | 0 | 1 | 0 |  |
| Module | `ir.module.module` | `ir_module_module` | Platform Foundation | 26 | 3 | 2 | 1 | 0 |  |
| Module dependency | `ir.module.module.dependency` | `ir_module_module_dependency` | Platform Foundation | 3 | 0 | 1 | 0 | 0 | no audit columns |
| Module exclusion | `ir.module.module.exclusion` | `ir_module_module_exclusion` | Platform Foundation | 2 | 0 | 1 | 0 | 0 |  |
| Multi Website Published Mixin | `website.published.multi.mixin` | `website_published_multi_mixin` | Website and Storefront | 0 | 0 | 0 | 0 | 0 | no relation in the observed installation |
| MyInvois Document | `myinvois.document` | `myinvois_document` | Fiscal Localizations | 18 | 2 | 2 | 0 | 0 |  |
| OAuth2 provider | `auth.oauth.provider` | `auth_oauth_provider` | Identity and Access | 10 | 0 | 0 | 0 | 0 |  |
| Odometer log for a vehicle | `fleet.vehicle.odometer` | `fleet_vehicle_odometer` | Fleet | 5 | 0 | 0 | 0 | 0 |  |
| Onboarding | `onboarding.onboarding` | `onboarding_onboarding` | Automation and Integration | 5 | 1 | 0 | 1 | 0 |  |
| Onboarding Progress Step Tracker | `onboarding.progress.step` | `onboarding_progress_step` | Automation and Integration | 3 | 1 | 1 | 0 | 1 |  |
| Onboarding Progress Tracker | `onboarding.progress` | `onboarding_progress` | Automation and Integration | 4 | 1 | 1 | 0 | 1 |  |
| Onboarding Step | `onboarding.onboarding.step` | `onboarding_onboarding_step` | Automation and Integration | 10 | 1 | 0 | 0 | 0 |  |
| Opp. Lost Reason | `crm.lost.reason` | `crm_lost_reason` | Customer Relationship Management | 2 | 0 | 0 | 0 | 0 |  |
| Optional Holidays | `l10n.in.hr.leave.optional.holiday` | `l10n_in_hr_leave_optional_holiday` | Time Off | 3 | 0 | 0 | 0 | 0 |  |
| Outgoing Mails | `mail.mail` | `mail_mail` | Messaging and Activities | 14 | 1 | 2 | 0 | 0 |  |
| Outgoing text message | `sms.sms` | `sms_sms` | Messaging and Activities | 10 | 0 | 1 | 1 | 0 |  |
| Overtime Rule | `hr.attendance.overtime.rule` | `hr_attendance_overtime_rule` | Attendances and Working Time | 17 | 1 | 1 | 2 | 0 |  |
| Overtime Ruleset | `hr.attendance.overtime.ruleset` | `hr_attendance_overtime_ruleset` | Attendances and Working Time | 6 | 0 | 0 | 0 | 0 |  |
| Package | `stock.package` | `stock_package` | Inventory Operations | 10 | 1 | 7 | 0 | 0 |  |
| Page | `website.page` | `website_page` | Website and Storefront | 13 | 0 | 4 | 0 | 0 |  |
| Paper Format Config | `report.paperformat` | `report_paperformat` | Platform Foundation | 15 | 0 | 0 | 0 | 0 |  |
| Partial Reconcile | `account.partial.reconcile` | `account_partial_reconcile` | General Ledger | 12 | 0 | 4 | 0 | 0 |  |
| Partner Activation | `res.partner.activation` | `res_partner_activation` | Customer Relationship Management | 3 | 0 | 0 | 0 | 0 |  |
| Partner Grade | `res.partner.grade` | `res_partner_grade` | Customer Relationship Management | 7 | 0 | 1 | 0 | 0 |  |
| Partner in-app purchase | `res.partner.iap` | `res_partner_iap` | Messaging and Activities | 3 | 0 | 0 | 1 | 0 |  |
| Partner Tags | `res.partner.category` | `res_partner_category` | Contacts and Organizations | 5 | 3 | 2 | 0 | 0 |  |
| Partner Tags - These tags can be used on website to find customers by sector, or ... | `res.partner.tag` | `res_partner_tag` | Website and Storefront | 4 | 1 | 1 | 0 | 0 |  |
| Passkey | `auth.passkey.key` | `auth_passkey_key` | Identity and Access | 4 | 0 | 1 | 1 | 0 |  |
| Payment Method | `payment.method` | `payment_method` | Payment Providers | 10 | 3 | 1 | 0 | 0 |  |
| Payment Methods | `account.payment.method` | `account_payment_method` | General Ledger | 3 | 0 | 0 | 1 | 0 |  |
| Payment Methods | `account.payment.method.line` | `account_payment_method_line` | General Ledger | 7 | 1 | 1 | 0 | 0 |  |
| Payment Provider | `payment.provider` | `payment_provider` | Payment Providers | 100 | 4 | 1 | 1 | 0 |  |
| Payment Terms | `account.payment.term` | `account_payment_term` | General Ledger | 10 | 0 | 0 | 0 | 0 |  |
| Payment Terms Line | `account.payment.term.line` | `account_payment_term_line` | General Ledger | 6 | 0 | 1 | 0 | 0 |  |
| Payment Token | `payment.token` | `payment_token` | Payment Providers | 14 | 0 | 2 | 0 | 0 |  |
| Payment Transaction | `payment.transaction` | `payment_transaction` | Payment Providers | 32 | 3 | 5 | 1 | 0 |  |
| Payment withholding line | `account.payment.withholding.line` | `account_payment_withholding_line` | Taxes | 17 | 0 | 0 | 0 | 0 |  |
| Payments | `account.payment` | `account_payment` | General Ledger | 35 | 3 | 9 | 1 | 2 |  |
| People Role | `crm.iap.lead.role` | `crm_iap_lead_role` | Customer Relationship Management | 3 | 2 | 0 | 1 | 0 |  |
| People Seniority | `crm.iap.lead.seniority` | `crm_iap_lead_seniority` | Customer Relationship Management | 2 | 0 | 0 | 1 | 0 |  |
| Peppol clarifications used for rejection | `account.peppol.clarification` | `account_peppol_clarification` | Electronic Invoicing and Document Exchange | 4 | 2 | 0 | 0 | 0 |  |
| Personal Filters on Employees for the Calendar view | `account.analytic.line.calendar.employee` | `account_analytic_line_calendar_employee` | Timesheets | 4 | 0 | 0 | 0 | 0 |  |
| Personal Task Stage | `project.task.stage.personal` | `project_task_user_rel` | Projects and Tasks | 3 | 0 | 2 | 1 | 0 |  |
| Phone Blacklist | `phone.blacklist` | `phone_blacklist` | Customer Relationship Management | 2 | 0 | 0 | 1 | 0 |  |
| Picking Type | `stock.picking.type` | `stock_picking_type` | Inventory Operations | 75 | 4 | 2 | 0 | 0 |  |
| PL Bank Account Verification | `l10n_pl.bank.account.verification` | `l10n_pl_bank_account_verification` | Fiscal Localizations | 8 | 0 | 0 | 0 | 0 |  |
| Point of Sale Category | `pos.category` | `pos_category` | Point of Sale | 6 | 4 | 1 | 0 | 0 |  |
| Point of Sale Configuration | `pos.config` | `pos_config` | Point of Sale | 92 | 20 | 1 | 0 | 0 |  |
| Point of Sale Note | `pos.note` | `pos_note` | Point of Sale | 3 | 1 | 0 | 1 | 0 |  |
| Point of Sale Order Lines | `pos.order.line` | `pos_order_line` | Point of Sale | 36 | 2 | 8 | 1 | 0 |  |
| Point of Sale Orders | `pos.order` | `pos_order` | Point of Sale | 71 | 3 | 8 | 1 | 0 |  |
| Point of Sale Orders Report | `report.pos.order` | `report_pos_order` | Platform Foundation | 26 | 0 | 0 | 0 | 0 | no table: read from a stored query, materialized as a view |
| Point of Sale Payment Methods | `pos.payment.method` | `pos_payment_method` | Point of Sale | 76 | 3 | 1 | 0 | 0 |  |
| Point of Sale Payments | `pos.payment` | `pos_payment` | Point of Sale | 30 | 0 | 4 | 1 | 0 |  |
| Point of Sale Printer | `pos.printer` | `pos_printer` | Point of Sale | 5 | 2 | 0 | 0 | 0 |  |
| point of sale Restaurant Order Course | `restaurant.order.course` | `restaurant_order_course` | Point of Sale | 5 | 0 | 1 | 0 | 0 |  |
| Point of Sale Session | `pos.session` | `pos_session` | Point of Sale | 17 | 0 | 4 | 0 | 0 |  |
| Post Closing Reason | `forum.post.reason` | `forum_post_reason` | Website and Storefront | 2 | 0 | 0 | 0 | 0 |  |
| Post Vote | `forum.post.vote` | `forum_post_vote` | Website and Storefront | 5 | 0 | 3 | 1 | 0 |  |
| Preferred myDATA classification combinations for a particular product | `l10n_gr_edi.preferred_classification` | `l10n_gr_edi_preferred_classification` | Fiscal Localizations | 7 | 0 | 0 | 0 | 0 |  |
| Preset to create journal entries during a invoices and payments matching | `account.reconcile.model` | `account_reconcile_model` | General Ledger | 13 | 2 | 0 | 0 | 0 |  |
| Pricelist | `product.pricelist` | `product_pricelist` | Products and Catalog | 8 | 4 | 0 | 0 | 0 |  |
| Pricelist Rule | `product.pricelist.item` | `product_pricelist_item` | Products and Catalog | 22 | 0 | 4 | 0 | 0 |  |
| Privacy Log | `privacy.log` | `privacy_log` | Automation and Integration | 7 | 0 | 0 | 0 | 0 |  |
| Privileges | `res.groups.privilege` | `res_groups_privilege` | Identity and Access | 5 | 0 | 1 | 0 | 0 |  |
| Product | `product.template` | `product_template` | Products and Catalog | 100 | 15 | 21 | 0 | 6 |  |
| Product Attribute | `product.attribute` | `product_attribute` | Products and Catalog | 9 | 1 | 2 | 1 | 0 |  |
| Product Attribute Category | `product.attribute.category` | `product_attribute_category` | Learning, Surveys and Gamification | 2 | 0 | 1 | 0 | 0 |  |
| Product Attribute Custom Value | `product.attribute.custom.value` | `product_attribute_custom_value` | Products and Catalog | 4 | 0 | 2 | 1 | 0 |  |
| Product categorization according to E-Faktur | `l10n_id_efaktur_coretax.product.code` | `l10n_id_efaktur_coretax_product_code` | Fiscal Localizations | 2 | 0 | 0 | 0 | 0 |  |
| Product Category | `product.category` | `product_category` | Products and Catalog | 15 | 2 | 10 | 0 | 0 |  |
| Product Combo | `product.combo` | `product_combo` | Products and Catalog | 5 | 1 | 1 | 0 | 0 |  |
| Product Combo Item | `product.combo.item` | `product_combo_item` | Products and Catalog | 4 | 0 | 1 | 0 | 0 |  |
| Product Document | `product.document` | `product_document` | Products and Catalog | 7 | 2 | 0 | 0 | 0 |  |
| Product Feed | `product.feed` | `product_feed` | Website and Storefront | 8 | 1 | 0 | 0 | 0 |  |
| Product Image | `product.image` | `product_image` | Website and Storefront | 6 | 0 | 2 | 0 | 0 |  |
| Product Moves (Stock Move Line) | `stock.move.line` | `stock_move_line` | Inventory Operations | 23 | 5 | 10 | 0 | 1 |  |
| Product ribbon | `product.ribbon` | `product_ribbon` | Website and Storefront | 8 | 0 | 0 | 0 | 0 |  |
| Product Tag | `product.tag` | `product_tag` | Products and Catalog | 6 | 4 | 1 | 1 | 0 |  |
| Product Template Attribute Exclusion | `product.template.attribute.exclusion` | `product_template_attribute_exclusion` | Products and Catalog | 2 | 1 | 1 | 0 | 0 |  |
| Product Template Attribute Line | `product.template.attribute.line` | `product_template_attribute_line` | Products and Catalog | 5 | 1 | 2 | 0 | 0 |  |
| Product Template Attribute Value | `product.template.attribute.value` | `product_template_attribute_value` | Products and Catalog | 7 | 10 | 3 | 1 | 0 |  |
| Product Unit of Measure | `uom.uom` | `uom_uom` | Units of Measure and Packaging | 17 | 1 | 2 | 1 | 0 |  |
| Product Value | `product.value` | `product_value` | Inventory Valuation and Costing | 8 | 0 | 2 | 0 | 0 |  |
| Product Variant | `product.product` | `product_product` | Products and Catalog | 18 | 7 | 6 | 0 | 2 |  |
| Product Wishlist | `product.wishlist` | `product_wishlist` | Website and Storefront | 6 | 0 | 1 | 1 | 0 |  |
| Production Group | `mrp.production.group` | `mrp_production_group` | Manufacturing | 1 | 1 | 1 | 0 | 0 |  |
| Profiling results | `ir.profile` | `ir_profile` | Platform Foundation | 12 | 0 | 1 | 0 | 0 | no audit columns |
| Progress of Scheduled Actions | `ir.cron.progress` | `ir_cron_progress` | Platform Foundation | 5 | 0 | 1 | 0 | 0 |  |
| Project | `project.project` | `project_project` | Projects and Tasks | 30 | 4 | 7 | 1 | 0 |  |
| Project Milestone | `project.milestone` | `project_milestone` | Projects and Tasks | 8 | 0 | 2 | 0 | 0 |  |
| Project Role | `project.role` | `project_role` | Projects and Tasks | 4 | 1 | 0 | 0 | 0 |  |
| Project Sales line, employee mapping | `project.sale.line.employee.map` | `project_sale_line_employee_map` | Timesheets | 7 | 0 | 1 | 1 | 0 |  |
| Project Stage | `project.project.stage` | `project_project_stage` | Projects and Tasks | 8 | 1 | 0 | 0 | 0 |  |
| Project Tags | `project.tags` | `project_tags` | Projects and Tasks | 2 | 2 | 0 | 1 | 0 |  |
| Project Update | `project.update` | `project_update` | Projects and Tasks | 13 | 0 | 1 | 0 | 0 |  |
| Properties Base Definition | `properties.base.definition` | `properties_base_definition` | Platform Foundation | 2 | 0 | 0 | 1 | 0 |  |
| Public Employee | `hr.employee.public` | `hr_employee_public` | Human Resources Core | 34 | 0 | 0 | 0 | 0 | no table: read from a stored query, materialized as a view |
| Purchase Line and Vendor Bill line matching view | `purchase.bill.line.match` | `purchase_bill_line_match` | Purchasing | 0 | 0 | 0 | 0 | 0 | no table: read from a stored query built at read time |
| Purchase Order | `purchase.order` | `purchase_order` | Purchasing | 38 | 5 | 11 | 0 | 0 |  |
| Purchase Order Line | `purchase.order.line` | `purchase_order_line` | Purchasing | 30 | 3 | 6 | 2 | 0 |  |
| Purchase Report | `purchase.report` | `purchase_report` | Purchasing | 0 | 0 | 0 | 0 | 0 | no table: read from a stored query built at read time |
| Purchase Requisition | `purchase.requisition` | `purchase_requisition` | Purchasing | 14 | 0 | 0 | 0 | 0 |  |
| Purchase Requisition Line | `purchase.requisition.line` | `purchase_requisition_line` | Purchasing | 9 | 0 | 2 | 0 | 0 |  |
| Purchases & Bills Union | `purchase.bill.union` | `purchase_bill_union` | Purchasing | 9 | 0 | 0 | 0 | 0 | no table: read from a stored query, materialized as a view |
| Push Notification Device | `mail.push.device` | `mail_push_device` | Messaging and Activities | 4 | 0 | 1 | 1 | 0 |  |
| Push Notifications | `mail.push` | `mail_push` | Messaging and Activities | 2 | 0 | 0 | 0 | 0 |  |
| Putaway Rule | `stock.putaway.rule` | `stock_putaway_rule` | Inventory Operations | 9 | 1 | 4 | 0 | 0 |  |
| Quants | `stock.quant` | `stock_quant` | Inventory Operations | 17 | 6 | 5 | 0 | 0 |  |
| Question's Answer | `event.quiz.answer` | `event_quiz_answer` | Events | 6 | 0 | 1 | 0 | 0 |  |
| Quiz | `event.quiz` | `event_quiz` | Events | 4 | 0 | 1 | 0 | 0 |  |
| Quotation Template | `sale.order.template` | `sale_order_template` | Sales | 11 | 1 | 1 | 0 | 0 |  |
| Quotation Template Line | `sale.order.template.line` | `sale_order_template_line` | Sales | 9 | 0 | 2 | 2 | 0 |  |
| Quotation's Headers & Footers | `quotation.document` | `quotation_document` | Sales | 5 | 3 | 0 | 0 | 0 |  |
| Rank based on karma | `gamification.karma.rank` | `gamification_karma_rank` | Learning, Surveys and Gamification | 4 | 0 | 0 | 1 | 0 |  |
| Rating | `rating.rating` | `rating_rating` | Projects and Tasks | 21 | 0 | 8 | 1 | 2 |  |
| Record of QRIS transactions | `l10n_id.qris.transaction` | `l10n_id_qris_transaction` | Fiscal Localizations | 8 | 2 | 0 | 0 | 0 |  |
| Record Rule | `ir.rule` | `ir_rule` | Platform Foundation | 9 | 1 | 1 | 1 | 0 |  |
| Recruitment Stages | `hr.recruitment.stage` | `hr_recruitment_stage` | Recruitment | 11 | 1 | 0 | 0 | 0 |  |
| Recycling Model | `data_recycle.model` | `data_recycle_model` | Automation and Integration | 14 | 1 | 0 | 1 | 0 |  |
| Recycling Record | `data_recycle.record` | `data_recycle_record` | Automation and Integration | 6 | 0 | 2 | 0 | 0 |  |
| Reference between stock documents | `stock.reference` | `stock_reference` | Inventory Operations | 1 | 6 | 0 | 0 | 0 |  |
| Refuse Reason of Applicant | `hr.applicant.refuse.reason` | `hr_applicant_refuse_reason` | Recruitment | 4 | 0 | 0 | 0 | 0 |  |
| Registration Mail Scheduler | `event.mail.registration` | `event_mail_registration` | Events | 4 | 0 | 2 | 0 | 0 |  |
| Relation Model | `ir.model.relation` | `ir_model_relation` | Platform Foundation | 3 | 0 | 3 | 0 | 0 |  |
| Removal Strategy | `product.removal` | `product_removal` | Inventory Operations | 2 | 0 | 0 | 0 | 0 |  |
| Repair Order | `repair.order` | `repair_order` | Repair and Maintenance | 27 | 2 | 14 | 0 | 0 |  |
| Repair Tags | `repair.tags` | `repair_tags` | Repair and Maintenance | 2 | 1 | 0 | 1 | 0 |  |
| Report Action | `ir.actions.report` | `ir_act_report_xml` | Platform Foundation | 18 | 2 | 1 | 0 | 0 |  |
| Report Layout | `report.layout` | `report_layout` | Platform Foundation | 5 | 0 | 0 | 0 | 0 |  |
| Represents a role in the system used to categorize users. Each role has a unique name and can be associated with multiple users. Roles can be mentioned in messages to notify all associated users. | `res.role` | `res_role` | Messaging and Activities | 1 | 1 | 0 | 0 | 1 |  |
| Resource Time Off Detail | `resource.calendar.leaves` | `resource_calendar_leaves` | Attendances and Working Time | 10 | 0 | 2 | 0 | 0 |  |
| Resource Working Time | `resource.calendar` | `resource_calendar` | Attendances and Working Time | 11 | 0 | 1 | 0 | 0 |  |
| Resources | `resource.resource` | `resource_resource` | Attendances and Working Time | 9 | 0 | 1 | 1 | 0 |  |
| Restaurant Floor | `restaurant.floor` | `restaurant_floor` | Point of Sale | 4 | 1 | 0 | 0 | 0 |  |
| Restaurant Table | `restaurant.table` | `restaurant_table` | Point of Sale | 12 | 0 | 1 | 0 | 0 |  |
| Resume line of an employee | `hr.resume.line` | `hr_resume_line` | Human Resources Core | 16 | 0 | 3 | 1 | 0 |  |
| Rich Text Editor Converter Subtest | `html_editor.converter.test.sub` | `html_editor_converter_test_sub` | Website and Storefront | 1 | 0 | 0 | 0 | 0 |  |
| Rich Text Editor Converter Test | `html_editor.converter.test` | `html_editor_converter_test` | Website and Storefront | 11 | 0 | 0 | 0 | 0 |  |
| Rules for the reconciliation model | `account.reconcile.model.line` | `account_reconcile_model_line` | General Ledger | 10 | 1 | 1 | 0 | 0 |  |
| Salary Structure Type | `hr.payroll.structure.type` | `hr_payroll_structure_type` | Human Resources Core | 3 | 0 | 0 | 0 | 0 |  |
| Sale Closing | `account.sale.closing` | `account_sale_closing` | Fiscal Localizations | 11 | 0 | 0 | 0 | 0 |  |
| Sale Order Coupon Points - Keeps track of how a sale order impacts a coupon | `sale.order.coupon.points` | `sale_order_coupon_points` | Loyalty and Promotions | 3 | 0 | 1 | 1 | 0 |  |
| Sales Analysis Report | `sale.report` | `sale_report` | Sales | 0 | 0 | 0 | 0 | 0 | no table: read from a stored query built at read time |
| Sales Order | `sale.order` | `sale_order` | Sales | 73 | 9 | 14 | 1 | 1 |  |
| Sales Order Line | `sale.order.line` | `sale_order_line` | Sales | 59 | 5 | 8 | 2 | 1 |  |
| Sales Team | `crm.team` | `crm_team` | Sales | 13 | 2 | 1 | 0 | 0 |  |
| Sales Team Member | `crm.team.member` | `crm_team_member` | Sales | 7 | 0 | 2 | 0 | 0 |  |
| Save favorite Animated Image from Tenor application programming interface | `discuss.gif.favorite` | `discuss_gif_favorite` | Messaging and Activities | 1 | 0 | 0 | 1 | 0 |  |
| Scheduled Message | `mail.scheduled.message` | `mail_scheduled_message` | Messaging and Activities | 10 | 2 | 0 | 0 | 0 |  |
| Scheduled Messages | `mail.message.schedule` | `mail_message_schedule` | Messaging and Activities | 3 | 0 | 0 | 0 | 0 |  |
| Scrap | `stock.scrap` | `stock_scrap` | Inventory Operations | 18 | 1 | 2 | 0 | 0 |  |
| Scrap Reason Tag | `stock.scrap.reason.tag` | `stock_scrap_reason_tag` | Inventory Operations | 3 | 1 | 0 | 1 | 0 |  |
| Sequence | `ir.sequence` | `ir_sequence` | Platform Foundation | 11 | 0 | 0 | 0 | 0 |  |
| Sequence Date Range | `ir.sequence.date_range` | `ir_sequence_date_range` | Platform Foundation | 4 | 0 | 0 | 1 | 0 |  |
| Server Action History | `ir.actions.server.history` | `ir_actions_server_history` | Platform Foundation | 2 | 0 | 0 | 0 | 0 |  |
| Server Actions | `ir.actions.server` | `ir_act_server` | Platform Foundation | 46 | 3 | 3 | 0 | 0 |  |
| Services for vehicles | `fleet.vehicle.log.services` | `fleet_vehicle_log_services` | Fleet | 18 | 0 | 2 | 0 | 0 |  |
| Shipping Methods | `delivery.carrier` | `delivery_carrier` | Delivery and Shipping | 29 | 7 | 2 | 2 | 0 |  |
| SInvoice symbol | `l10n_vn_edi_viettel.sinvoice.symbol` | `l10n_vn_edi_viettel_sinvoice_symbol` | Fiscal Localizations | 2 | 0 | 1 | 1 | 0 |  |
| SInvoice template | `l10n_vn_edi_viettel.sinvoice.template` | `l10n_vn_edi_viettel_sinvoice_template` | Fiscal Localizations | 2 | 0 | 0 | 1 | 0 |  |
| Skill | `hr.skill` | `hr_skill` | Human Resources Core | 3 | 3 | 1 | 0 | 0 |  |
| Skill Level | `hr.skill.level` | `hr_skill_level` | Human Resources Core | 4 | 0 | 1 | 1 | 0 |  |
| Skill level for an applicant | `hr.applicant.skill` | `hr_applicant_skill` | Human Resources Core | 7 | 0 | 1 | 0 | 0 |  |
| Skill level for employee | `hr.employee.skill` | `hr_employee_skill` | Human Resources Core | 7 | 0 | 1 | 0 | 0 |  |
| Skill Type | `hr.skill.type` | `hr_skill_type` | Human Resources Core | 6 | 0 | 0 | 0 | 0 |  |
| Skills for job positions | `hr.job.skill` | `hr_job_skill` | Human Resources Core | 7 | 0 | 1 | 0 | 0 |  |
| Slide / Partner decorated m2m | `slide.slide.partner` | `slide_slide_partner` | Learning, Surveys and Gamification | 7 | 0 | 3 | 2 | 0 |  |
| Slide Question's Answer | `slide.answer` | `slide_answer` | Learning, Surveys and Gamification | 5 | 0 | 1 | 0 | 0 |  |
| Slide Tag | `slide.tag` | `slide_tag` | Learning, Surveys and Gamification | 1 | 1 | 0 | 1 | 0 |  |
| Slides | `slide.slide` | `slide_slide` | Learning, Surveys and Gamification | 41 | 1 | 4 | 3 | 0 |  |
| Slot Mail Scheduler | `event.mail.slot` | `event_mail_slot` | Events | 6 | 0 | 1 | 0 | 0 |  |
| Snailmail Letter | `snailmail.letter` | `snailmail_letter` | Messaging and Activities | 20 | 0 | 2 | 0 | 0 |  |
| Source of Applicants | `hr.recruitment.source` | `hr_recruitment_source` | Recruitment | 5 | 0 | 1 | 0 | 0 |  |
| Specify product lot/serial number in point of sale order line | `pos.pack.operation.lot` | `pos_pack_operation_lot` | Point of Sale | 2 | 0 | 1 | 0 | 0 |  |
| Spreadsheet Dashboard | `spreadsheet.dashboard` | `spreadsheet_dashboard` | Spreadsheets and Dashboards | 5 | 4 | 1 | 0 | 0 |  |
| SRI Payment Method | `l10n_ec.sri.payment` | `l10n_ec_sri_payment` | Fiscal Localizations | 4 | 0 | 0 | 0 | 0 |  |
| Stock Landed Cost | `stock.landed.cost` | `stock_landed_cost` | Inventory Valuation and Costing | 10 | 2 | 2 | 0 | 0 |  |
| Stock Landed Cost Line | `stock.landed.cost.lines` | `stock_landed_cost_lines` | Inventory Valuation and Costing | 6 | 0 | 1 | 0 | 0 |  |
| Stock Move | `stock.move` | `stock_move` | Inventory Operations | 65 | 8 | 24 | 0 | 1 |  |
| Stock Package History | `stock.package.history` | `stock_package_history` | Inventory Operations | 10 | 1 | 0 | 0 | 0 |  |
| Stock package type | `stock.package.type` | `stock_package_type` | Inventory Operations | 13 | 2 | 1 | 5 | 0 |  |
| Stock Quantity Report | `report.stock.quantity` | `report_stock_quantity` | Platform Foundation | 7 | 0 | 0 | 0 | 0 | no table: read from a stored query, materialized as a view |
| Stock Rule | `stock.rule` | `stock_rule` | Inventory Operations | 19 | 0 | 6 | 0 | 0 |  |
| Storage Category | `stock.storage.category` | `stock_storage_category` | Inventory Operations | 4 | 0 | 0 | 1 | 0 |  |
| Storage Category Capacity | `stock.storage.category.capacity` | `stock_storage_category_capacity` | Inventory Operations | 4 | 0 | 3 | 3 | 0 |  |
| Store link preview data | `mail.link.preview` | `mail_link_preview` | Messaging and Activities | 8 | 0 | 1 | 0 | 1 |  |
| Supplier Pricelist | `product.supplierinfo` | `product_supplierinfo` | Products and Catalog | 16 | 1 | 3 | 0 | 0 |  |
| Survey | `survey.survey` | `survey_survey` | Learning, Surveys and Gamification | 35 | 2 | 2 | 8 | 0 |  |
| Survey Label | `survey.question.answer` | `survey_question_answer` | Learning, Surveys and Gamification | 8 | 1 | 2 | 1 | 0 |  |
| Survey Question | `survey.question` | `survey_question` | Learning, Surveys and Gamification | 41 | 2 | 1 | 11 | 0 |  |
| Survey User Input | `survey.user_input` | `survey_user_input` | Learning, Surveys and Gamification | 22 | 1 | 4 | 1 | 0 |  |
| Survey User Input Line | `survey.user_input.line` | `survey_user_input_line` | Learning, Surveys and Gamification | 16 | 0 | 2 | 0 | 0 |  |
| System Parameter | `ir.config_parameter` | `ir_config_parameter` | Platform Foundation | 2 | 0 | 0 | 1 | 0 |  |
| Talent Pool | `hr.talent.pool` | `hr_talent_pool` | Recruitment | 6 | 3 | 0 | 0 | 0 |  |
| Task | `project.task` | `project_task` | Projects and Tasks | 45 | 3 | 14 | 2 | 1 |  |
| Task Recurrence | `project.task.recurrence` | `project_task_recurrence` | Projects and Tasks | 4 | 0 | 0 | 0 | 0 |  |
| Task Stage | `project.task.type` | `project_task_type` | Projects and Tasks | 15 | 2 | 1 | 0 | 0 |  |
| Tasks Analysis | `report.project.task.user` | `report_project_task_user` | Platform Foundation | 38 | 0 | 0 | 0 | 0 | no table: read from a stored query, materialized as a view |
| Tax | `account.tax` | `account_tax` | General Ledger | 76 | 14 | 0 | 0 | 0 |  |
| Tax Group | `account.tax.group` | `account_tax_group` | General Ledger | 12 | 0 | 2 | 0 | 0 |  |
| Tax office in Czech Republic | `l10n_cz.tax_office` | `l10n_cz_tax_office` | Fiscal Localizations | 4 | 0 | 0 | 1 | 0 |  |
| Tax Office in Poland | `l10n_pl.l10n_pl_tax_office` | `l10n_pl_l10n_pl_tax_office` | Fiscal Localizations | 2 | 0 | 0 | 1 | 0 |  |
| Tax Repartition Line | `account.tax.repartition.line` | `account_tax_repartition_line` | General Ledger | 8 | 1 | 1 | 0 | 0 |  |
| Technical model to group purchase order for call to tenders | `purchase.order.group` | `purchase_order_group` | Purchasing | 0 | 0 | 0 | 0 | 0 |  |
| text message Templates | `sms.template` | `sms_template` | Messaging and Activities | 7 | 1 | 1 | 0 | 0 |  |
| Theme Asset | `theme.ir.asset` | `theme_ir_asset` | Website and Storefront | 8 | 0 | 0 | 0 | 0 |  |
| Theme Attachments | `theme.ir.attachment` | `theme_ir_attachment` | Website and Storefront | 3 | 0 | 0 | 0 | 0 |  |
| Theme user interface View | `theme.ir.ui.view` | `theme_ir_ui_view` | Website and Storefront | 10 | 0 | 0 | 0 | 0 |  |
| Thumb drive used to sign invoices in Egypt | `l10n_eg_edi.thumb.drive` | `l10n_eg_edi_thumb_drive` | Fiscal Localizations | 4 | 0 | 0 | 1 | 0 |  |
| TicketBAI Document | `l10n_es_edi_tbai.document` | `l10n_es_edi_tbai_document` | Fiscal Localizations | 8 | 0 | 0 | 0 | 0 |  |
| Time Off | `hr.leave` | `hr_leave` | Time Off | 29 | 0 | 4 | 3 | 1 |  |
| Time Off Allocation | `hr.leave.allocation` | `hr_leave_allocation` | Time Off | 24 | 0 | 3 | 1 | 0 |  |
| Time Off Calendar | `hr.leave.report.calendar` | `hr_leave_report_calendar` | Time Off | 15 | 0 | 0 | 0 | 0 | no table: read from a stored query, materialized as a view |
| Time Off Summary / Report | `hr.leave.employee.type.report` | `hr_leave_employee_type_report` | Time Off | 11 | 0 | 0 | 0 | 0 | no table: read from a stored query, materialized as a view |
| Time Off Summary / Report | `hr.leave.report` | `hr_leave_report` | Time Off | 13 | 0 | 0 | 0 | 0 | no table: read from a stored query, materialized as a view |
| Time Off Type | `hr.leave.type` | `hr_leave_type` | Time Off | 28 | 1 | 1 | 1 | 0 |  |
| Timesheet Attendance Report | `hr.timesheet.attendance.report` | `hr_timesheet_attendance_report` | Attendances and Working Time | 9 | 0 | 0 | 0 | 0 | no table: read from a stored query, materialized as a view |
| Timesheets Analysis Report | `timesheets.analysis.report` | `timesheets_analysis_report` | Timesheets | 22 | 0 | 0 | 0 | 0 | no table: read from a stored query, materialized as a view |
| Tour's step | `web_tour.tour.step` | `web_tour_tour_step` | Automation and Integration | 6 | 0 | 1 | 0 | 0 |  |
| Tours | `web_tour.tour` | `web_tour_tour` | Automation and Integration | 5 | 1 | 0 | 1 | 0 |  |
| Track / Visitor Link | `event.track.visitor` | `event_track_visitor` | Events | 7 | 0 | 3 | 0 | 0 |  |
| Track Karma Changes | `gamification.karma.tracking` | `gamification_karma_tracking` | Learning, Surveys and Gamification | 8 | 0 | 2 | 0 | 0 |  |
| Transaction Lipa na M-PESA | `transaction.lipa.na.mpesa` | `transaction_lipa_na_mpesa` | Point of Sale | 5 | 0 | 0 | 0 | 0 |  |
| Transfer | `stock.picking` | `stock_picking` | Inventory Operations | 64 | 10 | 14 | 1 | 0 |  |
| Transport Document | `l10n_it.ddt` | `l10n_it_ddt` | Fiscal Localizations | 2 | 0 | 0 | 0 | 0 |  |
| Triggered actions | `ir.cron.trigger` | `ir_cron_trigger` | Platform Foundation | 2 | 0 | 2 | 0 | 0 |  |
| Turkish Tax Codes (GIB Codes) | `l10n_tr_nilvera_einvoice_extended.account.tax.code` | `l10n_tr_nilvera_einvoice_extended_account_tax_code` | Fiscal Localizations | 4 | 0 | 0 | 0 | 0 |  |
| Turkish Tax Office | `l10n_tr_nilvera_einvoice_extended.tax.office` | `l10n_tr_nilvera_einvoice_extended_tax_office` | Fiscal Localizations | 3 | 0 | 0 | 0 | 0 |  |
| Twilio Number | `sms.twilio.number` | `sms_twilio_number` | Messaging and Activities | 4 | 0 | 1 | 0 | 0 |  |
| Type of a resume line | `hr.resume.line.type` | `hr_resume_line_type` | Human Resources Core | 4 | 0 | 0 | 0 | 0 |  |
| Unbuild Order | `mrp.unbuild` | `mrp_unbuild` | Manufacturing | 11 | 0 | 2 | 1 | 0 |  |
| unit of measure categorization according to E-Faktur | `l10n_id_efaktur_coretax.uom.code` | `l10n_id_efaktur_coretax_uom_code` | Fiscal Localizations | 2 | 0 | 0 | 0 | 0 |  |
| Unit of Measure for price per unit on Electronic Commerce products. | `website.base.unit` | `website_base_unit` | Website and Storefront | 1 | 0 | 0 | 0 | 0 |  |
| User | `res.users` | `res_users` | Identity and Access | 31 | 29 | 4 | 4 | 0 |  |
| User Settings | `res.users.settings` | `res_users_settings` | Identity and Access | 18 | 2 | 0 | 1 | 0 |  |
| User Settings for Embedded Actions | `res.users.settings.embedded.action` | `res_users_settings_embedded_action` | Identity and Access | 7 | 0 | 1 | 1 | 0 |  |
| User Settings Volumes | `res.users.settings.volumes` | `res_users_settings_volumes` | Messaging and Activities | 4 | 0 | 3 | 1 | 2 |  |
| User/Guest Presence | `mail.presence` | `mail_presence` | Messaging and Activities | 5 | 0 | 0 | 1 | 2 | no audit columns |
| Users application programming interface Keys | `res.users.apikeys` | `res_users_apikeys` | Identity and Access | 6 | 0 | 0 | 0 | 0 |  |
| Users Deletion Request | `res.users.deletion` | `res_users_deletion` | Identity and Access | 3 | 0 | 0 | 0 | 0 |  |
| Users Log | `res.users.log` | `res_users_log` | Identity and Access | 1 | 0 | 1 | 0 | 0 |  |
| Valuation Adjustment Lines | `stock.valuation.adjustment.lines` | `stock_valuation_adjustment_lines` | Inventory Valuation and Costing | 11 | 0 | 1 | 0 | 0 |  |
| Vehicle | `fleet.vehicle` | `fleet_vehicle` | Fleet | 48 | 2 | 1 | 0 | 0 |  |
| Vehicle Contract | `fleet.vehicle.log.contract` | `fleet_vehicle_log_contract` | Fleet | 16 | 1 | 2 | 0 | 0 |  |
| Vehicle Status | `fleet.vehicle.state` | `fleet_vehicle_state` | Fleet | 3 | 0 | 0 | 1 | 0 |  |
| Vehicle Tag | `fleet.vehicle.tag` | `fleet_vehicle_tag` | Fleet | 2 | 1 | 0 | 1 | 0 |  |
| Vendor Delay Report | `vendor.delay.report` | `vendor_delay_report` | Replenishment and Procurement | 7 | 0 | 0 | 0 | 0 | no table: read from a stored query, materialized as a view |
| Veri*Factu Document | `l10n_es_edi_verifactu.document` | `l10n_es_edi_verifactu_document` | Fiscal Localizations | 9 | 0 | 0 | 0 | 0 |  |
| Version | `hr.version` | `hr_version` | Human Resources Core | 53 | 0 | 3 | 1 | 1 |  |
| View | `ir.ui.view` | `ir_ui_view` | Platform Foundation | 24 | 1 | 4 | 2 | 1 |  |
| Visited Pages | `website.track` | `website_track` | Website and Storefront | 5 | 0 | 4 | 0 | 0 | no audit columns |
| Warehouse | `stock.warehouse` | `stock_warehouse` | Inventory Operations | 48 | 4 | 1 | 2 | 0 |  |
| Website | `website` | `website` | Website and Storefront | 79 | 2 | 1 | 1 | 0 |  |
| Website Checkout Step | `website.checkout.step` | `website_checkout_step` | Website and Storefront | 7 | 0 | 2 | 0 | 0 |  |
| Website Configurator Feature | `website.configurator.feature` | `website_configurator_feature` | Website and Storefront | 11 | 0 | 0 | 0 | 0 |  |
| Website Event Menu | `website.event.menu` | `website_event_menu` | Events | 10 | 0 | 1 | 0 | 0 |  |
| Website Menu | `website.menu` | `website_menu` | Website and Storefront | 12 | 1 | 5 | 0 | 0 |  |
| Website Product Category | `product.public.category` | `product_public_category` | Website and Storefront | 16 | 2 | 4 | 0 | 0 |  |
| Website rewrite | `website.rewrite` | `website_rewrite` | Website and Storefront | 8 | 0 | 2 | 0 | 0 |  |
| Website Snippet Filter | `website.snippet.filter` | `website_snippet_filter` | Website and Storefront | 9 | 0 | 2 | 0 | 0 |  |
| Website Technical Page | `website.technical.page` | `website_technical_page` | Website and Storefront | 0 | 0 | 0 | 0 | 0 | no table: read from a stored query built at read time |
| Website Theme Menu | `theme.website.menu` | `theme_website_menu` | Website and Storefront | 9 | 0 | 2 | 0 | 0 |  |
| Website Theme Page | `theme.website.page` | `theme_website_page` | Website and Storefront | 10 | 0 | 1 | 0 | 0 |  |
| Website Visitor | `website.visitor` | `website_visitor` | Website and Storefront | 9 | 1 | 2 | 1 | 0 |  |
| Work Center | `mrp.workcenter` | `mrp_workcenter` | Manufacturing | 17 | 3 | 3 | 0 | 0 |  |
| Work Center Capacity | `mrp.workcenter.capacity` | `mrp_workcenter_capacity` | Manufacturing | 6 | 0 | 1 | 1 | 1 |  |
| Work Center Usage | `mrp.routing.workcenter` | `mrp_routing_workcenter` | Manufacturing | 9 | 2 | 2 | 0 | 0 |  |
| Work Detail | `resource.calendar.attendance` | `resource_calendar_attendance` | Attendances and Working Time | 12 | 0 | 3 | 0 | 0 |  |
| Work Entries Employees | `hr.user.work.entry.employee` | `hr_user_work_entry_employee` | Work Entries | 4 | 0 | 0 | 1 | 0 |  |
| Work Location | `hr.work.location` | `hr_work_location` | Human Resources Core | 6 | 0 | 0 | 0 | 0 |  |
| Work Order | `mrp.workorder` | `mrp_workorder` | Manufacturing | 20 | 3 | 4 | 0 | 0 |  |
| Workcenter Productivity Log | `mrp.workcenter.productivity` | `mrp_workcenter_productivity` | Manufacturing | 10 | 0 | 3 | 0 | 0 |  |
| Workcenter Productivity Losses | `mrp.workcenter.productivity.loss` | `mrp_workcenter_productivity_loss` | Manufacturing | 4 | 0 | 0 | 0 | 0 |  |

### 12.2 Interactive assistant entities

Assistant tables follow the same rules with two differences: their links to persistent entities default to cascade deletion when required (section 9.2), and their rows are removed by the periodic cleanup. 222 entities, 1,098 value columns.

| Entity | Transport name | Table | Specified in | Value columns | Association tables | Indexed columns | Table constraints | Declared indexes | Notes |
|---|---|---|---|---|---|---|---|---|---|
| 2-Factor Setup Wizard | `auth_totp.wizard` | `auth_totp_wizard` | Identity and Access | 4 | 0 | 0 | 0 | 0 |  |
| Account merge wizard | `account.merge.wizard` | `account_merge_wizard` | General Ledger | 1 | 1 | 0 | 0 | 0 |  |
| Account merge wizard line | `account.merge.wizard.line` | `account_merge_wizard_line` | General Ledger | 6 | 0 | 0 | 0 | 0 |  |
| Account move line to be created when posting work in progress account move | `mrp.account.wip.accounting.line` | `mrp_account_wip_accounting_line` | Manufacturing | 6 | 0 | 0 | 1 | 0 |  |
| Account Move Reversal | `account.move.reversal` | `account_move_reversal` | General Ledger | 16 | 2 | 0 | 0 | 0 |  |
| Account Move Send Batch Wizard | `account.move.send.batch.wizard` | `account_move_send_batch_wizard` | General Ledger | 0 | 1 | 0 | 0 | 0 |  |
| Account Move Send Wizard | `account.move.send.wizard` | `account_move_send_wizard` | General Ledger | 12 | 1 | 0 | 0 | 0 |  |
| Accrued Orders Wizard | `account.accrued.orders.wizard` | `account_accrued_orders_wizard` | General Ledger | 7 | 0 | 0 | 0 | 0 |  |
| Activity schedule plan Wizard | `mail.activity.schedule` | `mail_activity_schedule` | Messaging and Activities | 11 | 1 | 0 | 0 | 0 |  |
| Add applicants to a job | `job.add.applicants` | `job_add_applicants` | Recruitment | 0 | 2 | 0 | 0 | 0 |  |
| Add applicants to talent pool | `talent.pool.add.applicants` | `talent_pool_add_applicants` | Recruitment | 0 | 3 | 0 | 0 | 0 |  |
| Add Contacts to Mailing List | `mailing.contact.to.list` | `mailing_contact_to_list` | Marketing and Mass Mailing | 1 | 1 | 0 | 0 | 0 |  |
| Add Debit Note wizard | `account.debit.note` | `account_debit_note` | Accounts Receivable | 5 | 1 | 0 | 0 | 0 |  |
| application programming interface Key Description | `res.users.apikeys.description` | `res_users_apikeys_description` | Identity and Access | 3 | 0 | 0 | 0 | 0 |  |
| Approved Dematerialization Platform Registration | `pdp.registration` | `pdp_registration` | Fiscal Localizations | 3 | 0 | 0 | 0 | 0 |  |
| Approved Dematerialization Platform Response wizard | `pdp.response.wizard` | `pdp_response_wizard` | Fiscal Localizations | 7 | 1 | 0 | 0 | 0 |  |
| Assign serial numbers to production order | `mrp.production.serials` | `mrp_production_serials` | Manufacturing | 5 | 0 | 0 | 0 | 0 |  |
| Autopost Bills Wizard | `account.autopost.bills.wizard` | `account_autopost_bills_wizard` | General Ledger | 2 | 0 | 0 | 0 | 0 |  |
| Backorder Confirmation | `stock.backorder.confirmation` | `stock_backorder_confirmation` | Inventory Operations | 1 | 1 | 0 | 0 | 0 |  |
| Backorder Confirmation Line | `mrp.production.backorder.line` | `mrp_production_backorder_line` | Manufacturing | 3 | 0 | 0 | 0 | 0 |  |
| Backorder Confirmation Line | `stock.backorder.confirmation.line` | `stock_backorder_confirmation_line` | Inventory Operations | 3 | 0 | 0 | 0 | 0 |  |
| Bank Account Allocation Line (Wizard) | `hr.bank.account.allocation.wizard.line` | `hr_bank_account_allocation_wizard_line` | Human Resources Core | 6 | 0 | 0 | 0 | 0 |  |
| Bank Account Allocation Wizard | `hr.bank.account.allocation.wizard` | `hr_bank_account_allocation_wizard` | Human Resources Core | 1 | 0 | 0 | 0 | 0 |  |
| Bank setup manual config | `account.setup.bank.manual.config` | `account_setup_bank_manual_config` | General Ledger | 4 | 0 | 0 | 0 | 0 |  |
| Base Import | `base_import.import` | `base_import_import` | Automation and Integration | 4 | 0 | 0 | 0 | 0 |  |
| Batch Transfer Lines | `stock.picking.to.batch` | `stock_picking_to_batch` | Inventory Operations | 5 | 0 | 0 | 0 | 0 |  |
| Bill to Purchase Order | `bill.to.po.wizard` | `bill_to_po_wizard` | Purchasing | 2 | 0 | 0 | 0 | 0 |  |
| Calendar Popover Delete Wizard | `calendar.popover.delete.wizard` | `calendar_popover_delete_wizard` | Calendar and Scheduling | 6 | 0 | 0 | 0 | 0 |  |
| Calendar Provider Configuration Wizard | `calendar.provider.config` | `calendar_provider_config` | Calendar and Scheduling | 7 | 0 | 0 | 0 | 0 |  |
| Cancel Electronic Invoice | `l10n_in_edi.cancel` | `l10n_in_edi_cancel` | Fiscal Localizations | 3 | 0 | 0 | 0 | 0 |  |
| Cancel Electronic Waybill | `l10n.in.ewaybill.cancel` | `l10n_in_ewaybill_cancel` | Fiscal Localizations | 3 | 0 | 0 | 0 | 0 |  |
| Cancel multiple quotations | `sale.mass.cancel.orders` | `sale_mass_cancel_orders` | Sales | 0 | 1 | 0 | 0 | 0 |  |
| Cancel Time Off Wizard | `hr.holidays.cancel.leave` | `hr_holidays_cancel_leave` | Time Off | 2 | 0 | 0 | 0 | 0 |  |
| Change Password Wizard | `change.password.wizard` | `change_password_wizard` | Identity and Access | 0 | 0 | 0 | 0 | 0 |  |
| Change Production Qty | `change.production.qty` | `change_production_qty` | Platform Foundation | 2 | 0 | 0 | 0 | 0 |  |
| Channel Invitation Wizard | `slide.channel.invite` | `slide_channel_invite` | Learning, Surveys and Gamification | 7 | 2 | 0 | 0 | 0 |  |
| Checks Mass Transfers | `l10n_latam.payment.mass.transfer` | `l10n_latam_payment_mass_transfer` | Payments and Bank Reconciliation | 3 | 1 | 0 | 0 | 0 |  |
| Choose the sheet layout to print lot labels | `lot.label.layout` | `lot_label_layout` | Inventory Operations | 2 | 1 | 0 | 0 | 0 |  |
| Choose the sheet layout to print the labels | `product.label.layout` | `product_label_layout` | Products and Catalog | 6 | 3 | 0 | 0 | 0 |  |
| Choose whether to print product or lot/sn labels | `picking.label.type` | `picking_label_type` | Inventory Operations | 1 | 2 | 0 | 0 | 0 |  |
| Close Session Wizard | `pos.close.session.wizard` | `pos_close_session_wizard` | Point of Sale | 4 | 0 | 0 | 0 | 0 |  |
| Company Document Layout | `base.document.layout` | `base_document_layout` | Contacts and Organizations | 3 | 0 | 0 | 0 | 0 |  |
| Compliance Letter for EXO Number | `compliance.letter.wizard` | `compliance_letter_wizard` | Fiscal Localizations | 1 | 0 | 0 | 0 | 0 |  |
| Config | `res.config` | `res_config` | Platform Foundation | 0 | 0 | 0 | 0 | 0 |  |
| Config Settings | `res.config.settings` | `res_config_settings` | Platform Foundation | 278 | 3 | 0 | 0 | 0 |  |
| Confirm Expiry | `expiry.picking.confirmation` | `expiry_picking_confirmation` | Products and Catalog | 1 | 3 | 0 | 0 | 0 |  |
| Confirm Stock text message | `confirm.stock.sms` | `confirm_stock_sms` | Inventory Operations | 0 | 1 | 0 | 0 | 0 |  |
| Confirmation Wizard | `pos.confirmation.wizard` | `pos_confirmation_wizard` | Point of Sale | 1 | 0 | 0 | 0 | 0 |  |
| Conflict in Inventory | `stock.inventory.conflict` | `stock_inventory_conflict` | Inventory Operations | 0 | 2 | 0 | 0 | 0 |  |
| Consolidate Invoice Wizard | `myinvois.consolidate.invoice.wizard` | `myinvois_consolidate_invoice_wizard` | Fiscal Localizations | 3 | 0 | 0 | 0 | 0 |  |
| Contract Template Wizard | `hr.version.wizard` | `hr_version_wizard` | Human Resources Core | 1 | 0 | 0 | 0 | 0 |  |
| Convert Lead to Opportunity (in mass) | `crm.lead2opportunity.partner.mass` | `crm_lead2opportunity_partner_mass` | Customer Relationship Management | 9 | 3 | 0 | 0 | 0 |  |
| Convert Lead to Opportunity (not in mass) | `crm.lead2opportunity.partner` | `crm_lead2opportunity_partner` | Customer Relationship Management | 8 | 1 | 0 | 0 | 0 |  |
| Create a Passkey | `auth.passkey.key.create` | `auth_passkey_key_create` | Identity and Access | 1 | 0 | 0 | 0 | 0 |  |
| Create activity and to-do at the same time | `mail.activity.todo.create` | `mail_activity_todo_create` | Projects and Tasks | 4 | 0 | 1 | 0 | 0 |  |
| Create Automatic Entries | `account.automatic.entry.wizard` | `account_automatic_entry_wizard` | General Ledger | 7 | 1 | 0 | 0 | 0 |  |
| Create links that apply a coupon and redirect to a specific page | `coupon.share` | `coupon_share` | Loyalty and Promotions | 4 | 0 | 0 | 0 | 0 |  |
| Create Menu Wizard | `wizard.ir.model.menu.create` | `wizard_ir_model_menu_create` | Platform Foundation | 2 | 0 | 0 | 0 | 0 |  |
| Create new or use existing Customer on new Quotation | `crm.quotation.partner` | `crm_quotation_partner` | Sales | 3 | 0 | 0 | 0 | 0 |  |
| Delivery Carrier Selection Wizard | `choose.delivery.carrier` | `choose_delivery_carrier` | Delivery and Shipping | 7 | 0 | 0 | 0 | 0 |  |
| Demo | `ir.demo` | `ir_demo` | Platform Foundation | 0 | 0 | 0 | 0 | 0 |  |
| Demo failure | `ir.demo_failure` | `ir_demo_failure` | Platform Foundation | 3 | 0 | 0 | 0 | 0 |  |
| Demo Failure wizard | `ir.demo_failure.wizard` | `ir_demo_failure_wizard` | Platform Foundation | 0 | 0 | 0 | 0 | 0 |  |
| Departure Wizard | `hr.departure.wizard` | `hr_departure_wizard` | Human Resources Core | 7 | 1 | 0 | 0 | 0 |  |
| Discount Wizard | `sale.order.discount` | `sale_order_discount` | Sales | 4 | 0 | 0 | 0 | 0 |  |
| Document Status Update Wizard | `myinvois.document.status.update.wizard` | `myinvois_document_status_update_wizard` | Fiscal Localizations | 3 | 0 | 0 | 0 | 0 |  |
| Edit Attendee Details on Sales Confirmation | `registration.editor` | `registration_editor` | Events | 1 | 0 | 0 | 0 | 0 |  |
| Edit Attendee Line on Sales Confirmation | `registration.editor.line` | `registration_editor_line` | Events | 9 | 0 | 0 | 0 | 0 |  |
| Electronic Invoice cancellation wizard | `l10n_vn_edi_viettel.cancellation` | `l10n_vn_edi_viettel_cancellation` | Fiscal Localizations | 4 | 0 | 0 | 0 | 0 |  |
| Email composition wizard | `mail.compose.message` | `mail_compose_message` | Messaging and Activities | 35 | 3 | 0 | 0 | 0 |  |
| Email Template Preview | `mail.template.preview` | `mail_template_preview` | Messaging and Activities | 3 | 0 | 0 | 0 | 0 |  |
| Employee Delete Wizard | `hr.employee.delete.wizard` | `hr_employee_delete_wizard` | Timesheets | 0 | 1 | 0 | 0 | 0 |  |
| Enable profiling for some time | `base.enable.profiling.wizard` | `base_enable_profiling_wizard` | Platform Foundation | 2 | 0 | 0 | 0 | 0 |  |
| Event Booth Configurator | `event.booth.configurator` | `event_booth_configurator` | Events | 4 | 1 | 0 | 0 | 0 |  |
| Event Configurator | `event.event.configurator` | `event_event_configurator` | Events | 4 | 0 | 0 | 0 | 0 |  |
| Expense Approve Duplicate | `hr.expense.approve.duplicate` | `hr_expense_approve_duplicate` | Expenses | 0 | 1 | 0 | 0 | 0 |  |
| Expense Posting Wizard | `hr.expense.post.wizard` | `hr_expense_post_wizard` | Expenses | 3 | 0 | 0 | 0 | 0 |  |
| Expense Refuse Reason Wizard | `hr.expense.refuse.wizard` | `hr_expense_refuse_wizard` | Expenses | 1 | 1 | 0 | 0 | 0 |  |
| Expense Split | `hr.expense.split` | `hr_expense_split` | Expenses | 14 | 1 | 0 | 0 | 0 |  |
| Expense Split Wizard | `hr.expense.split.wizard` | `hr_expense_split_wizard` | Expenses | 1 | 0 | 0 | 0 | 0 |  |
| Exports 2307 data to a XLS file. | `l10n_ph_2307.wizard` | `l10n_ph_2307_wizard` | Fiscal Localizations | 0 | 1 | 0 | 0 | 0 |  |
| Fichier Echange Informatise | `l10n_fr.fec.export.wizard` | `l10n_fr_fec_export_wizard` | Fiscal Localizations | 5 | 1 | 0 | 0 | 0 |  |
| Followers edit wizard | `mail.followers.edit` | `mail_followers_edit` | Messaging and Activities | 5 | 1 | 0 | 0 | 0 |  |
| Gamification Goal Wizard | `gamification.goal.wizard` | `gamification_goal_wizard` | Learning, Surveys and Gamification | 2 | 0 | 0 | 0 | 0 |  |
| Gamification User Badge Wizard | `gamification.badge.user.wizard` | `gamification_badge_user_wizard` | Learning, Surveys and Gamification | 4 | 0 | 0 | 0 | 0 |  |
| Generate Coupons | `loyalty.generate.wizard` | `loyalty_generate_wizard` | Loyalty and Promotions | 6 | 2 | 0 | 0 | 0 |  |
| Generate Payment Link | `payment.link.wizard` | `payment_link_wizard` | Payment Providers | 11 | 0 | 0 | 0 | 0 |  |
| Generate time off allocations for multiple employees | `hr.leave.allocation.generate.multi.wizard` | `hr_leave_allocation_generate_multi_wizard` | Time Off | 12 | 1 | 0 | 0 | 0 |  |
| Generate time off for multiple employees | `hr.leave.generate.multi.wizard` | `hr_leave_generate_multi_wizard` | Time Off | 8 | 1 | 0 | 0 | 0 |  |
| Get Lost Reason | `crm.lead.lost` | `crm_lead_lost` | Customer Relationship Management | 2 | 1 | 0 | 0 | 0 |  |
| Get Refuse Reason | `applicant.get.refuse.reason` | `applicant_get_refuse_reason` | Recruitment | 8 | 3 | 0 | 0 | 0 |  |
| Google Calendar Account Reset | `google.calendar.account.reset` | `google_calendar_account_reset` | Calendar and Scheduling | 3 | 0 | 0 | 0 | 0 |  |
| Grant Portal Access | `portal.wizard` | `portal_wizard` | Website and Storefront | 1 | 1 | 0 | 0 | 0 |  |
| Handles problems occurring while creating multiple quick response-invoices at once | `l10n_ch.qr_invoice.wizard` | `l10n_ch_qr_invoice_wizard` | Fiscal Localizations | 4 | 0 | 0 | 0 | 0 |  |
| human resources Time Off Summary Report By Employee | `hr.holidays.summary.employee` | `hr_holidays_summary_employee` | Time Off | 2 | 1 | 0 | 0 | 0 |  |
| Implements cancelling an ecpay invoice. | `l10n_tw_edi.invoice.cancel` | `l10n_tw_edi_invoice_cancel` | Fiscal Localizations | 2 | 0 | 0 | 0 | 0 |  |
| Implements printingan ecpay invoice. | `l10n_tw_edi.invoice.print` | `l10n_tw_edi_invoice_print` | Fiscal Localizations | 3 | 0 | 0 | 0 | 0 |  |
| Import Module | `base.import.module` | `base_import_module` | Platform Foundation | 6 | 0 | 0 | 0 | 0 |  |
| Install Language | `base.language.install` | `base_language_install` | Platform Foundation | 1 | 2 | 0 | 0 | 0 |  |
| Inventory Adjustment Reference / Reason | `stock.inventory.adjustment.name` | `stock_inventory_adjustment_name` | Inventory Operations | 3 | 1 | 0 | 0 | 0 |  |
| Inventory Adjustment Warning | `stock.inventory.warning` | `stock_inventory_warning` | Inventory Operations | 0 | 1 | 0 | 0 | 0 |  |
| Language Export | `base.language.export` | `base_language_export` | Platform Foundation | 8 | 1 | 0 | 0 | 0 |  |
| Language Import | `base.language.import` | `base_language_import` | Platform Foundation | 5 | 0 | 0 | 0 | 0 |  |
| Lead Assignation | `crm.lead.assignation` | `crm_lead_assignation` | Customer Relationship Management | 6 | 0 | 0 | 0 | 0 |  |
| Lead forward to partner | `crm.lead.forward.to.partner` | `crm_lead_forward_to_partner` | Customer Relationship Management | 3 | 0 | 0 | 0 | 0 |  |
| Line of issue consumption | `mrp.consumption.warning.line` | `mrp_consumption_warning_line` | Manufacturing | 5 | 0 | 0 | 0 | 0 |  |
| Mail Activity Schedule Line | `mail.activity.schedule.line` | `mail_activity_schedule_line` | Messaging and Activities | 4 | 0 | 0 | 0 | 0 |  |
| Mail Template Reset | `mail.template.reset` | `mail_template_reset` | Messaging and Activities | 0 | 1 | 0 | 0 | 0 |  |
| Mailing Contact Import | `mailing.contact.import` | `mailing_contact_import` | Marketing and Mass Mailing | 1 | 1 | 0 | 0 | 0 |  |
| Merge Mass Mailing List | `mailing.list.merge` | `mailing_list_merge` | Marketing and Mass Mailing | 4 | 1 | 0 | 0 | 0 |  |
| Merge Opportunities | `crm.merge.opportunity` | `crm_merge_opportunity` | Customer Relationship Management | 2 | 1 | 0 | 0 | 0 |  |
| Merge Partner Line | `base.partner.merge.line` | `base_partner_merge_line` | Contacts and Organizations | 3 | 0 | 0 | 0 | 0 |  |
| Merge Partner Wizard | `base.partner.merge.automatic.wizard` | `base_partner_merge_automatic_wizard` | Contacts and Organizations | 12 | 1 | 0 | 0 | 0 |  |
| Microsoft Calendar Account Reset | `microsoft.calendar.account.reset` | `microsoft_calendar_account_reset` | Calendar and Scheduling | 3 | 0 | 0 | 0 | 0 |  |
| Module Activation Request | `base.module.install.request` | `base_module_install_request` | Platform Foundation | 3 | 0 | 0 | 0 | 0 |  |
| Module Activation Review | `base.module.install.review` | `base_module_install_review` | Platform Foundation | 1 | 0 | 0 | 0 | 0 |  |
| Module Uninstall | `base.module.uninstall` | `base_module_uninstall` | Platform Foundation | 1 | 1 | 0 | 0 | 0 |  |
| MojEracun Reject Invoice Wizard | `l10n_hr_edi.mojeracun_reject_wizard` | `l10n_hr_edi_mojeracun_reject_wizard` | Fiscal Localizations | 3 | 0 | 0 | 0 | 0 |  |
| Multiple order invoice creation | `pos.make.invoice` | `pos_make_invoice` | Point of Sale | 1 | 0 | 0 | 0 | 0 |  |
| Nemhandel Registration | `nemhandel.registration` | `nemhandel_registration` | Fiscal Localizations | 1 | 0 | 0 | 0 | 0 |  |
| Nemhandel Rejection wizard | `nemhandel.rejection.wizard` | `nemhandel_rejection_wizard` | Fiscal Localizations | 1 | 1 | 0 | 0 | 0 |  |
| Opening Balance of Financial Year | `account.financial.year.op` | `account_financial_year_op` | General Ledger | 1 | 0 | 0 | 0 | 0 |  |
| Page Properties | `website.page.properties` | `website_page_properties` | Website and Storefront | 3 | 0 | 0 | 0 | 0 |  |
| Page Properties Base | `website.page.properties.base` | `website_page_properties_base` | Website and Storefront | 3 | 0 | 0 | 0 | 0 |  |
| Password Check Wizard | `res.users.identitycheck` | `res_users_identitycheck` | Identity and Access | 2 | 0 | 0 | 0 | 0 |  |
| Pay | `account.payment.register` | `account_payment_register` | General Ledger | 27 | 2 | 0 | 0 | 0 |  |
| Payment Capture Wizard | `payment.capture.wizard` | `payment_capture_wizard` | Payment Providers | 2 | 1 | 0 | 0 | 0 |  |
| Payment Refund Wizard | `payment.refund.wizard` | `payment_refund_wizard` | Accounts Receivable | 2 | 0 | 0 | 0 | 0 |  |
| Payment register check | `l10n_latam.payment.register.check` | `l10n_latam_payment_register_check` | Payments and Bank Reconciliation | 6 | 0 | 0 | 0 | 0 |  |
| Payment register withholding line | `account.payment.register.withholding.line` | `account_payment_register_withholding_line` | Taxes | 17 | 0 | 0 | 0 | 0 |  |
| Payment register withholding lines | `l10n_ar.payment.register.withholding` | `l10n_ar_payment_register_withholding` | Fiscal Localizations | 5 | 0 | 0 | 0 | 0 |  |
| Peppol Configuration Wizard | `pdp.config.wizard` | `pdp_config_wizard` | Fiscal Localizations | 1 | 0 | 0 | 0 | 0 |  |
| Peppol Configuration Wizard | `peppol.config.wizard` | `peppol_config_wizard` | Electronic Invoicing and Document Exchange | 3 | 0 | 0 | 0 | 0 |  |
| Peppol Registration | `peppol.registration` | `peppol_registration` | Electronic Invoicing and Document Exchange | 2 | 0 | 0 | 0 | 0 |  |
| Peppol Rejection wizard | `account.peppol.rejection.wizard` | `account_peppol_rejection_wizard` | Electronic Invoicing and Document Exchange | 0 | 3 | 0 | 0 | 0 |  |
| Peppol Service | `account_peppol.service` | `account_peppol_service` | Electronic Invoicing and Document Exchange | 4 | 0 | 0 | 0 | 0 |  |
| Point of Sale Daily Report | `pos.daily.sales.reports.wizard` | `pos_daily_sales_reports_wizard` | Point of Sale | 2 | 0 | 0 | 0 | 0 |  |
| Point of Sale Details Report | `pos.details.wizard` | `pos_details_wizard` | Point of Sale | 2 | 1 | 0 | 0 | 0 |  |
| Point of Sale Make Payment Wizard | `pos.make.payment` | `pos_make_payment` | Point of Sale | 5 | 0 | 0 | 0 | 0 |  |
| Portal Sharing | `portal.share` | `portal_share` | Website and Storefront | 3 | 1 | 0 | 0 | 0 |  |
| Portal User Config | `portal.wizard.user` | `portal_wizard_user` | Website and Storefront | 3 | 0 | 0 | 0 | 0 |  |
| Print Pre-numbered Checks | `print.prenumbered.checks` | `print_prenumbered_checks` | Accounts Payable | 1 | 0 | 0 | 0 | 0 |  |
| Print Resume | `hr.employee.cv.wizard` | `hr_employee_cv_wizard` | Human Resources Core | 5 | 1 | 0 | 0 | 0 |  |
| Privacy Lookup Wizard | `privacy.lookup.wizard` | `privacy_lookup_wizard` | Automation and Integration | 4 | 0 | 0 | 0 | 0 |  |
| Privacy Lookup Wizard Line | `privacy.lookup.wizard.line` | `privacy_lookup_wizard_line` | Automation and Integration | 9 | 0 | 0 | 0 | 0 |  |
| Product Margin | `product.margin` | `product_margin` | Pricing and Pricelists | 3 | 0 | 0 | 0 | 0 |  |
| Product Replenish | `product.replenish` | `product_replenish` | Inventory Operations | 11 | 0 | 0 | 0 | 0 |  |
| Project role to users mapping | `project.template.role.to.users.map` | `project_template_role_to_users_map` | Projects and Tasks | 2 | 1 | 0 | 0 | 0 |  |
| Project Sharing | `project.share.wizard` | `project_share_wizard` | Projects and Tasks | 3 | 1 | 0 | 0 | 0 |  |
| Project Sharing Collaborator Wizard | `project.share.collaborator.wizard` | `project_share_collaborator_wizard` | Projects and Tasks | 4 | 0 | 0 | 0 | 0 |  |
| Project Stage Delete Wizard | `project.project.stage.delete.wizard` | `project_project_stage_delete_wizard` | Projects and Tasks | 0 | 1 | 0 | 0 | 0 |  |
| Project Task Stage Delete Wizard | `project.task.type.delete.wizard` | `project_task_type_delete_wizard` | Projects and Tasks | 0 | 2 | 0 | 0 | 0 |  |
| Project Template create Wizard | `project.template.create.wizard` | `project_template_create_wizard` | Projects and Tasks | 7 | 0 | 0 | 0 | 0 |  |
| Put In Pack Wizard | `stock.put.in.pack` | `stock_put_in_pack` | Inventory Operations | 5 | 2 | 0 | 0 | 0 |  |
| Receive Bills Wizard | `l10n_hu_edi_receive.bills.wizard` | `l10n_hu_edi_receive_bills_wizard` | Fiscal Localizations | 2 | 0 | 0 | 0 | 0 |  |
| Regenerate Employee Work Entries | `hr.work.entry.regeneration.wizard` | `hr_work_entry_regeneration_wizard` | Work Entries | 2 | 1 | 0 | 0 | 0 |  |
| Reject Group Message | `mail.group.message.reject` | `mail_group_message_reject` | Messaging and Activities | 4 | 0 | 0 | 0 | 0 |  |
| Remake the sequence of Journal Entries. | `account.resequence.wizard` | `account_resequence_wizard` | General Ledger | 4 | 1 | 0 | 0 | 0 |  |
| Remove email from blacklist wizard | `mail.blacklist.remove` | `mail_blacklist_remove` | Messaging and Activities | 2 | 0 | 0 | 0 | 0 |  |
| Remove phone from blacklist | `phone.blacklist.remove` | `phone_blacklist_remove` | Customer Relationship Management | 2 | 0 | 0 | 0 | 0 |  |
| Request Zakat Tax and Customs Authority one-time password | `l10n_sa_edi.otp.wizard` | `l10n_sa_edi_otp_wizard` | Fiscal Localizations | 3 | 0 | 0 | 0 | 0 |  |
| Reset View Architecture Wizard | `reset.view.arch.wizard` | `reset_view_arch_wizard` | Platform Foundation | 3 | 0 | 0 | 0 | 0 |  |
| Return Picking | `stock.return.picking` | `stock_return_picking` | Inventory Operations | 1 | 0 | 0 | 0 | 0 |  |
| Return Picking Line | `stock.return.picking.line` | `stock_return_picking_line` | Inventory Operations | 5 | 0 | 0 | 0 | 0 |  |
| Robots.txt Editor | `website.robots` | `website_robots` | Website and Storefront | 1 | 0 | 0 | 0 | 0 |  |
| Sale Loyalty - Apply Coupon Wizard | `sale.loyalty.coupon.wizard` | `sale_loyalty_coupon_wizard` | Loyalty and Promotions | 2 | 0 | 0 | 0 | 0 |  |
| Sale Loyalty - Reward Selection Wizard | `sale.loyalty.reward.wizard` | `sale_loyalty_reward_wizard` | Loyalty and Promotions | 3 | 0 | 0 | 0 | 0 |  |
| Sales Advance Payment Invoice | `sale.advance.payment.inv` | `sale_advance_payment_inv` | Sales | 10 | 1 | 0 | 0 | 0 |  |
| Sample Mail Wizard | `mailing.mailing.test` | `mailing_mailing_test` | Marketing and Mass Mailing | 2 | 0 | 0 | 0 | 0 |  |
| schedule a mailing | `mailing.mailing.schedule.date` | `mailing_mailing_schedule_date` | Marketing and Mass Mailing | 2 | 0 | 0 | 0 | 0 |  |
| Secure Journal Entries | `account.secure.entries.wizard` | `account_secure_entries_wizard` | General Ledger | 2 | 0 | 0 | 0 | 0 |  |
| Send Approved Dematerialization Platform Flow Wizard | `l10n.fr.pdp.reports.send.wizard` | `l10n_fr_pdp_reports_send_wizard` | Fiscal Localizations | 2 | 0 | 0 | 0 | 0 |  |
| Send mails to applicants | `applicant.send.mail` | `applicant_send_mail` | Recruitment | 5 | 2 | 0 | 0 | 0 |  |
| Send mails to Drivers | `fleet.vehicle.send.mail` | `fleet_vehicle_send_mail` | Fleet | 5 | 2 | 0 | 0 | 0 |  |
| Send text message Wizard | `sms.composer` | `sms_composer` | Messaging and Activities | 15 | 0 | 0 | 0 | 0 |  |
| Server Action History Wizard | `server.action.history.wizard` | `server_action_history_wizard` | Platform Foundation | 2 | 0 | 0 | 0 | 0 |  |
| Set Homework Location Wizard | `homework.location.wizard` | `homework_location_wizard` | Human Resources Core | 4 | 0 | 0 | 0 | 0 |  |
| Snooze Orderpoint | `stock.orderpoint.snooze` | `stock_orderpoint_snooze` | Inventory Operations | 2 | 1 | 0 | 0 | 0 |  |
| Sparse fields Test | `sparse_fields.test` | `sparse_fields_test` | Automation and Integration | 1 | 0 | 0 | 0 | 0 |  |
| Split Production Detail | `mrp.production.split.line` | `mrp_production_split_line` | Manufacturing | 4 | 0 | 0 | 0 | 0 |  |
| Stock Package Destination | `stock.package.destination` | `stock_package_destination` | Inventory Operations | 1 | 1 | 0 | 0 | 0 |  |
| Stock Quantity History | `stock.quantity.history` | `stock_quantity_history` | Inventory Operations | 1 | 0 | 0 | 0 | 0 |  |
| Stock Quantity Relocation | `stock.quant.relocate` | `stock_quant_relocate` | Inventory Operations | 3 | 1 | 0 | 0 | 0 |  |
| Stock Request an Inventory Count | `stock.request.count` | `stock_request_count` | Inventory Operations | 2 | 1 | 0 | 0 | 0 |  |
| Stock Rules report | `stock.rules.report` | `stock_rules_report` | Inventory Operations | 3 | 2 | 0 | 0 | 0 |  |
| Stock supplier replenishment information | `stock.replenishment.info` | `stock_replenishment_info` | Inventory Operations | 3 | 2 | 0 | 0 | 0 |  |
| Stock warehouse replenishment option | `stock.replenishment.option` | `stock_replenishment_option` | Inventory Operations | 3 | 0 | 0 | 0 | 0 |  |
| Survey Invitation Wizard | `survey.invite` | `survey_invite` | Learning, Surveys and Gamification | 11 | 2 | 1 | 0 | 0 |  |
| Task Sharing | `task.share.wizard` | `task_share_wizard` | Projects and Tasks | 4 | 1 | 0 | 0 | 0 |  |
| Tax audit export - Adóhatósági Ellenőrzési Adatszolgáltatás | `l10n_hu_edi.tax_audit_export` | `l10n_hu_edi_tax_audit_export` | Fiscal Localizations | 5 | 0 | 0 | 0 | 0 |  |
| Technical Annulment Wizard | `l10n_hu_edi.cancellation` | `l10n_hu_edi_cancellation` | Fiscal Localizations | 3 | 0 | 0 | 0 | 0 |  |
| Test text message Mailing | `mailing.sms.test` | `mailing_sms_test` | Marketing and Mass Mailing | 2 | 0 | 0 | 0 | 0 |  |
| text message Account Registration Phone Number Wizard | `sms.account.phone` | `sms_account_phone` | Messaging and Activities | 2 | 0 | 0 | 0 | 0 |  |
| text message Account Sender Name Wizard | `sms.account.sender` | `sms_account_sender` | Messaging and Activities | 2 | 0 | 0 | 0 | 0 |  |
| text message Account Verification Code Wizard | `sms.account.code` | `sms_account_code` | Messaging and Activities | 2 | 0 | 0 | 0 | 0 |  |
| text message Template Preview | `sms.template.preview` | `sms_template_preview` | Messaging and Activities | 3 | 0 | 0 | 0 | 0 |  |
| text message Template Reset | `sms.template.reset` | `sms_template_reset` | Messaging and Activities | 0 | 1 | 0 | 0 | 0 |  |
| text message Twilio Connection Wizard | `sms.twilio.account.manage` | `sms_twilio_account_manage` | Messaging and Activities | 2 | 0 | 0 | 0 | 0 |  |
| time-based one-time password rate limit logs | `auth.totp.rate.limit.log` | `auth_totp_rate_limit_log` | Identity and Access | 3 | 0 | 0 | 0 | 1 |  |
| Traceability Report | `stock.traceability.report` | `stock_traceability_report` | Inventory Operations | 0 | 0 | 0 | 0 | 0 |  |
| Update Loyalty Card Points | `loyalty.card.update.balance` | `loyalty_card_update_balance` | Loyalty and Promotions | 3 | 0 | 0 | 0 | 0 |  |
| Update Module | `base.module.update` | `base_module_update` | Platform Foundation | 3 | 0 | 0 | 0 | 0 |  |
| Update product attribute value | `update.product.attribute.value` | `update_product_attribute_value` | Products and Catalog | 2 | 0 | 0 | 0 | 0 |  |
| Update Tax Tags Wizard | `account.update.tax.tags.wizard` | `account_update_tax_tags_wizard` | Taxes | 2 | 0 | 0 | 0 | 0 |  |
| Update the probabilities | `crm.lead.pls.update` | `crm_lead_pls_update` | Customer Relationship Management | 1 | 1 | 0 | 0 | 0 |  |
| Upgrade Module | `base.module.upgrade` | `base_module_upgrade` | Platform Foundation | 1 | 0 | 0 | 0 | 0 |  |
| User list of blocked 3rd-party domains | `website.custom_blocked_third_party_domains` | `website_custom_blocked_third_party_domains` | Website and Storefront | 1 | 0 | 0 | 0 | 0 |  |
| User, change own password wizard | `change.password.own` | `change_password_own` | Identity and Access | 2 | 0 | 0 | 0 | 0 |  |
| User, Change Password Wizard | `change.password.user` | `change_password_user` | Identity and Access | 4 | 0 | 0 | 0 | 0 |  |
| Validate Account Move | `validate.account.move` | `validate_account_move` | General Ledger | 4 | 1 | 0 | 0 | 0 |  |
| Warn Insufficient Repair Quantity | `stock.warn.insufficient.qty.repair` | `stock_warn_insufficient_qty_repair` | Repair and Maintenance | 5 | 0 | 0 | 0 | 0 |  |
| Warn Insufficient Scrap Quantity | `stock.warn.insufficient.qty.scrap` | `stock_warn_insufficient_qty_scrap` | Inventory Operations | 5 | 0 | 0 | 0 | 0 |  |
| Warn Insufficient Unbuild Quantity | `stock.warn.insufficient.qty.unbuild` | `stock_warn_insufficient_qty_unbuild` | Manufacturing | 5 | 0 | 0 | 0 | 0 |  |
| Wave Transfer Lines | `stock.add.to.wave` | `stock_add_to_wave` | Inventory Operations | 3 | 2 | 0 | 0 | 0 |  |
| Withhold Wizard | `l10n_in.withhold.wizard` | `l10n_in_withhold_wizard` | Fiscal Localizations | 7 | 0 | 0 | 0 | 0 |  |
| Wizard in case of consumption in warning/strict and more component has been used for a manufacturing order (related to the bill of materials) | `mrp.consumption.warning` | `mrp_consumption_warning` | Manufacturing | 0 | 1 | 0 | 0 | 0 |  |
| Wizard in case purchase order still has open alternative requests for quotation | `purchase.requisition.alternative.warning` | `purchase_requisition_alternative_warning` | Purchasing | 0 | 2 | 0 | 0 | 0 |  |
| Wizard to mark as done or create back order | `mrp.production.backorder` | `mrp_production_backorder` | Manufacturing | 0 | 1 | 0 | 0 | 0 |  |
| Wizard to post Manufacturing work in progress account move | `mrp.account.wip.accounting` | `mrp_account_wip_accounting` | Manufacturing | 4 | 1 | 0 | 0 | 0 |  |
| Wizard to preset values for alternative purchase order | `purchase.requisition.create.alternative` | `purchase_requisition_create_alternative` | Purchasing | 2 | 1 | 0 | 0 | 0 |  |
| Wizard to Split a Production | `mrp.production.split` | `mrp_production_split` | Manufacturing | 2 | 0 | 0 | 0 | 0 |  |
| Wizard to Split Multiple Productions | `mrp.production.split.multi` | `mrp_production_split_multi` | Manufacturing | 0 | 0 | 0 | 0 | 0 |  |

### 12.3 Worked examples: two tables in full

Every column of two representative tables, one master-data table and one document-configuration table, as a replacement must create them.

#### The table `res_currency`, entity Currency (`res.currency`)

| Column | Storage type | Required | Index | Foreign key | Field label |
|---|---|---|---|---|---|
| `id` | `integer` — thirty-two bit integer | yes | primary key |  | Identifier, the surrogate key |
| `name` | `character varying` — variable length text | yes |  |  | Currency |
| `symbol` | `character varying` — variable length text | yes |  |  | Symbol |
| `iso_numeric` | `integer` — thirty-two bit integer | no |  |  | Currency numeric code. |
| `decimal_places` | `integer` — thirty-two bit integer | no |  |  | Decimal Places |
| `create_uid` | `integer` — thirty-two bit integer | no |  | `res_users`, on deletion set null | Created by, the acting user at creation |
| `write_uid` | `integer` — thirty-two bit integer | no |  | `res_users`, on deletion set null | Last updated by, the acting user of the last write |
| `full_name` | `character varying` — variable length text | no |  |  | Name |
| `position` | `character varying` — variable length text | no |  |  | Symbol Position |
| `currency_unit_label` | `jsonb` — structured document | no |  |  | Currency Unit |
| `currency_subunit_label` | `jsonb` — structured document | no |  |  | Currency Subunit |
| `rounding` | `numeric` — exact decimal number | no |  |  | Rounding Factor |
| `active` | `boolean` — boolean | no |  |  | Active |
| `create_date` | `timestamp without time zone` — date and time without zone offset, stored in coordinated universal time | no |  |  | Created on, the timestamp of the creating transaction |
| `write_date` | `timestamp without time zone` — date and time without zone offset, stored in coordinated universal time | no |  |  | Last updated on, the timestamp of the last write |
| `l10n_ar_afip_code` | `character varying` — variable length text, at most 4 characters | no |  |  | ARCA Code |
| `l10n_cl_currency_code` | `jsonb` — structured document | no |  |  | Currency Code |
| `l10n_cl_short_name` | `jsonb` — structured document | no |  |  | Short Name |

#### The table `account_payment_term`, entity Payment Terms (`account.payment.term`)

| Column | Storage type | Required | Index | Foreign key | Field label |
|---|---|---|---|---|---|
| `id` | `integer` — thirty-two bit integer | yes | primary key |  | Identifier, the surrogate key |
| `company_id` | `integer` — thirty-two bit integer | no |  | `res_company`, on deletion set null | Company |
| `sequence` | `integer` — thirty-two bit integer | yes |  |  | Sequence |
| `discount_days` | `integer` — thirty-two bit integer | no |  |  | Discount Days |
| `create_uid` | `integer` — thirty-two bit integer | no |  | `res_users`, on deletion set null | Created by, the acting user at creation |
| `write_uid` | `integer` — thirty-two bit integer | no |  | `res_users`, on deletion set null | Last updated by, the acting user of the last write |
| `early_pay_discount_computation` | `character varying` — variable length text | no |  |  | Cash Discount Tax Reduction |
| `name` | `jsonb` — structured document | yes |  |  | Payment Terms |
| `note` | `jsonb` — structured document | no |  |  | Description on the Invoice |
| `active` | `boolean` — boolean | no |  |  | Active |
| `display_on_invoice` | `boolean` — boolean | no |  |  | Show installment dates |
| `early_discount` | `boolean` — boolean | no |  |  | Early Discount |
| `create_date` | `timestamp without time zone` — date and time without zone offset, stored in coordinated universal time | no |  |  | Created on, the timestamp of the creating transaction |
| `write_date` | `timestamp without time zone` — date and time without zone offset, stored in coordinated universal time | no |  |  | Last updated on, the timestamp of the last write |
| `discount_percentage` | `double precision` — binary floating point number | no |  |  | Discount % |

## 13. Totals

| Measure | Count |
|---|---|
| Relations | 1,240 |
| Of which tables | 1,217 |
| Of which stored queries materialized as views | 23 |
| Relations backing an entity | 803 |
| Association tables | 422 |
| Columns, all relations | 13,518 |
| Persistent entities | 600 |
| Interactive assistant entities | 222 |
| Shared behaviour entities, which have no table | 161 |
| Persistent entities read from a stored query instead of a table | 32 |
| Of which materialized as a view | 22 |
| Of which built at read time, with no relation in the schema | 10 |
| Entities classified persistent with no relation for another reason | 2 |
| Entities with no audit columns | 10 |
| Value columns on persistent tables, system columns excluded | 7,623 |
| Value columns on interactive assistant tables, system columns excluded | 1,098 |
| Indexes | 2,933 |
| Column indexes created from a field declaration | 1,000 |
| Column indexes of kind plain | 603 |
| Column indexes of kind partial | 360 |
| Column indexes of kind text similarity | 37 |
| Foreign keys | 4,575 |
| Declared table constraints | 291 |
| Declared unique indexes | 34 |
| Declared indexes | 65 |
| Unique and check constraints reported by the store | 296 |
| Identifier generators | 790 |
| Document numbering generators | 35 |
| Stored columns carrying per-language text | 319 |
| Stored columns carrying per-company values | 52 |
| Stored columns carrying binary content inline | 12 |
| Entities carrying a materialized ancestor path | 11 |

## 14. The complete column-level catalogue

This file states the rules and the per-entity totals. The exhaustive, machine-readable catalogue is in [`../../schemas/data/`](../../schemas/data/). Every document listed below is a structured-data document, and the folder carries its own index that describes each one.

| Document | Content |
|---|---|
| `schemas/data/entity-index.json` | Every entity with its transport name, storage name, full name, kind, defining package, extending packages and field, operation, constraint, message and view counts. |
| `schemas/data/entities/<transport name>.json` | One document per entity: every field with its type, target entity, deletion behaviour, default, precision, index kind, translatability, company dependence, copy behaviour, tracking, access restriction and derivation; the constraints; the validation rules; the state fields; the operations; the access rights and the record rules; the views, actions, menus, scheduled jobs, sequences and message templates. |
| `schemas/data/physical-tables.json` | Every table with every column, its storage type, its nullability, its default, its length and precision, and the table's indexes and constraints. |
| `schemas/data/association-tables.json` | Every association table with its two columns, its foreign keys and its indexes. |
| `schemas/data/foreign-keys.json` | Every foreign key with its table, its definition and its referential action. |
| `schemas/data/indexes.json` | Every index with its table, its uniqueness and its definition. |
| `schemas/data/table-constraints.json` | Every unique and check constraint the store reports. |
| `schemas/data/constraints.json` | Every constraint and index an entity declares, with its message. |
| `schemas/data/database-sequences.json` | Every generator with its start value and its increment. |
| `schemas/data/relations.json` | Every relational field with source, field, cardinality, target, deletion behaviour, inverse field and association table. |
| `schemas/data/selection-values.json` and `schemas/data/resolved-selection-values.json` | Every closed value list with its values and labels. |
| `schemas/data/state-machines.json` | Every state field with its values and the operations that move a record into each value. |
| `schemas/data/computed-fields.json` | Every derived field with its dependencies. |
| `schemas/data/reference-data/` | Every shipped record per entity, keyed by external identifier, plus the shipped data files and the country chart templates. |

When this file and the catalogues disagree, the catalogues are the enumeration of record and this file is the statement of the rules.

## 15. Reconciliation notes

The target branch carried no file for this topic, so the structure and the prose come from the working branch. Every enumeration was regenerated from the machine-readable catalogues of this repository, which are observed on a live installation, and the following differences were resolved against them and against the source tree.

| Point | Working branch | Resolution |
|---|---|---|
| Table, column, index, constraint and association-table names | Written as de-identified paraphrases: tables named after the full name of the entity, association tables named `<entity>__<field>__association`, association columns named `<entity>_identifier`, system columns named `identifier`, `created_on`, `created_by_user`, `last_updated_on`, `last_updated_by_user`. | Replaced throughout by the names the observed schema reports: `<transport name with underscores>` for a table, `<first table>_<second table>_rel` for an association table, `<table>_id` for an association column, and `id`, `create_date`, `create_uid`, `write_date`, `write_uid` for the system columns. A physical catalogue whose names cannot be found in the database it describes is unusable, and rule three of the documentation rules requires identifiers to be reproduced exactly. |
| Number of tables | Counted only the tables of entities: 601 persistent and 222 assistant. | The observed schema holds 1,240 relations, of which 1,217 are tables and 23 are views, 803 back an entity and 422 are association tables. The entity counts themselves are 600 persistent and 222 interactive assistants; the difference of one on the persistent side is that one entity classified as persistent is a shared behaviour with no records of its own. |
| Binary columns held inline | "Seven fields opt out of attachment storage." | Twelve do, and the observed schema carries exactly twelve `bytea` columns. All twelve are named in section 4.3. |
| Association tables | 428 were counted. | The observed schema carries 422, and each is listed in section 8.2 with both foreign keys. |
| Indexed columns | 877 were counted, of which 555 plain and 285 partial. | The observed schema carries 1,000 column indexes, of which 603 plain, 360 partial and 37 text similarity. The count of text-similarity indexes is identical in both, which is what identifies the difference as coverage rather than definition. |
| Declared constraints | 291 table constraints, 34 unique indexes, 65 indexes. | Identical in both; the catalogue of section 11.2 reproduces all 390 with their exact messages. |
| Column type counts | Counted per declared field type over persistent tables only. | Both counts are given: by declared field type in the last column of section 3, and by the type the storage engine reports across all tables, because a rebuild needs the second to size its schema and the first to map its field definitions. |
| Domain attribution of an entity | Named after the working taxonomy, whose keys differ from the folder names of this repository. | Replaced by the folder that specifies the entity in this repository. For the entities of the foundation package the repository's own attribution names the first domain whose scope lists that package, which is not the folder that specifies them; the writing charter overrides it, so the generic platform entities are attributed to the platform foundation, the shared party, company, country, language and bank entities to contacts and organizations, the currency entities to multi-currency, and the user, group, device and key entities to identity and access. |
