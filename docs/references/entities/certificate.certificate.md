# Certificate (`certificate.certificate`)

**Transport name:** `certificate.certificate`  
**Storage name:** `certificate_certificate`  
**Kind:** persistent entity (one table)  
**Defined by package:** `certificate`  
**Extended by packages:** `l10n_es_edi_facturae`, `l10n_es_edi_sii`, `l10n_es_edi_tbai`, `l10n_es_edi_verifactu`, `l10n_sa_edi`

Description: Certificate

## Identity and behavior

- Default ordering: `date_end DESC`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (18)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  |  |
| `content` | Certificate | binary |  | required |
| `pkcs12_password` | Certificate Password | single line text |  | Help: Password to decrypt the PKS file. |
| `private_key_id` | Private Key | many to one | `certificate.key` | computed by rule `_compute_private_key` and stored; restricted by domain `[["public", "=", false]]`; must belong to the same company |
| `public_key_id` | Public Key | many to one | `certificate.key` | restricted by domain `[["public", "=", true]]`; must belong to the same company; Help: Used to set a public key in case the one self-contained in the certificate is erroneus.                 When a public key is set this way, it will be used instead of the one in the certificate. |
| `scope` | Certificate scope | selection |  | extended by packages `l10n_es_edi_facturae`, `l10n_es_edi_sii`, `l10n_es_edi_tbai`, `l10n_es_edi_verifactu` |
| `content_format` | Original certificate format | selection |  | computed by rule `_compute_pem_certificate` and stored |
| `pem_certificate` | Certificate in PEM format | binary |  | computed by rule `_compute_pem_certificate` and stored |
| `subject_common_name` | Subject Name | single line text |  | computed by rule `_compute_pem_certificate` and stored |
| `serial_number` | Serial number | single line text |  | computed by rule `_compute_pem_certificate` and stored; Help: The serial number to add to electronic documents |
| `date_start` | Available date | date and time |  | computed by rule `_compute_pem_certificate` and stored; Help: The date on which the certificate starts to be valid |
| `date_end` | Expiration date | date and time |  | computed by rule `_compute_pem_certificate` and stored; Help: The date on which the certificate expires |
| `loading_error` | Loading error | multi line text |  | computed by rule `_compute_pem_certificate` and stored |
| `is_valid` | Valid | boolean |  | computed by rule `_compute_is_valid` (not stored); searchable through a search rule |
| `Active` | Active | boolean |  | default `True`; Help: Set active to false to archive the certificate |
| `company_id` | Company | many to one | `res.company` | required; default computed dynamically (lambda self: self.env.company); on delete of the target: cascade |
| `country_code` | Country Code | single line text |  | related through path `company_id.country_code` |
| `issuer_cert_id` | Issuer Certificate | many to one | `certificate.certificate` | computed by rule `_compute_issuer_cert_id` (not stored); must belong to the same company |

## Selection values

### `scope` (Certificate scope)

| Value | Label |
|---|---|
| `general` | General |
| `facturae` | Facturae |
| `sii` | SII |
| `tbai` | TBAI |
| `verifactu` | Veri*Factu |

### `content_format` (Original certificate format)

| Value | Label |
|---|---|
| `der` | DER |
| `pem` | PEM |
| `pkcs12` | PKCS12 |

## Operations (31)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_issuer_cert_id` | computation | self | `certificate` | depends: `pem_certificate`, `subject_common_name`, `company_id` |  |
| `_compute_private_key` | computation | self | `certificate` | depends: `pem_certificate` |  |
| `_compute_pem_certificate` | computation | self | `certificate` | depends: `content`, `pkcs12_password` |  |
| `_compute_is_valid` | computation | self | `certificate` | depends: `date_start`, `date_end`, `loading_error` |  |
| `_search_is_valid` | search rule | self, operator, value | `certificate` |  |  |
| `_constrains_certificate_key_compatibility` | validation | self | `certificate` | constrains: `pem_certificate`, `private_key_id`, `public_key_id` |  |
| `_constrains_certificate_loaded` | validation | self | `certificate` | constrains: `content`, `pem_certificate` |  |
| `_get_subject_key_identifier` | preparation rule | self, x509_cert | `certificate` | model | Helper to safely extract the Subject Key Identifier (SKI) |
| `_get_authority_key_identifier` | preparation rule | self, x509_cert | `certificate` | model | Helper to safely extract the Authority Key Identifier (AKI) |
| `_get_common_name` | preparation rule | self, cert, issuer | `certificate` | model | Helper to safely extract the common name from a certificate's subject (or issuer). |
| `_is_issued_by` | internal rule | self, x509_certificate, x509_issuer_certificate | `certificate` | model | Cryptographically check that `certificate` was directly issued by `issuer_certificate`: the issuer distinguished name must match and the signature must verify against the issuer's public key.  :return: `True` if the issuance is cryptographically proven, `False` if it     is disproven, and `None` if it could not be checked (unsupported scheme or     parameters). :rtype: bool \| None |
| `_parse_pem_certificate_bundle` | internal rule | self, decoded_content, password | `certificate` | model | Parses a PEM-encoded bundle to extract individual certificate blocks and orders them based on the provided private key.  The function attempts to load a private key from the bundle. If successful, it compares public keys to identify the main target certificate. The first certificate in the returned list is guaranteed to be the leaf certificate, followed by the rest of the CA chain. If no private key is found, it returns the certificates in their original order. |
| `_parse_certificate_content` | internal rule | self, content, password | `certificate` | model |  |
| `_extract_and_filter_chain` | internal rule | self, content_bytes, password | `certificate` | model | Parses a bundle and returns only the certificates forming the leaf's chain |
| `create` | lifecycle override | self, vals_list | `certificate` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `certificate` |  |  |
| `_parse_chain_missing_ca_vals` | internal rule | self, vals | `certificate` | model | Parses the certificate content to extract its CA chain and identifies which Certificate Authorities are missing from the database.  :return: A list of field dictionaries ready to be passed to `create()` for the missing CAs. |
| `_get_der_certificate_bytes` | preparation rule | self, formatting | `certificate` |  | Get the DER bytes of the certificate.  :param optional,default='encodebytes' formatting: The formatting of the returned bytes     - 'encodebytes' returns a base64-encoded block of 76 characters lines     - 'base64' returns the raw base64-encoded data     - other returns non-encoded data :return: The formatted DER bytes of the certificate :rtype: bytes |
| `_get_fingerprint_bytes` | preparation rule | self, hashing_algorithm, formatting | `certificate` |  | Get the fingerprint bytes of the certificate.  :param optional,default='sha256' hashing_algorithm: The digest algorithm to use. Currently, only 'sha1' and 'sha256' are available. :param optional,default='encodebytes' formatting: The formatting of the returned bytes     - 'encodebytes' returns a base64-encoded block of 76 characters lines     - 'base64' returns the raw base64-encoded data     - other returns non-encoded data :return: The formatted fingerprint bytes of the certificate :rtype: bytes |
| `_get_signature_bytes` | preparation rule | self, formatting | `certificate` |  | Get the signature bytes of the certificate.  :param optional,default='encodebytes' formatting: The formatting of the returned bytes     - 'encodebytes' returns a base64-encoded block of 76 characters lines     - 'base64' returns the raw base64-encoded data     - other returns non-encoded data :return: The formatted signature bytes of the certificate :rtype: bytes |
| `_get_public_key_numbers_bytes` | preparation rule | self, formatting | `certificate` |  | Get the certificate public key's public numbers bytes.  :param optional,default='encodebytes' formatting: The formatting of the returned bytes     - 'encodebytes' returns a base64-encoded block of 76 characters lines     - 'base64' returns the raw base64-encoded data     - other returns non-encoded data :return: A tuple containing the formatted public number bytes of the certificate's public key  :rtype: tuple(bytes,bytes) |
| `_get_public_key_bytes` | preparation rule | self, encoding, formatting | `certificate` |  | Get the certificate's public key bytes.  :param optional,default='der' encoding: The formatting of the returned bytes     - 'der' returns DER public key bytes     - other returns PEM public key bytes :param optional,default='encodebytes' formatting: The formatting of the returned bytes     - 'encodebytes' returns a base64-encoded block of 76 characters lines     - 'base64' returns the raw base64-encoded data     - other returns non-encoded data :return: The formatted certificate public key bytes in the corresponding format :rtype: bytes |
| `_sign` | internal rule | self, message, hashing_algorithm, formatting | `certificate` |  | Compute and return the message's signature.  :param str\|bytes message: The message to sign :param optional,default='sha256' hashing_algorithm: The digest algorithm to use. Currently, only 'sha1' and 'sha256' are available. :param optional,default='encodebytes' formatting: The formatting of the returned bytes     - 'encodebytes' returns a base64-encoded block of 76 characters lines     - 'base64' returns the raw base64-encoded data     - other returns non-encoded data :return: The formatted signature bytes of the message :rtype: bytes |
| `_get_certificate_chain` | preparation rule | self | `certificate` |  | Retrieves the full certificate chain as a recordset, starting from the current certificate and walking up the issuer_cert_id links.  :return: A recordset of certificate.certificate objects ordered from Leaf to Root. |
| `_l10n_es_edi_facturae_get_issuer` | internal rule | self | `l10n_es_edi_facturae` |  |  |
| `_l10n_es_edi_tbai_get_issuer` | internal rule | self | `l10n_es_edi_tbai` |  |  |
| `_l10n_es_edi_verifactu_is_sello_certificate` | internal rule | self | `l10n_es_edi_verifactu` |  |  |
| `_l10n_sa_get_issuer_name` | internal rule | self | `l10n_sa_edi` |  |  |
| `_l10n_sa_get_csr_vals` | internal rule | self, journal | `l10n_sa_edi` | model |  |
| `_l10n_sa_validate_csr_vals` | internal rule | self, journal | `l10n_sa_edi` | model |  |
| `_l10n_sa_get_csr_str` | internal rule | self, journal | `l10n_sa_edi` | model | Return a string representation of a ZATCA compliant CSR that will be sent to the Compliance API in order to get back a signed X509 certificate |

## Validation and error messages (10)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_constrains_certificate_key_compatibility` | ValidationError | certificate.private_key_id.loading_error | `certificate` |
| `_constrains_certificate_key_compatibility` | ValidationError | The certificate and private key are not compatible. | `certificate` |
| `_constrains_certificate_key_compatibility` | ValidationError | certificate.public_key_id.loading_error | `certificate` |
| `_constrains_certificate_key_compatibility` | ValidationError | The certificate and public key are not compatible. | `certificate` |
| `_constrains_certificate_loaded` | ValidationError | certificate.loading_error or _('This certificate could not be loaded. Please provide the certificate password.') | `certificate` |
| `_get_fingerprint_bytes` | UserError | f"Unsupported hashing algorithm '{hashing_algorithm}'. Currently supported: sha1 and sha256." | `certificate` |
| `_get_public_key_bytes` | UserError | The public key from the certificate could not be loaded. | `certificate` |
| `_sign` | UserError | self.loading_error or _('This certificate is not valid, its validity has expired.') | `certificate` |
| `_sign` | UserError | No private key linked to the certificate, it is required to sign documents. | `certificate` |
| `_l10n_sa_validate_csr_vals` | UserError | Please make sure the following fields are shorter than %(max_length)d bytes (note that Arabic or special characters take more space): %(error_fields_msg)s | `l10n_sa_edi` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `certificate` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Certificate multi-company | global (all users) | `['\|', ('company_id', '=', False), ('company_id', 'parent_of', company_ids)]` | True | True | True | True |

## Views (11)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `certificate.certificate_certificate_view_form` | form |  | `loading_error`, `name`, `company_id`, `content`, `pkcs12_password`, `subject_common_name`, `private_key_id`, `public_key_id`, `scope`, `date_start`, `date_end`, `serial_number` |  |  | `certificate` |
| `certificate.certificate_certificate_view_list` | list |  | `name`, `subject_common_name`, `is_valid`, `company_id` |  |  | `certificate` |
| `certificate.certificate_certificate_view_search` | search |  | `name`, `scope` |  | `General`, `Valid`, `Invalid`, `Archived` | `certificate` |
| `l10n_es_edi_facturae.certificate_certificate_view_search` | filter | `certificate.certificate_certificate_view_search` |  |  | `scope_general`, `Facturae` | `l10n_es_edi_facturae` |
| `l10n_es_edi_facturae.certificate_certificate_view_form` | field | `certificate.certificate_certificate_view_form` | `scope` |  |  | `l10n_es_edi_facturae` |
| `l10n_es_edi_sii.certificate_certificate_view_search` | filter | `certificate.certificate_certificate_view_search` |  |  | `scope_general`, `SII` | `l10n_es_edi_sii` |
| `l10n_es_edi_sii.certificate_certificate_view_form` | field | `certificate.certificate_certificate_view_form` | `scope` |  |  | `l10n_es_edi_sii` |
| `l10n_es_edi_tbai.certificate_certificate_view_search` | filter | `certificate.certificate_certificate_view_search` |  |  | `scope_general`, `TBAI` | `l10n_es_edi_tbai` |
| `l10n_es_edi_tbai.certificate_certificate_view_form` | field | `certificate.certificate_certificate_view_form` | `scope` |  |  | `l10n_es_edi_tbai` |
| `l10n_es_edi_verifactu.certificate_certificate_view_search` | filter | `certificate.certificate_certificate_view_search` |  |  | `scope_general`, `Veri*Factu` | `l10n_es_edi_verifactu` |
| `l10n_es_edi_verifactu.certificate_certificate_view_form` | field | `certificate.certificate_certificate_view_form` | `scope` |  |  | `l10n_es_edi_verifactu` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `certificate.certificate_certificate_action_view_list` | Certificates | list,form |  |  |  | `certificate` |
| `l10n_es_edi_facturae.l10n_es_edi_facturae_certificate_action` | Certificates for Facturae EDI invoices on Spain | list,form |  | `{'scope': 'facturae', 'search_default_scope_facturae': 1}` |  | `l10n_es_edi_facturae` |
| `l10n_es_edi_sii.l10n_es_edi_sii_certificate_action` | Certificates for SII EDI invoices on Spain | list,form |  | `{'scope': 'sii', 'search_default_scope_sii': 1}` |  | `l10n_es_edi_sii` |
| `l10n_es_edi_tbai.l10n_es_edi_tbai_certificate_action` | Certificates for EDI TicketBAI invoices on Spain | list,form |  | `{'scope': 'tbai', 'search_default_scope_tbai': 1}` |  | `l10n_es_edi_tbai` |
| `l10n_es_edi_verifactu.l10n_es_edi_verifactu_certificate_action` | Certificates for Veri*Factu | list,form |  | `{'scope': 'verifactu', 'search_default_scope_verifactu': 1}` |  | `l10n_es_edi_verifactu` |

Machine-readable definition: `../../../schemas/data/entities/certificate.certificate.json`; views: `../../../schemas/interfaces/views/certificate.certificate.json`.
