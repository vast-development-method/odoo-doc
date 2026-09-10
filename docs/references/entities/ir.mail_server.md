# Mail Server (`ir.mail_server`)

**Transport name:** `ir.mail_server`  
**Storage name:** `ir_mail_server`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`  
**Extended by packages:** `mail`, `google_gmail`, `mass_mailing`, `microsoft_outlook`

Description: Mail Server

## Identity and behavior

- Mixins (classical inheritance): `google.gmail.mixin`, `microsoft.outlook.mixin`
- Default ordering: `sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (20)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; indexed |
| `from_filter` | FROM Filtering | single line text |  | Help: Comma-separated list of addresses or domains for which this server can be used. e.g.: "notification@odoo.com" or "odoo.com" |
| `smtp_host` | mail transfer protocol Server | single line text |  | Help: Hostname or IP of SMTP server |
| `smtp_port` | mail transfer protocol Port | integer |  | default `25`; Help: SMTP Port. Usually 465 for SSL, and 25 or 587 for other cases. |
| `smtp_authentication` | Authenticate with | selection |  | required; default `login`; on delete of the target: {"outlook": "set default"}; extended by packages `google_gmail`, `microsoft_outlook` |
| `smtp_authentication_info` | Authentication Info | multi line text |  | computed by rule `_compute_smtp_authentication_info` (not stored) |
| `smtp_user` | Username | single line text |  | visible only to groups `base.group_system`; Help: Optional username for SMTP authentication |
| `smtp_pass` | Password | single line text |  | visible only to groups `base.group_system`; Help: Optional password for SMTP authentication |
| `smtp_encryption` | Connection Encryption | selection |  | required; default `none`; Help: Choose the connection encryption scheme: - None: SMTP sessions are done in cleartext. - TLS (STARTTLS): TLS encryption is requested at start of SMTP session (Recommended) - SSL/TLS: SMTP sessions are encrypted with SSL/TLS through a dedicated port (default: 465)  Choose an additionnal variant for SSL or TLS: - encryption and validation: encrypt the data and authentify the server using its SSL certificate (Recommended) - encryption only: encrypt the data but skip server authentication |
| `smtp_ssl_certificate` | SSL Certificate | binary |  | visible only to groups `base.group_system`; Help: SSL certificate used for authentication |
| `smtp_ssl_private_key` | SSL Private Key | binary |  | visible only to groups `base.group_system`; Help: SSL private key used for authentication |
| `smtp_debug` | Debugging | boolean |  | Help: If enabled, the full output of SMTP sessions will be written to the server log at DEBUG level (this is very verbose and may include confidential info!) |
| `max_email_size` | Max Email Size | float |  |  |
| `sequence` | Priority | integer |  | default `10`; Help: When no specific mail server is requested for a mail, the highest priority one is used. Default priority is 10 (smaller number = higher priority) |
| `active` | Active | boolean |  | default `True` |
| `mail_template_ids` | Mail template using this mail server | one to many | `mail.template` | read only; inverse field `mail_server_id` |
| `owner_user_id` | Owner | many to one | `res.users` | not copied on duplication |
| `owner_limit_time` | Owner Limit Time | date and time |  | not copied on duplication |
| `owner_limit_count` | Owner Limit Count | integer |  | not copied on duplication |
| `active_mailing_ids` | Active mailing using this mail server | one to many | `mailing.mailing` | read only; restricted by domain `[["state", "!=", "done"], ["active", "=", true]]`; inverse field `mail_server_id` |

## Selection values

### `smtp_authentication` (Authenticate with)

| Value | Label |
|---|---|
| `login` | Username |
| `certificate` | SSL Certificate |
| `cli` | Command Line Interface |
| `gmail` | Gmail OAuth Authentication |
| `outlook` | Outlook OAuth Authentication |

### `smtp_encryption` (Connection Encryption)

| Value | Label |
|---|---|
| `none` | None |
| `starttls_strict` | TLS (STARTTLS), encryption and validation |
| `starttls` | TLS (STARTTLS), encryption only |
| `ssl_strict` | SSL/TLS, encryption and validation |
| `ssl` | SSL/TLS, encryption only |

## Database constraints and indexes (2)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_certificate_requires_tls` | Constraint | `CHECK(smtp_encryption != 'none' OR smtp_authentication != 'certificate')` | Certificate-based authentication requires a TLS transport | `base` |
| `_unique_owner_user_id` | Constraint | `UNIQUE(owner_user_id)` | owner_user_id must be unique | `mail` |

## Operations (35)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_smtp_authentication_info` | computation | self | `base`, `google_gmail`, `microsoft_outlook` | depends: `smtp_authentication` |  |
| `_check_smtp_ssl_files` | validation | self | `base` | constrains: `smtp_authentication`, `smtp_ssl_certificate`, `smtp_ssl_private_key` |  |
| `write` | lifecycle override | self, vals | `base` |  | Ensure we cannot archive a server in-use |
| `_active_usages_compute` | internal rule | self | `base`, `mail`, `mass_mailing` |  | Compute a dict server id to list of user-friendly outgoing mail servers usage of this record set.  This method must be overridden by all modules that uses this class in order to complete the list with user-friendly string describing the active elements that could send mail through the instance of this class. :return dict: { ir_mail_server.id: usage_str_list }. |
| `_get_max_email_size` | preparation rule | self | `base` |  |  |
| `_get_test_email_from` | preparation rule | self | `base`, `mail` |  |  |
| `_get_test_email_to` | preparation rule | self | `base` |  |  |
| `test_smtp_connection` | operation | self, autodetect_max_email_size | `base` |  | Test the connection and if autodetect_max_email_size, set auto-detected max email size.  :param bool autodetect_max_email_size: whether to autodetect the max email size :return: client action to notify the user of the result of the operation (connection test or     auto-detection successful depending on the ``autodetect_max_email_size`` parameter) :rtype: dict  :raises UserError: if the connection fails and if ``autodetect_max_email_size`` and     the server doesn't support the auto-detection of email max size |
| `action_retrieve_max_email_size` | user action | self | `base` |  |  |
| `_disable_send` | internal rule | cls | `base` |  | Whether to disable sending e-mails |
| `_connect__` | internal rule | self, host, port, user, password, encryption, smtp_from, ssl_certificate, ssl_private_key, smtp_debug, mail_server_id, allow_archived | `base` |  | Returns a new SMTP connection to the given SMTP server. When running in test mode, this method does nothing and returns `None`.  :param host: host or IP of SMTP server to connect to, if mail_server_id not passed :param int port: SMTP port to connect to :param user: optional username to authenticate with :param password: optional password to authenticate with :param str encryption: optional, ``'none'`` \| ``'ssl'`` \| ``'ssl_strict'`` \| ``'starttls'`` \| ``'starttls_strict'``.     The 'strict' variants verify the remote server's certificate against the operating system trust store. :param smtp |
| `_check_forced_mail_server` | validation | self, mail_server, allow_archived, smtp_from | `base`, `mail` |  |  |
| `_smtp_login__` | internal rule | self, connection, smtp_user, smtp_password | `base`, `google_gmail`, `microsoft_outlook` |  | Authenticate the SMTP connection.  Can be overridden in other module for different authentication methods.Can be called on the model itself or on a singleton.  :param connection: The SMTP connection to authenticate :param smtp_user: The user to used for the authentication :param smtp_password: The password to used for the authentication |
| `_build_email__` | internal rule | self, email_from, email_to, subject, body, email_cc, email_bcc, reply_to, attachments, message_id, references, object_id, subtype, headers, body_alternative, subtype_alternative | `base` |  | Constructs an RFC2822 email.message.Message object based on the keyword arguments passed, and returns it.  :param string email_from: sender email address :param list email_to: list of recipient addresses (to be joined with commas) :param string subject: email subject (no pre-encoding/quoting necessary) :param string body: email body, of the type ``subtype`` (by default, plaintext).                     If html subtype is used, the message will be automatically converted                     to plaintext and wrapped in multipart/alternative, unless an explicit                     ``body_alternati |
| `_get_default_bounce_address` | preparation rule | self | `base`, `mail` | model | Computes the default bounce address. It is used to set the envelop address if no envelop address is provided in the message.  :return: defaults to the ``--email-from`` CLI/config parameter. :rtype: str \| None |
| `_get_default_from_address` | preparation rule | self | `base`, `mail` | model | Computes the default from address. It is used for the "header from" address when no other has been received.  :return: defaults to the ``--email-from`` CLI/config parameter. :rtype: str \| None |
| `_get_default_from_filter` | preparation rule | self | `base` | model | Computes the default from_filter. It is used when no specific ir.mail_server is used when sending emails, hence having no value for from_filter.  :return: defaults to 'mail.default.from_filter', then   ``--from-filter`` CLI/config parameter. :rtype: str \| None |
| `_prepare_email_message__` | preparation rule | self, message, smtp_session | `base` |  | Prepare the SMTP information (from, to, message) before sending.  :param message: the email.message.Message to send, information like the     Return-Path, the From, etc... will be used to find the smtp_from and to smtp_to :param smtp_session: the opened SMTP session to use to authenticate the sender  :return: smtp_from, smtp_to_list, message     smtp_from: email to used during the authentication to the mail server     smtp_to_list: list of email address which will receive the email     message: the email.message.Message to send |
| `_alter_message__` | internal rule | self, message, smtp_from, smtp_to_list | `base` | model |  |
| `_prepare_smtp_to_list` | preparation rule | self, message, smtp_session | `base` | model | Prepare SMTP To address list, based on To / Cc / Bcc.  Optional 'send_validated_to' context key filter restricts addresses to be part of that list.  Optional 'send_smtp_skip_to' context key holds a recipients block list |
| `send_email` | operation | self, message, mail_server_id, smtp_server, smtp_port, smtp_user, smtp_password, smtp_encryption, smtp_ssl_certificate, smtp_ssl_private_key, smtp_debug, smtp_session | `base` | model | Sends an email directly (no queuing).  No retries are done, the caller should handle MailDeliveryException in order to ensure that the mail is never lost.  If the mail_server_id is provided, sends using this mail server, ignoring other smtp_* arguments. If mail_server_id is None and smtp_server is None, use the default mail server (highest priority). If mail_server_id is None and smtp_server is not None, use the provided smtp_* arguments. If both mail_server_id and smtp_server are None, look for an 'smtp_server' value in server config, and fails if not found.  :param message: the email.message |
| `_find_mail_server_allowed_domain` | internal rule | self | `base`, `mail` |  | Overridable domain getter for all mail servers that may be used as default. |
| `_find_mail_server` | internal rule | self, email_from, mail_servers | `base` |  | Find the appropriate mail server for the given email address.  :rtype: tuple[IrMail_Server \| None, str] :returns: A two-elements tuple: ``(Record<ir.mail_server>, email_from)``    1. Mail server to use to send the email (``None`` if we use the odoo-bin arguments)   2. Email FROM to use to send the email (in some case, it might be impossible      to use the given email address directly if no mail server is configured for) |
| `_filter_mail_servers_fallback` | internal rule | self, servers | `base`, `mail` | model | Filter the mail servers that can be used as fallback, or for default email from. |
| `_match_from_filter` | internal rule | self, email_from, from_filter | `base` | model | Return True is the given email address match the "from_filter" field.  The from filter can be Falsy (always match), a domain name or an full email address. |
| `_parse_from_filter` | internal rule | self, from_filter | `base` | model |  |
| `_onchange_encryption` | on change | self | `base`, `google_gmail`, `microsoft_outlook` | onchange: `smtp_encryption` | Do not change the SMTP configuration if it's a Gmail server (e.g. the port which is already set) |
| `_get_personal_mail_servers_limit` | preparation rule | self | `mail`, `microsoft_outlook` |  | Return the number of email we can send in 1 minutes for this outgoing server.  0 fallbacks to 30 to avoid blocking servers. |
| `_onchange_smtp_authentication_gmail` | on change | self | `google_gmail` | onchange: `smtp_authentication` |  |
| `_on_change_smtp_user_gmail` | on change | self | `google_gmail` | onchange: `smtp_user`, `smtp_authentication` | The Gmail mail servers can only be used for the user personal email address. |
| `_check_use_google_gmail_service` | validation | self | `google_gmail` | constrains: `smtp_authentication`, `smtp_pass`, `smtp_encryption`, `from_filter`, `smtp_user` |  |
| `_check_owner_user_id_not_mass_mailing` | validation | self | `mass_mailing` | constrains: `owner_user_id` |  |
| `_check_use_microsoft_outlook_service` | validation | self | `microsoft_outlook` | constrains: `smtp_authentication`, `smtp_pass`, `smtp_encryption`, `smtp_user` |  |
| `_onchange_smtp_authentication_outlook` | on change | self | `microsoft_outlook` | onchange: `smtp_authentication` |  |
| `_on_change_smtp_user_outlook` | on change | self | `microsoft_outlook` | onchange: `smtp_user`, `smtp_authentication` | The Outlook mail servers can only be used for the user personal email address. |

## Validation and error messages (35)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_smtp_ssl_files` | UserError | SSL private key is missing for %s. | `base` |
| `_check_smtp_ssl_files` | UserError | SSL certificate is missing for %s. | `base` |
| `write` | UserError | You cannot archive this Outgoing Mail Server (%(server_usage)s) because it is still used in the following case(s): %(usage_details)s | `base` |
| `write` | UserError | You cannot archive these Outgoing Mail Servers (%(server_usage)s) because they are still used in the following case(s): %(usage_details)s | `base` |
| `_get_test_email_from` | UserError | Please configure an email on the current user to simulate sending an email message via this outgoing server | `base` |
| `test_smtp_connection` | UserError | The server refused the sender address (%(email_from)s) with error %(repl)s | `base` |
| `test_smtp_connection` | UserError | The server refused the test recipient (%(email_to)s) with error %(repl)s | `base` |
| `test_smtp_connection` | UserError | The server refused the test connection with error %(repl)s | `base` |
| `test_smtp_connection` | UserError | Invalid server name!  %s | `base` |
| `test_smtp_connection` | UserError | No response received. Check server address and port number.  %s | `base` |
| `test_smtp_connection` | UserError | The server has closed the connection unexpectedly. Check configuration served on this port number.  %s | `base` |
| `test_smtp_connection` | UserError | Server replied with following exception:  %s | `base` |
| `test_smtp_connection` | UserError | An option is not supported by the server:  %s | `base` |
| `test_smtp_connection` | UserError | An SMTP exception occurred. Check port number and connection security type.  %s | `base` |
| `test_smtp_connection` | UserError | An SSL exception occurred. Check connection security type.  CertificateError: %s | `base` |
| `test_smtp_connection` | UserError | An SSL exception occurred. Check connection security type.  %s | `base` |
| `test_smtp_connection` | UserError | Connection Test Failed! Here is what we got instead:  %s | `base` |
| `test_smtp_connection` | UserError | The server "%(server_name)s" doesn't return the maximum email size. | `base` |
| `_connect__` | UserError | Missing SMTP Server Please define at least one SMTP server, or provide the SMTP parameters explicitly. | `base` |
| `_connect__` | UserError | The private key or the certificate is not a valid file.  %s | `base` |
| `_connect__` | UserError | Could not load your certificate / private key.  %s | `base` |
| `_connect__` | UserError | The private key or the certificate is not a valid file.  %s | `base` |
| `_connect__` | UserError | Could not load your certificate / private key.  %s | `base` |
| `_check_forced_mail_server` | UserError | The server "%s" cannot be used because it is archived. | `base` |
| `_check_forced_mail_server` | UserError | The server "%s" cannot be forced as it belongs to a user. | `mail` |
| `_check_forced_mail_server` | UserError | The server "%s" cannot be forced as it belongs to a user and is archived. | `mail` |
| `_check_forced_mail_server` | UserError | The server "%s" cannot be forced as the owner does not use it anymore. | `mail` |
| `_check_use_google_gmail_service` | UserError | Please leave the password field empty for Gmail mail server “%s”. The OAuth process does not require it | `google_gmail` |
| `_check_use_google_gmail_service` | UserError | Incorrect Connection Security for Gmail mail server “%s”. Please set it to "TLS (STARTTLS)". | `google_gmail` |
| `_check_use_google_gmail_service` | UserError | Please fill the "Username" field with your Gmail username (your email address). This should be the same account as the one used for the Gmail OAuthentication Token. | `google_gmail` |
| `_check_owner_user_id_not_mass_mailing` | ValidationError | Cannot set an owner on '%(server)s': it is configured as the dedicated Email Marketing server. | `mass_mailing` |
| `_check_owner_user_id_not_mass_mailing` | ValidationError | Cannot set an owner on '%(server)s': it is used by mailing '%(mailing)s'. | `mass_mailing` |
| `_check_use_microsoft_outlook_service` | UserError | Please leave the password field empty for Outlook mail server “%s”. The OAuth process does not require it | `microsoft_outlook` |
| `_check_use_microsoft_outlook_service` | UserError | Incorrect Connection Security for Outlook mail server “%s”. Please set it to "TLS (STARTTLS)". | `microsoft_outlook` |
| `_check_use_microsoft_outlook_service` | UserError | Please fill the "Username" field with your Outlook/Office365 username (your email address). This should be the same account as the one used for the Outlook OAuthentication Token. | `microsoft_outlook` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_system` | yes | yes | yes | yes | `base` |
| `mass_mailing.group_mass_mailing_user` | no | yes | no | no | `mass_mailing` |

## Views (6)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.ir_mail_server_form` | form |  | `name`, `from_filter`, `sequence`, `max_email_size`, `smtp_authentication`, `smtp_authentication_info`, `smtp_encryption`, `smtp_debug`, `smtp_host`, `smtp_port`, `smtp_user`, `smtp_pass`, `smtp_ssl_certificate`, `smtp_ssl_private_key` | `Test Connection`, `action_retrieve_max_email_size` |  | `base` |
| `base.ir_mail_server_list` | list |  | `sequence`, `name`, `smtp_host`, `smtp_user`, `smtp_encryption`, `from_filter` |  |  | `base` |
| `base.view_ir_mail_server_search` | search |  | `name`, `smtp_encryption`, `from_filter` |  | `Archived` | `base` |
| `google_gmail.ir_mail_server_view_form` | field | `base.ir_mail_server_form` | `smtp_ssl_private_key` | `open_google_gmail_uri`, `open_google_gmail_uri` |  | `google_gmail` |
| `mail.ir_mail_server_view_form` | xpath | `base.ir_mail_server_form` |  |  |  | `mail` |
| `microsoft_outlook.ir_mail_server_view_form` | field | `base.ir_mail_server_form` | `smtp_ssl_private_key` | `open_microsoft_outlook_uri`, `open_microsoft_outlook_uri` |  | `microsoft_outlook` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.action_ir_mail_server_list` | Outgoing Mail Servers | list,form |  |  |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/ir.mail_server.json`; views: `../../../schemas/interfaces/views/ir.mail_server.json`.
