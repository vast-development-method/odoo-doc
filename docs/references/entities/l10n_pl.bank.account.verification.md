# PL Bank Account Verification (`l10n_pl.bank.account.verification`)

**Transport name:** `l10n_pl.bank.account.verification`  
**Storage name:** `l10n_pl_bank_account_verification`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_pl_bank_verification`

Description: PL Bank Account Verification

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (8)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `verification_status` | Verification Status | selection |  | required; read only; Help: Flag the payment verification status with one of the following: - Valid: The partner VAT is linked to the bank account used for this payment. - Invalid: The partner VAT is not linked to the bank account used for this payment. - Incomplete partner: The partner has no VAT or no bank account. - Partner not found: Partner VAT not found in Government files. - Error: An error occurred during check with Government API. |
| `verification_timestamp` | Verification Timestamp | date and time |  | read only |
| `verification_date` | Verification Date | date |  | computed by rule `_compute_verification_date` and stored |
| `verification_request_id` | Correlation identifier | single line text |  | read only |
| `partner_bank_id` | Bank Account | many to one | `res.partner.bank` | read only |
| `partner_bank_account_number` | Partner Bank Account Number | single line text |  | read only; computed by rule `_compute_partner_bank_account_number` and stored |
| `partner_id` | Partner | many to one | `res.partner` | read only |
| `partner_vat` | Partner Value-added tax | single line text |  | read only; computed by rule `_compute_partner_vat` and stored |

## Selection values

### `verification_status` (Verification Status)

| Value | Label |
|---|---|
| `valid` | Valid |
| `invalid` | Invalid |
| `incomplete_partner` | Incomplete partner |
| `not_found_partner` | Partner not found |
| `error` | An error occurred during check with Government API |

## State fields

State machine fields of this entity: `verification_status`. Transitions are specified in the domain documents.

## Operations (10)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_auto_init` | lifecycle override | self | `l10n_pl_bank_verification` |  |  |
| `_gc_bank_account_verification` | background operation | self | `l10n_pl_bank_verification` | autovacuum |  |
| `_compute_verification_date` | computation | self | `l10n_pl_bank_verification` | depends: `verification_timestamp` |  |
| `_compute_partner_bank_account_number` | computation | self | `l10n_pl_bank_verification` | depends: `partner_bank_id` |  |
| `_compute_partner_vat` | computation | self | `l10n_pl_bank_verification` | depends: `partner_id` |  |
| `_l10n_pl_get_verification` | internal rule | self, partner_bank_data, date | `l10n_pl_bank_verification` |  | :param partner_bank_data: list(tuple(partner_id, partner_banks)): recordset of partner bank to get verification for by partner id :returns: A recordset of l10n_pl.bank.account.verification for all res.partner.bank in param |
| `_make_request` | internal rule | self, endpoint, params | `l10n_pl_bank_verification` | model | Send request to the government API :param endpoint: The endpoint to call in the API :param params: Params to include in request :return: response |
| `_handle_response` | internal rule | self, response | `l10n_pl_bank_verification` | model | Handle response given by the API :param response: The response received by the API :return: Response content or raise an error |
| `_get_creation_vals` | preparation rule | self, status, partner_banks, partners, timestamp, request_id | `l10n_pl_bank_verification` |  | partners should be filled only for partners without bank accounts ('incomplete_partner') |
| `_get_partner_from_identifier` | preparation rule | self, identifier | `l10n_pl_bank_verification` | model |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_user` | no | yes | no | no | `l10n_pl_bank_verification` |

Machine-readable definition: `../../../schemas/data/entities/l10n_pl.bank.account.verification.json`.
