# User (`res.users`)

**Transport name:** `res.users`  
**Storage name:** `res_users`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`  
**Extended by packages:** `web`, `base_setup`, `bus`, `web_tour`, `mail`, `mail`, `auth_signup`, `resource`, `digest`, `auth_ldap`, `auth_oauth`, `auth_passkey`, `auth_password_policy`, `auth_totp`, `auth_totp_mail`, `auth_timeout`, `auth_totp_portal`, `contacts`, `phone_validation`, `base_import`, `calendar`, `sales_team`, `crm`, `im_livechat`, `crm_livechat`, `stock`, `sale_stock`, `gamification`, `google_calendar`, `google_gmail`, `hr`, `hr_attendance`, `hr_gamification`, `hr_holidays`, `hr_homeworking`, `hr_recruitment`, `website`, `website_profile`, `website_slides`, `project`, `point_of_sale`, `lunch`, `mail_bot`, `mass_mailing`, `mass_mailing_sms`, `microsoft_account`, `microsoft_calendar`, `microsoft_outlook`, `project_hr_skills`, `project_todo`, `web_unsplash`, `website_forum`, `website_sale_wishlist`

Description: User

## Identity and behavior

- Mixins (classical inheritance): `bus.listener.mixin`, `pos.load.mixin`
- Delegation inheritance: embeds `res.partner` through field `partner_id`
- Default ordering: `name, login`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (140)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `partner_id` | Related Partner | many to one | `res.partner` | required; indexed; on delete of the target: restrict; Help: Partner-related data of the user |
| `login` | Login | single line text |  | required; Help: Used to log into the system |
| `password` | Password | single line text |  | computed by rule `_compute_password` (not stored); writable through an inverse rule; not copied on duplication; Help: Keep empty if you don't want the user to be able to connect on the system. |
| `new_password` | Set Password | single line text |  | computed by rule `_compute_password` (not stored); writable through an inverse rule; Help: Specify a value only when creating a user or if you're changing the user's password, otherwise leave empty. After a change of password, the user has to login again. |
| `api_key_ids` | application programming interface Keys | one to many | `res.users.apikeys` | inverse field `user_id` |
| `signature` | Email Signature | rich text |  | computed by rule `_compute_signature` and stored |
| `active` | Active | boolean |  | default `True` |
| `active_partner` | Partner is Active | boolean |  | read only; related through path `partner_id.active` |
| `action_id` | Home Action | many to one | `ir.actions.actions` | Help: If specified, this action will be opened at log on for this user, in addition to the standard menu. |
| `log_ids` | User log entries | one to many | `res.users.log` | inverse field `create_uid` |
| `device_ids` | User devices | one to many | `res.device` | inverse field `user_id` |
| `login_date` | Latest Login | date and time |  | related through path `log_ids.create_date` |
| `share` | Share User | boolean |  | computed by rule `_compute_share` and stored; Help: External user with limited access, created only for the purpose of sharing data. |
| `companies_count` | Number of Companies | integer |  | computed by rule `_compute_companies_count` (not stored) |
| `tz_offset` | Timezone offset | single line text |  | computed by rule `_compute_tz_offset` (not stored) |
| `res_users_settings_ids` | Resource Users Settings | one to many | `res.users.settings` | inverse field `user_id` |
| `res_users_settings_id` | Settings | many to one | `res.users.settings` | computed by rule `_compute_res_users_settings_id` (not stored); searchable through a search rule |
| `company_id` | Company | many to one | `res.company` | required; default computed dynamically (lambda self: self.env.company.id); Help: The default company for this user. |
| `company_ids` | Companies | many to many | `res.company` | default computed dynamically (lambda self: self.env.company.ids); association table `res_company_users_rel` |
| `name` | Name | single line text |  | related through path `partner_id.name` |
| `email` | Email | single line text |  | related through path `partner_id.email` |
| `email_domain_placeholder` | Email Domain Placeholder | single line text |  | computed by rule `_compute_email_domain_placeholder` (not stored) |
| `phone` | Phone | single line text |  | related through path `partner_id.phone` |
| `group_ids` | Groups | many to many | `res.groups` | default computed dynamically (lambda s: s._default_groups()); association table `res_groups_users_rel`; Help: Groups explicitly assigned to the user |
| `all_group_ids` | Groups and implied groups | many to many | `res.groups` | computed by rule `_compute_all_group_ids` (not stored); searchable through a search rule |
| `accesses_count` | # Access Rights | integer |  | computed by rule `_compute_accesses_count` (not stored); Help: Number of access rights that apply to the current user |
| `rules_count` | # Record Rules | integer |  | computed by rule `_compute_accesses_count` (not stored); Help: Number of record rules that apply to the current user |
| `groups_count` | # Groups | integer |  | computed by rule `_compute_accesses_count` (not stored); Help: Number of groups that apply to the current user |
| `view_group_hierarchy` | Technical field for user group setting | structured document |  | default computed dynamically (_default_view_group_hierarchy); not copied on duplication |
| `role` | Role | selection |  | computed by rule `_compute_role` (not stored) |
| `tour_enabled` | Onboarding | boolean |  | computed by rule `_compute_tour_enabled` and stored |
| `is_in_call` | Is in call | boolean |  | related through path `partner_id.is_in_call` |
| `role_ids` | User Roles | many to many | `res.role` | association table `res_role_res_users_rel`; Help: Users are notified whenever one of their roles is @-mentioned in a conversation. |
| `can_edit_role` | Can Edit Role | boolean |  | computed by rule `_compute_can_edit_role` (not stored) |
| `notification_type` | Notification | selection |  | required; computed by rule `_compute_notification_type` and stored; writable through an inverse rule; default `email`; Help: Policy on how to handle Chatter notifications: - By Emails: notifications are sent to your email address - In Odoo: notifications appear in your Odoo Inbox |
| `presence_ids` | Presence | one to many | `mail.presence` | visible only to groups `base.group_system`; inverse field `user_id` |
| `out_of_office_from` | Out Of Office From | date and time |  |  |
| `out_of_office_to` | Out Of Office To | date and time |  |  |
| `out_of_office_message` | Vacation Responder | rich text |  |  |
| `is_out_of_office` | Out of Office | boolean |  | computed by rule `_compute_is_out_of_office` (not stored) |
| `im_status` | IM Status | single line text |  | computed by rule `_compute_im_status` (not stored) |
| `manual_im_status` | IM status manually set by the user | selection |  |  |
| `outgoing_mail_server_id` | Outgoing Mail Server | many to one | `ir.mail_server` | computed by rule `_compute_outgoing_mail_server_id` (not stored); visible only to groups `base.group_user` |
| `outgoing_mail_server_type` | Outgoing Mail Server Type | selection |  | required; computed by rule `_compute_outgoing_mail_server_id` (not stored); default `default`; visible only to groups `base.group_user`; on delete of the target: {"outlook": "set default"}; extended by packages `google_gmail`, `microsoft_outlook` |
| `has_external_mail_server` | Has External Mail Server | boolean |  | computed by rule `_compute_has_external_mail_server` (not stored) |
| `state` | Status | selection |  | computed by rule `_compute_state` (not stored); searchable through a search rule |
| `resource_ids` | Resources | one to many | `resource.resource` | inverse field `user_id` |
| `resource_calendar_id` | Default Working Hours | many to one | `resource.calendar` | related through path `resource_ids.calendar_id` |
| `oauth_provider_id` | open authorization Provider | many to one | `auth.oauth.provider` |  |
| `oauth_uid` | open authorization User identifier | single line text |  | not copied on duplication; Help: Oauth Provider user_id |
| `oauth_access_token` | open authorization Access Token Store | single line text |  | read only; not copied on duplication; visible only to groups `{"expression": "fields.NO_ACCESS"}` |
| `has_oauth_access_token` | Has open authorization Access Token | boolean |  | computed by rule `_compute_has_oauth_access_token` (not stored); visible only to groups `base.group_erp_manager` |
| `auth_passkey_key_ids` | Auth Passkey Key | one to many | `auth.passkey.key` | inverse field `create_uid` |
| `totp_secret` | Time-based one-time password Secret | single line text |  | computed by rule `_compute_totp_secret` (not stored); writable through an inverse rule; not copied on duplication; visible only to groups `{"expression": "fields.NO_ACCESS"}` |
| `totp_last_counter` | Time-based one-time password Last Counter | integer |  | not copied on duplication; visible only to groups `{"expression": "fields.NO_ACCESS"}` |
| `totp_enabled` | Two-factor authentication | boolean |  | computed by rule `_compute_totp_enabled` (not stored); searchable through a search rule |
| `totp_trusted_device_ids` | Trusted Devices | one to many | `auth_totp.device` | inverse field `user_id` |
| `calendar_default_privacy` | Calendar Default Privacy | selection |  | computed by rule `_compute_calendar_default_privacy` (not stored); writable through an inverse rule |
| `crm_team_ids` | Sales Teams | many to many | `crm.team` | read only; computed by rule `_compute_crm_team_ids` (not stored); searchable through a search rule; not copied on duplication; must belong to the same company; association table `crm_team_member` |
| `crm_team_member_ids` | Sales Team Members | one to many | `crm.team.member` | inverse field `user_id` |
| `sale_team_id` | User Sales Team | many to one | `crm.team` | read only; computed by rule `_compute_sale_team_id` and stored; Help: Main user sales team. Used notably for pipeline, or to set sales team in invoicing or subscription. |
| `livechat_channel_ids` | Livechat Channel | many to many | `im_livechat.channel` | not copied on duplication; association table `im_livechat_channel_im_user` |
| `livechat_username` | Livechat Username | single line text |  | computed by rule `_compute_livechat_username` (not stored); writable through an inverse rule; visible only to groups `im_livechat.im_livechat_group_user,base.group_erp_manager` |
| `livechat_lang_ids` | Livechat Languages | many to many | `res.lang` | computed by rule `_compute_livechat_lang_ids` (not stored); writable through an inverse rule; visible only to groups `im_livechat.im_livechat_group_user,base.group_erp_manager` |
| `livechat_expertise_ids` | Live Chat Expertise | many to many | `im_livechat.expertise` | computed by rule `_compute_livechat_expertise_ids` (not stored); writable through an inverse rule; visible only to groups `im_livechat.im_livechat_group_user,base.group_erp_manager`; Help: When forwarding live chat conversations, the chatbot will prioritize users with matching expertise. |
| `livechat_ongoing_session_count` | Number of Ongoing sessions | integer |  | computed by rule `_compute_livechat_ongoing_session_count` (not stored); visible only to groups `im_livechat.im_livechat_group_user` |
| `livechat_is_in_call` | Livechat Is In Call | boolean |  | computed by rule `_compute_livechat_is_in_call` (not stored); visible only to groups `im_livechat.im_livechat_group_user`; Help: Whether the user is in a call, only available if the user is in a live chat agent |
| `has_access_livechat` | Has access to Livechat | boolean |  | read only; computed by rule `_compute_has_access_livechat` (not stored) |
| `property_warehouse_id` | Default Warehouse | many to one | `stock.warehouse` | value is company dependent; must belong to the same company |
| `karma` | Karma | integer |  | computed by rule `_compute_karma` and stored |
| `karma_tracking_ids` | Karma Changes | one to many | `gamification.karma.tracking` | visible only to groups `base.group_system`; inverse field `user_id` |
| `badge_ids` | Badges | one to many | `gamification.badge.user` | not copied on duplication; inverse field `user_id`; extended by packages `hr_gamification` |
| `gold_badge` | Gold badges count | integer |  | computed by rule `_get_user_badge_level` (not stored) |
| `silver_badge` | Silver badges count | integer |  | computed by rule `_get_user_badge_level` (not stored) |
| `bronze_badge` | Bronze badges count | integer |  | computed by rule `_get_user_badge_level` (not stored) |
| `rank_id` | Rank | many to one | `gamification.karma.rank` | indexed (btree_not_null) |
| `next_rank_id` | Next Rank | many to one | `gamification.karma.rank` |  |
| `google_calendar_rtoken` | Google Calendar Rtoken | single line text |  | related through path `res_users_settings_id.google_calendar_rtoken`; visible only to groups `base.group_system` |
| `google_calendar_token` | Google Calendar Token | single line text |  | related through path `res_users_settings_id.google_calendar_token`; visible only to groups `base.group_system` |
| `google_calendar_token_validity` | Google Calendar Token Validity | date and time |  | related through path `res_users_settings_id.google_calendar_token_validity`; visible only to groups `base.group_system` |
| `google_calendar_sync_token` | Google Calendar Sync Token | single line text |  | related through path `res_users_settings_id.google_calendar_sync_token`; visible only to groups `base.group_system` |
| `google_calendar_cal_id` | Google Calendar Cal | single line text |  | related through path `res_users_settings_id.google_calendar_cal_id`; visible only to groups `base.group_system` |
| `google_synchronization_stopped` | Google Synchronization Stopped | boolean |  | related through path `res_users_settings_id.google_synchronization_stopped`; visible only to groups `base.group_system` |
| `employee_ids` | Related employee | one to many | `hr.employee` | restricted by domain `_employee_ids_domain`; inverse field `user_id` |
| `employee_id` | Company employee | many to one | `hr.employee` | computed by rule `_compute_company_employee` (not stored); searchable through a search rule |
| `job_title` | Job Title | single line text |  | related through path `employee_id.job_title` |
| `work_phone` | Work Phone | single line text |  | related through path `employee_id.work_phone` |
| `mobile_phone` | Mobile Phone | single line text |  | related through path `employee_id.mobile_phone` |
| `work_email` | Work Email | single line text |  | related through path `employee_id.work_email` |
| `category_ids` | Employee Tags | many to many |  | related through path `employee_id.category_ids` |
| `work_contact_id` | Work Contact | many to one |  | related through path `employee_id.work_contact_id` |
| `work_location_id` | Work Location | many to one |  | related through path `employee_id.work_location_id` |
| `work_location_name` | Work Location Name | single line text |  | related through path `employee_id.work_location_name` |
| `work_location_type` | Work Location Type | selection |  | related through path `employee_id.work_location_type` |
| `private_street` | Private Street | single line text |  | related through path `employee_id.private_street` |
| `private_street2` | Private Street2 | single line text |  | related through path `employee_id.private_street2` |
| `private_city` | Private City | single line text |  | related through path `employee_id.private_city` |
| `private_state_id` | Private State | many to one |  | related through path `employee_id.private_state_id`; restricted by domain `[('country_id', '=?', private_country_id)]` |
| `private_zip` | Private Zip | single line text |  | related through path `employee_id.private_zip` |
| `private_country_id` | Private Country | many to one |  | related through path `employee_id.private_country_id` |
| `private_phone` | Private Phone | single line text |  | related through path `employee_id.private_phone` |
| `private_email` | Private Email | single line text |  | related through path `employee_id.private_email` |
| `km_home_work` | Km Home Work | integer |  | related through path `employee_id.km_home_work` |
| `employee_bank_account_ids` | Employee's Bank Accounts | many to many | `res.partner.bank` | related through path `employee_id.bank_account_ids` |
| `emergency_contact` | Emergency Contact | single line text |  | related through path `employee_id.emergency_contact` |
| `emergency_phone` | Emergency Phone | single line text |  | related through path `employee_id.emergency_phone` |
| `visa_expire` | Visa Expire | date |  | related through path `employee_id.visa_expire` |
| `additional_note` | Additional Note | multi line text |  | related through path `employee_id.additional_note` |
| `barcode` | Barcode | single line text |  | related through path `employee_id.barcode` |
| `pin` | Pin | single line text |  | related through path `employee_id.pin` |
| `employee_count` | Employee Count | integer |  | computed by rule `_compute_employee_count` (not stored) |
| `employee_resource_calendar_id` | Employee's Working Hours | many to one |  | read only; related through path `employee_id.resource_calendar_id` |
| `bank_account_ids` | Bank Account | many to many |  | related through path `employee_id.bank_account_ids` |
| `create_employee` | Technical field, whether to create an employee | boolean |  | default ; not copied on duplication |
| `create_employee_id` | Technical field, bind user to this employee on create | many to one | `hr.employee` | not copied on duplication |
| `is_system` | Is System | boolean |  | computed by rule `_compute_is_system` (not stored) |
| `is_hr_user` | Is Human resources User | boolean |  | computed by rule `_compute_is_hr_user` (not stored) |
| `goal_ids` | Goal | one to many | `gamification.goal` | inverse field `user_id` |
| `leave_date_to` | Leave Date To | date |  | related through path `employee_id.leave_date_to` |
| `monday_location_id` | Mondays | many to one | `hr.work.location` | related through path `employee_id.monday_location_id` |
| `tuesday_location_id` | Tuesdays | many to one | `hr.work.location` | related through path `employee_id.tuesday_location_id` |
| `wednesday_location_id` | Wednesdays | many to one | `hr.work.location` | related through path `employee_id.wednesday_location_id` |
| `thursday_location_id` | Thursdays | many to one | `hr.work.location` | related through path `employee_id.thursday_location_id` |
| `friday_location_id` | Fridays | many to one | `hr.work.location` | related through path `employee_id.friday_location_id` |
| `saturday_location_id` | Saturdays | many to one | `hr.work.location` | related through path `employee_id.saturday_location_id` |
| `sunday_location_id` | Sundays | many to one | `hr.work.location` | related through path `employee_id.sunday_location_id` |
| `website_id` | Website | many to one | `website` | related through path `partner_id.website_id` and stored |
| `favorite_project_ids` | Favorite Projects | many to many | `project.project` | not copied on duplication; association table `project_favorite_user_rel` |
| `last_lunch_location_id` | Last Lunch Location | many to one | `lunch.location` | not copied on duplication; visible only to groups `lunch.group_lunch_user` |
| `favorite_lunch_product_ids` | Favorite Lunch Product | many to many | `lunch.product` | not copied on duplication; visible only to groups `lunch.group_lunch_user`; association table `lunch_product_favorite_user_rel` |
| `odoobot_state` | OdooBot Status | selection |  | read only |
| `odoobot_failed` | Odoobot Failed | boolean |  | read only |
| `microsoft_calendar_rtoken` | Microsoft Refresh Token | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `microsoft_calendar_token` | Microsoft User token | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `microsoft_calendar_token_validity` | Microsoft Token Validity | date and time |  | not copied on duplication |
| `microsoft_calendar_sync_token` | Microsoft Calendar Sync Token | single line text |  | related through path `res_users_settings_id.microsoft_calendar_sync_token`; visible only to groups `base.group_system` |
| `microsoft_synchronization_stopped` | Microsoft Synchronization Stopped | boolean |  | related through path `res_users_settings_id.microsoft_synchronization_stopped`; visible only to groups `base.group_system` |
| `microsoft_last_sync_date` | Microsoft Last Sync Date | date and time |  | related through path `res_users_settings_id.microsoft_last_sync_date`; visible only to groups `base.group_system` |
| `employee_skill_ids` | Employee Skill | one to many |  | related through path `employee_id.employee_skill_ids` |
| `create_date` | Create Date | date and time |  | read only; indexed |

## Selection values

### `role` (Role)

| Value | Label |
|---|---|
| `group_user` | User |
| `group_system` | Administrator |

### `notification_type` (Notification)

| Value | Label |
|---|---|
| `email` | By Emails |
| `inbox` | In Odoo |

### `manual_im_status` (IM status manually set by the user)

| Value | Label |
|---|---|
| `away` | Away |
| `busy` | Do Not Disturb |
| `offline` | Offline |

### `outgoing_mail_server_type` (Outgoing Mail Server Type)

| Value | Label |
|---|---|
| `default` | Default |
| `gmail` | Gmail |
| `outlook` | Outlook |

### `state` (Status)

| Value | Label |
|---|---|
| `new` | Invited |
| `active` | Confirmed |

### `calendar_default_privacy` (Calendar Default Privacy)

| Value | Label |
|---|---|
| `public` | Public by default |
| `private` | Private by default |
| `confidential` | Internal users only |

### `odoobot_state` (OdooBot Status)

| Value | Label |
|---|---|
| `not_initialized` | Not initialized |
| `onboarding_emoji` | Onboarding emoji |
| `onboarding_attachement` | Onboarding attachment |
| `onboarding_command` | Onboarding command |
| `onboarding_ping` | Onboarding ping |
| `onboarding_canned` | Onboarding canned |
| `idle` | Idle |
| `disabled` | Disabled |

## State fields

State machine fields of this entity: `manual_im_status`, `state`, `odoobot_state`. Transitions are specified in the domain documents.

## Database constraints and indexes (4)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_login_key` | Constraint | `UNIQUE (login)` | You can not have two users with the same login! | `base` |
| `_notification_type` | Constraint | `CHECK (notification_type = 'email' OR NOT share)` | Only internal user can receive notifications in Odoo | `mail` |
| `_uniq_users_oauth_provider_oauth_uid` | Constraint | `unique(oauth_provider_id, oauth_uid)` | OAuth UID must be unique per provider | `auth_oauth` |
| `_login_key` | Constraint | `unique (login, website_id)` | You can not have two users with the same login! | `website` |

## Operations (263)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_company_domain` | validation | self, companies | `base` |  |  |
| `SELF_READABLE_FIELDS` | operation | self | `auth_oauth`, `auth_passkey`, `auth_totp`, `base`, `calendar`, `hr_holidays`, `hr_homeworking`, `hr`, `im_livechat`, `mail_bot`, `mail`, `sale_stock`, `website_profile` |  | The list of fields a user can read on their own user record. In order to add fields, please override this property on model extensions. |
| `SELF_WRITEABLE_FIELDS` | operation | self | `base`, `calendar`, `hr_homeworking`, `hr`, `im_livechat`, `mail`, `sale_stock`, `website_profile` |  | The list of fields a user can write on their own user record. In order to add fields, please override this property on model extensions. |
| `_self_accessible_fields` | internal rule | self | `base` | model | Readable and writable fields by portal users. |
| `_default_groups` | preparation rule | self | `base` |  | Default groups for employees  All the groups of the Default User Group |
| `_default_view_group_hierarchy` | preparation rule | self | `base` |  |  |
| `init` | lifecycle override | self | `auth_totp`, `base` |  |  |
| `_set_password` | internal rule | self | `auth_password_policy`, `base` |  |  |
| `_set_encrypted_password` | internal rule | self, uid, pw | `base` |  |  |
| `_rpc_api_keys_only` | internal rule | self | `auth_totp_mail`, `auth_totp`, `base` |  | To be overridden if RPC access needs to be restricted to API keys, e.g. for 2FA |
| `_check_credentials` | validation | self, credential, env | `auth_ldap`, `auth_oauth`, `auth_passkey`, `auth_totp_mail`, `auth_totp`, `base`, `website_sale_wishlist` |  | Validates the current user's password.  Override this method to plug additional authentication methods.  Overrides should:  * call ``super`` to delegate to parents for credentials-checking * catch :class:`~odoo.exceptions.AccessDenied` and perform their   own checking * (re)raise :class:`~odoo.exceptions.AccessDenied` if the   credentials are still invalid according to their own   validation method * return the ``auth_info``  When trying to check for credentials validity, call :meth:`_check_credentials` instead.  Credentials are considered to be untrusted user input, for more information pleas |
| `_compute_email_domain_placeholder` | computation | self | `base` | depends_context: `uid` |  |
| `_compute_password` | computation | self | `base` |  |  |
| `_set_new_password` | internal rule | self | `base` |  |  |
| `_compute_role` | computation | self | `base` | depends: `group_ids` |  |
| `_onchange_role` | on change | self | `base` | onchange: `role` |  |
| `_compute_all_group_ids` | computation | self | `base` | depends: `group_ids.all_implied_ids` |  |
| `_search_all_group_ids` | search rule | self, operator, value | `base` |  |  |
| `_compute_signature` | computation | self | `base` | depends: `name` |  |
| `_compute_share` | computation | self | `base` | depends: `all_group_ids` |  |
| `_compute_companies_count` | computation | self | `base` | depends: `company_id` |  |
| `_compute_tz_offset` | computation | self | `base` | depends: `tz` |  |
| `_compute_accesses_count` | computation | self | `base` | depends: `all_group_ids` |  |
| `_compute_res_users_settings_id` | computation | self | `base` | depends: `res_users_settings_ids` |  |
| `_search_res_users_settings_id` | search rule | self, operator, operand | `base` | model |  |
| `on_change_login` | on change | self | `base` | onchange: `login` |  |
| `onchange_parent_id` | on change | self | `base` | onchange: `parent_id` |  |
| `_check_user_company` | validation | self | `base` | constrains: `company_id`, `company_ids`, `active` |  |
| `_check_action_id` | validation | self | `base` | constrains: `action_id` |  |
| `_check_disjoint_groups` | validation | self | `base`, `website` | constrains: `group_ids` | We check that no users are both portal and users (same with public). This could typically happen because of implied groups. |
| `_check_at_least_one_administrator` | validation | self | `base` | constrains: `group_ids` |  |
| `onchange` | lifecycle override | self, values, field_names, fields_spec | `base` |  |  |
| `read` | lifecycle override | self, fields, load | `base` |  |  |
| `_has_field_access` | internal rule | self, field, operation | `base` |  |  |
| `create` | lifecycle override | self, vals_list | `auth_signup`, `base`, `calendar`, `digest`, `gamification`, `hr_holidays`, `hr`, `mail`, `project`, `website_slides` | model_create_multi | Automatically subscribe employee users to default digest if activated |
| `write` | lifecycle override | self, vals | `auth_signup`, `auth_totp_mail`, `base`, `calendar`, `gamification`, `hr`, `im_livechat`, `mail`, `resource`, `website_slides` |  | Forbid the calendar default privacy update from different users for keeping private events secured. |
| `_unlink_except_master_data` | internal rule | self | `base` | ondelete |  |
| `name_search` | operation | self, name, domain, operator, limit | `base`, `web` | model |  |
| `_search_display_name` | search rule | self, operator, value | `base` | model |  |
| `copy_data` | lifecycle override | self, default | `base` |  |  |
| `context_get` | operation | self | `base` | model |  |
| `_get_company_ids` | preparation rule | self | `base` |  |  |
| `action_get` | user action | self | `base`, `hr` | model |  |
| `_get_invalidation_fields` | preparation rule | self | `base` | model |  |
| `_update_last_login` | internal rule | self | `base` | model |  |
| `_get_login_domain` | preparation rule | self, login | `base`, `website` | model |  |
| `_get_email_domain` | preparation rule | self, email | `base`, `website` | model |  |
| `_get_login_order` | preparation rule | self | `base`, `website` | model |  |
| `_login` | internal rule | self, credential, user_agent_env | `auth_ldap`, `auth_passkey`, `base` |  |  |
| `authenticate` | operation | self, credential, user_agent_env | `auth_totp_mail`, `base`, `website` |  | Verifies and returns the user ID corresponding to the given ``credential``, or False if there was no matching user.  :param dict credential: a dictionary where the `type` key defines the authentication method and     additional keys are passed as required per authentication method.     For example:     - { 'type': 'password', 'login': 'username', 'password': '123456' }     - { 'type': 'webauthn', 'webauthn_response': '{json data}' } :param dict user_agent_env: environment dictionary describing any     relevant environment attributes :return: auth_info :rtype: dict |
| `_check_uid_passwd` | validation | self, uid, passwd | `base` | model | Verifies that the given (uid, password) is authorized and raise an exception if it is not. |
| `_get_session_token_fields` | preparation rule | self | `auth_oauth`, `auth_passkey`, `auth_totp`, `base` |  |  |
| `_get_session_token_query_params` | preparation rule | self | `auth_passkey`, `base` |  |  |
| `_compute_session_token` | computation | self, sid | `base` |  | Compute a session token given a session id and a user id |
| `_session_token_get_values` | internal rule | self | `base` |  |  |
| `_session_token_hash_compute` | internal rule | self, sid, field_values | `base` |  |  |
| `_legacy_session_token_hash_compute` | internal rule | self, sid | `base` |  |  |
| `change_password` | operation | self, old_passwd, new_passwd | `auth_ldap`, `auth_totp`, `base` | model | Change current user password. Old password must be provided explicitly to prevent hijacking an existing user session, or for cases where the cleartext password is not used to authenticate requests.  :return: True :raise: odoo.exceptions.AccessDenied when old password is wrong :raise: odoo.exceptions.UserError when new password is not set or empty |
| `_change_password` | internal rule | self, new_passwd | `base` |  |  |
| `_deactivate_portal_user` | internal rule | self, **post | `base`, `mail`, `phone_validation` |  | Try to remove the current portal user.  This is used to give the opportunity to portal users to de-activate their accounts. Indeed, as the portal users can easily create accounts, they will sometimes wish it removed because they don't use this Odoo portal anymore.  Before this feature, they would have to contact the website or the support to get their account removed, which could be tedious. |
| `preference_save` | operation | self | `base` |  |  |
| `action_change_password_wizard` | user action | self | `base` |  |  |
| `preference_change_password` | operation | self | `base` |  |  |
| `api_key_wizard` | operation | self | `base` |  |  |
| `action_revoke_all_devices` | user action | self | `base` |  |  |
| `_action_revoke_all_devices` | internal rule | self | `base` |  |  |
| `has_groups` | operation | self, group_spec | `base` | readonly | Return whether user ``self`` satisfies the given group restrictions ``group_spec``, i.e., whether it is member of at least one of the groups, and is not a member of any of the groups preceded by ``!``.  Note that the group ``"base.group_no_one"`` is only effective in debug mode, just like method :meth:`~.has_group` does.  :param str group_spec: comma-separated list of fully-qualified group     external IDs, optionally preceded by ``!``.     Example:``"base.group_user,base.group_portal,!base.group_system"``. |
| `has_group` | operation | self, group_ext_id | `base` | readonly | Return whether user ``self`` belongs to the given group (given by its fully-qualified external ID).  Note that the group ``"base.group_no_one"`` is only effective in debug mode: the method returns ``True`` if the user belongs to the group and the current request is in debug mode. |
| `_has_group` | internal rule | self, group_ext_id | `base` |  | Return whether user ``self`` belongs to the given group.  :param str group_ext_id: external ID (XML ID) of the group.    Must be provided in fully-qualified form (``module.ext_id``), as there    is no implicit module to use.. :return: True if user ``self`` is a member of the group with the    given external ID (XML ID), else False. |
| `_get_group_ids` | preparation rule | self | `base` |  | Return ``self``'s group ids (as a tuple). |
| `_action_show` | internal rule | self | `base` |  | If self is a singleton, directly access the form view. If it is a recordset, open a list view |
| `action_show_groups` | user action | self | `base` |  |  |
| `action_show_accesses` | user action | self | `base` |  |  |
| `action_show_rules` | user action | self | `base` |  |  |
| `_is_internal` | internal rule | self | `base` |  |  |
| `_is_portal` | internal rule | self | `base` |  |  |
| `_is_public` | internal rule | self | `base` |  |  |
| `_is_system` | internal rule | self | `base` |  |  |
| `_is_admin` | internal rule | self | `base` |  |  |
| `_is_superuser` | internal rule | self | `base` |  |  |
| `get_company_currency_id` | operation | self | `base` | model |  |
| `_crypt_context` | internal rule | self | `base` |  | Passlib CryptContext instance used to encrypt and verify passwords. Can be overridden if technical, legal or political matters require different kdfs than the provided default.  The work factor of the default KDF can be configured using the ``password.hashing.rounds`` ICP. |
| `_assert_can_auth` | internal rule | self, user | `base` |  | Checks that the current environment even allows the current auth request to happen.  The baseline implementation is a simple linear login cooldown: after a number of failures trying to log-in, the user (by login) is put on cooldown. During the cooldown period, login *attempts* are ignored and logged.  :param user: user id or login, for logging purpose  .. warning::      The login counter is not shared between workers and not     specifically thread-safe, the feature exists mostly for     rate-limiting on large number of login attempts (brute-forcing     passwords) so that should not be much of |
| `_on_login_cooldown` | internal rule | self, failures, previous | `base` |  | Decides whether the user trying to log in is currently "on cooldown" and not even allowed to attempt logging in.  The default cooldown function simply puts the user on cooldown for <login_cooldown_duration> seconds after each failure following the <login_cooldown_after>th (0 to disable).  Can be overridden to implement more complex backoff strategies, or e.g. wind down or reset the cooldown period as the previous failure recedes into the far past.  :param int failures: number of recorded failures (since last success) :param previous: timestamp of previous failure :type previous:  datetime.date |
| `_register_hook` | internal rule | self | `base` |  |  |
| `_mfa_type` | internal rule | self | `auth_totp_mail`, `auth_totp`, `base` |  | If an MFA method is enabled, returns its type as a string. |
| `_mfa_url` | internal rule | self | `auth_totp_mail`, `auth_totp`, `base` |  | If an MFA method is enabled, returns the URL for its second step. |
| `fields_get` | lifecycle override | self, allfields, attributes | `base` | model |  |
| `_get_view_postprocessed` | preparation rule | self, view, arch, **options | `base` |  |  |
| `new` | operation | self, values, origin, ref | `base` | model |  |
| `_on_webclient_bootstrap` | internal rule | self | `mail_bot`, `web` |  |  |
| `_should_captcha_login` | internal rule | self, credential | `web` |  |  |
| `web_create_users` | operation | self, emails | `auth_signup`, `base_setup` | model |  |
| `_bus_channel` | internal rule | self | `bus` |  |  |
| `_compute_tour_enabled` | computation | self | `web_tour` | depends: `create_date` |  |
| `switch_tour_enabled` | operation | self, val | `web_tour` | model |  |
| `unlink` | lifecycle override | self | `mail` |  |  |
| `_unsubscribe_from_non_public_channels` | internal rule | self | `mail` |  | This method un-subscribes users from group restricted channels. Main purpose of this method is to prevent sending internal communication to archived / deleted users. |
| `_init_messaging` | internal rule | self, store | `mail` |  |  |
| `_init_store_data` | internal rule | self, store | `crm_livechat`, `im_livechat`, `mail` | model | Initialize the store of the user. |
| `_compute_has_external_mail_server` | computation | self | `mail` |  |  |
| `_compute_notification_type` | computation | self | `mail` | depends: `share`, `all_group_ids` |  |
| `_compute_is_out_of_office` | computation | self | `mail` | depends: `out_of_office_from`, `out_of_office_to` | Out-of-office is considered as activated once out_of_office_from is set in the past. "To" is not mandatory, as users could simply deactivate it when coming back if the leave timerange is unknown. |
| `_compute_im_status` | computation | self | `hr_holidays`, `hr_homeworking`, `mail` | depends: `manual_im_status`, `presence_ids.status` |  |
| `_inverse_notification_type` | inverse computation | self | `mail` |  |  |
| `_compute_can_edit_role` | computation | self | `mail` | depends_context: `uid` |  |
| `_compute_outgoing_mail_server_id` | computation | self | `mail` | depends: `email` |  |
| `action_archive` | lifecycle override | self | `mail`, `sales_team` |  |  |
| `_notify_security_setting_update` | internal rule | self, subject, content, mail_values, **kwargs | `mail` |  | This method is meant to be called whenever a sensitive update is done on the user's account. It will send an email to the concerned user warning him about this change and making some security suggestions.  :param str subject: The subject of the sent email (e.g: 'Security Update: Password Changed') :param str content: The text to embed within the email template (e.g: 'Your password has been changed') :param kwargs: 'suggest_password_reset' key:     Whether or not to suggest the end-user to reset     his password in the email sent.     Defaults to True. |
| `_notify_security_setting_update_prepare_values` | internal rule | self, content, **kwargs | `auth_totp_mail`, `mail` |  | "Prepare rendering values for the 'mail.account_security_alert' qweb template. |
| `_get_portal_access_update_body` | preparation rule | self, access_granted | `mail` |  |  |
| `_get_activity_groups` | preparation rule | self | `calendar`, `contacts`, `mail`, `mass_mailing_sms`, `mass_mailing`, `project_todo` | model | Update the systray icon of res.partner activities to use the contact application one instead of base icon. |
| `_get_store_avatar_card_fields` | preparation rule | self, target | `mail` |  |  |
| `_gc_personal_mail_servers` | background operation | self | `mail` | autovacuum | In case the user change their email, we need to delete the old personal servers. |
| `_get_mail_server_values` | preparation rule | self, server_type | `google_gmail`, `mail`, `microsoft_outlook` | model |  |
| `action_setup_outgoing_mail_server` | user action | self, server_type | `mail` | model | Configure the outgoing mail servers. |
| `action_test_outgoing_mail_server` | user action | self | `mail` | model |  |
| `_get_mail_server_setup_end_action` | preparation rule | self, smtp_server | `google_gmail`, `mail`, `microsoft_outlook` | model |  |
| `_search_state` | search rule | self, operator, value | `auth_signup` |  |  |
| `_compute_state` | computation | self | `auth_signup` |  |  |
| `signup` | operation | self, values, token | `auth_signup` | model | signup a user, to either: - create a new user (no token), or - create a user for a partner (with token, but no user for partner), or - change the password of a user (with token, and existing user). :param values: a dictionary with field values that are written on user :param token: signup token (optional) :return: (dbname, login, password) for the signed up user |
| `_get_signup_invitation_scope` | preparation rule | self | `auth_signup`, `website` | model |  |
| `_signup_create_user` | internal rule | self, values | `auth_signup`, `website` | model | signup a new user using the template user |
| `_notify_inviter` | internal rule | self | `auth_signup` |  |  |
| `_create_user_from_template` | internal rule | self, values | `auth_signup` |  |  |
| `reset_password` | operation | self, login | `auth_signup` |  | retrieve the user corresponding to login (login or email), and reset their password |
| `action_reset_password` | user action | self | `auth_signup` |  |  |
| `_action_reset_password` | internal rule | self, signup_type | `auth_signup` |  | create signup token for each user, and send their signup url by email |
| `send_unregistered_user_reminder` | operation | self, after_days, batch_size | `auth_signup` |  |  |
| `_ondelete_signup_cancel` | internal rule | self | `auth_signup` | ondelete |  |
| `copy` | lifecycle override | self, default | `auth_signup` |  |  |
| `_set_empty_password` | internal rule | self | `auth_ldap` |  |  |
| `_compute_has_oauth_access_token` | computation | self | `auth_oauth` | depends: `oauth_access_token` |  |
| `remove_oauth_access_token` | operation | self | `auth_oauth` |  |  |
| `_auth_oauth_rpc` | internal rule | self, endpoint, access_token | `auth_oauth` |  |  |
| `_auth_oauth_validate` | internal rule | self, provider, access_token | `auth_oauth` | model | return the validation data corresponding to the access token |
| `_generate_signup_values` | internal rule | self, provider, validation, params | `auth_oauth` | model |  |
| `_auth_oauth_signin` | internal rule | self, provider, validation, params | `auth_oauth` | model | retrieve and sign in the user corresponding to provider and validated access token :param provider: oauth provider id (int) :param validation: result of validation of access token (dict) :param params: oauth parameters (dict) :return: user login (str) :raise: AccessDenied if signin failed  This method can be overridden to add alternative signin methods. |
| `auth_oauth` | operation | self, provider, params | `auth_oauth` | model |  |
| `action_create_passkey` | user action | self | `auth_passkey` |  |  |
| `get_password_policy` | operation | self | `auth_password_policy` | model |  |
| `_check_password_policy` | validation | self, passwords | `auth_password_policy` |  |  |
| `_compute_totp_enabled` | computation | self | `auth_totp` | depends: `totp_secret` |  |
| `_totp_try_setting` | internal rule | self, secret, code | `auth_totp` |  |  |
| `_totp_rate_limit` | internal rule | self, limit_type | `auth_totp` |  |  |
| `_totp_rate_limit_purge` | internal rule | self, limit_type | `auth_totp` |  |  |
| `action_totp_disable` | user action | self | `auth_totp` |  |  |
| `action_totp_enable_wizard` | user action | self | `auth_totp` |  |  |
| `revoke_all_devices` | operation | self | `auth_totp` |  |  |
| `_revoke_all_devices` | internal rule | self | `auth_totp` |  |  |
| `_compute_totp_secret` | computation | self | `auth_totp` |  |  |
| `_inverse_token` | inverse computation | self | `auth_totp` |  |  |
| `_totp_enable_search` | internal rule | self, operator, value | `auth_totp` |  |  |
| `_notify_security_new_connection` | internal rule | self, auth_info | `auth_totp_mail` |  |  |
| `action_open_my_account_settings` | user action | self | `auth_totp_mail` |  |  |
| `get_totp_invite_url` | operation | self | `auth_totp_mail`, `auth_totp_portal` |  |  |
| `action_totp_invite` | user action | self | `auth_totp_mail` |  |  |
| `_get_totp_mail_key` | preparation rule | self | `auth_totp_mail` |  |  |
| `_get_totp_mail_code` | preparation rule | self | `auth_totp_mail` |  |  |
| `_send_totp_mail_code` | internal rule | self | `auth_totp_mail` |  |  |
| `_get_auth_methods` | preparation rule | self | `auth_timeout` |  | Return the list of authentication methods available to the user.  This includes passkeys (WebAuthn), TOTP (app or mail), and password, depending on the user's configured credentials and MFA policy.  :return: A list of enabled authentication method types (e.g., ["webauthn", "totp", "password"]). :rtype: list[str] |
| `_get_lock_timeouts` | preparation rule | self | `auth_timeout` |  | Return the user's configured session and inactivity timeouts.  Delegates to the group-level `_get_lock_timeouts`, using the user's group membership to determine applicable timeout settings.  :return: A dictionary of timeout types and values, as defined by `_get_lock_timeouts` on groups. :rtype: dict |
| `_get_lock_timeout_inactivity` | preparation rule | self | `auth_timeout` |  | Return the shortest applicable inactivity timeout for the user.  Extracts the first (i.e., shortest) timeout from the "lock_timeout_inactivity" entry in the user's timeout configuration, if present.  :return: Inactivity timeout in seconds, or None if not configured. :rtype: float or None |
| `_can_import_remote_urls` | internal rule | self | `base_import` |  | Hook to decide whether the current user is allowed to import images via URL (as such an import can DOS a worker). By default, allows the administrator group.  :rtype: bool |
| `get_selected_calendars_partner_ids` | operation | self, include_user | `calendar` |  | Retrieves the partner IDs of the attendees selected in the calendar view.  :param bool include_user: Determines whether to include the current user's partner ID in the results. :return: A list of integer IDs representing the partners selected in the calendar view.          If 'include_user' is True, the list will also include the current user's partner ID. :rtype: list |
| `_default_user_calendar_default_privacy` | preparation rule | self | `calendar` | model | Get the calendar default privacy from the Default User Template, set public as default. |
| `_compute_calendar_default_privacy` | computation | self | `calendar` | depends: `res_users_settings_id.calendar_default_privacy` | Compute the calendar default privacy of the users, pointing to its ResUsersSettings. When any user doesn't have its setting from ResUsersSettings defined, fallback to Default User Template's. |
| `_inverse_calendar_res_users_settings` | inverse computation | self | `calendar` |  | Updates the values of the calendar fields in 'res_users_settings_ids' to have the same values as their related fields in 'res.users'. If there is no 'res.users.settings' record for the user, then the record is created. |
| `_get_user_calendar_configuration_fields` | preparation rule | self | `calendar` | model | Return the list of configurable fields for the user related to the res.users.settings table. |
| `_systray_get_calendar_event_domain` | internal rule | self | `calendar` |  |  |
| `check_calendar_credentials` | operation | self | `calendar`, `google_calendar`, `microsoft_calendar` | model |  |
| `check_synchronization_status` | operation | self | `calendar`, `google_calendar`, `microsoft_calendar` |  |  |
| `_has_any_active_synchronization` | internal rule | self | `calendar`, `google_calendar`, `microsoft_calendar` |  | Overridable method for checking if user has any synchronization active in inherited modules.  :return: boolean indicating if any synchronization is active. |
| `_compute_crm_team_ids` | computation | self | `sales_team` | depends: `crm_team_member_ids.active` |  |
| `_search_crm_team_ids` | search rule | self, operator, value | `sales_team` |  |  |
| `_compute_sale_team_id` | computation | self | `sales_team` | depends: `crm_team_member_ids.crm_team_id`, `crm_team_member_ids.create_date`, `crm_team_member_ids.active` |  |
| `_compute_display_name` | computation | self | `crm`, `hr_holidays` | depends_context: `crm_formatted_display_name_team`, `formatted_display_name`; depends: `leave_date_to`; depends_context: `formatted_display_name` |  |
| `_compute_livechat_is_in_call` | computation | self | `im_livechat` | depends: `livechat_channel_ids`, `is_in_call` |  |
| `_compute_livechat_ongoing_session_count` | computation | self | `im_livechat` | depends_context: `im_livechat_channel_id`; depends: `livechat_channel_ids.channel_ids.livechat_end_dt`, `partner_id` |  |
| `_compute_livechat_username` | computation | self | `im_livechat` | depends: `res_users_settings_id.livechat_username` |  |
| `_inverse_livechat_username` | inverse computation | self | `im_livechat` |  |  |
| `_compute_livechat_lang_ids` | computation | self | `im_livechat` | depends: `res_users_settings_id.livechat_lang_ids` |  |
| `_inverse_livechat_lang_ids` | inverse computation | self | `im_livechat` |  |  |
| `_compute_livechat_expertise_ids` | computation | self | `im_livechat` | depends: `res_users_settings_id.livechat_expertise_ids` |  |
| `_inverse_livechat_expertise_ids` | inverse computation | self | `im_livechat` |  |  |
| `_compute_has_access_livechat` | computation | self | `im_livechat` | depends: `group_ids` |  |
| `_get_default_warehouse_id` | preparation rule | self | `sale_stock`, `stock` |  |  |
| `_compute_karma` | computation | self | `gamification` | depends: `karma_tracking_ids.new_value` |  |
| `_get_user_badge_level` | computation | self | `gamification` | depends: `badge_ids` | Return total badge per level of users TDE CLEANME: shouldn't check type is forum ? |
| `_add_karma` | internal rule | self, gain, source, reason | `gamification` |  |  |
| `_add_karma_batch` | internal rule | self, values_per_user | `gamification` |  |  |
| `_get_user_ids_ranked_by_karma` | preparation rule | self, user_domain, from_date, to_date, limit, offset | `gamification` | model | Return the list of user_ids satisfying the domain, sorted by their karma gain during the specified period.  This method is designed for the website leaderboard pagination. It computes the ranking based on the sum of gamification.karma.tracking values within the given dates. |
| `_get_tracking_karma_gain_position` | preparation rule | self, user_domain, from_date, to_date | `gamification` |  | Get absolute position in term of gained karma for users. First a ranking of all users is done given a user_domain; then the position of each user belonging to the current record set is extracted.  Example: in website profile, search users with name containing Norbert. Their positions should not be 1 to 4 (assuming 4 results), but their actual position in the karma gain ranking (with example user_domain being karma > 1, website published True).  :param user_domain: general domain (i.e. active, karma > 1, website, ...)   to compute the absolute position of the current record set :param from_date |
| `_get_karma_position` | preparation rule | self, user_domain | `gamification` |  | Get absolute position in term of total karma for users. First a ranking of all users is done given a user_domain; then the position of each user belonging to the current record set is extracted.  Example: in website profile, search users with name containing Norbert. Their positions should not be 1 to 4 (assuming 4 results), but their actual position in the total karma ranking (with example user_domain being karma > 1, website published True).  :param user_domain: general domain (i.e. active, karma > 1, website, ...)   to compute the absolute position of the current record set  :rtype: list[di |
| `_rank_changed` | internal rule | self | `gamification` |  | Method that can be called on a batch of users with the same new rank |
| `_recompute_rank` | internal rule | self | `gamification` |  | The caller should filter the users on karma > 0 before calling this method to avoid looping on every single users  Compute rank of each user by user. For each user, check the rank of this user |
| `_recompute_rank_bulk` | internal rule | self | `gamification` |  | Compute rank of each user by rank. For each rank, check which users need to be ranked |
| `_get_next_rank` | preparation rule | self | `gamification` |  | For fresh users with 0 karma that don't have a rank_id and next_rank_id yet this method returns the first karma rank (by karma ascending). This acts as a default value in related views.  TDE FIXME in post-12.4: make next_rank_id a non-stored computed field correctly computed |
| `get_gamification_redirection_data` | operation | self | `gamification`, `website_forum`, `website_slides` |  | Hook for other modules to add redirect button(s) in new rank reached mail Must return a list of dictionnary including url and label. E.g. return [{'url': '/forum', label: 'Go to Forum'}] |
| `action_karma_report` | user action | self | `gamification` |  |  |
| `_get_google_calendar_token` | preparation rule | self | `google_calendar` |  |  |
| `_get_google_sync_status` | preparation rule | self | `google_calendar` |  | Returns the calendar synchronization status (active, paused or stopped). |
| `_check_pending_odoo_records` | validation | self | `google_calendar` |  | Returns True if sync is active and there are records to be synchronized to Google. |
| `_sync_google_calendar` | internal rule | self, calendar_service | `google_calendar` |  |  |
| `_sync_google_calendar_filter_remote_events` | internal rule | self, google_events | `google_calendar` |  | Filter out events coming from google which should not be synced into odoo. |
| `_sync_single_event` | internal rule | self, calendar_service, odoo_event, event_id | `google_calendar` |  |  |
| `_sync_request` | internal rule | self, calendar_service, event_id | `google_calendar` |  |  |
| `_sync_all_google_calendar` | internal rule | self | `google_calendar` | model | Cron job |
| `is_google_calendar_synced` | operation | self | `google_calendar` |  | True if Google Calendar settings are filled (Client ID / Secret) and user calendar is synced meaning we can make API calls, false otherwise. |
| `stop_google_synchronization` | operation | self | `google_calendar` |  |  |
| `restart_google_synchronization` | operation | self | `google_calendar` |  |  |
| `unpause_google_synchronization` | operation | self | `google_calendar` |  |  |
| `pause_google_synchronization` | operation | self | `google_calendar` |  |  |
| `_has_setup_credentials` | internal rule | self | `google_calendar` | model | Checks if both Client ID and Client Secret are defined in the database. |
| `_employee_ids_domain` | internal rule | self | `hr` |  |  |
| `_compute_is_system` | computation | self | `hr` | depends_context: `uid` |  |
| `_compute_is_hr_user` | computation | self | `hr` |  |  |
| `_compute_employee_count` | computation | self | `hr` | depends: `employee_ids` |  |
| `_onchange_private_state_id` | on change | self | `hr` | onchange: `private_state_id` |  |
| `get_views` | operation | self, views, options | `hr` | model |  |
| `get_view` | lifecycle override | self, view_id, view_type, **options | `hr` | model |  |
| `_get_employee_fields_to_sync` | preparation rule | self | `hr_homeworking`, `hr` |  | Get values to sync to the related employee when the User is changed. |
| `_get_personal_info_partner_ids_to_notify` | preparation rule | self, employee | `hr` |  |  |
| `_compute_company_employee` | computation | self | `hr` | depends: `employee_ids`; depends_context: `company` |  |
| `_search_company_employee` | search rule | self, operator, value | `hr` |  |  |
| `action_create_employee` | user action | self | `hr` |  |  |
| `action_open_employees` | user action | self | `hr` |  |  |
| `action_related_contact` | user action | self | `hr` |  |  |
| `get_formview_action` | operation | self, access_uid | `hr` |  | Override this method in order to redirect many2one towards the full user form view incase the user is ERP manager and the request coming from employee form. |
| `_clean_attendance_officers` | internal rule | self | `hr_attendance` |  |  |
| `_get_on_leave_ids` | preparation rule | self, partner | `hr_holidays` | model |  |
| `_clean_leave_responsible_users` | internal rule | self | `hr_holidays` |  |  |
| `_create_recruitment_interviewers` | internal rule | self | `hr_recruitment` |  |  |
| `_remove_recruitment_interviewers` | internal rule | self | `hr_recruitment` |  |  |
| `_check_login` | validation | self | `website` | constrains: `login`, `website_id` | Do not allow two users with the same login without website |
| `website_publish_button` | operation | self | `website` |  |  |
| `_generate_profile_token` | internal rule | self, user_id, email | `website_profile` | model | Return a token for email validation. This token is valid for the day and is a hash based on a (secret) uuid generated by the forum module, the user_id, the email and currently the day (to be updated if necessary). |
| `_send_profile_validation_email` | internal rule | self, **kwargs | `website_profile` |  |  |
| `_process_profile_validation_token` | background operation | self, token, email | `website_profile` |  |  |
| `_onboard_users_into_project` | internal rule | self, users | `project_todo`, `project` |  |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `point_of_sale` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `point_of_sale` | model |  |
| `_load_pos_data_read` | internal rule | self, records, config | `point_of_sale` | model |  |
| `_has_cash_move_permission` | internal rule | self | `point_of_sale` |  |  |
| `_has_cash_delete_permission` | internal rule | self | `point_of_sale` |  |  |
| `_init_odoobot` | internal rule | self | `mail_bot` |  |  |
| `_set_microsoft_auth_tokens` | internal rule | self, access_token, refresh_token, ttl | `microsoft_account` |  |  |
| `_microsoft_calendar_authenticated` | internal rule | self | `microsoft_calendar` |  |  |
| `_get_microsoft_calendar_token` | preparation rule | self | `microsoft_calendar` |  |  |
| `_is_microsoft_calendar_valid` | internal rule | self | `microsoft_calendar` |  |  |
| `_refresh_microsoft_calendar_token` | internal rule | self, service | `microsoft_calendar` |  |  |
| `_get_microsoft_sync_status` | preparation rule | self | `microsoft_calendar` |  | Returns the calendar synchronization status (active, paused or stopped). |
| `_sync_microsoft_calendar` | internal rule | self | `microsoft_calendar` |  |  |
| `_sync_all_microsoft_calendar` | internal rule | self | `microsoft_calendar` | model | Cron job |
| `stop_microsoft_synchronization` | operation | self | `microsoft_calendar` |  |  |
| `restart_microsoft_synchronization` | operation | self | `microsoft_calendar` |  |  |
| `unpause_microsoft_synchronization` | operation | self | `microsoft_calendar` |  |  |
| `pause_microsoft_synchronization` | operation | self | `microsoft_calendar` |  |  |
| `_has_setup_microsoft_credentials` | internal rule | self | `microsoft_calendar` | model | Checks if both Client ID and Client Secret are defined in the database. |
| `_set_ICP_first_synchronization_date` | internal rule | self, now | `microsoft_calendar` |  | Set the first synchronization date as an ICP parameter when applicable (param not defined yet and calendar never synchronized before). This parameter is used for not synchronizing previously created Odoo events and thus avoid spamming invitations for those events. |
| `_generate_onboarding_todo` | internal rule | self | `project_todo` |  |  |
| `_can_manage_unsplash_settings` | internal rule | self | `web_unsplash` |  |  |
| `open_website_url` | operation | self | `website_forum` |  |  |

## Validation and error messages (49)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_set_new_password` | UserError | Please use the change password wizard (in User Preferences or User menu) to change your own password. | `base` |
| `_check_user_company` | ValidationError | Company %(company_name)s is not in the allowed companies for user %(user_name)s (%(company_allowed)s). | `base` |
| `_check_action_id` | ValidationError | The "App Switcher" action cannot be selected as home action. | `base` |
| `_check_action_id` | ValidationError | The "%s" action cannot be selected as home action. | `base` |
| `_check_action_id` | ValidationError | The action "%s" cannot be set as the home action because it requires a record to be selected beforehand. | `base` |
| `_check_disjoint_groups` | ValidationError | User %(user)s cannot be at the same time in exclusive groups %(groups)s. | `base` |
| `_check_at_least_one_administrator` | ValidationError | You must have at least an administrator user. | `base` |
| `write` | UserError | You cannot activate the superuser. | `base` |
| `write` | UserError | You cannot deactivate the user you're currently logged in as. | `base` |
| `_unlink_except_master_data` | UserError | You can not remove the admin user as it is used internally for resources created by the system (updates, module installation, ...) | `base` |
| `_unlink_except_master_data` | UserError | You cannot delete the admin user because it is utilized in various places (such as security configurations,...). Instead, archive it. | `base` |
| `_unlink_except_master_data` | UserError | Deleting the template users is not allowed. Deleting this profile will compromise critical functionalities. | `base` |
| `_unlink_except_master_data` | UserError | Deleting the public user is not allowed. Deleting this profile will compromise critical functionalities. | `base` |
| `_change_password` | UserError | Setting empty passwords is not allowed for security reasons! | `base` |
| `_deactivate_portal_user` | AccessDenied | Only the portal users can delete their accounts. The user(s) %s can not be deleted. | `base` |
| `has_group` | AccessError | You can ony call user.has_group() with your current user. | `base` |
| `_assert_can_auth` | AccessDenied | Too many login failures, please wait a bit before trying again. | `base` |
| `web_create_users` | UserError | You have to install the Discuss application to use this feature. | `base_setup` |
| `action_setup_outgoing_mail_server` | UserError | You are not allowed to create a personal mail server. | `mail` |
| `action_setup_outgoing_mail_server` | UserError | Only internal users can configure a personal mail server. | `mail` |
| `action_setup_outgoing_mail_server` | UserError | Please set your email before connecting your mail server. | `mail` |
| `action_setup_outgoing_mail_server` | UserError | Wrong email address %s. | `mail` |
| `action_setup_outgoing_mail_server` | UserError | Your email address is used by an alias domain, and so you can not create a mail server for it. | `mail` |
| `action_test_outgoing_mail_server` | UserError | You are not allowed to test personal mail servers. | `mail` |
| `action_test_outgoing_mail_server` | UserError | Only internal users can configure personal mail servers. | `mail` |
| `action_test_outgoing_mail_server` | UserError | No mail server configured | `mail` |
| `_signup_create_user` | UserError | Another user is already registered using this email address. | `auth_signup` |
| `action_reset_password` | UserError | Could not contact the mail server, please check your outgoing email server configuration | `auth_signup` |
| `action_reset_password` | UserError | There was an error when trying to deliver your Email, please check your configuration | `auth_signup` |
| `_action_reset_password` | UserError | You cannot perform this action on an archived user. | `auth_signup` |
| `_action_reset_password` | UserError | Cannot send email: user %s has no email address. | `auth_signup` |
| `remove_oauth_access_token` | AccessError | You do not have permissions to remove the access token | `auth_oauth` |
| `_auth_oauth_validate` | AccessDenied | Missing subject identity | `auth_oauth` |
| `_login` | AccessDenied | Unknown passkey | `auth_passkey` |
| `_check_credentials` | AccessDenied | Unknown passkey | `auth_passkey` |
| `_check_credentials` | AccessDenied | e.args[0] | `auth_passkey` |
| `_check_password_policy` | UserError | u'\n\n '.join(failures) | `auth_password_policy` |
| `_check_credentials` | AccessDenied | Verification failed, please double-check the 6-digit code | `auth_totp` |
| `_check_credentials` | AccessDenied | Verification failed, please use the latest 6-digit code | `auth_totp` |
| `_totp_rate_limit` | AccessDenied | description | `auth_totp` |
| `action_totp_enable_wizard` | UserError | Two-factor authentication can only be enabled for yourself | `auth_totp` |
| `action_totp_enable_wizard` | UserError | Two-factor authentication already enabled | `auth_totp` |
| `_check_credentials` | AccessDenied | Verification failed, please double-check the 6-digit code | `auth_totp_mail` |
| `_send_totp_mail_code` | UserError | Cannot send email: user %s has no email address. | `auth_totp_mail` |
| `write` | AccessError | You are not allowed to change the calendar default privacy of another user due to privacy constraints. | `calendar` |
| `action_create_employee` | AccessError | You are not allowed to create an employee because the user does not have access rights for %s | `hr` |
| `_check_login` | ValidationError | You can not have two users with the same login! | `website` |
| `_check_disjoint_groups` | ValidationError | Remove website on related partner before they become internal user. | `website` |
| `_refresh_microsoft_calendar_token` | UserError | error_msg | `microsoft_calendar` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_public` | no | yes | no | no | `base` |
| `base.group_portal` | no | yes | no | no | `base` |
| `base.group_user` | no | yes | no | no | `base` |
| `group_erp_manager` | yes | yes | yes | yes | `base` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| user rule | global (all users) | `['\|', ('share', '=', False), ('company_ids', 'in', company_ids)]` | True | True | True | True |
| portal user access | `[Command.set([ref('base.group_portal')])]` | `[('commercial_partner_id', '=', user.commercial_partner_id.id)]` | True | True | True | True |

## Views (39)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `auth_passkey.auth_passkey_users_form` | xpath | `base.view_users_form` | `auth_passkey_key_ids` | `Add Passkey` |  | `auth_passkey` |
| `auth_passkey.auth_passkey_users_preferences` | xpath | `base.view_users_form_simple_modif` | `auth_passkey_key_ids` | `Add Passkey` |  | `auth_passkey` |
| `auth_signup.res_users_view_form` | xpath | `base.view_users_form` | `state` | `Send an Invitation Email` |  | `auth_signup` |
| `auth_signup.view_users_state_tree` | list | `base.view_users_tree` | `state` |  |  | `auth_signup` |
| `auth_totp.res_users_view_search` | xpath | `base.view_users_search` |  |  | `Two-factor authentication Enabled`, `Two-factor authentication Disabled` | `auth_totp` |
| `auth_totp.view_totp_form` | xpath | `base.view_users_form` | `totp_enabled` | `Enable 2FA`, `action_totp_disable` |  | `auth_totp` |
| `auth_totp.view_totp_field` | xpath | `base.view_users_form_simple_modif` | `totp_enabled` | `Enable 2FA`, `action_totp_disable` |  | `auth_totp` |
| `auth_totp_mail.view_users_form` | xpath | `auth_totp.view_totp_form` |  | `Invite to use 2FA` |  | `auth_totp_mail` |
| `auth_totp_mail.res_users_view_form` | form | `base.view_users_form_simple_modif` |  |  |  | `auth_totp_mail` |
| `base.view_users_simple_form` | form |  | `image_1920`, `avatar_128`, `name`, `email`, `email_domain_placeholder`, `login`, `phone`, `company_id`, `group_ids` |  |  | `base` |
| `base.view_users_form` | form |  | `groups_count`, `accesses_count`, `rules_count`, `partner_id`, `image_1920`, `avatar_128`, `name`, `login`, `email`, `phone`, `partner_id`, `role`, `company_ids`, `company_id`, `group_ids`, `group_ids`, `lang`, `signature`, `tz`, `tz_offset`, `api_key_ids`, `device_ids` | `action_show_groups`, `action_show_accesses`, `action_show_rules`, `%(base.action_view_base_language_install)d`, `Change password`, `Change password`, `Add API Key`, `Log out from all devices` |  | `base` |
| `base.view_users_tree` | list |  | `avatar_128`, `name`, `login`, `lang`, `login_date`, `role` |  |  | `base` |
| `base.view_res_users_kanban` | kanban |  | `active`, `login_date`, `avatar_128`, `name`, `login`, `lang` |  |  | `base` |
| `base.view_users_search` | search |  | `name`, `company_ids`, `share` |  | `Internal Users`, `Portal Users`, `Inactive Users` | `base` |
| `base.view_users_form_simple_modif` | form |  | `image_1920`, `avatar_128`, `name`, `email`, `phone`, `lang`, `signature`, `tz`, `tz_offset`, `api_key_ids`, `device_ids` | `%(base.action_view_base_language_install)d`, `Change password`, `Add API Key`, `Log out from all devices`, `Update Preferences`, `Discard` |  | `base` |
| `calendar.res_users_view_form` | xpath | `base.view_users_form` |  |  |  | `calendar` |
| `calendar.res_users_form_view_calendar_default_privacy` | xpath | `base.view_users_form_simple_modif` | `calendar_default_privacy` |  |  | `calendar` |
| `calendar.res_users_form_view` | xpath | `base.view_users_form` | `calendar_default_privacy` |  |  | `calendar` |
| `gamification.res_users_view_form` | xpath | `base.view_users_form` | `karma` | `action_karma_report` |  | `gamification` |
| `google_calendar.view_users_form` | xpath | `calendar.res_users_view_form` |  |  |  | `google_calendar` |
| `hr.res_users_view_form_simple_modif` | form | `base.view_users_form_simple_modif` |  |  |  | `hr` |
| `hr.view_users_form_simple_modif_resource` | field | `base.view_users_form_simple_modif` | `tz` |  |  | `hr` |
| `hr.res_users_view_form_preferences` | form | `res_users_view_form_simple_modif` |  |  |  | `hr` |
| `hr.view_users_simple_form_inherit_hr` | xpath | `base.view_users_simple_form` | `employee_count` | `action_open_employees` |  | `hr` |
| `hr.view_users_simple_form` | sheet | `base.view_users_simple_form` |  | `Save`, `Cancel` |  | `hr` |
| `hr.res_users_view_form` | xpath | `base.view_users_form` | `share`, `employee_ids`, `employee_id` | `Create employee` |  | `hr` |
| `hr_homeworking.res_users_view_form` | xpath | `hr.res_users_view_form` | `monday_location_id`, `tuesday_location_id`, `wednesday_location_id`, `thursday_location_id`, `friday_location_id`, `saturday_location_id`, `sunday_location_id` |  |  | `hr_homeworking` |
| `hr_homeworking.res_useurs_view_form_profile` | xpath | `hr.res_users_view_form_preferences` | `monday_location_id`, `tuesday_location_id`, `wednesday_location_id`, `thursday_location_id`, `friday_location_id`, `saturday_location_id`, `sunday_location_id` |  |  | `hr_homeworking` |
| `im_livechat.res_users_form_view_simple_modif` | xpath | `base.view_users_form_simple_modif` | `has_access_livechat`, `livechat_username`, `livechat_lang_ids`, `livechat_expertise_ids` |  |  | `im_livechat` |
| `im_livechat.res_users_form_view` | xpath | `base.view_users_form` | `has_access_livechat`, `livechat_username`, `livechat_lang_ids`, `livechat_expertise_ids` |  |  | `im_livechat` |
| `mail.view_users_form_simple_modif_mail` | data | `base.view_users_form_simple_modif` | `notification_type`, `outgoing_mail_server_id`, `outgoing_mail_server_type`, `signature`, `out_of_office_from`, `out_of_office_to`, `out_of_office_message` |  |  | `mail` |
| `mail.view_users_form_mail` | data | `base.view_users_form` | `notification_type`, `outgoing_mail_server_id`, `outgoing_mail_server_type`, `signature`, `out_of_office_from`, `out_of_office_to`, `out_of_office_message` |  |  | `mail` |
| `mail_bot.res_users_view_form` | data | `mail.view_users_form_mail` | `notification_type`, `odoobot_state` |  |  | `mail_bot` |
| `mail_bot_hr.res_users_view_form_simple_modif` | widget | `hr.res_users_view_form_simple_modif` |  |  |  | `mail_bot_hr` |
| `mail_bot_hr.res_users_view_form_preferences` | sheet | `hr.res_users_view_form_preferences` |  |  |  | `mail_bot_hr` |
| `microsoft_calendar.view_users_form` | xpath | `calendar.res_users_view_form` |  |  |  | `microsoft_calendar` |
| `sale_stock.res_users_view_form_preferences` | group | `base.view_users_form_simple_modif` | `property_warehouse_id` |  |  | `sale_stock` |
| `sale_stock.res_users_view_simple_form` | group | `base.view_users_simple_form` | `property_warehouse_id` |  |  | `sale_stock` |
| `sale_stock.res_users_view_form` | group | `base.view_users_form` | `property_warehouse_id` |  |  | `sale_stock` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.action_res_users` | Users | list,kanban,form |  | `{'search_default_filter_no_share': 1, 'is_action_res_users': True}` |  | `base` |
| `base.action_res_users_my` | Change My Preferences | form |  |  | new | `base` |
| `gamification.action_current_rank_users` | Users | list,form | `[('rank_id', '=', active_id)]` |  |  | `gamification` |
| `gamification.action_new_simplified_res_users` | Create User |  |  | `{}` | current | `gamification` |
| `hr.res_users_action_my` | Change my Preferences | form |  |  | new | `hr` |
| `mail.action_res_users_my_fullpage` | Change My Preferences | form |  |  |  | `mail` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `auth_signup.action_send_password_reset_instructions` | Send Password Reset Instructions | code |  | yes |
| `auth_totp.action_disable_totp` | Disable two-factor authentication | code |  | yes |
| `auth_totp_mail.action_invite_totp` | Invite to use two-factor authentication | code |  | yes |
| `auth_totp_mail.action_activate_two_factor_authentication` | Open two-factor authentication configuration | code |  | yes |
| `privacy_lookup.ir_action_server_action_privacy_lookup_user` | Privacy Lookup | code |  | yes |

## Printable reports

| Report | Name | Type | Template | File name rule | Attachment rule |
|---|---|---|---|---|---|
| `point_of_sale.report_user_label` | User Labels | qweb-pdf | `point_of_sale.report_userlabel` |  |  |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `auth_signup.ir_cron_auth_signup_send_pending_user_reminder` | Users: Notify About Unregistered Users | 1 days | `send_unregistered_user_reminder` | 6 |
| `google_calendar.ir_cron_sync_all_cals` | Google Calendar: synchronization | 12 hours | `_sync_all_google_calendar` |  |
| `microsoft_calendar.ir_cron_sync_all_cals` | Outlook: synchronization | 12 hours | `_sync_all_microsoft_calendar` |  |

## Email templates

| Template | Name | Subject |
|---|---|---|
| `auth_signup.set_password_email` | Settings: New User Invite | {{ object.create_uid.name }} from {{ object.company_id.name }} invites you to connect to the system |
| `auth_signup.mail_template_data_unregistered_users` | Settings: Unregistered User Reminder | Reminder for unregistered users |
| `auth_signup.mail_template_user_signup_account_created` | Settings: New Portal Sign Up | Welcome to {{ object.company_id.name }}! |
| `auth_signup.portal_set_password_email` | Settings: New Portal User Invite | Your account at {{ object.company_id.name }} |
| `auth_totp_mail.mail_template_totp_invite` | Settings: 2Fa Invitation | Invitation to activate two-factor authentication on your the system account |
| `auth_totp_mail.mail_template_totp_mail_code` | Settings: 2Fa New Login | Your two-factor authentication code |
| `gamification.mail_template_data_new_rank_reached` | Gamification: New Rank Reached | New rank: {{ object.rank_id.name }} |
| `website_profile.validation_email` | Forum: Email Verification | {{ object.company_id.name }} Profile validation |

Machine-readable definition: `../../../schemas/data/entities/res.users.json`; views: `../../../schemas/interfaces/views/res.users.json`.
