# Configuration parameters

A **System Parameter** is a per-database key-value pair that changes behaviour at run time without changing data, without changing a user's settings and without restarting anything. This document specifies the entity, its read and write semantics, its caching, the parameters seeded when a database is created, the ones the platform reserves, and the ones every capability package adds, each with its type, its default and its effect.

Every key in this document is a **reproduced identifier** and is written exactly, in code font, with its meaning in words beside it. A replacement must honour every parameter listed here with the same key, the same default and the same meaning, because behaviour documented elsewhere in this repository refers to them by key.

## 1. The System Parameter entity

The transport name is `ir.config_parameter` and the full name is System Parameter. Records are ordered by key ascending and the display name is the key.

| Identifier | Full name | Type | Required | Meaning |
|---|---|---|---|---|
| `key` | Key | Text | yes | The parameter name. Unique across the database. |
| `value` | Value | Long text | yes | The parameter value, always stored as text. Callers convert it. |

| Rule | Statement |
|---|---|
| Uniqueness | The key is unique; the violation message is `Key must be unique.` |
| Default ordering | By key ascending. |
| Display name | The key. |
| Privileged commands | Forbidden on this entity: an elevated caller may not execute list-editing commands on behalf of another user here. |
| Company scoping | None. Parameters are database-wide. |
| Archiving | Not supported. |

### 1.1 Reading

1. Check that the acting user may read System Parameters.
2. Look the key up through the memoized lookup.
3. Return the stored value when it is non-empty, and the caller's fallback otherwise. The fallback defaults to false.

The memoized lookup is held in the registry cache container named `stable`, catalogued in [`caching.md`](caching.md), section 10. Before reading it flushes the pending changes of the key and value fields, so that a parameter written earlier in the same transaction is visible. It reads with a **direct statement**, deliberately bypassing the record layer, because parameter reads occur inside field-dependency computations that run while the entity definitions are being rebuilt and the record layer is not yet usable.

Two consequences must be reproduced.

1. An **empty** stored value is indistinguishable from an absent parameter: both yield the fallback. Storing an empty value is therefore not a way to force a falsy setting; deleting the parameter is.
2. The value is always text. A caller that wants a number converts it and is responsible for the failure path. Every platform caller that converts logs a warning and falls back to the documented default rather than failing the request.

### 1.2 Writing

1. Find the parameter with that key.
2. If it exists: remember its value as the previous one. If the new value is neither false nor unset, write it when its text form differs from the previous one; otherwise delete the parameter. Return the previous value.
3. If it does not exist: create it when the new value is neither false nor unset. Return false.

Writing a false or unset value therefore **deletes** the parameter rather than storing a falsy text. This is the documented way to reset a parameter to its default.

Writing the same value issues no statement at all, which is what keeps a settings form that saves every field from invalidating every cache on every save.

### 1.3 Cache invalidation

Creating, writing or deleting any System Parameter clears the `stable` cache key, which empties the containers `stable`, `default` and `templates.cached_values`, and announces the invalidation to the other worker processes after the commit. The signalling protocol is in [`caching.md`](caching.md), section 10.

Every memoized value derived from configuration is therefore dropped by any parameter change. This is a deliberate over-invalidation in exchange for a single, simple rule: no caller has to declare which memoized values its parameter affects.

### 1.4 Protected parameters

The six parameters seeded at database creation, listed in section 2, are protected twice.

| Attempt | Outcome |
|---|---|
| Renaming one of them, that is writing a new key on the record | Refused with `You cannot rename config parameters with keys %s`, whose placeholder is the comma-separated list of the offending keys. |
| Deleting one of them | Refused with `You cannot delete the %s record.`, whose placeholder is the key. |

Their **values** may be changed. The deletion guard is declared as a deletion hook that does not run at package removal, so that removing a package never trips it.

### 1.5 Value sanitizing and side effects

Some parameters are not inert: writing them changes something else. The platform defines two such behaviours, both in the messaging area.

| Key | Behaviour on write |
|---|---|
| `mail.restrict_template_rendering` | Setting it to a false value grants the template-editing group to every internal user; setting it to a true value removes that group from internal users. Administrators keep it through the administration group. |
| `mail.catchall.domain.allowed` | The value is normalized before being stored: the list of permitted mail domains is sanitized, both when written through the write operation and when created or written directly. |

## 2. Parameters seeded at database creation

| Key | Full name | Type | Seeded value |
|---|---|---|---|
| `database.secret` | Database secret | Text | A freshly generated random universally unique value. |
| `database.uuid` | Database universally unique identifier | Text | A freshly generated time-based unique value. |
| `database.create_date` | Database creation instant | Date and time | The creation instant. |
| `web.base.url` | Base web address | Text | The loopback address followed by the configured listening port. |
| `base.login_cooldown_after` | Login cooldown threshold | Integer | 10 |
| `base.login_cooldown_duration` | Login cooldown duration | Integer, seconds | 60 |

The seeding operation may be asked to overwrite existing values; by default it only fills in the ones that are absent. It runs with prefetching disabled, because during a package installation the user table may not yet have every column the prefetch would read.

## 3. Platform parameters

### 3.1 Identity, secrets and lifecycle

| Key | Full name | Type | Default | Effect |
|---|---|---|---|---|
| `database.secret` | Database secret | Text | seeded | The keying material for every derived token: cross-site request forgery tokens, session binding tokens, scoped field access tokens, sign-up and password-reset tokens, unsubscribe tokens. Changing it invalidates every one of them at once. |
| `database.uuid` | Database universally unique identifier | Text | seeded | The stable identity of this database, reported to external services. |
| `database.create_date` | Database creation instant | Date and time | seeded | When the database was created. |
| `database.is_neutralized` | Database is neutralized | Boolean | unset | Marks a copy that must not act on the outside world. A settings switch may not re-enable a scheduled job while it is set, and integrations refuse to contact their services. |
| `database.enterprise_code` | Subscription code | Text | unset | The subscription code sent by the subscription status check job. |

The subscription status check job additionally writes five values, each under a fixed key of its own: the expiry instant under `database.expiration_date`, the expiry reason under `database.expiration_reason` (the word `trial` when the subscription service names no reason) and, when the code is already bound to another database, the address at which that other database is managed under `database.already_linked_subscription_url`, the electronic-mail address of its owner under `database.already_linked_email` and the address to which a contact request is sent under `database.already_linked_send_mail_url`. The key of each of those five is fixed by the platform; only the value comes from the subscription service. All five are read only by the screens that report the subscription state.

### 3.2 Request layer

| Key | Full name | Type | Default | Effect |
|---|---|---|---|---|
| `web.base.url` | Base web address | Text | seeded | The absolute base address used to build links in messages, documents and external callbacks. It is overwritten at the first login of a settings administrator unless frozen. |
| `web.base.url.freeze` | Freeze the base web address | Boolean | unset | When set, the base address is never overwritten automatically. |
| `report.url` | Document rendering base address | Text | unset | The base address the document conversion engine uses to fetch the deployment's own resources. It falls back to the base address. See [`report-rendering.md`](report-rendering.md), section 4.10. |
| `web.max_file_upload_size` | Maximum request body size | Integer, bytes | unset, therefore 134 217 728 | The database-wide request body limit. An endpoint-level declaration overrides it. A value that is not an integer is logged and ignored. |
| `web.active_ids_limit` | Active record key limit | Integer | 20 000 | The maximum number of record keys an action carries in its context. |
| `web.quick_login` | Quick login | Boolean | true | Whether the login page offers the list of known logins. |
| `web.web_app_name` | Installable application name | Text | the platform's own name | The application name published in the installable web application description. |
| `web.json.enabled` | Structured-data endpoints enabled | Boolean | unset | Enables the direct structured-data endpoints for users who are not internal users. |
| `report.print_delay` | Print delay | Integer, milliseconds | 1 000 | The delay before the print dialog opens after a document has been produced. |

### 3.3 Sessions, authentication and keys

| Key | Full name | Type | Default | Effect |
|---|---|---|---|---|
| `sessions.max_inactivity_seconds` | Maximum session inactivity | Integer, seconds | 604 800, seven days | The cookie maximum age and the age after which an untouched stored session is deleted. A value that is not an integer is logged and the default is used. |
| `base.login_cooldown_after` | Login cooldown threshold | Integer | seeded 10; the rule's own fallback when the parameter is absent is 5 | The number of consecutive failed logins from one client address after which the cooldown applies. A value of 0 disables the feature. |
| `base.login_cooldown_duration` | Login cooldown duration | Integer, seconds | 60 | How long the cooldown lasts after each failure past the threshold. |
| `password.hashing.rounds` | Password derivation rounds | Integer | 0 | The iteration count of the password derivation. The effective value is the greater of this and 600 000. |
| `auth_password_policy.minlength` | Minimum password length | Integer | 0 | The minimum password length. A shorter password is refused with `Your password must contain at least %(minimal_length)d characters and only has %(current_count)d.` |
| `base.enable_programmatic_api_keys` | Programmatic key creation enabled | Boolean | unset | Allows a user who is not an administrator to create application keys from another key. |
| `base.programmatic_api_keys_limit` | Programmatic key limit | Integer | 10 | The maximum number of live keys a user may hold when creating them programmatically. |
| `portal.allow_api_keys` | Portal key management allowed | Boolean | unset | Allows portal users to manage application keys. |
| `auth_totp.policy` | Second-factor policy | Selection: `employee_required`, `all_required` | unset | Enforces a second authentication factor for internal users, or for everybody. |
| `auth_totp.trusted_device_age` | Trusted browser lifetime | Integer, days | 90 | The lifetime of a trusted-browser credential. A non-positive or unparseable value falls back to the default with a warning. |
| `auth_signup.reset_password` | Password reset offered | Boolean | unset | Whether the login page offers password reset. |
| `auth_signup.reset_password.validity.hours` | Password-reset link lifetime | Integer, hours | 4 | The lifetime of a password-reset link. |
| `auth_signup.signup.validity.hours` | Invitation link lifetime | Integer, hours | 144, six days | The lifetime of an invitation link. |
| `auth_signup.invitation_scope` | Sign-up scope | Selection: `b2b`, `b2c` | `b2b`, meaning by invitation only | Whether anybody may create an account or only invited people may. The two stored values are reproduced; the first means by invitation and the second means free sign-up. |
| `base.template_portal_user_id` | Portal template user | Reference to a User | the shipped portal template user | The user record new portal accounts are copied from. |
| `auth_ldap.disable_chase_ref` | Directory referral chasing disabled | Boolean | true | Whether the external directory lookup follows referrals. |
| `auth_oauth.authorization_header` | External assertion header | Text | unset | The header name carrying the external authentication assertion. |

### 3.4 Storage

| Key | Full name | Type | Default | Effect |
|---|---|---|---|---|
| `ir_attachment.location` | Attachment storage location | Selection: `file`, `db` | `file` | Where attachment content lives. See [`attachments-and-file-store.md`](attachments-and-file-store.md), section 2. |
| `base.image_autoresize_extensions` | Image reduction subtypes | Comma-separated text | `png,jpeg,bmp,tiff` | The image subtypes eligible for automatic reduction. |
| `base.image_autoresize_max_px` | Image reduction bounding box | Text of the form width, the letter `x`, height | `1920x1920` | The bounding box of automatic reduction. A value that reads as false disables it. |
| `base.image_autoresize_quality` | Image reduction quality | Integer | 80 | The re-encoding quality for the lossy subtype. |
| `cloud_storage_provider` | Remote object store provider | Selection | unset | Which remote object store, if any, holds large attachments. |
| `cloud_storage_min_file_size` | Remote object store threshold | Integer, bytes | the provider's default | Below this size, content stays in the local file store. |
| `cloud_storage_azure_account_name`, `cloud_storage_azure_container_name`, `cloud_storage_azure_tenant_id`, `cloud_storage_azure_client_id`, `cloud_storage_azure_client_secret`, `cloud_storage_azure_user_delegation_key_sequence` | Credentials and container of the first supported remote object store | Text, and an integer for the last | unset, and 0 for the last | Identify and authenticate against that store. |
| `cloud_storage_google_account_info`, `cloud_storage_google_bucket_name` | Credentials and container of the second supported remote object store | Text | unset | Identify and authenticate against that store. |
| `cloud_storage_migration_max_file_size` | Migration maximum file size | Integer, bytes | 1 000 000 000 | The largest attachment the migration job moves. |
| `cloud_storage_migration_max_batch_file_size` | Migration maximum batch size | Integer, bytes | 1 000 000 000 | The total size moved per batch. |
| `cloud_storage_migration_all_models` | Migrate every entity | Boolean | unset | Whether the migration covers every entity. |
| `cloud_storage_migration_message_models` | Migrate message attachments | Boolean | unset | Whether the migration covers message attachments. |
| `cloud_storage_migration_min_attachment_id` | Migration lower key bound | Integer | 0 | The lower bound of the key range to migrate. |
| `cloud_storage_migration_max_attachment_id` | Migration upper key bound | Integer | 0 | The upper bound of the key range to migrate. |

### 3.5 Diagnostics

| Key | Full name | Type | Default | Effect |
|---|---|---|---|---|
| `base.profiling_enabled_until` | Profiling enabled until | Date and time as text | empty | Profiling is possible only while the current instant is before this value. See [`logging-and-audit.md`](logging-and-audit.md), section 7. |
| `speedscope_cdn` | Flame viewer base address | Text | the publisher's default viewer address | Where the flame-graph viewer used to display a performance profile is fetched from. |
| `bus.gc_retention_seconds` | Notification retention | Integer, seconds | 86 400 | How long notification bus rows are kept. See [`notification-bus.md`](notification-bus.md). |

## 4. Messaging parameters

| Key | Full name | Type | Default | Effect |
|---|---|---|---|---|
| `mail.mail.queue.batch.size` | Outgoing queue batch size | Integer | 1 000 | The maximum number of outgoing messages one queue run selects. |
| `mail.session.batch.size` | Mail connection batch size | Integer | 1 000 | The maximum number of messages sent over one connection to the outgoing mail server before it is reopened. |
| `mail.mail.force.send.limit` and `mail.mail_force_send_limit` | Inline sending limit | Integer | 100 | Above this number of messages in one operation, the operation queues them instead of sending them inline; 0 means always queue. Two keys with this meaning and this default exist, the first read by the composer and notification flows and the second by the calendar flows. This is a **compatibility finding**: the corrected behaviour is one key read by both, and a replacement that collapses them must keep reading both so that an existing configuration keeps working. |
| `mail.batch_size` | Generation batch size | Integer | 50 | The batch size used when *generating* messages from a template or from a notification flow. |
| `mail.render.cron.limit` | Rendering job limit | Integer | 1 000 | The maximum number of records whose content a rendering job renders in one run. |
| `mail.server.personal.limit.minutes` | Personal mail server throttle | Integer | 30 when unset or zero | The per-minute throttle of an individually owned outgoing mail server. See [`mail-gateway.md`](mail-gateway.md), section 6. |
| `mail.server.personal.limit.minutes_outlook` | Personal hosted mail server throttle | Integer | as above | The same throttle for the externally hosted personal mail service. |
| `mail.default.from` | Default sender local part | Text | unset | The local part used as sender when no other rule applies. |
| `mail.default.from.filter` | Default sender filter | Text | unset | The sender filter used when no specific outgoing mail server is chosen. |
| `mail.catchall.alias` | Catchall local part | Text | unset | The local part that receives replies. |
| `mail.catchall.domain` | Mail domain | Text | unset | The mail domain of the deployment. |
| `mail.catchall.domain.allowed` | Permitted alias domains | Comma-separated text | unset | Restricts the right-hand part of an alias to these domains. It is sanitized on write. |
| `mail.bounce.alias` | Bounce local part | Text | unset | The local part that receives delivery failures. |
| `mail.gateway.loop.minutes` | Loop detection window | Integer, minutes | 120 | The window used to detect a mail loop. |
| `mail.gateway.loop.threshold` | Loop detection threshold | Integer | 20 | How many records created from the same sender within that window block further processing. |
| `mail.disable_personal_mail_servers` | Personal mail servers forbidden | Boolean | false | Forbids individually owned outgoing mail servers. |
| `base.default_max_email_size` | Maximum outgoing message size | Decimal, mebibytes | 10 | The largest outgoing message an outgoing mail server accepts. |
| `mail.activity.gc.delete_overdue_years` | Overdue activity retention | Integer, years | 0, meaning disabled | Activities overdue for more than this many years are deleted by the cleanup. |
| `mail.activity.systray.limit` | Activity notification area limit | Integer | 1 000 | How many activities the notification area loads. |
| `mail.restrict_template_rendering` | Template editing restricted | Boolean | unset | Restricts template editing to a dedicated group. Writing it moves that group in or out of the internal user group. |
| `mail.link_preview_throttle` | Link preview throttle | Integer | 99 | Above this many stored link previews for one domain in the last 10 seconds, no new preview is stored. |
| `mail.chat_from_token` | Conversation from a token | Boolean | unset | Allows starting a conversation from a token rather than from a session. |
| `mail.google_translate_api_key` | Translation service credential | Text | unset | The credential of the translation service used for message translation. |
| `mail.web_push_vapid_private_key`, `mail.web_push_vapid_public_key` | Browser push signing keys | Text | unset | The signing key pair of browser push notifications. Both must be present for the push job to deliver anything. See [`background-workers.md`](background-workers.md), section 6.2. |
| `mail.use_twilio_rtc_servers` | Relay servers in use | Boolean | unset | Whether real-time conversations use relay servers. |
| `mail.twilio_account_sid`, `mail.twilio_account_token` | Relay service credentials | Text | unset | The account identifier and token of the relay service. |
| `mail.use_sfu_server` | Forwarding server in use | Boolean | unset | Whether real-time conversations use a media forwarding server. |
| `mail.sfu_server_url`, `mail.sfu_server_key` | Forwarding server address and credential | Text | unset | The address of that server and the credential used to reach it. |
| `discuss.klipy_api_key` | Animated image service credential | Text | unset | The credential of the animated image search service. |
| `microsoft_outlook_client_id`, `microsoft_outlook_client_secret` | First hosted mail provider credentials | Text | unset | The delegated authorization credentials of the first supported hosted mail provider. |
| `google_gmail_client_id`, `google_gmail_client_secret` | Second hosted mail provider credentials | Text | unset | The same for the second supported hosted mail provider. |
| `microsoft_account.auth_endpoint`, `microsoft_account.token_endpoint` | First provider authorization endpoints | Text | the provider's defaults | The authorization and token addresses of that provider. |
| `mass_mailing.mail_server_id` | Campaign mail server | Reference to an Outgoing Mail Server | unset | The outgoing mail server used for campaigns. |
| `mass_mailing.outgoing_mail_server` | Dedicated campaign mail server | Boolean | unset | Whether campaigns use a dedicated outgoing mail server. |
| `mass_mailing.mass_mailing_reports` | Campaign summaries | Boolean | seeded with the text `True` when the mass mailing package is installed | Whether a summary is sent after a campaign completes. See [`background-workers.md`](background-workers.md), section 10. |
| `mass_mailing.show_blacklist_buttons` | Blocked-list buttons offered | Boolean | seeded with the text `True` when the mass mailing package is installed | Whether the unsubscription page offers the recipient the buttons that add its address to, or remove it from, the blocked list, so that the recipient manages its own blocked-list state instead of only leaving the mailing lists it is subscribed to. The page reads the parameter with the fallback true and then treats whatever text it gets as true, so an absent parameter, an empty parameter and the stored text `False` all leave the buttons visible; clearing the settings checkbox deletes the parameter and therefore also leaves them visible. This is a **compatibility finding**: the switch as shipped cannot be turned off. The corrected behaviour reads the stored text as a boolean, treating the texts `False`, `0` and the empty text as off, and applies the fallback true only when the parameter is absent. |
| `mass_mailing.cancelled_mails_months_limit` | Cancelled message retention | Integer, months | 6 | How long cancelled outgoing messages are kept; 0 or less disables the cleanup. |
| `sms.endpoint` | Text message service address | Text | the service's default | The address of the text message service. |
| `sms.session.batch.size` | Text message batch size | Integer | 500 | The number of text messages one queue run attempts. See [`background-workers.md`](background-workers.md), section 5. |
| `sms_twilio.session.batch.size` | Relay text message batch size | Integer | 10 | The same, for the relay-provider variant. |
| `snailmail.endpoint` | Postal letter service address | Text | the service's default | The address of the postal letter service. |
| `snailmail.timeout` | Postal letter service timeout | Number, seconds | 30 | The request timeout of that service. |
| `link_tracker.no_external_tracking` | External tracking suppressed | Boolean | unset | Suppresses the addition of campaign tracking parameters to outgoing links. |
| `digest.default_digest_emails` | Digests offered by default | Boolean | unset | Whether new users are subscribed to the default digest. |
| `digest.default_digest_id` | Default digest | Reference to a Digest Email | unset | Which digest new users are subscribed to. See [`background-workers.md`](background-workers.md), section 7. |
| `mail_mobile.disable_redirect_firebase_dynamic_link` | Mobile dynamic link redirection suppressed | Boolean | unset | Suppresses the dynamic-link redirection of the mobile application. |
| `mail_mobile.enable_ocn` | Mobile cloud notifications | Boolean | unset | Enables cloud notifications for the mobile application. |

## 5. Accounting and invoicing parameters

| Key | Full name | Type | Default | Effect |
|---|---|---|---|---|
| `account.use_invoice_terms` | Invoice terms appended | Boolean | unset | Whether the company's terms are appended to invoices. |
| `account.show_sale_receipts` | Sales receipts offered | Boolean | unset | Whether sales receipts are offered as a document type. |
| `account.product_name_similarity_threshold` | Product matching threshold | Number between 0 and 1 | 0.9 | The similarity above which an imported line is matched to an existing product. |
| `account.pdf_generation_batch` | Document rendering batch | Integer | 80 | How many documents are rendered per batch when producing printable invoices. |
| `sequence.mixin.constraint_start_date` | Numbering continuity start date | Date as text | `1970-01-01` | The date from which numbering-sequence continuity is enforced. A numbered document whose sequence date lies strictly after this date and whose number does not match the period that its numbering style implies is refused with `The %(date_field)s (%(date)s) you've entered isn't aligned with the existing sequence number (%(sequence)s). Clear the sequence number to proceed.` followed on the next line by `To maintain date-based sequences, select entries and use the resequence option from the actions menu, available in developer mode.` The three placeholders are, in order, the label of the document's date field, that date formatted for the user's language and the number already carried by the document. Moving the date forward lets documents that are already inconsistent be edited; it is not a supported way to switch the check off, because the numbering chain stops being reliable once the check is bypassed wholesale. |
| `account.custom_templates_facturx_list` | Extra electronic invoice templates | Comma-separated text | empty | Extra electronic invoice templates offered. |
| `account.display_name_in_footer` | Company name in the footer | Boolean | unset | Whether the company name is repeated in the document footer. See [`report-rendering.md`](report-rendering.md), section 6.6. |
| `account.skip_create_bank_account_on_reconcile` | Bank account creation suppressed | Boolean | unset | Suppresses the automatic creation of a bank account when reconciling a statement line. |
| `account.tests_shared_js_python` | Shared calculation test mode | Boolean | unset | Enables the mode in which the client and the server run the same tax calculation for comparison. |
| `account_payment.enable_portal_payment` | Portal payment enabled | Boolean | true | Whether customers may pay from the portal. |
| `stock_account.skip_lock_date_check` | Valuation lock-date check suppressed | Boolean | unset | Suppresses the lock-date check on inventory valuation entries. |
| `account_edi_ubl_cii.disable_pdf_in_xml` | Printable document not embedded | Boolean | false | Suppresses embedding the printable document inside the electronic invoice. |
| `account_edi_ubl_cii.use_new_dict_to_xml_helpers` | Current document builder | Boolean | true | Selects the current electronic document builder. |
| `account_edi_proxy_client.demo` | Document-exchange proxy demonstration mode | Text | seeded with the value `demo` when the document-exchange proxy package is installed | Marks that the document-exchange proxy runs in demonstration mode. In that mode the registration of a proxy user is simulated locally instead of being requested from the proxy: the client identifier becomes the word `demo` followed by the company key and the proxy type, and the refresh token becomes the word `demo`. Any attempt to reach the proxy while a proxy user is in that mode is refused with `Can't access the proxy in demo mode` under the error code `block_demo_mode`, which is the last barrier for a caller that failed to handle the mode itself. |
| `account_peppol.edi.mode` | Exchange network environment | Selection: `demo`, `test`, `prod` | unset, except that installing the exchange network package on a database marked neutralized sets it to `demo` | The environment of the document exchange network. |
| `account_peppol.disable_pdp_warning` | Exchange network banner suppressed | Boolean | false | Suppresses the banner inviting the user to register on the exchange network. |
| `analytic.project_plan` | Project analytic plan | Reference to an Analytic Plan | 0 | The analytic plan used for project analytics. |
| `analytic.analytic_plan_projects` | Project account plan | Reference to an Analytic Plan | unset | The plan holding project accounts. |

Localization-specific parameters follow the same pattern and are documented with their localization. The keys the platform ships are `l10n_dk_nemhandel.edi.mode`, the environment of the Danish exchange network; `l10n_eg_eta.sign.host`, the address of the Egyptian signing service; `l10n_es_edi_tbai.epigrafe`, the activity code of the Spanish sealed-record service; `l10n_fr_fec.batch_size`, the batch size of the French ledger export; `l10n_fr_pdp.kyc_siren`, the registration number used on the French exchange platform; `l10n_in.gsp_provider`, the Indian service provider; `l10n_in_edi.endpoint`, the address of the Indian electronic invoice service, which every call of that service is issued against; `l10n_it_edi.proxy_user_edi_mode`, the environment of the Italian exchange system; `l10n_my_edi_test_server_url`, the test address of the Malaysian exchange service; and `l10n_pl_edi_ksef.mode`, the environment of the Polish national system.

## 6. Sales, purchase and commerce parameters

| Key | Full name | Type | Default | Effect |
|---|---|---|---|---|
| `sale.automatic_invoice` | Automatic invoicing | Boolean | unset | Creates and sends an invoice automatically once an order is paid online. |
| `sale.default_confirmation_template` | Order confirmation template | Reference to a Message Template | the shipped order confirmation template | The template used to confirm an order. |
| `sale.default_invoice_email_template` | Invoice sending template | Reference to a Message Template | the shipped invoice template | The template used to send an invoice. |
| `sale.async_emails` | Deferred order messages | Boolean | false | Defers order confirmation messages to a job instead of sending them inline. |
| `sale.always_include_selected_documents` | Always attach selected documents | Boolean | unset | Always attaches the selected documents to a quotation. |
| `sale.disable_sale_update` | Order update from delivery suppressed | Boolean | unset | Suppresses the automatic update of an order from its delivery. |
| `sale.invoiced_timesheet` | Invoiced recorded time | Selection | unset | Which recorded time is invoiced. |
| `sale_stock.use_security_lead` | Security lead time applied | Boolean | unset | Whether the security lead time is applied to sales deliveries. |
| `sales_team.membership_multi` | Multiple team membership | Boolean | false | Whether a salesperson may belong to several teams. |
| `purchase_stock.on_time_delivery_days` | Vendor punctuality window | Integer, days | 365 | The window over which vendor punctuality is computed. |
| `product.dynamic_variant_limit` | Dynamic variant threshold | Integer | 1 000 | Above this number of possible combinations, variants are created on demand instead of upfront. |
| `product.weight_in_lbs` | Weights in pounds | Boolean | unset | Displays weights in pounds. |
| `product.volume_in_cubic_feet` | Volumes in cubic feet | Boolean | unset | Displays volumes in cubic feet. |
| `website_sale.require_billing_details_for_services` | Billing address required for services | Boolean | true | Whether a billing address is required for baskets containing only services. |
| `website_sale.markup_data_limit_variants` | Structured markup variant limit | Integer | false, meaning unlimited | The maximum number of variants described in the structured page markup. |
| `website_sale_coupon.abandonned_coupon_validity` | Abandoned basket coupon validity | Integer, days | 4 | How long a coupon offered on an abandoned basket stays valid. The key's spelling is reproduced, including its doubled letter. |
| `website.visitor.live.days` | Visitor liveness window | Integer, days | 60 | How long a visitor record is considered live. |
| `website.apply_new_theme` | New visual theme | Boolean | 0 | Applies the new visual theme. |
| `website.disable_delay_translations` | Immediate translation rendering | Boolean | unset | Renders translations immediately instead of deferring them. See [`translation.md`](translation.md), section 6.5. |
| `website_form_enable_metadata` | Form metadata stored | Boolean | unset | Stores the visitor metadata of submitted forms. |
| `website_profile.uuid` | Public profile area identity | Text | generated | The identity of the public profile area. |
| `loyalty.compute_all_discount_product_ids` | Discount product precomputation | Selection | enabled | Whether every discount product is precomputed. |
| `loyalty.timezone` | Loyalty time zone | Text | the coordinated universal time zone | The time zone used to evaluate loyalty validity windows. |
| `base_setup.show_effect` | Celebration effects | Boolean | unset | Whether the interface plays a celebration effect on certain confirmations. |

## 7. Inventory, manufacturing and logistics parameters

| Key | Full name | Type | Default | Effect |
|---|---|---|---|---|
| `stock.no_auto_scheduler` | Automatic scheduler suppressed | Boolean | unset | Suppresses the automatic run of the replenishment scheduler after stock moves. |
| `stock.picking_no_auto_reserve` | Automatic reservation suppressed | Boolean | unset | Suppresses the automatic reservation of transfers. |
| `stock.skip_quant_tasks` | Quantity maintenance suppressed | Boolean | unset | Suppresses the maintenance work performed on stock quantity records. |
| `stock.merge_only_same_date` | Merge only equal dates | Boolean | unset | Merges stock moves only when their scheduled dates are equal. |
| `stock.merge_ignore_date_deadline` | Ignore the deadline when merging | Boolean | unset | Ignores the deadline when merging stock moves. |
| `stock.propagate_uom` | Propagate the unit of measure | Boolean | unset | Propagates the unit of measure of a move to the moves it generates. |
| `stock.cancel_moves_origin` | Cancel originating moves | Boolean | unset | Cancels the originating moves when a move is cancelled. |
| `stock.intercompany_auto_unpack` | Automatic inter-company unpacking | Boolean | unset | Unpacks inter-company receipts automatically. |
| `stock.barcode_separator` | Aggregated code separator | Text | `,` | The separator used in aggregated scanning codes. |
| `stock.agg_barcode_max_length` | Aggregated code maximum length | Integer | 400 | The maximum length of an aggregated scanning code. |
| `stock.report_stock_quantity_period` | Forecast horizon | Integer, months | 3 | The horizon of the forecast report. |
| `stock.show_expected_quantity_count` | Expected quantity counter | Boolean | false | Shows the expected quantity counter on transfers. |
| `mrp.workcenter_max_planning_iterations` | Work centre planning attempts | Integer | 50 | The maximum number of attempts when placing an operation on a work centre. |
| `barcode.max_time_between_keys_in_ms` | Scan character interval | Integer, milliseconds | 150 | The maximum delay between two characters for a scan to be recognized as one code. |

## 8. Customer relationship and marketing parameters

| Key | Full name | Type | Default | Effect |
|---|---|---|---|---|
| `crm.pls_fields` | Predictive scoring fields | Comma-separated text | the shipped list of scoring fields | Which fields feed the predictive lead score. |
| `crm.pls_start_date` | Predictive scoring start date | Date | 8 days before installation | The earliest date whose leads feed the model. |
| `crm.lead.auto.assignment` | Automatic lead assignment | Boolean | false | Enables automatic lead assignment. |
| `crm.assignment.delay` | Assignment delay | Integer | 0 | The delay applied between two assignment runs. |
| `crm.assignment.commit.bundle` | Assignment commit bundle | Integer | 100 | How many assignments are committed per batch. |
| `crm.lead.rot.max.months` | Stale lead threshold | Integer, months | 12 | After how long an untouched lead is considered stale. |
| `crm.iap.lead.enrich.setting` | Lead enrichment mode | Selection: `auto`, `manual` | `auto` | Whether leads are enriched automatically. |
| `crm.lead_mining_in_pipeline` | Generated leads in the pipeline | Boolean | unset | Whether generated leads land directly in the pipeline. |
| `enrich.endpoint` | Enrichment service address | Text | the service's default | The address of the enrichment service. |
| `reveal.endpoint` | Visitor identification service address | Text | the service's default | The address of the visitor identification service. |
| `reveal.lead_month_valid` | Generated lead validity | Integer, months | the service's default | How long a generated lead stays valid. |
| `reveal.view_weeks_valid` | Recorded visit validity | Integer, weeks | the service's default | How long a recorded visit stays valid. |
| `reveal.already_notified` | Credit warning already sent | Boolean | unset | Whether the credit warning has already been sent. |
| `marketing_card.card_image_cleanup_interval_days` | Card image retention | Integer, days | 60 | How long generated card images are kept. |

## 9. People, time and events parameters

| Key | Full name | Type | Default | Effect |
|---|---|---|---|---|
| `hr_expense.use_mailgateway` | Expenses by electronic mail | Boolean | unset | Enables creating expenses by electronic mail. |
| `hr_fleet.delay_alert_contract` | Vehicle contract alert lead time | Integer, days | 30 | How many days before a vehicle contract expires the alert is raised. |
| `calendar.default_privacy` | Default event privacy | Selection: `public`, `private`, `confidential` | `public` | The privacy of a newly created event. |
| `calendar.block_mail` | Event notifications suppressed | Boolean | unset | Suppresses event notification messages. |
| `calendar.max_recurrence_years` | Recurrence expansion horizon | Integer, years | 15 | How far an unbounded recurrence is expanded. |
| `event.use_event_barcode` | Registration scanning codes | Boolean | unset | Enables scanning codes on registrations. |
| `event.event_mail_async` | Deferred event communications | Boolean | unset | Defers event communications to a job. |

## 10. Integration and external service parameters

| Key | Full name | Type | Default | Effect |
|---|---|---|---|---|
| `iap.endpoint` | Metered service platform address | Text | the service's default | The address of the metered service platform. |
| `iap.partner_autocomplete.endpoint` | Company lookup service address | Text | the service's default | The address of the company lookup service. |
| `iap_vies.endpoint`, `iap_vies.client_identifier`, `iap_vies.client_token` | Tax number validation service | Text | the service's default, then unset | The address and credentials of the tax number validation service. |
| `recaptcha_public_key`, `recaptcha_private_key`, `recaptcha_min_score`, `enable_recaptcha` | First challenge service | Text, text, number, boolean | unset, unset, the provider's default, unset | The credentials, the score threshold and the switch of the first supported challenge service used on public forms. |
| `cf.turnstile_site_key`, `cf.turnstile_secret_key` | Second challenge service | Text | unset | The credentials of the second supported challenge service. |
| `google_address_autocomplete.google_places_api_key` | Address completion credential | Text | unset | The credential of the address completion service. |
| `google_address_autocomplete.minimal_partial_address_size` | Address completion minimum length | Integer | 5 | The least number of characters before completion is requested. |
| `google_maps.signed_static_api_key`, `google_maps.signed_static_api_secret` | Static map signing credentials | Text | unset | The credentials used to sign static map images. |
| `base_geolocalize.geo_provider` | Geolocation provider | Text | unset | Which service resolves an address to coordinates. |
| `base_geolocalize.google_map_api_key` | Geolocation credential | Text | unset | The credential of that service. |
| `google_calendar_client_id`, `google_calendar_client_secret`, `google_calendar_sync_paused` | First external calendar service | Text, text, boolean | unset | The credentials and the pause switch of the first supported external calendar service. |
| `google_calendar.sync.range_days` | First calendar synchronization horizon | Integer, days | 365 | How far ahead events are synchronized. |
| `microsoft_calendar_client_id`, `microsoft_calendar_client_secret`, `microsoft_calendar_sync_paused` | Second external calendar service | Text, text, boolean | unset | The credentials and the pause switch of the second supported external calendar service. |
| `microsoft_calendar.sync.range_days` | Second calendar synchronization horizon | Integer, days | 365 | How far ahead events are synchronized. |
| `microsoft_calendar.sync.lower_bound_range`, `microsoft_calendar.sync.first_synchronization_date` | Second calendar lower bound | Integer, date | unset | The lower bound of the synchronized window. |
| `microsoft_calendar.graph_timeout` | Second calendar request timeout | Number, seconds | 5 | The timeout of a synchronization request. |
| `microsoft_calendar.microsoft_guid` | Second calendar tenant | Text | false | The tenant of that external calendar service. |
| `html_editor.media_library_endpoint` | Shared media library address | Text | the service's default | The address of the shared media library. |
| `html_editor.olg_api_endpoint` | Text generation service address | Text | the service's default | The address of the text generation service. |
| `unsplash.access_key`, `unsplash.app_id` | Stock image service credentials | Text | unset | The credentials of the stock image service. |
| `transifex.project_url` | Translation project address | Text | the shipped project address | The address of the external translation project. See [`translation.md`](translation.md), section 11. |
| `pos_pine_labs.pine_labs_proxy_endpoint`, `pos_pine_labs.payment_auto_cancel_duration` | Payment terminal address and cancellation delay | Text, integer | unset, the provider's default | The address and the automatic cancellation delay of one supported payment terminal. One pair exists per supported terminal. |
| `point_of_sale.limited_product_count`, `point_of_sale.limited_customer_count` | Preloaded product and customer counts | Integer | the shipped defaults | How many products and customers a session preloads. |
| `point_of_sale.use_lna` | Local network agent in use | Boolean | unset | Whether the local network agent is used for peripherals. |
| `point_of_sale.log_order_data` | Raw order payloads logged | Boolean | false | Whether raw order payloads are logged. |

## 11. Worked examples

### 11.1 Resetting a parameter to its default

| Step | Effect |
|---|---|
| 1 | `mail.batch_size` is set to `10`. Every message generation now works in batches of 10. |
| 2 | An administrator writes the value false for that key. |
| 3 | The parameter record is **deleted**, not set to a falsy text. |
| 4 | The next read returns the caller's fallback, which is 50. |

Setting the value to the empty text has the same observable effect on readers, because an empty value yields the fallback, but it leaves a record behind that appears in the parameter list. Deleting is the documented way.

### 11.2 Changing the upload limit

| Step | Effect |
|---|---|
| 1 | The administrator sets `web.max_file_upload_size` to `2000000`. |
| 2 | The write clears the `stable` cache key and announces the invalidation after the commit. |
| 3 | Every worker, at its next request, empties its `stable`, `default` and `templates.cached_values` containers. |
| 4 | The pre-dispatch of every subsequent request reads 2 000 000 and sets the request's body limit. |
| 5 | A request posting 3 000 000 bytes to an endpoint that declares no limit of its own is refused with the payload-too-large status. |
| 6 | A request posting 3 000 000 bytes to an endpoint that declares a 50 000 000-byte limit succeeds, because the endpoint declaration is applied after the database-wide value. |

### 11.3 A malformed numeric parameter

| Step | Effect |
|---|---|
| 1 | `sessions.max_inactivity_seconds` is set to `two weeks`. |
| 2 | The next request tries to convert it, fails, logs the warning `Invalid value for 'sessions.max_inactivity_seconds', using default value.` and uses 604 800. |
| 3 | No request fails. The session cookie maximum age is the default. |

Every platform reader of a numeric parameter behaves this way: convert, and on failure log and use the documented default.

### 11.4 Rotating the database secret

| Step | Effect |
|---|---|
| 1 | The administrator changes `database.secret`. |
| 2 | Every registry cache container is dropped by the `stable` invalidation, so every memoized session binding token disappears. |
| 3 | The next request of every session recomputes the expected token with the new secret. It differs from the stored one, so every session is logged out. |
| 4 | Every outstanding cross-site token, password-reset link, invitation link, unsubscribe link and scoped field access token stops validating. |

This is the intended emergency procedure and must remain available. Note that the parameter is one of the six protected ones: it may be given a new value but neither renamed nor deleted.

### 11.5 A settings form that saves every field

| Step | Effect |
|---|---|
| 1 | An administrator opens the general settings, changes one switch and saves. |
| 2 | The form writes every parameter it owns, most of them with their existing values. |
| 3 | Each write whose text form equals the stored one issues no statement, so only the changed parameter is written. |
| 4 | The cache key is nonetheless cleared once by that single write, and announced once after the commit. |
| 5 | A form that saved values through direct record writes rather than through the write operation would issue one statement per field and would still clear the cache once, because the clearing happens in the record write as well. |

## 12. Acceptance criteria

1. **Given** a parameter that does not exist, **when** it is read with a fallback, **then** the fallback is returned and no record is created.
2. **Given** a parameter whose stored value is the empty text, **when** it is read with a fallback, **then** the fallback is returned.
3. **Given** an existing parameter, **when** it is written with a false or unset value, **then** the record is deleted and the previous value is returned.
4. **Given** an existing parameter, **when** it is written with the same value, **then** no write statement is issued and the previous value is returned.
5. **Given** a key that does not exist, **when** it is written with a false or unset value, **then** nothing is created and false is returned.
6. **Given** any parameter creation, write or deletion, **when** the transaction commits, **then** the `stable` cache key is invalidated locally and announced to the other processes.
7. **Given** a parameter written earlier in the same transaction, **when** it is read again in that transaction, **then** the new value is returned, because the lookup flushes first.
8. **Given** one of the six seeded parameters, **when** a caller tries to rename it, **then** the operation is refused with `You cannot rename config parameters with keys %s` naming that key.
9. **Given** one of the six seeded parameters, **when** a caller tries to delete it, **then** the operation is refused with `You cannot delete the %s record.` naming that key.
10. **Given** one of the six seeded parameters, **when** a caller changes its value, **then** the change succeeds.
11. **Given** a package being removed that owns a seeded parameter, **when** the removal runs, **then** the deletion guard does not fire, because it is declared not to run at package removal.
12. **Given** two parameters with the same key, **when** the second is created, **then** the operation is refused with `Key must be unique.`
13. **Given** a user without read access on System Parameters, **when** they read one, **then** the access check refuses the call.
14. **Given** `mail.restrict_template_rendering` written with a false value, **when** the write completes, **then** every internal user holds the template-editing group; **given** it written with a true value, **then** they do not.
15. **Given** `mail.catchall.domain.allowed` written with an unsanitized list, **when** the write completes, **then** the stored value is the sanitized list; the same holds for a direct create or write.
16. **Given** a numeric parameter whose stored value cannot be converted, **when** a platform reader reads it, **then** a warning is logged, the documented default is used, and the operation continues.
17. **Given** `base.login_cooldown_after` set to `0`, **when** logins fail repeatedly from one address, **then** no cooldown is applied.
18. **Given** `ir_attachment.location` changed from `file` to `db`, **when** the migration operation runs, **then** every binary attachment ends with its content in the database and an empty stored file name.
19. **Given** a fresh database, **when** it is created, **then** exactly the six parameters of section 2 exist with the stated values.
20. **Given** an existing database, **when** the seeding operation runs without being asked to overwrite, **then** the existing values are left unchanged and only absent keys are created.
21. **Given** `base.image_autoresize_max_px` set to a value that reads as false, **when** an oversized image is uploaded, **then** it is stored unchanged.
22. **Given** both `mail.mail.force.send.limit` and `mail.mail_force_send_limit` set to different values, **when** a composer flow and a calendar flow each send, **then** each reads its own key, which is the compatibility finding recorded in section 4.
23. **Given** `sequence.mixin.constraint_start_date` at its default `1970-01-01` and a posted document numbered for the year 2025, **when** its accounting date is changed to a date in 2026, **then** the change is refused with the message of section 5, because the date lies after the start date and no longer matches the number.
24. **Given** `sequence.mixin.constraint_start_date` set to `2026-12-31` and the same document, **when** its accounting date is changed to a date in 2026, **then** the change is accepted, because the date does not lie after the start date.
25. **Given** the mass mailing package freshly installed, **when** a recipient opens the unsubscription page, **then** the blocked-list buttons are shown, because `mass_mailing.show_blacklist_buttons` holds the text `True`.
26. **Given** the settings checkbox for the blocked-list buttons cleared, which deletes the parameter, **when** a recipient opens the unsubscription page, **then** the buttons are still shown, because the fallback used when the parameter is absent is true; this is the compatibility finding recorded in section 4.
27. **Given** `mass_mailing.show_blacklist_buttons` set by hand to the text `False`, **when** a recipient opens the unsubscription page, **then** the buttons are still shown, because any non-empty stored text reads as true; under the corrected behaviour of section 4 they would be hidden and only the mailing-list choices would remain.
28. **Given** the document-exchange proxy package freshly installed, **when** `account_edi_proxy_client.demo` is read, **then** its value is `demo`; **when** a proxy user in demonstration mode is registered, **then** no request leaves the deployment and the user receives the locally built client identifier and the refresh token `demo`.
29. **Given** a proxy user in demonstration mode, **when** any caller tries to reach the proxy, **then** the call is refused with `Can't access the proxy in demo mode` under the code `block_demo_mode`.

## 13. Reconciliation notes

1. Every parameter key was given in a paraphrased full-word form. A parameter key is a contractual identifier: an existing database holds these exact strings and an integration writes them. Every key in this document is therefore the reproduced one, and the full name in words is carried in its own column, as the documentation rules require for a field table.
2. The keys whose paraphrase differed most from the reproduced form, and which a reader of the earlier text would otherwise not find, are: the database secret, now `database.secret`; the base address, now `web.base.url` and `web.base.url.freeze`; the document rendering base address, now `report.url`; the session inactivity limit, now `sessions.max_inactivity_seconds`; the attachment storage location, now `ir_attachment.location`; the programmatic key parameters, now `base.enable_programmatic_api_keys` and `base.programmatic_api_keys_limit`; the trusted browser lifetime, now `auth_totp.trusted_device_age`; the browser push signing keys, now `mail.web_push_vapid_private_key` and `mail.web_push_vapid_public_key`; the text message batch size, now `sms.session.batch.size`; the campaign summary switch, now `mass_mailing.mass_mailing_reports`; and the translation project address, now `transifex.project_url`.
3. Several keys the earlier text listed under invented names do not exist. Where the behaviour exists under a different key, the reproduced key is given. The values the subscription status check job writes were said to sit under keys the subscription service supplies; they do not. The platform fixes all five keys and only the values come from the service, so section 3.1 now names each of the five keys.
4. The two keys with the same meaning and default for the inline sending limit were noted as collapsible. They are recorded as a **compatibility finding** in section 4, with the corrected behaviour and the migration constraint.
5. The abandoned-basket coupon validity key contains a spelling slip in the original. It is reproduced exactly, and the slip is pointed out so that a reader does not correct it.
6. The delete protection of the seeded parameters was attributed to the same rule as the rename protection. They are two different rules with two different messages, and the deletion guard additionally does not run at package removal; both are specified in section 1.4.
7. The notification retention parameter was named without its package prefix. It is `bus.gc_retention_seconds`.
8. The parameter that fixes the date from which numbering-sequence continuity is enforced was given as an accounting-prefixed key. It belongs to the numbering behaviour shared by every numbered document, and its reproduced key is `sequence.mixin.constraint_start_date`, with the default `1970-01-01` rather than an unset value. Section 5 carries it with its refusal message.
9. The switch that puts the document-exchange proxy into demonstration mode was given under a shortened key and described as a boolean left unset. Its reproduced key is `account_edi_proxy_client.demo`, it holds text, and installing the package seeds it with the value `demo`. Section 5 carries it with the effects of the mode.
10. The switch for the blocked-list buttons was given under a key spelled in full words and was said to govern contact forms. Its reproduced key is `mass_mailing.show_blacklist_buttons`, the package ships it with the text `True`, and it governs the unsubscription page, not a contact form. Section 4 carries the corrected statement together with the **compatibility finding** that the switch cannot in fact be turned off.
11. The list of localization keys dropped the address of the Indian electronic invoice service. It is `l10n_in_edi.endpoint` and it is restored to the closing paragraph of section 5, which now lists ten localization keys.
12. Two seeded values were recorded as unset. Installing the mass mailing package seeds `mass_mailing.mass_mailing_reports` and `mass_mailing.show_blacklist_buttons` with the text `True`, and installing the exchange network package seeds `account_peppol.edi.mode` with `demo` only when the database is marked neutralized. The three rows now state those conditions.
