# Cryptographic Keys (`certificate.key`)

**Transport name:** `certificate.key`  
**Storage name:** `certificate_key`  
**Kind:** persistent entity (one table)  
**Defined by package:** `certificate`  
**Extended by packages:** `account_edi_proxy_client`

Description: Cryptographic Keys

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (8)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | default `New key` |
| `content` | Key file | binary |  | required |
| `password` | Private key password | single line text |  |  |
| `pem_key` | Key bytes in PEM format | binary |  | computed by rule `_compute_pem_key` and stored |
| `public` | Public/Private key | boolean |  | computed by rule `_compute_pem_key` and stored |
| `loading_error` | Loading error | multi line text |  | computed by rule `_compute_pem_key` and stored |
| `Active` | Active | boolean |  | default `True`; Help: Set active to false to archive the key. |
| `company_id` | Company | many to one | `res.company` | required; default computed dynamically (lambda self: self.env.company); on delete of the target: cascade |

## Operations (13)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_pem_key` | computation | self | `certificate` | depends: `content`, `password` |  |
| `_sign` | internal rule | self, message, hashing_algorithm, formatting | `certificate` |  | Compute and return the message's signature.  :param str\|bytes message: The message to sign :param optional,default='sha256' hashing_algorithm: The digest algorithm to use. Currently, only 'sha1' and 'sha256' are available. :param optional,default='encodebytes' formatting: The formatting of the returned bytes     - 'encodebytes' returns a base64-encoded block of 76 characters lines     - 'base64' returns the raw base64-encoded data     - other returns non-encoded data :return: The formatted signature bytes of the message :rtype: bytes |
| `_verify` | internal rule | self, signed_message, signature, hashing_algorithm | `certificate` |  | Return the verification of the signature |
| `_get_public_key_numbers_bytes` | preparation rule | self, formatting | `certificate` |  | Get the public key's public numbers bytes.  :param optional,default='encodebytes' formatting: The formatting of the returned bytes     - 'encodebytes' returns a base64-encoded block of 76 characters lines     - 'base64' returns the raw base64-encoded data     - other returns non-encoded data :return: A tuple containing formatted public number bytes of the public key :rtype: tuple(bytes,bytes) |
| `_get_public_key_bytes` | preparation rule | self, encoding, formatting | `certificate` |  | Get the public key bytes.  :param optional,default='der' encoding: The formatting of the returned bytes     - 'der' returns DER public key bytes     - other returns PEM public key bytes :param optional,default='encodebytes' formatting: The formatting of the returned bytes     - 'encodebytes' returns a base64-encoded block of 76 characters lines     - 'base64' returns the raw base64-encoded data     - other returns non-encoded data :return: The formatted public key bytes in the corresponding format :rtype: bytes |
| `_decrypt` | internal rule | self, message, hashing_algorithm | `certificate` |  | Decrypt the given message using the provided digest.  :param str\|bytes message: The message to encode :param optional,default='sha256' hashing_algorithm: The digest algorithm to use. Currently, only 'sha1' and 'sha256' are available. :return: The decrypted text :rtype: str |
| `_sign_with_key` | internal rule | self, message, pem_key, pwd, hashing_algorithm, formatting | `certificate` | model | Compute and return the message's signature for a given private key.  :param str\|bytes message: The message to sign :param str\|bytes pem_key: A base64 encoded private key in the PEM format :param str\|bytes pwd: A password to decrypt the PEM key :param optional,default='sha1' hashing_algorithm: The digest algorithm to use. Currently, only 'sha1' and 'sha256' are available. :param optional,default='encodebytes' formatting: The formatting of the returned bytes     - 'encodebytes' returns a base64-encoded block of 76 characters lines     - 'base64' returns the raw base64-encoded data     - other |
| `_verify_with_key` | internal rule | self, signed_message, signature, pem_key, signature_algorithm | `certificate` | model | Return the verification of the signature |
| `_numbers_public_key_bytes_with_key` | internal rule | self, pem_key, formatting | `certificate` | model | Get the given public key's public numbers bytes.  :param str\|bytes pem_key: A base64 encoded public key in the PEM format :param optional,default='encodebytes' formatting: The formatting of the returned bytes     - 'encodebytes' returns a base64-encoded block of 76 characters lines     - 'base64' returns the raw base64-encoded data     - other returns non-encoded data :return: A tuple containing the formatted public number bytes of the public key :rtype: tuple(bytes,bytes) |
| `_generate_ec_private_key` | internal rule | self, company, name, curve, password | `certificate` | model | Generate an elliptic curve private key.  :param res.company company: A company record :param str,optional,default='id_ec' name: The name of the newly created key. :param optional,default='SECP256R1' curve: The type of elliptic curve algorithm. Currently, only SECP256R1 is supported. :param str \| bytes \| None password: Encrypts (best available algorithm) the key with the given password :return: A certificate.key record :rtype: certificate.key |
| `_generate_rsa_private_key` | internal rule | self, company, name, public_exponent, key_size, password | `certificate` | model | Generate an RSA private key.  :param res.company company: A company record :param str,optional,default='id_rsa' name: The name of the newly created key. :param int,optional,default=65537 public_exponent: The public exponent of the new key: either 65537 or 3 (for legacy purposes) :param int,optional,default=2048 key_size: The length of the modulus in bits; it is strongly recommended to be at least 2048 and must not be less than 512 :param str \| bytes \| None password: Encrypts (best available algorithm) the key with the given password :return: A certificate.key record :rtype: certificate.key |
| `_generate_ed25519_private_key` | internal rule | self, company, name, password | `certificate` |  | Generate an Ed25519 private key.  :param res.company company: A company record :param str,optional,default='id_ed25519' name: The name of the newly created key. :param str \| bytes \| None password: Encrypts (best available algorithm) the key with the given password :return: A certificate.key record :rtype: certificate.key |
| `_account_edi_fernet_decrypt` | internal rule | self, key, message | `account_edi_proxy_client` | model |  |

## Validation and error messages (19)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_sign` | UserError | Make sure to use a private key to sign documents. | `certificate` |
| `_sign` | UserError | The private key could not be loaded. | `certificate` |
| `_sign` | UserError | self.name + ' - ' + self.loading_error | `certificate` |
| `_verify` | UserError | Make sure to use a public key to verify the signature of documents. | `certificate` |
| `_verify` | UserError | self.name + ' - ' + self.loading_error | `certificate` |
| `_decrypt` | UserError | A private key is required to decrypt data. | `certificate` |
| `_decrypt` | UserError | f"Unsupported hashing algorithm '{hashing_algorithm}'. Currently supported: sha1 and sha256." | `certificate` |
| `_decrypt` | UserError | Unsupported asymmetric cryptography algorithm '%s'. Currently supported for decryption: RSA. | `certificate` |
| `_sign_with_key` | UserError | f"Unsupported hashing algorithm '{hashing_algorithm}'. Currently supported: sha1 and sha256." | `certificate` |
| `_sign_with_key` | UserError | The private key could not be loaded. | `certificate` |
| `_sign_with_key` | UserError | Unsupported asymmetric cryptography algorithm '%s'. Currently supported for signature: ED25519, EC and RSA. | `certificate` |
| `_verify_with_key` | UserError | f"Unsupported signature algorithm '{signature_algorithm}'. Currently supported: sha1 and sha256." | `certificate` |
| `_verify_with_key` | UserError | The public key could not be loaded. | `certificate` |
| `_verify_with_key` | UserError | Unsupported asymmetric cryptography algorithm '%s'. Currently supported for signature: EC and RSA. | `certificate` |
| `_numbers_public_key_bytes_with_key` | UserError | The public key could not be loaded. | `certificate` |
| `_numbers_public_key_bytes_with_key` | UserError | Unsupported asymmetric cryptography algorithm '%s'. Currently supported: EC, RSA. | `certificate` |
| `_generate_ec_private_key` | UserError | f"Unsupported curve algorithm '{curve}'. Currently supported: SECP256R1." | `certificate` |
| `_generate_rsa_private_key` | UserError | The public exponent should be 65537 (or 3 for legacy purposes). | `certificate` |
| `_generate_rsa_private_key` | UserError | The key size should be at least 512 bytes. | `certificate` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `certificate` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Key multi-company | global (all users) | `['\|', ('company_id', '=', False), ('company_id', 'parent_of', company_ids)]` | True | True | True | True |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `certificate.certificate_key_view_form` | form |  | `loading_error`, `name`, `company_id`, `content`, `password` |  |  | `certificate` |
| `certificate.certificate_key_view_list` | list |  | `name`, `company_id` |  |  | `certificate` |
| `certificate.certificate_key_view_search` | search |  |  |  | `Archived`, `Private`, `Public`, `Invalid` | `certificate` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `certificate.certificate_key_action_view_list` | Keys | list,form |  |  |  | `certificate` |

Machine-readable definition: `../../../schemas/data/entities/certificate.key.json`; views: `../../../schemas/interfaces/views/certificate.key.json`.
