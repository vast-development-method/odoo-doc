# Bank Accounts (`res.partner.bank`)

**Transport name:** `res.partner.bank`  
**Storage name:** `res_partner_bank`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`  
**Extended by packages:** `account`, `account_qr_code_emv`, `base_iban`, `account_qr_code_sepa`, `hr`, `l10n_ar`, `l10n_au`, `l10n_br`, `l10n_ch`, `l10n_hk`, `l10n_id`, `l10n_kh`, `l10n_mx`, `l10n_sg`, `l10n_th`, `l10n_us`, `l10n_vn`

Description: Bank Accounts

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `mail.activity.mixin`
- Default ordering: `sequence, id`
- Display name field: `acc_number`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (56)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `active` | Active | boolean |  | default `True`; changes are tracked in the message thread; extended by packages `account` |
| `acc_type` | Type | selection |  | computed by rule `_compute_acc_type` (not stored); Help: Bank account type: Normal or IBAN. Inferred from the bank account number. |
| `acc_number` | Account Number | single line text |  | required; searchable through a search rule; changes are tracked in the message thread; extended by packages `account` |
| `clearing_number` | Clearing Number | single line text |  | changes are tracked in the message thread; extended by packages `account` |
| `sanitized_acc_number` | Sanitized Account Number | single line text |  | read only; computed by rule `_compute_sanitized_acc_number` and stored |
| `acc_holder_name` | Account Holder Name | single line text |  | computed by rule `_compute_account_holder_name` and stored; changes are tracked in the message thread; Help: Account holder name, in case it is different than the name of the Account Holder; extended by packages `account` |
| `partner_id` | Account Holder | many to one | `res.partner` | required; changes are tracked in the message thread; indexed; on delete of the target: cascade; restricted by domain `["\|", ["is_company", "=", true], ["parent_id", "=", false]]`; extended by packages `account` |
| `allow_out_payment` | Send Money | boolean |  | default ; changes are tracked in the message thread; not copied on duplication; Help: Sending fake invoices with a fraudulent account number is a common phishing practice. To protect yourself, always verify new bank account numbers, preferably by calling the vendor, as phishing usually happens when their emails are compromised. Once verified, you can activate the ability to send money.; extended by packages `account` |
| `bank_id` | Bank | many to one | `res.bank` | changes are tracked in the message thread; extended by packages `account` |
| `bank_name` | Bank Name | single line text |  | related through path `bank_id.name` |
| `bank_bic` | Bank Bank identifier code | single line text |  | related through path `bank_id.bic` |
| `sequence` | Sequence | integer |  | default `10` |
| `currency_id` | Currency | many to one | `res.currency` | changes are tracked in the message thread; extended by packages `account` |
| `company_id` | Company | many to one | `res.company` | read only; related through path `partner_id.company_id` and stored |
| `country_code` | Country Code | single line text |  | related through path `partner_id.country_code` |
| `note` | Notes | multi line text |  |  |
| `color` | Color | integer |  | computed by rule `_compute_color` (not stored) |
| `journal_id` | Account Journal | one to many | `account.journal` | read only; restricted by domain `[["type", "=", "bank"]]`; must belong to the same company; inverse field `bank_account_id`; Help: The accounting journal corresponding to this bank account. |
| `has_iban_warning` | Has International bank account number Warning | boolean |  | computed by rule `_compute_display_account_warning` and stored; Help: Technical field used to display a warning if the IBAN country is different than the holder country. |
| `partner_country_name` | Partner Country Name | single line text |  | related through path `partner_id.country_id.name` |
| `has_money_transfer_warning` | Has Money Transfer Warning | boolean |  | computed by rule `_compute_display_account_warning` and stored; Help: Technical field used to display a warning if the account is a transfer service account. |
| `money_transfer_service` | Money Transfer Service | single line text |  | computed by rule `_compute_money_transfer_service_name` (not stored) |
| `partner_supplier_rank` | Partner Supplier Rank | integer |  | related through path `partner_id.supplier_rank` |
| `partner_customer_rank` | Partner Customer Rank | integer |  | related through path `partner_id.customer_rank` |
| `related_moves` | Related Moves | one to many | `account.move` | inverse field `partner_bank_id` |
| `user_has_group_validate_bank_account` | User Has Group Validate Bank Account | boolean |  | computed by rule `_compute_user_has_group_validate_bank_account` (not stored) |
| `lock_trust_fields` | Lock Trust Fields | boolean |  | computed by rule `_compute_lock_trust_fields` (not stored) |
| `duplicate_bank_partner_ids` | Duplicate Bank Partner | many to many | `res.partner` | computed by rule `_compute_duplicate_bank_partner_ids` (not stored) |
| `display_qr_setting` | Display Quick response Setting | boolean |  | computed by rule `_compute_display_qr_setting` (not stored) |
| `include_reference` | Include Reference | boolean |  | Help: Include the reference in the QR code. |
| `proxy_type` | Proxy Type | selection |  | default `none`; on delete of the target: {"merchant_id": "set default", "payment_service": "set default", "atm_card": "set default", "bank_acc": "set default"}; extended by packages `l10n_br`, `l10n_hk`, `l10n_kh`, `l10n_sg`, `l10n_th`, `l10n_vn` |
| `country_proxy_keys` | Country Proxy Keys | single line text |  | computed by rule `_compute_country_proxy_keys` (not stored) |
| `proxy_value` | Proxy Value | single line text |  |  |
| `bank_street` | Bank Street | single line text |  | related through path `bank_id.street` |
| `bank_street2` | Bank Street2 | single line text |  | related through path `bank_id.street2` |
| `bank_zip` | Bank Zip | single line text |  | related through path `bank_id.zip` |
| `bank_city` | Bank City | single line text |  | related through path `bank_id.city` |
| `bank_state` | Bank State | many to one |  | related through path `bank_id.state` |
| `bank_country` | Bank Country | many to one |  | related through path `bank_id.country` |
| `bank_email` | Bank Email | single line text |  | related through path `bank_id.email` |
| `bank_phone` | Bank Phone | single line text |  | related through path `bank_id.phone` |
| `employee_id` | Employee | many to many | `hr.employee` | computed by rule `_compute_employee_id` (not stored); searchable through a search rule; association table `Employee` |
| `employee_salary_amount` | Salary Allocation | float |  | read only; computed by rule `_compute_salary_amount` (not stored); precision `[16, 4]` |
| `employee_salary_amount_is_percentage` | Employee Salary Amount Is Percentage | boolean |  | read only; computed by rule `_compute_salary_amount` (not stored) |
| `currency_symbol` | Currency Symbol | single line text |  | related through path `currency_id.symbol` |
| `employee_has_multiple_bank_accounts` | Employee Has Multiple Bank Accounts | boolean |  | related through path `employee_id.has_multiple_bank_accounts` |
| `aba_bsb` | BSB | single line text |  | Help: Bank State Branch code - needed if payment is to be made using ABA files |
| `l10n_ch_qr_iban` | quick response-international bank account number | single line text |  | computed by rule `_compute_l10n_ch_qr_iban` and stored; Help: Put the QR-IBAN here for your own bank accounts.  That way, you can still use the main IBAN in the Account Number while you will see the QR-IBAN for the barcode. |
| `l10n_ch_display_qr_bank_options` | Localization Ch Display Quick response Bank Options | boolean |  | computed by rule `_compute_l10n_ch_display_qr_bank_options` (not stored) |
| `l10n_id_qris_api_key` | QRIS application programming interface Key | single line text |  | visible only to groups `base.group_system` |
| `l10n_id_qris_mid` | QRIS Merchant identifier | single line text |  | visible only to groups `base.group_system` |
| `l10n_kh_merchant_id` | Merchant identifier | single line text |  |  |
| `l10n_mx_edi_clabe` | CLABE | single line text |  | Help: Standardized banking cipher for Mexico. More info wikipedia.org/wiki/CLABE |
| `fiscal_country_codes` | Fiscal Country Codes | single line text |  | default computed dynamically (_get_fiscal_country_codes) |
| `show_aba_routing` | Show Aba Routing | boolean |  | computed by rule `_compute_show_aba_routing` (not stored) |
| `l10n_us_bank_account_type` | Bank Account Type | selection |  | required; default `checking` |

## Selection values

### `proxy_type` (Proxy Type)

| Value | Label |
|---|---|
| `none` | None |
| `email` | Email Address |
| `mobile` | Mobile Number |
| `br_cpf_cnpj` | CPF/CNPJ (BR) |
| `br_random` | Random Key (BR) |
| `id` | FPS ID |
| `bakong_id_solo` | Bakong Account ID (Solo Merchant) |
| `bakong_id_merchant` | Bakong Account ID (Corporate Merchant) |
| `uen` | UEN |
| `ewallet_id` | Ewallet ID |
| `merchant_tax_id` | Merchant Tax ID |
| `merchant_id` | Merchant ID |
| `payment_service` | Payment Service |
| `atm_card` | ATM Card Number |
| `bank_acc` | Bank Account |

### `l10n_us_bank_account_type` (Bank Account Type)

| Value | Label |
|---|---|
| `checking` | Checking |
| `savings` | Savings |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_unique_number` | Constraint | `unique(sanitized_acc_number, partner_id)` | The combination Account Number/Partner must be unique. | `base` |

## Operations (72)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `get_supported_account_types` | operation | self | `base` | model |  |
| `_get_supported_account_types` | preparation rule | self | `base_iban`, `base`, `l10n_ar`, `l10n_au` | model | Add new account type named cbu used in Argentina |
| `_compute_sanitized_acc_number` | computation | self | `base` | depends: `acc_number` |  |
| `_search_acc_number` | search rule | self, operator, value | `base` |  |  |
| `_compute_acc_type` | computation | self | `base`, `l10n_au` | depends: `acc_number` | Criteria to be an ABA account: - Spaces, hypens, digits are valid. - Total length must be 9 or less. - Cannot be only spaces, zeros or hyphens (must have at least one digit in range 1-9) |
| `_compute_account_holder_name` | computation | self | `base` | depends: `partner_id` |  |
| `retrieve_acc_type` | operation | self, acc_number | `base_iban`, `base`, `l10n_ar` | model | To be overridden by subclasses in order to support other account_types. |
| `_compute_display_name` | computation | self | `account`, `base`, `hr` | depends: `acc_number`, `bank_id`; depends: `allow_out_payment`, `acc_number`, `bank_id`; depends_context: `display_account_trust` |  |
| `_compute_color` | computation | self | `base` | depends: `allow_out_payment` |  |
| `_sanitize_vals` | internal rule | self, vals | `base` |  |  |
| `create` | lifecycle override | self, vals_list | `account`, `base_iban`, `base`, `l10n_ch` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `account`, `base_iban`, `base`, `l10n_ch` |  |  |
| `action_archive_bank` | user action | self | `base` |  | Custom archive function because the basic action_archive don't trigger a re-rendering of the page, so the archived value is still visible in the view. |
| `unlink` | lifecycle override | self | `account`, `base` |  | Instead of deleting a bank account, we want to archive it since we cannot delete bank account that is linked to any entries |
| `_user_can_trust` | internal rule | self | `account`, `base` |  |  |
| `_find_or_create_bank_account` | internal rule | self, account_number, partner, company, allow_company_account_creation, extra_create_vals | `base` |  | Find a bank account for the given partner and number. Create it if it doesn't exist.  Manage different corner cases:  - make sure that we don't try to create the bank number if we look for it but it exists restricted in another   company; because of the unique constraint - make sure that we don't create a bank account number for one of the database's companies, unless   `allow_company_account_creation` is specified  :param account_number: the bank account number to search for (or to create) :param partner: the partner linked to the account number :param company: the company that the bank needs |
| `_check_journal_id` | validation | self | `account` | constrains: `journal_id` |  |
| `_check_allow_out_payment` | validation | self | `account` |  | Block enabling the setting, but it can be set to false without the group. (For example, at creation) |
| `_compute_duplicate_bank_partner_ids` | computation | self | `account` | depends: `acc_number` |  |
| `_compute_display_account_warning` | computation | self | `account` | depends: `partner_id.country_id`, `sanitized_acc_number`, `allow_out_payment`, `acc_type` |  |
| `_compute_money_transfer_service_name` | computation | self | `account` | depends: `sanitized_acc_number`, `allow_out_payment` |  |
| `_get_money_transfer_services` | preparation rule | self | `account` |  |  |
| `_compute_user_has_group_validate_bank_account` | computation | self | `account` | depends: `acc_number`; depends_context: `uid` |  |
| `_compute_lock_trust_fields` | computation | self | `account` | depends: `allow_out_payment` |  |
| `_build_qr_code_vals` | internal rule | self, amount, free_communication, structured_communication, currency, debtor_partner, qr_method, silent_errors | `account` |  | Returns the QR-code vals needed to generate the QR-code report link to pay this account with the given parameters, or None if no QR-code could be generated.  :param amount: The amount to be paid :param free_communication: Free communication to add to the payment when generating one with the QR-code :param structured_communication: Structured communication to add to the payment when generating one with the QR-code :param currency: The currency in which amount is expressed :param debtor_partner: The partner to which this QR-code is aimed (so the one who will have to pay) :param qr_method: The QR |
| `build_qr_code_url` | operation | self, amount, free_communication, structured_communication, currency, debtor_partner, qr_method, silent_errors | `account` |  |  |
| `build_qr_code_base64` | operation | self, amount, free_communication, structured_communication, currency, debtor_partner, qr_method, silent_errors | `account` |  |  |
| `_get_qr_vals` | preparation rule | self, qr_method, amount, currency, debtor_partner, free_communication, structured_communication | `account_qr_code_emv`, `account_qr_code_sepa`, `account`, `l10n_ch`, `l10n_id` |  | Getting content for the QR through calling QRIS API and storing the QRIS transaction as a record |
| `_get_qr_code_generation_params` | preparation rule | self, qr_method, amount, currency, debtor_partner, free_communication, structured_communication | `account_qr_code_emv`, `account_qr_code_sepa`, `account`, `l10n_ch`, `l10n_id` |  |  |
| `_get_qr_code_url` | preparation rule | self, qr_method, amount, currency, debtor_partner, free_communication, structured_communication | `account` |  | Hook for extension, to support the different QR generation methods. This function uses the provided qr_method to try generation a QR-code for the given data. It it succeeds, it returns the report URL to make this QR-code; else None.  :param qr_method: The QR generation method to be used to make the QR-code. :param amount: The amount to be paid :param currency: The currency in which amount is expressed :param debtor_partner: The partner to which this QR-code is aimed (so the one who will have to pay) :param free_communication: Free communication to add to the payment when generating one with th |
| `_get_qr_code_base64` | preparation rule | self, qr_method, amount, currency, debtor_partner, free_communication, structured_communication | `account` |  | Hook for extension, to support the different QR generation methods. This function uses the provided qr_method to try generation a QR-code for the given data. It it succeeds, it returns QR code in base64 url; else None.  :param qr_method: The QR generation method to be used to make the QR-code. :param amount: The amount to be paid :param currency: The currency in which amount is expressed :param debtor_partner: The partner to which this QR-code is aimed (so the one who will have to pay) :param free_communication: Free communication to add to the payment when generating one with the QR-code :par |
| `_get_available_qr_methods` | preparation rule | self | `account_qr_code_emv`, `account_qr_code_sepa`, `account`, `l10n_ch`, `l10n_id` | model | Returns the QR-code generation methods that are available on this db, in the form of a list of (code, name, sequence) elements, where 'code' is a unique string identifier, 'name' the name to display to the user to designate the method, and 'sequence' is a positive integer indicating the order in which those mehtods need to be checked, to avoid shadowing between them (lower sequence means more prioritary). |
| `get_available_qr_methods_in_sequence` | operation | self | `account` | model | Same as _get_available_qr_methods but without returning the sequence, and using it directly to order the returned list. |
| `_get_error_messages_for_qr` | preparation rule | self, qr_method, debtor_partner, currency | `account_qr_code_emv`, `account_qr_code_sepa`, `account`, `l10n_br`, `l10n_ch`, `l10n_hk`, `l10n_id`, `l10n_kh`, `l10n_sg`, `l10n_th`, `l10n_vn` |  | Tells whether or not the criteria to apply QR-generation method qr_method are met for a payment on this account, in the given currency, by debtor_partner. This does not impeach generation errors, it only checks that this type of QR-code *should be* possible to generate. If not, returns an adequate error message to be displayed to the user if need be. Consistency of the required field needs then to be checked by _check_for_qr_code_errors(). :returns:  None if the qr method is eligible, or the error message |
| `_check_for_qr_code_errors` | validation | self, qr_method, amount, currency, debtor_partner, free_communication, structured_communication | `account_qr_code_emv`, `account_qr_code_sepa`, `account`, `l10n_br`, `l10n_ch`, `l10n_hk`, `l10n_id`, `l10n_kh`, `l10n_sg`, `l10n_th`, `l10n_vn` |  | Checks the data before generating a QR-code for the specified qr_method (this method must have been checked for eligbility by _get_error_messages_for_qr() first).  Returns None if no error was found, or a string describing the first error encountered so that it can be reported to the user. |
| `action_open_business_doc` | user action | self | `account` |  |  |
| `default_get` | lifecycle override | self, fields | `account` | model |  |
| `_serialize` | internal rule | self, header, value | `account_qr_code_emv` | model |  |
| `_remove_accents` | internal rule | self, string | `account_qr_code_emv` | model |  |
| `_compute_country_proxy_keys` | computation | self | `account_qr_code_emv`, `l10n_br`, `l10n_hk`, `l10n_kh`, `l10n_sg`, `l10n_th`, `l10n_vn` | depends: `country_code` |  |
| `_compute_display_qr_setting` | computation | self | `account_qr_code_emv`, `l10n_br`, `l10n_hk`, `l10n_kh`, `l10n_sg`, `l10n_th`, `l10n_vn` | depends: `country_code`; depends_context: `company` | Override. |
| `_get_crc16` | preparation rule | self, data, poly, init | `account_qr_code_emv` |  |  |
| `_get_merchant_account_info` | preparation rule | self | `account_qr_code_emv`, `l10n_br`, `l10n_hk`, `l10n_kh`, `l10n_sg`, `l10n_th`, `l10n_vn` |  | Override. |
| `_get_additional_data_field` | preparation rule | self, comment | `account_qr_code_emv`, `l10n_br`, `l10n_hk`, `l10n_kh`, `l10n_sg`, `l10n_vn` |  | Override. |
| `_get_merchant_category_code` | preparation rule | self | `account_qr_code_emv`, `l10n_kh` |  |  |
| `_get_qr_code_vals_list` | preparation rule | self, qr_method, amount, currency, debtor_partner, free_communication, structured_communication | `account_qr_code_emv`, `l10n_br`, `l10n_kh`, `l10n_vn` |  | Override. Force the amount field to always have two decimals. Uppercase the merchant name and merchant city. Although not specified explicitly in the spec, not uppercasing causes errors when scanning the code. Also ensure there is always some comment set. |
| `get_bban` | operation | self | `base_iban` |  |  |
| `_check_iban` | validation | self | `base_iban` | constrains: `acc_number` |  |
| `check_iban` | operation | self, iban | `base_iban` |  |  |
| `_compute_salary_amount` | computation | self | `hr` | depends: `employee_id.salary_distribution` |  |
| `_search_employee_id` | search rule | self, operator, value | `hr` |  |  |
| `action_open_allocation_wizard` | user action | self | `hr` |  |  |
| `_compute_employee_id` | computation | self | `hr` | depends: `partner_id` |  |
| `_validate_aba_bsb` | validation | self | `l10n_au` | constrains: `aba_bsb` |  |
| `_check_br_proxy` | validation | self | `l10n_br` | constrains: `proxy_type`, `proxy_value`, `partner_id` |  |
| `_compute_l10n_ch_display_qr_bank_options` | computation | self | `l10n_ch` | depends: `partner_id`, `company_id` |  |
| `_compute_l10n_ch_qr_iban` | computation | self | `l10n_ch` | depends: `acc_number` |  |
| `_l10n_ch_filter_text` | internal rule | self, value | `l10n_ch` |  |  |
| `_l10n_ch_get_qr_vals` | internal rule | self, amount, currency, debtor_partner, free_communication, structured_communication | `l10n_ch` |  |  |
| `_get_partner_address_lines` | preparation rule | self, partner | `l10n_ch` |  | Retrieves the partner's address fields, truncated to respect the line specs. :returns: tuple(street, street_number, zip, city) |
| `_is_qr_reference` | internal rule | self, reference | `l10n_ch` | model | Checks whether the given reference is a QR-reference, i.e. it is made of 27 digits, the 27th being a mod10r check on the 26 previous ones. |
| `_is_iso11649_reference` | internal rule | self, reference | `l10n_ch` | model | Checks whether the given reference is a ISO11649 (SCOR) reference. |
| `_l10n_ch_qr_debtor_check` | internal rule | self, debtor_partner | `l10n_ch` |  | This method should be used in _get_error_messages_for_qr and _check_for_qr_code_errors It allows is to permit to set this qr method if a partner is not yet provided when executing _get_error_messages_for_qr while preventing to print qr code when executing _check_for_qr_code_errors if the partner is not provided |
| `_check_hk_proxy` | validation | self | `l10n_hk` | constrains: `proxy_type`, `proxy_value`, `partner_id` |  |
| `_l10n_id_qris_fetch_status` | internal rule | self, qr_data | `l10n_id` |  | using self and the given data, fetches the status of a specific QR code generated by QRIS Expected values in the qr_data dict are:     - invoice_id returned when generating a QR code     - the amount present in the qr code     - the datetime at which the QR code was generated |
| `_check_kh_proxy` | validation | self | `l10n_kh` | constrains: `proxy_type`, `proxy_value` |  |
| `_get_fiscal_country_codes` | preparation rule | self | `l10n_mx` |  |  |
| `_check_sg_proxy` | validation | self | `l10n_sg` | constrains: `proxy_type`, `proxy_value`, `partner_id` |  |
| `_check_th_proxy` | validation | self | `l10n_th` | constrains: `proxy_type`, `proxy_value`, `partner_id` |  |
| `_compute_show_aba_routing` | computation | self | `l10n_us` | depends: `country_code`, `acc_type` |  |
| `_check_clearing_number_us` | validation | self | `l10n_us` | constrains: `clearing_number` |  |
| `_check_vn_proxy` | validation | self | `l10n_vn` | constrains: `proxy_type` |  |

## Validation and error messages (29)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_find_or_create_bank_account` | UserError | Please add your own bank account manually: %(account_number)s (%(partner)s) | `base` |
| `_check_journal_id` | ValidationError | A bank account can belong to only one journal. | `account` |
| `_check_allow_out_payment` | ValidationError | You do not have the right to trust or un-trust a bank account. | `account` |
| `_build_qr_code_vals` | UserError | Currency must always be provided in order to generate a QR-code | `account` |
| `_build_qr_code_vals` | UserError | The following error prevented '%(candidate)s' QR-code to be generated though it was detected as eligible: | `account` |
| `create` | UserError | A bank account with Account Number %(number)s already exists for Partner %(partner)s, but is archived. Please unarchive it instead. | `account` |
| `write` | UserError | You cannot modify the account number or partner of an account that has been trusted. | `account` |
| `write` | UserError | You do not have the rights to trust or un-trust accounts. | `account` |
| `get_bban` | UserError | Cannot compute the BBAN because the account number is not an IBAN. | `base_iban` |
| `_validate_aba_bsb` | ValidationError | BSB is not valid (expected format is "NNN-NNN"). Please rectify. | `l10n_au` |
| `_check_br_proxy` | ValidationError | The proxy type must be Email Address, Mobile Number, CPF/CNPJ (BR) or Random Key (BR) for Pix code generation. | `l10n_br` |
| `_check_br_proxy` | ValidationError | %s is not a valid email. | `l10n_br` |
| `_check_br_proxy` | ValidationError | %s is not a valid CPF or CNPJ (don't include periods or dashes). | `l10n_br` |
| `_check_br_proxy` | ValidationError | The mobile number %s is invalid. It must start with +55, contain a 2 digit territory or state code followed by a 9 digit number. | `l10n_br` |
| `_check_br_proxy` | ValidationError | The random key %s is invalid, the format looks like this: 71d6c6e1-64ea-4a11-9560-a10870c40ca2 | `l10n_br` |
| `_check_hk_proxy` | ValidationError | The FPS Type must be either ID, Mobile or Email to generate a FPS QR code for account number %s. | `l10n_hk` |
| `_check_hk_proxy` | ValidationError | Invalid FPS ID! Please enter a valid FPS ID with length 7 or 9 for account number %s. | `l10n_hk` |
| `_check_hk_proxy` | ValidationError | Invalid Mobile! Please enter a valid mobile number with format +852-67891234 for account number %s. | `l10n_hk` |
| `_check_hk_proxy` | ValidationError | Invalid Email! Please enter a valid email address for account number %s. | `l10n_hk` |
| `_get_qr_vals` | ValidationError | response.get('data') | `l10n_id` |
| `_check_kh_proxy` | ValidationError | The proxy type must be Bakong Account ID | `l10n_kh` |
| `_check_kh_proxy` | ValidationError | Please enter a valid Bakong Account ID. | `l10n_kh` |
| `_check_kh_proxy` | ValidationError | Merchant ID is missing. | `l10n_kh` |
| `_check_sg_proxy` | ValidationError | The PayNow Type must be either Mobile or UEN to generate a PayNow QR code for account number %s. | `l10n_sg` |
| `_check_th_proxy` | ValidationError | The QR Code Type must be either Ewallet ID, Merchant Tax ID or Mobile Number to generate a Thailand Bank QR code for account number %s. | `l10n_th` |
| `_check_th_proxy` | ValidationError | The Merchant Tax ID must be in the format 1234567890123 for account number %s. | `l10n_th` |
| `_check_th_proxy` | ValidationError | The Mobile Number must be in the format 0812345678 for account number %s. | `l10n_th` |
| `_check_clearing_number_us` | ValidationError | ABA/Routing should only contain numbers (maximum 9 digits). | `l10n_us` |
| `_check_vn_proxy` | ValidationError | The QR Code Type must be either Merchant ID, ATM Card Number or Bank Account to generate a Vietnam Bank QR code for account number %s. | `l10n_vn` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_user` | no | yes | no | no | `base` |
| `group_partner_manager` | yes | yes | yes | yes | `base` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Billing: Allow accessing employee bank accounts | `[(4, ref('account.group_account_invoice'))]` | `[(1, '=', 1)]` | True | True | True | True |
| Partner bank company rule | global (all users) | `['\|', ('company_id', 'parent_of', company_ids), ('company_id', '=', False)]` | True | True | True | True |
| HR: Prevent non HR officers from accessing employee bank accounts | `[(4, ref('base.group_user'))]` | `[('partner_id.employee_ids', '=', False)]` | True | True | True | True |
| HR: Allow HR officers from accessing employee bank accounts | `[(4, ref('hr.group_hr_user'))]` | `[(1, '=', 1)]` | True | True | True | True |

## Views (20)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.view_partner_bank_form_inherit_account` | xpath | `base.view_partner_bank_form` | `duplicate_bank_partner_ids` |  |  | `account` |
| `account.view_partner_bank_search_inherit` | xpath | `base.view_partner_bank_search` |  |  | `Trusted`, `Untrusted`, `To validate`, `Customers`, `Vendors`, `Phishing risk: High`, `Phishing risk: Medium`, `Created On`, `Created On`, `Created By` | `account` |
| `account_qr_code_emv.view_partner_bank_form_inherit_account` | xpath | `base.view_partner_bank_form` | `display_qr_setting`, `country_proxy_keys`, `proxy_type`, `proxy_value`, `include_reference` |  |  | `account_qr_code_emv` |
| `base.view_partner_bank_form` | form |  | `acc_number`, `clearing_number`, `partner_id`, `acc_holder_name`, `bank_id`, `allow_out_payment`, `company_id`, `currency_id`, `note` | `Archive` |  | `base` |
| `base.view_partner_bank_tree` | list |  | `sequence`, `acc_number`, `partner_id`, `bank_name`, `company_id`, `allow_out_payment`, `active` |  |  | `base` |
| `base.view_partner_bank_search` | search |  | `bank_name`, `company_id`, `partner_id` |  | `Archived` | `base` |
| `base_iban.view_partner_property_iban_form` | xpath | `account.view_partner_bank_form_inherit_account` |  |  |  | `base_iban` |
| `hr.view_partner_bank_form_inherit_hr` | xpath | `base.view_partner_bank_form` | `employee_has_multiple_bank_accounts`, `employee_salary_amount`, `currency_id`, `employee_id` |  |  | `hr` |
| `l10n_au.view_partner_bank_form` | field | `base.view_partner_bank_form` | `partner_id`, `aba_bsb` |  |  | `l10n_au` |
| `l10n_br.view_partner_bank_form_inherit_account` | field | `base.view_partner_bank_form` | `include_reference` |  |  | `l10n_br` |
| `l10n_ch.isr_partner_bank_form` | xpath | `base.view_partner_bank_form` | `l10n_ch_qr_iban` |  |  | `l10n_ch` |
| `l10n_hk.view_partner_bank_form_inherit_account` | field | `base.view_partner_bank_form` | `include_reference` |  |  | `l10n_hk` |
| `l10n_id.view_partner_bank_form_inherit_account` | xpath | `base.view_partner_bank_form` | `l10n_id_qris_api_key`, `l10n_id_qris_mid` |  |  | `l10n_id` |
| `l10n_kh.view_partner_bank_form_inherit_account` | field | `account_qr_code_emv.view_partner_bank_form_inherit_account` | `proxy_value`, `l10n_kh_merchant_id` |  |  | `l10n_kh` |
| `l10n_mx.view_res_partner_bank_inherit_l10n_mx_edi_bank` | xpath | `account.view_partner_bank_form_inherit_account` | `l10n_mx_edi_clabe` |  |  | `l10n_mx` |
| `l10n_mx.view_partner_bank_form_l10n_mx_edi_bank` | xpath | `base.view_partner_bank_form` | `fiscal_country_codes`, `l10n_mx_edi_clabe` |  |  | `l10n_mx` |
| `l10n_mx.view_partner_bank_tree_l10n_mx_edi_bank` | xpath | `base.view_partner_bank_tree` | `l10n_mx_edi_clabe` |  |  | `l10n_mx` |
| `l10n_sg.view_partner_bank_form_inherit_account` | field | `base.view_partner_bank_form` | `include_reference` |  |  | `l10n_sg` |
| `l10n_us.view_partner_bank_form_inherit_l10n_us` | xpath | `base.view_partner_bank_form` |  |  |  | `l10n_us` |
| `l10n_vn.view_partner_bank_form_inherit_account` | field | `base.view_partner_bank_form` | `include_reference` |  |  | `l10n_vn` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `account.action_account_supplier_accounts` | Bank Accounts | list,form |  |  |  | `account` |
| `base.action_res_partner_bank_account_form` | Bank Accounts | list,form |  |  |  | `base` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `contacts.menu_action_res_partner_bank_form` |  | `menu_config_bank_accounts` | `base.action_res_partner_bank_account_form` | 2 |  |

Machine-readable definition: `../../../schemas/data/entities/res.partner.bank.json`; views: `../../../schemas/interfaces/views/res.partner.bank.json`.
