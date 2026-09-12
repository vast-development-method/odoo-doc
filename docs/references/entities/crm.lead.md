# Lead (`crm.lead`)

**Transport name:** `crm.lead`  
**Storage name:** `crm_lead`  
**Kind:** persistent entity (one table)  
**Defined by package:** `crm`  
**Extended by packages:** `iap_crm`, `crm_iap_enrich`, `crm_iap_mine`, `crm_livechat`, `crm_mail_plugin`, `event_crm`, `sale_crm`, `mass_mailing_crm`, `survey_crm`, `website_crm`, `website_crm_iap_reveal`, `website_crm_livechat`, `website_crm_partner_assign`

Description: Lead

## Identity and behavior

- Mixins (classical inheritance): `mail.thread.cc`, `mail.thread.blacklist`, `mail.thread.phone`, `mail.activity.mixin`, `utm.mixin`, `format.address.mixin`, `mail.tracking.duration.mixin`
- Default ordering: `priority desc, id desc`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (95)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Opportunity | single line text |  | required; computed by rule `_compute_name` and stored; indexed (trigram) |
| `user_id` | Salesperson | many to one | `res.users` | default computed dynamically (lambda self: self.env.user); changes are tracked in the message thread; indexed; restricted by domain `[('share', '=', False)]`; must belong to the same company |
| `user_company_ids` | User Company | many to many | `res.company` | computed by rule `_compute_user_company_ids` (not stored); Help: UX: Limit to lead company or all if no company |
| `team_id` | Sales Team | many to one | `crm.team` | computed by rule `_compute_team_id` and stored; changes are tracked in the message thread; indexed; on delete of the target: set null; must belong to the same company; precomputed before insertion |
| `lead_properties` | Properties | properties |  |  |
| `company_id` | Company | many to one | `res.company` | computed by rule `_compute_company_id` and stored; indexed |
| `referred` | Referred By | single line text |  |  |
| `description` | Notes | rich text |  |  |
| `active` | Active | boolean |  | default `True`; changes are tracked in the message thread |
| `type` | Type | selection |  | required; default computed dynamically (lambda self: 'lead' if self.env.user.has_group('crm.group_use_lead') else 'opportunity'); changes are tracked in the message thread; indexed |
| `priority` | Priority | selection |  | default computed dynamically (crm_stage.AVAILABLE_PRIORITIES[0][0]); indexed |
| `stage_id` | Stage | many to one | `crm.stage` | computed by rule `_compute_stage_id` and stored; changes are tracked in the message thread; indexed; not copied on duplication; on delete of the target: restrict; restricted by domain `['\|', ('team_ids', '=', False), ('team_ids', 'in', team_id)]` |
| `stage_id_color` | Stage Color | integer |  | related through path `stage_id.color` |
| `tag_ids` | Tags | many to many | `crm.tag` | association table `crm_tag_rel`; Help: Classify and analyze your lead/opportunity categories like: Training, Service |
| `color` | Color Index | integer |  | default  |
| `expected_revenue` | Expected Revenue | monetary |  | default ; changes are tracked in the message thread; currency taken from `company_currency` |
| `prorated_revenue` | Prorated Revenue | monetary |  | computed by rule `_compute_prorated_revenue` and stored; currency taken from `company_currency` |
| `recurring_revenue` | Recurring Revenues | monetary |  | default ; changes are tracked in the message thread; currency taken from `company_currency` |
| `recurring_plan` | Recurring Plan | many to one | `crm.recurring.plan` |  |
| `recurring_revenue_monthly` | Expected MRR | monetary |  | computed by rule `_compute_recurring_revenue_monthly` and stored; currency taken from `company_currency` |
| `recurring_revenue_monthly_prorated` | Prorated MRR | monetary |  | computed by rule `_compute_recurring_revenue_monthly_prorated` and stored; currency taken from `company_currency` |
| `recurring_revenue_prorated` | Prorated Recurring Revenues | monetary |  | computed by rule `_compute_recurring_revenue_prorated` and stored; currency taken from `company_currency` |
| `company_currency` | Currency | many to one | `res.currency` | computed by rule `_compute_company_currency` (not stored) |
| `date_closed` | Closed Date | date and time |  | read only; not copied on duplication |
| `date_automation_last` | Last Action | date and time |  | read only |
| `date_open` | Assignment Date | date and time |  | read only; computed by rule `_compute_date_open` and stored |
| `day_open` | Days to Assign | float |  | computed by rule `_compute_day_open` and stored |
| `day_close` | Days to Close | float |  | computed by rule `_compute_day_close` and stored |
| `date_last_stage_update` | Last Stage Update | date and time |  | read only; computed by rule `_compute_date_last_stage_update` and stored; indexed |
| `date_conversion` | Conversion Date | date and time |  | read only |
| `date_deadline` | Expected Closing | date |  | Help: Estimate of the date on which the opportunity will be won. |
| `commercial_partner_id` | Customer Company | many to one | `res.partner` | computed by rule `_compute_commercial_partner_id` (not stored); restricted by domain `[('is_company', '=', True)]` |
| `partner_id` | Contact | many to one | `res.partner` | changes are tracked in the message thread; indexed; must belong to the same company; Help: Linked partner (optional). Usually created when converting the lead. You can find a partner by its Name, TIN, Email or Internal Reference. |
| `partner_is_blacklisted` | Partner is blacklisted | boolean |  | read only; related through path `partner_id.is_blacklisted` |
| `contact_name` | Contact Name | single line text |  | computed by rule `_compute_contact_name` and stored; changes are tracked in the message thread; indexed (trigram) |
| `partner_name` | Company Name | single line text |  | computed by rule `_compute_partner_name` and stored; changes are tracked in the message thread; indexed (trigram); Help: The name of the future partner company that will be created while converting the lead into opportunity |
| `function` | Job Position | single line text |  | computed by rule `_compute_function` and stored |
| `email_from` | Email | single line text |  | computed by rule `_compute_email_from` and stored; writable through an inverse rule; changes are tracked in the message thread; indexed (trigram) |
| `email_normalized` | Email Normalized | single line text |  | indexed (trigram) |
| `email_domain_criterion` | Email Domain Criterion | single line text |  | computed by rule `_compute_email_domain_criterion` and stored; indexed (btree_not_null) |
| `phone` | Phone | single line text |  | computed by rule `_compute_phone` and stored; writable through an inverse rule; changes are tracked in the message thread |
| `phone_sanitized` | Phone Sanitized | single line text |  | indexed (btree_not_null) |
| `phone_state` | Phone Quality | selection |  | computed by rule `_compute_phone_state` and stored |
| `email_state` | Email Quality | selection |  | computed by rule `_compute_email_state` and stored |
| `website` | Website | single line text |  | computed by rule `_compute_website` and stored; Help: Website of the contact |
| `lang_id` | Language | many to one | `res.lang` | computed by rule `_compute_lang_id` and stored |
| `lang_code` | Lang Code | single line text |  | related through path `lang_id.code` |
| `lang_active_count` | Lang Active Count | integer |  | computed by rule `_compute_lang_active_count` (not stored) |
| `street` | Street | single line text |  | computed by rule `_compute_partner_address_values` and stored |
| `street2` | Street2 | single line text |  | computed by rule `_compute_partner_address_values` and stored |
| `zip` | Zip | single line text |  | computed by rule `_compute_partner_address_values` and stored |
| `city` | City | single line text |  | computed by rule `_compute_partner_address_values` and stored |
| `state_id` | State | many to one | `res.country.state` | computed by rule `_compute_partner_address_values` and stored; restricted by domain `[('country_id', '=?', country_id)]` |
| `country_id` | Country | many to one | `res.country` | computed by rule `_compute_partner_address_values` and stored |
| `probability` | Probability | float |  | computed by rule `_compute_probabilities` and stored; not copied on duplication; aggregated with avg |
| `automated_probability` | Automated Probability | float |  | read only; computed by rule `_compute_probabilities` and stored |
| `is_automated_probability` | Is automated probability? | boolean |  | computed by rule `_compute_is_automated_probability` (not stored) |
| `won_status` | Won/Lost | selection |  | computed by rule `_compute_won_status` and stored; changes are tracked in the message thread |
| `lost_reason_id` | Lost Reason | many to one | `crm.lost.reason` | changes are tracked in the message thread; indexed; on delete of the target: restrict |
| `calendar_event_ids` | Meetings | one to many | `calendar.event` | inverse field `opportunity_id` |
| `duplicate_lead_ids` | Potential Duplicate Lead | many to many | `crm.lead` | computed by rule `_compute_potential_lead_duplicates` (not stored) |
| `duplicate_lead_count` | Potential Duplicate Lead Count | integer |  | computed by rule `_compute_potential_lead_duplicates` (not stored) |
| `meeting_display_date` | Meeting Display Date | date |  | computed by rule `_compute_meeting_display` (not stored) |
| `meeting_display_label` | Meeting Display Label | single line text |  | computed by rule `_compute_meeting_display` (not stored) |
| `partner_email_update` | Partner Email will Update | boolean |  | computed by rule `_compute_partner_email_update` (not stored) |
| `partner_phone_update` | Partner Phone will Update | boolean |  | computed by rule `_compute_partner_phone_update` (not stored) |
| `is_partner_visible` | Is Partner Visible | boolean |  | computed by rule `_compute_is_partner_visible` (not stored) |
| `campaign_id` | Campaign | many to one |  | on delete of the target: set null |
| `medium_id` | Medium | many to one |  | on delete of the target: set null |
| `source_id` | Source | many to one |  | on delete of the target: set null |
| `reveal_id` | Reveal identifier | single line text |  | indexed (btree_not_null) |
| `iap_enrich_done` | Enrichment done | boolean |  | Help: Whether IAP service for lead enrichment based on email has been performed on this lead. |
| `show_enrich_button` | Allow manual enrich | boolean |  | computed by rule `_compute_show_enrich_button` (not stored) |
| `lead_mining_request_id` | Lead Mining Request | many to one | `crm.iap.lead.mining.request` | indexed (btree_not_null) |
| `origin_channel_id` | Live chat from which the lead was created | many to one | `discuss.channel` | read only; indexed (btree_not_null) |
| `event_lead_rule_id` | Registration Rule | many to one | `event.lead.rule` | indexed (btree_not_null); Help: Rule that created this lead |
| `event_id` | Source Event | many to one | `event.event` | indexed (btree_not_null); Help: Event triggering the rule that created this lead |
| `registration_ids` | Source Registrations | many to many | `event.registration` | visible only to groups `event.group_event_registration_desk`; Help: Registrations triggering the rule that created this lead |
| `registration_count` | # Registrations | integer |  | computed by rule `_compute_registration_count` (not stored); visible only to groups `event.group_event_registration_desk`; Help: Counter for the registrations linked to this lead |
| `sale_amount_total` | Sum of Orders | monetary |  | computed by rule `_compute_sale_data` (not stored); currency taken from `company_currency`; Help: Untaxed Total of Confirmed Orders |
| `quotation_count` | Number of Quotations | integer |  | computed by rule `_compute_sale_data` (not stored) |
| `sale_order_count` | Number of Sale Orders | integer |  | computed by rule `_compute_sale_data` (not stored) |
| `order_ids` | Orders | one to many | `sale.order` | inverse field `opportunity_id` |
| `origin_survey_id` | Survey | many to one | `survey.survey` | indexed (btree_not_null); on delete of the target: set null |
| `visitor_ids` | Web Visitors | many to many | `website.visitor` |  |
| `visitor_page_count` | # Page Views | integer |  | computed by rule `_compute_visitor_page_count` (not stored) |
| `reveal_ip` | internet protocol Address | single line text |  |  |
| `reveal_iap_credits` | in-app purchase Credits | integer |  |  |
| `reveal_rule_id` | Lead Generation Rule | many to one | `crm.reveal.rule` | indexed (btree_not_null) |
| `visitor_sessions_count` | # Sessions | integer |  | computed by rule `_compute_visitor_sessions_count` (not stored); visible only to groups `im_livechat.im_livechat_group_user` |
| `partner_latitude` | Geo Latitude | float |  | precision `[10, 7]` |
| `partner_longitude` | Geo Longitude | float |  | precision `[10, 7]` |
| `partner_assigned_id` | Assigned Partner | many to one | `res.partner` | changes are tracked in the message thread; indexed (btree_not_null); restricted by domain `['\|', ('company_id', '=', False), ('company_id', '=', company_id)]` |
| `partner_declined_ids` | Partner not interested | many to many | `res.partner` | association table `crm_lead_declined_partner` |
| `date_partner_assign` | Partner Assignment Date | date |  | computed by rule `_compute_date_partner_assign` and stored; Help: Last date this case was forwarded/assigned to a partner |

## Selection values

### `type` (Type)

| Value | Label |
|---|---|
| `lead` | Lead |
| `opportunity` | Opportunity |

### `phone_state` (Phone Quality)

| Value | Label |
|---|---|
| `correct` | Correct |
| `incorrect` | Incorrect |

### `email_state` (Email Quality)

| Value | Label |
|---|---|
| `correct` | Correct |
| `incorrect` | Incorrect |

### `won_status` (Won/Lost)

| Value | Label |
|---|---|
| `won` | Won |
| `lost` | Lost |
| `pending` | Pending |

## State fields

State machine fields of this entity: `phone_state`, `email_state`, `won_status`. Transitions are specified in the domain documents.

## Database constraints and indexes (4)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_check_probability` | Constraint | `check(probability >= 0 and probability <= 100)` | The probability of closing the deal should be between 0% and 100%! | `crm` |
| `_user_id_team_id_type_index` | Index | `(user_id, team_id, type)` |  | `crm` |
| `_create_date_team_id_idx` | Index | `(create_date, team_id)` |  | `crm` |
| `_default_order_idx` | Index | `(priority DESC, id DESC) WHERE active IS TRUE` |  | `crm` |

## Operations (157)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_won_validity` | validation | self | `crm` | constrains: `probability`, `stage_id` |  |
| `_compute_user_company_ids` | computation | self | `crm` | depends: `company_id` |  |
| `_compute_company_currency` | computation | self | `crm` | depends: `company_id` |  |
| `_field_to_sql` | internal rule | self, alias, field_expr, query | `crm` |  |  |
| `_compute_team_id` | computation | self | `crm` | depends: `user_id`, `type` | When changing the user, also set a team_id or restrict team id to the ones user_id is member of. |
| `_compute_company_id` | computation | self | `crm` | depends: `user_id`, `team_id`, `partner_id` | Compute company_id coherency. |
| `_compute_stage_id` | computation | self | `crm` | depends: `team_id`, `type` |  |
| `_compute_date_open` | computation | self | `crm` | depends: `user_id` |  |
| `_compute_date_last_stage_update` | computation | self | `crm` | depends: `stage_id` |  |
| `_compute_day_open` | computation | self | `crm` | depends: `create_date`, `date_open` | Compute difference between create date and open date |
| `_compute_day_close` | computation | self | `crm` | depends: `create_date`, `date_closed` | Compute difference between current date and log date |
| `_get_rotting_depends_fields` | preparation rule | self | `crm` |  |  |
| `_get_rotting_domain` | preparation rule | self | `crm` |  |  |
| `_compute_name` | computation | self | `crm` | depends: `partner_id` |  |
| `_compute_commercial_partner_id` | computation | self | `crm` | depends: `partner_id`, `partner_name` |  |
| `_onchange_commercial_partner_id` | on change | self | `crm` | onchange: `commercial_partner_id` |  |
| `_compute_contact_name` | computation | self | `crm` | depends: `partner_id` | compute the new values when partner_id has changed |
| `_compute_partner_name` | computation | self | `crm` | depends: `partner_id` | compute the new values when partner_id has changed |
| `_compute_function` | computation | self | `crm` | depends: `partner_id` | compute the new values when partner_id has changed |
| `_compute_website` | computation | self | `crm` | depends: `partner_id` | compute the new values when partner_id has changed |
| `_compute_lang_id` | computation | self | `crm` | depends: `partner_id` | compute the lang based on partner, erase any value to force the partner one if set. |
| `_compute_lang_active_count` | computation | self | `crm` | depends: `lang_id` |  |
| `_compute_partner_address_values` | computation | self | `crm` | depends: `partner_id` | Sync all or none of address fields |
| `_compute_email_from` | computation | self | `crm` | depends: `partner_id.email` |  |
| `_inverse_email_from` | inverse computation | self | `crm` |  |  |
| `_compute_email_domain_criterion` | computation | self | `crm` | depends: `email_normalized` |  |
| `_compute_phone` | computation | self | `crm` | depends: `partner_id.phone` |  |
| `_inverse_phone` | inverse computation | self | `crm` |  |  |
| `_compute_phone_state` | computation | self | `crm` | depends: `phone`, `country_id.code` |  |
| `_compute_email_state` | computation | self | `crm` | depends: `email_from` |  |
| `_compute_is_automated_probability` | computation | self | `crm` | depends: `probability`, `automated_probability` | If probability and automated_probability are equal probability computation is considered as automatic, aka probability is sync with automated_probability |
| `_compute_probabilities` | computation | self | `crm` | depends: |  |
| `_compute_prorated_revenue` | computation | self | `crm` | depends: `expected_revenue`, `probability` |  |
| `_compute_recurring_revenue_monthly` | computation | self | `crm` | depends: `recurring_revenue`, `recurring_plan.number_of_months` |  |
| `_compute_recurring_revenue_monthly_prorated` | computation | self | `crm` | depends: `recurring_revenue_monthly`, `probability` |  |
| `_compute_recurring_revenue_prorated` | computation | self | `crm` | depends: `recurring_revenue`, `probability` |  |
| `_compute_meeting_display` | computation | self | `crm` | depends: `calendar_event_ids`, `calendar_event_ids.start` |  |
| `_compute_won_status` | computation | self | `crm` | depends: `active`, `probability`, `stage_id` |  |
| `_compute_potential_lead_duplicates` | computation | self | `crm` | depends: `email_domain_criterion`, `email_normalized`, `partner_id`, `phone_sanitized` | Override potential lead duplicates computation to be more efficient with high lead volume. Criterions:   * email domain exact match;   * phone_sanitized exact match;   * same commercial entity; |
| `_compute_partner_email_update` | computation | self | `crm` | depends: `email_from`, `partner_id` |  |
| `_compute_partner_phone_update` | computation | self | `crm` | depends: `phone`, `partner_id` |  |
| `_compute_is_partner_visible` | computation | self | `crm` | depends_context: `uid`; depends: `partner_id`, `type` | When the crm.lead is of type 'lead', we don't want to display the "Customer" field on the form view unless it's set (or debug mode).  Indeed, most of the times leads will not have this information set, since when we assign a Customer we usually convert the lead to an opportunity as well.  This means that on the lead form, we don't want to display this field since it may be misleading for the end user. When it's set however, we want to display it, mainly because there are a few automatic synchronizations between the lead and its partner (phone and email for examples), and this needs to be clear |
| `_onchange_phone_validation` | on change | self | `crm` | onchange: `phone`, `country_id`, `company_id` |  |
| `_prepare_values_from_partner` | preparation rule | self, partner | `crm` |  | Get a dictionary with values coming from partner information to copy on a lead. Non-address fields get the current lead values to avoid being reset if partner has no value for them. |
| `_prepare_address_values_from_partner` | preparation rule | self, partner | `crm` |  |  |
| `_prepare_contact_name_from_partner` | preparation rule | self, partner | `crm` |  |  |
| `_prepare_partner_name_from_partner` | preparation rule | self, partner | `crm` |  | Company name: name of partner parent (if set) or name of partner (if company) or company_name of partner (if not a company). |
| `_get_partner_email_update` | preparation rule | self, force_void | `crm`, `website_crm_partner_assign` |  | Calculate if we should write the email on the related partner. When the email of the lead / partner is an empty string, we force it to False to not propagate a False on an empty string.  Done in a separate method so it can be used in both ribbon and inverse and compute of email update methods.  :param bool force_void: if False, skip when lead has a void email value.   This is used notably to avoid propagating void lead value to a valid   partner value. |
| `_get_partner_phone_update` | preparation rule | self, force_void | `crm` |  | Calculate if we should write the phone on the related partner. When the phone of the lead / partner is an empty string, we force it to False to not propagate a False on an empty string.  Done in a separate method so it can be used in both ribbon and inverse and compute of phone update methods.  :param bool force_void: if False, skip when lead has a void phone value.   This is used notably to avoid propagating void lead value to a valid   partner value. |
| `create` | lifecycle override | self, vals_list | `crm_iap_enrich`, `crm_livechat`, `crm` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `crm_livechat`, `crm`, `website_crm_partner_assign` |  |  |
| `search_fetch` | operation | self, domain, field_names, offset, limit, order | `crm` | model | Override to support ordering on my_activity_date_deadline.  Ordering through web client calls search_read() with an order parameter set. Method search_read() then calls search_fetch(). Here we override search_fetch() to intercept a search with an order on field my_activity_date_deadline. In that case we do the search in two steps.  First step: fill with deadline-based results    * Perform a read_group on my activities to get a mapping lead_id / deadline     Remember date_deadline is required, we always have a value for it. Only     the earliest deadline per lead is kept.   * Search leads linke |
| `_handle_won_lost` | internal rule | self, old_status_by_lead, new_status_by_lead | `crm` |  | This method handles all changes of won / lost status of leads on creation / writing, and update the scoring frequency table accordingly: - To lost : Increment corresponding lost count - To won : Increment corresponding won count - Leaving lost : Decrement corresponding lost count - Leaving won : Decrement corresponding won count More than one operation can happen simultaneously, for instance, going from lost to won: Decrement corresponding lost count + increment corresponding won count.  A lead is WON when in won stage (and probability = 100% but that is implied and constrained) A lead is LOST |
| `copy_data` | lifecycle override | self, default | `crm` |  |  |
| `unlink` | lifecycle override | self | `crm` |  | Update meetings when removing opportunities, otherwise you have a link to a record that does not lead anywhere. |
| `_read_group_stage_ids` | internal rule | self, stages, domain | `crm` | model |  |
| `_stage_find` | internal rule | self, team_id, domain, order, limit | `crm` |  | Determine the stage of the current lead with its teams, the given domain and the given team_id :param team_id :param domain : base search domain for stage :param order : base search order for stage :param limit : base search limit for stage :returns crm.stage recordset |
| `action_unarchive` | lifecycle override | self | `crm` |  | When re-activating, force update probability for both leads and opportunities. Note that archiving triggers nothing more, as a lead can be archived and not lost. |
| `action_restore` | user action | self | `crm` |  | Restoring a lost lead means that it should go back to its normal life cycle. This should reactivate the lead but also force the recompute of its probability, for the stage where the lead is currently at. During toggle_active, when reactivating a lost lead,only the automated probability will be recomputed, because the probability is not automated anymore. Restore will reset this automation. |
| `action_set_lost` | user action | self, **additional_values | `crm` |  | Lost semantic: probability = 0 AND active = False |
| `action_set_won` | user action | self | `crm` |  | Won semantic: stage.is_won (AND probability = 100 but implied) |
| `action_set_automated_probability` | user action | self | `crm` |  | Update the automated probability and align probability to that value |
| `action_set_won_rainbowman` | user action | self | `crm` |  |  |
| `get_rainbowman_message` | operation | self | `crm` |  |  |
| `_get_rainbowman_message` | preparation rule | self | `crm` |  |  |
| `action_schedule_meeting` | user action | self, smart_calendar | `crm` |  | Open meeting's calendar view to schedule meeting on current opportunity.  :param bool smart_calendar: to set to False if the view should not try to choose relevant   mode and initial date for calendar view, see `_get_opportunity_meeting_view_parameters` :returns: dictionary value for created Meeting view :rtype: dict |
| `_get_opportunity_meeting_view_parameters` | preparation rule | self | `crm` |  | Return the most relevant parameters for calendar view when viewing meetings linked to an opportunity. If there are any meetings that are not finished yet, only consider those meetings, since the user would prefer no to see past meetings. Otherwise, consider all meetings. Allday events datetimes are used without taking tz into account. -If there is no event, return week mode and false (The calendar will target 'now' by default) -If there is only one, return week mode and date of the start of the event. -If there are several events entirely on the same week, return week mode and start of first e |
| `action_reschedule_meeting` | user action | self | `crm` |  |  |
| `action_show_potential_duplicates` | user action | self | `crm` |  | Open kanban view to display duplicate leads or opportunity. :return dict: dictionary value for created kanban view |
| `redirect_lead_opportunity_view` | operation | self | `crm` |  |  |
| `get_empty_list_help` | operation | self, help_message | `crm` | model | This method returns the action helpers for the leads. If help is already provided on the action, the same is returned. Otherwise, we build the help message which contains the alias responsible for creating the lead (if available) and return it. |
| `_assign_userless_lead_in_team` | internal rule | self, creation_source | `crm` |  | Assign userless leads to their team's leader. |
| `log_meeting` | operation | self, meeting | `crm` |  | Log the meeting info with a link to it in the chatter :param record meeting: the meeting we want to log |
| `_merge_data` | internal rule | self, fnames | `crm` |  | Prepare lead/opp data into a dictionary for merging. Different types of fields are processed in different ways:     - text: all the values are concatenated     - m2m and o2m: those fields aren't processed     - m2o: the first not null value prevails (the other are dropped)     - any other type of field: same as m2o  :param fnames: list of fields to process :returns: contains the merged values of the new opportunity :rtype: dict |
| `merge_opportunity` | operation | self, user_id, team_id, auto_unlink | `crm` |  | Merge opportunities in one. Different cases of merge:  - merge leads together = 1 new lead - merge at least 1 opp with anything else (lead or opp) = 1 new opp  The resulting lead/opportunity will be the most important one (based on its confidence level) updated with values from other opportunities to merge.  :param user_id: the id of the saleperson. If not given, will be determined by :meth:`_merge_data`. :param team_id: the id of the Sales Team. If not given, will be determined by :meth:`_merge_data`. :returns: crm.lead record resulting of th merge |
| `_merge_opportunity` | internal rule | self, user_id, team_id, auto_unlink, max_length | `crm` |  | Private merging method. This one allows to relax rules on record set length allowing to merge more than 5 opportunities at once if requested. This should not be called by action buttons.  See `merge_opportunity` for more details. |
| `_merge_get_fields_address` | internal rule | self | `crm` |  | The address fields are propagated as a whole.  The address is taken from the lead with the most non-empty address field (sorted by highest rank if multiple lead have the same amount of non-empty fields). |
| `_merge_get_fields_specific` | internal rule | self | `crm_iap_enrich`, `crm`, `sale_crm`, `website_crm` |  |  |
| `_merge_get_fields` | internal rule | self | `crm_iap_mine`, `crm`, `event_crm`, `iap_crm`, `website_crm_iap_reveal`, `website_crm_partner_assign` |  |  |
| `_merge_dependences` | internal rule | self, opportunities | `crm`, `event_crm` |  | Merge dependences (messages, attachments,activities, calendar events, ...). These dependences will be transfered to `self` considered as the master lead.  :param opportunities : recordset of opportunities to transfer. Does not   include `self` which is the target crm.lead being the result of the   merge; |
| `_merge_dependences_history` | internal rule | self, opportunities | `crm` |  | Move history from the given opportunities to the current one. `self` is the crm.lead record destination for message of `opportunities`.  This method moves   * messages   * activities  :param opportunities: see `_merge_dependences` |
| `_merge_dependences_attachments` | internal rule | self, opportunities | `crm` |  | Move attachments of given opportunities to the current one `self`, and rename     the attachments having same name than native ones.  :param opportunities: see `_merge_dependences` |
| `_merge_dependences_calendar_events` | internal rule | self, opportunities | `crm` |  | Move calender.event from the given opportunities to the current one. `self` is the     crm.lead record destination for event of `opportunities`. :param opportunities: see `merge_dependences` |
| `_merge_followers` | internal rule | self, opportunities | `crm` |  | Add the followers into the destination lead if they post a message in the last 30 days.  :param opportunities : Record<crm.lead> of opportunities to transfer :return: {old_lead_id: Record<mail.followers>} Followers which have been added in     the destination lead grouped by source lead ID. |
| `_merge_log_summary` | internal rule | self, merged_followers, opportunities_tail | `crm` |  | Log the merge message on the lead. |
| `_format_properties` | internal rule | self | `crm` |  | Format the properties to build the merge message.  Return a list of dict containing the label, and a value key if there's only one value, or a "values" key if we have multiple values (e.g. many2many, tags).  E.G.     [{         'label': 'My Partner',         'value': 'Alice',     }, {         'label': 'My Partners',         'values': [             {'name': 'Alice'},             {'name': 'Bob'},         ],     }, {         'label': 'My Tags',         'values': [             {'name': 'A', 'color': 1},             {'name': 'C', 'color': 3},         ],     }] |
| `_convert_opportunity_data` | internal rule | self, customer, team_id | `crm` |  | Extract the data from a lead to create the opportunity :param customer : res.partner record :param team_id : identifier of the Sales Team to determine the stage |
| `convert_opportunity` | operation | self, partner, user_ids, team_id | `crm` |  |  |
| `_handle_partner_assignment` | internal rule | self, force_partner_id, create_missing, with_parent | `crm` |  | Update customer (partner_id) of leads. Purpose is to set the same partner on most leads; either through a newly created partner either through a given partner_id.  :param int force_partner_id: if set, update all leads to that customer; :param create_missing: for leads without customer, create a new one   based on lead information; :param with_parent: if set, create the new partner with the given parent |
| `_handle_salesmen_assignment` | internal rule | self, user_ids, team_id | `crm` |  | Assign salesmen and salesteam to a batch of leads.  If there are more leads than salesmen, these salesmen will be assigned in round-robin. E.g. 4 salesmen (S1, S2, S3, S4) for 6 leads (L1, L2, ... L6) will assigned as following: L1 - S1, L2 - S2, L3 - S3, L4 - S4, L5 - S1, L6 - S2.  :param list user_ids: salesmen to assign :param int team_id: salesteam to assign |
| `_get_lead_duplicates` | preparation rule | self, partner, email, include_lost | `crm` |  | Search for leads that seem duplicated based on partner / email.  :param partner : optional customer when searching duplicated :param email: email (possibly formatted) to search :param boolean include_lost: if True, search includes archived opportunities   (still only active leads are considered). If False, search for active   and not won leads and opportunities; |
| `_sort_by_confidence_level` | internal rule | self, reverse | `crm` |  | Sorting the leads/opps according to the confidence level to it being won. It is sorted following this incremental heuristics :    * "not lost" first (inactive leads are lost); normally all leads     should be active but in case lost one, they are always last.     Inactive opportunities are considered as valid;   * opportunity is more reliable than a lead which is a pre-stage     used mainly for first classification;   * stage sequence: the higher the better as it indicates we are moving     towards won stage;   * probability: the higher the better as it is more likely to be won;   * ID: the hi |
| `_find_matching_partner` | internal rule | self | `crm` |  | Try to find a matching partner with available information on the lead, using currently customer's email  :return: partner browse record |
| `_create_customer` | internal rule | self, with_parent | `crm` |  | Create a partner from lead data and link it to the lead.  :param with_parent: if set, create the new partner with the given parent :return: newly-created partner browse record |
| `_get_customer_information` | preparation rule | self | `crm` |  |  |
| `_prepare_customer_values` | preparation rule | self, partner_name, is_company, parent_id | `crm`, `website_crm_partner_assign` |  | Extract data from lead to create a partner.  :param partner_name : future name of the partner :param is_company : True if the partner is a company :param parent_id : id of the parent partner (False if no parent)  :return: dictionary of values to give at res_partner.create() |
| `_is_rule_based_assignment_activated` | internal rule | self | `crm` |  | Returns whether a rule-based assignment method is activated (cron-enabled or manually-ran). |
| `_creation_subtype` | internal rule | self | `crm` |  |  |
| `_creation_message` | internal rule | self | `crm` |  |  |
| `_track_subtype` | messaging hook | self, init_values | `crm` |  |  |
| `_notify_by_email_prepare_rendering_context` | internal rule | self, message, msg_vals, model_description, force_email_company, force_email_lang, force_record_name | `crm` |  |  |
| `_notify_get_reply_to` | internal rule | self, default, author_id | `crm` |  |  |
| `message_new` | messaging hook | self, msg_dict, custom_values | `crm` | model |  |
| `_message_post_after_hook` | messaging hook | self, message, msg_vals | `crm` |  |  |
| `get_import_templates` | operation | self | `crm` | model |  |
| `_pls_get_naive_bayes_probabilities` | internal rule | self, batch_mode, is_tooltip | `crm` |  | In machine learning, naive Bayes classifiers (NBC) are a family of simple "probabilistic classifiers" based on applying Bayes theorem with strong (naive) independence assumptions between the variables taken into account. E.g: will TDE eat m&m's depending on his sleep status, the amount of work he has and the fullness of his stomach? As we use experience to compute the statistics, every day, we will register the variables state + the result. As the days pass, we will be able to determine, with more and more precision, if TDE will eat m&m's for a specific combination :     - did sleep very well, |
| `_pls_increment_frequencies` | internal rule | self, from_state, to_state | `crm` |  | When losing or winning a lead, this method is called to increment each PLS parameter related to the lead in won_count (if won) or in lost_count (if lost).  This method is also used when reactivating a mistakenly lost lead (using the decrement argument). In this case, the lost count should be de-increment by 1 for each PLS parameter linked to the lead.  Live increment must be done before writing the new values because we need to know the state change (from and to). This would not be an issue for the reach won or reach lost as we just need to increment the frequencies with the final state of the |
| `_cron_update_automated_probabilities` | background operation | self | `crm` |  | This cron will : - rebuild the lead scoring frequency table - recompute all the automated_probability and align probability if both were aligned |
| `_rebuild_pls_frequency_table` | internal rule | self | `crm` |  |  |
| `_update_automated_probabilities` | internal rule | self | `crm` |  | Recompute all the automated_probability (and align probability if both were aligned) for all the leads that are active (not won, nor lost).  For performance matter, as there can be a huge amount of leads to recompute, this cron proceed by batch. Each batch is performed into its own transaction, in order to minimise the lock time on the lead table (and to avoid complete lock if there was only 1 transaction that would last for too long -> several minutes). If a concurrent update occurs, it will simply be put in the queue to get the lock. |
| `_pls_prepare_update_frequency_table` | internal rule | self, rebuild, target_state | `crm` |  | This method is common to Live Increment or Full Rebuild mode, as it shares the main steps. This method will prepare the frequency dict needed to update the frequency table:     - New frequencies: frequencies that we need to add in the frequency table.     - Existing frequencies: frequencies that are already in the frequency table. In rebuild mode, only the new frequencies are needed as existing frequencies are truncated. For each team, each dict contains the frequency in won and lost for each field/value couple of the target leads. Target leads are :     - in Live increment mode : given ongoin |
| `_pls_update_frequency_table` | internal rule | self, new_frequencies_by_team, step, existing_frequencies_by_team | `crm` |  | Create / update the frequency table in a cross company way, per team_id |
| `_pls_get_safe_start_date` | internal rule | self | `crm` |  | As config_parameters does not accept Date field, we get directly the date formated string stored into the Char config field, as we directly use this string in the sql queries. To avoid sql injections when using this config param, we ensure the date string can be effectively a date. |
| `_pls_get_safe_fields` | internal rule | self | `crm` |  | As config_parameters does not accept M2M field, we the fields from the formated string stored into the Char config field. To avoid sql injections when using that list, we return only the fields that are defined on the model. |
| `_pls_get_won_lost_total_count` | internal rule | self, team_results | `crm` |  | Get all won and all lost + total :        first stage can be used to know how many lost and won there is        as won count are equals for all stage        and first stage is always incremented in lost_count :param team_results: :return: won count, lost count and total count for all records in frequencies |
| `_pls_prepare_frequencies` | internal rule | self, lead_values, leads_pls_fields, target_state | `crm` |  | new state is used when getting frequencies for leads that are changing to lost or won. Stays none if we are checking frequencies for leads already won or lost. |
| `_pls_increment_frequency_dict` | internal rule | self, frequencies, field, value, won, lost | `crm` |  |  |
| `_pls_get_lead_pls_values` | internal rule | self, domain | `crm` |  | This methods builds a dict where, for each lead in self or matching the given domain, we will get a list of field/value couple. Due to onchange and create, we don't always have the id of the lead to recompute. When we update few records (one, typically) with onchanges, we build the lead_values (= couple field/value) using the ORM. To speed up the computation and avoid making too much DB read inside loops, we can give a domain to make sql queries to bypass the ORM. This domain will be used in sql queries to get the values for every lead matching the domain. :param domain: If set, we get all the |
| `prepare_pls_tooltip_data` | operation | self | `crm` |  | Compute and return all necessary information to render CrmPlsTooltip, displayed when pressing the small AI button, located next to the label of probability when automated, in the crm.lead form view. This method first replaces ids with display names of relational fields before returning data, then also recomputes probabilities and writes them on self.  :returns:      ::         {             low_3_data: list of field-value couples for lowest 3 criterions, lowest first             probability: numerical value, used for display on tooltip             team_name: string, name of lead team if any    |
| `_compute_show_enrich_button` | computation | self | `crm_iap_enrich` | depends: `email_from`, `probability`, `iap_enrich_done`, `reveal_id` |  |
| `_iap_enrich_leads_cron` | internal rule | self, enrich_hours_delay, batch_size | `crm_iap_enrich` | model |  |
| `iap_enrich` | operation | self, batch_size | `crm_iap_enrich` |  |  |
| `_iap_enrich_from_response` | internal rule | self, iap_response | `crm_iap_enrich` | model | Handle from the service and enrich the lead accordingly  :param iap_response: dict{lead_id: company data or False} |
| `action_generate_leads` | user action | self | `crm_iap_mine` |  |  |
| `action_open_livechat` | user action | self | `crm_livechat` |  |  |
| `_form_view_auto_fill` | internal rule | self | `crm_mail_plugin` | model | deprecated as of saas-14.3, not needed for newer versions of the mail plugin but necessary for supporting older versions |
| `_compute_registration_count` | computation | self | `event_crm` | depends: `registration_ids` |  |
| `_compute_sale_data` | computation | self | `sale_crm` | depends: `order_ids.state`, `order_ids.currency_id`, `order_ids.amount_untaxed`, `order_ids.date_order`, `order_ids.company_id` |  |
| `action_sale_quotations_new` | user action | self | `sale_crm` |  |  |
| `action_new_quotation` | user action | self | `sale_crm` |  |  |
| `action_view_sale_quotation` | user action | self | `sale_crm` |  |  |
| `action_view_sale_order` | user action | self | `sale_crm` |  |  |
| `_get_action_view_sale_quotation_domain` | preparation rule | self | `sale_crm` |  |  |
| `_get_lead_quotation_domain` | preparation rule | self | `sale_crm` |  |  |
| `_get_lead_sale_order_domain` | preparation rule | self | `sale_crm` |  |  |
| `_prepare_opportunity_quotation_context` | preparation rule | self | `sale_crm` |  | Prepares the context for a new quotation (sale.order) by sharing the values of common fields |
| `_update_revenues_from_so` | internal rule | self, order | `sale_crm` |  |  |
| `_compute_visitor_page_count` | computation | self | `website_crm` | depends: `visitor_ids.page_ids` |  |
| `action_redirect_to_page_views` | user action | self | `website_crm` |  |  |
| `website_form_input_filter` | operation | self, request, values | `website_crm` |  |  |
| `_compute_visitor_sessions_count` | computation | self | `website_crm_livechat` | depends: `visitor_ids.discuss_channel_ids` |  |
| `action_redirect_to_livechat_sessions` | user action | self | `website_crm_livechat` |  |  |
| `_compute_date_partner_assign` | computation | self | `website_crm_partner_assign` | depends: `partner_assigned_id` |  |
| `_assert_portal_write_access` | internal rule | self | `website_crm_partner_assign` |  |  |
| `assign_salesman_of_assigned_partner` | operation | self | `website_crm_partner_assign` |  |  |
| `action_assign_partner` | user action | self | `website_crm_partner_assign` |  | While assigning a partner, geo-localization is performed only for leads having country set (see method 'assign_geo_localize' and 'search_geo_partner'). So for leads that does not have country set, we show the notification, and for the rest, we geo-localize them. |
| `assign_partner` | operation | self, partner_id | `website_crm_partner_assign` |  |  |
| `assign_geo_localize` | operation | self, latitude, longitude | `website_crm_partner_assign` |  |  |
| `search_geo_partner` | operation | self | `website_crm_partner_assign` |  |  |
| `partner_interested` | operation | self, comment | `website_crm_partner_assign` |  |  |
| `partner_desinterested` | operation | self, comment, contacted, spam | `website_crm_partner_assign` |  |  |
| `update_lead_portal` | operation | self, values | `website_crm_partner_assign` |  |  |
| `update_contact_details_from_portal` | operation | self, values | `website_crm_partner_assign` |  |  |
| `update_stage_from_portal` | operation | self, stage_id | `website_crm_partner_assign` |  | Allow portal users to update the stage of their assigned leads |
| `create_opp_portal` | operation | self, values | `website_crm_partner_assign` | model |  |
| `_get_access_action` | preparation rule | self, access_uid, force_website | `website_crm_partner_assign` |  | Instead of the classic form view, redirect to the online document for portal users or if force_website=True. |
| `_mail_get_operation_for_mail_message_operation` | messaging hook | self, message_operation | `website_crm_partner_assign` | model |  |

## Validation and error messages (9)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_won_validity` | ValidationError | A lead in a Won stage cannot be lost. Move it to another stage first. | `crm` |
| `_handle_won_lost` | ValidationError | The lead %s cannot be won and lost at the same time. | `crm` |
| `_merge_opportunity` | UserError | Select at least two Leads/Opportunities from the list to merge them. | `crm` |
| `_merge_opportunity` | UserError | To prevent data loss, Leads and Opportunities can only be merged by groups of %(max_length)s. | `crm` |
| `_rebuild_pls_frequency_table` | UserError | You don't have the access needed to run this cron. | `crm` |
| `create` | AccessError | You cannot create leads linked to channels you don't have access to. | `crm_livechat` |
| `write` | AccessError | You cannot update a lead and link it to a channel you don't have access to. | `crm_livechat` |
| `_assert_portal_write_access` | AccessError | Only users with commercial partner which is a parent of the assigned partner can edit this lead. | `website_crm_partner_assign` |
| `update_contact_details_from_portal` | UserError | Not allowed to update the following field(s): %s. | `website_crm_partner_assign` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `crm` |
| `sales_team.group_sale_salesman` | yes | yes | yes | no | `crm` |
| `base.group_portal` | no | yes | no | no | `website_crm_partner_assign` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Personal Leads | `[(4, ref('sales_team.group_sale_salesman'))]` | `['\|',('user_id','=',user.id),('user_id','=',False)]` | True | True | True | True |
| CRM Lead Multi-Company | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| All Leads | `[(4, ref('sales_team.group_sale_salesman_all_leads'))]` | `[(1,'=',1)]` | True | True | True | True |
| Portal Graded Partner: read and write assigned leads | `[(4, ref('base.group_portal'))]` | `[('partner_assigned_id','child_of',user.commercial_partner_id.id)]` | True | False | False | False |

## Views (59)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `crm.crm_lead_view_form` | form |  | `stage_id`, `won_status`, `active`, `company_id`, `is_rotting`, `rotting_days`, `meeting_display_label`, `meeting_display_date`, `duplicate_lead_count`, `name`, `company_currency`, `expected_revenue`, `recurring_revenue`, `recurring_plan`, `automated_probability`, `is_automated_probability`, `probability`, `is_partner_visible`, `partner_id`, `partner_name`, `street`, `street2`, `city`, `state_id`, `zip`, `country_id`, `website`, `lang_active_count`, `lang_code`, `lang_id`, `partner_id`, `is_blacklisted`, `partner_is_blacklisted`, `phone_blacklisted`, `email_state`, `phone_state`, `partner_email_update`, `partner_phone_update`, `email_from`, `phone`, `lost_reason_id`, `date_conversion`, `user_company_ids`, `contact_name`, `is_blacklisted`, `phone_blacklisted`, `email_state`, `phone_state`, `partner_email_update`, `partner_phone_update`, `email_from`, `email_cc`, `function`, `phone`, `type`, `user_id`, `date_deadline`, `priority`, `tag_ids`, `user_id` | `Won`, `Convert to Opportunity`, `Restore`, `Lost`, `action_schedule_meeting`, `action_show_potential_duplicates`, `mail_action_blacklist_remove`, `phone_action_blacklist_remove`, `mail_action_blacklist_remove`, `phone_action_blacklist_remove` |  | `crm` |
| `crm.crm_case_tree_view_leads` | list |  | `company_id`, `user_company_ids`, `date_deadline`, `create_date`, `name`, `contact_name`, `partner_name`, `email_from`, `phone`, `company_id`, `city`, `state_id`, `country_id`, `partner_id`, `user_id`, `team_id`, `active`, `campaign_id`, `referred`, `medium_id`, `source_id`, `probability`, `message_needaction`, `tag_ids`, `priority` | `Mark Lost` |  | `crm` |
| `crm.view_crm_lead_kanban` | kanban |  | `name`, `contact_name`, `tag_ids`, `priority`, `activity_ids`, `user_id` |  |  | `crm` |
| `crm.crm_case_calendar_view_leads` | calendar |  | `expected_revenue`, `partner_id`, `user_id`, `team_id`, `lead_properties` |  |  | `crm` |
| `crm.quick_create_opportunity_form` | form |  | `commercial_partner_id`, `partner_id`, `name`, `email_from`, `phone`, `expected_revenue`, `priority`, `recurring_revenue`, `recurring_plan`, `company_currency`, `company_id`, `user_id`, `user_company_ids`, `team_id`, `type`, `partner_name`, `contact_name`, `country_id`, `state_id`, `city`, `street`, `street2`, `zip`, `website`, `function`, `activity_ids` |  |  | `crm` |
| `crm.crm_lead_view_activity` | activity |  | `user_id`, `company_currency`, `user_id`, `name`, `expected_revenue`, `partner_id`, `stage_id_color`, `stage_id` |  |  | `crm` |
| `crm.crm_case_kanban_view_leads` | kanban |  | `stage_id`, `probability`, `active`, `company_currency`, `recurring_revenue_monthly`, `team_id`, `won_status`, `color`, `name`, `expected_revenue`, `recurring_revenue`, `recurring_plan`, `partner_id`, `partner_id`, `contact_name`, `partner_name`, `tag_ids`, `lead_properties`, `priority`, `activity_ids`, `is_rotting`, `rotting_days`, `user_id` |  |  | `crm` |
| `crm.crm_lead_view_kanban_forecast` | xpath | `crm.crm_case_kanban_view_leads` |  |  |  | `crm` |
| `crm.view_crm_case_leads_filter` | search |  | `name`, `tag_ids`, `user_id`, `team_id`, `country_id`, `city`, `phone_mobile_search`, `lang_id`, `create_date`, `source_id`, `medium_id`, `campaign_id`, `activity_state`, `lead_properties`, `activity_user_id`, `activity_type_id` |  | `My Leads`, `Unassigned`, `Lost`, `Creation Date`, `filter_date_closed`, `My Activities`, `Late Activities`, `Today Activities`, `Future Activities`, `Archived`, `Salesperson`, `Sales Team`, `City`, `Country`, `Company`, `Campaign`, `Medium`, `Source`, `Creation Date`, `Closed Date`, `Properties` | `crm` |
| `crm.crm_case_tree_view_oppor` | list |  | `company_id`, `user_company_ids`, `date_deadline`, `activity_calendar_event_id`, `won_status`, `create_date`, `name`, `partner_id`, `contact_name`, `email_from`, `phone`, `company_id`, `city`, `state_id`, `country_id`, `user_id`, `team_id`, `priority`, `activity_ids`, `activity_user_id`, `my_activity_date_deadline`, `campaign_id`, `medium_id`, `source_id`, `company_currency`, `expected_revenue`, `date_deadline`, `recurring_revenue_monthly`, `recurring_revenue`, `recurring_plan`, `stage_id_color`, `stage_id`, `active`, `probability`, `lost_reason_id`, `tag_ids`, `referred`, `message_needaction`, `lead_properties`, `is_rotting`, `rotting_days` | `Mark Lost`, `Email`, `Email` |  | `crm` |
| `crm.crm_lead_view_tree_forecast` | xpath | `crm.crm_case_tree_view_oppor` |  |  |  | `crm` |
| `crm.crm_lead_view_list_activities` | xpath | `crm.crm_case_tree_view_oppor` |  |  |  | `crm` |
| `crm.view_crm_case_my_activities_filter` | xpath | `crm.view_crm_case_leads_filter` |  |  | `Late Activities` | `crm` |
| `crm.crm_lead_view_graph` | graph |  | `stage_id`, `user_id`, `color`, `automated_probability`, `message_bounce`, `recurring_revenue_monthly`, `recurring_revenue_monthly_prorated`, `recurring_revenue`, `recurring_revenue_prorated`, `stage_id_color` |  |  | `crm` |
| `crm.crm_lead_view_graph_forecast` | graph |  | `date_deadline`, `prorated_revenue`, `automated_probability`, `color`, `day_open`, `day_close`, `message_bounce`, `probability`, `recurring_revenue_monthly`, `recurring_revenue_monthly_prorated`, `recurring_revenue`, `recurring_revenue_prorated`, `stage_id_color` |  |  | `crm` |
| `crm.crm_lead_view_pivot` | pivot |  | `create_date`, `stage_id`, `expected_revenue`, `color`, `stage_id_color`, `automated_probability`, `message_bounce`, `probability`, `recurring_revenue_monthly`, `recurring_revenue_monthly_prorated`, `recurring_revenue`, `recurring_revenue_prorated` |  |  | `crm` |
| `crm.crm_lead_view_pivot_forecast` | pivot |  | `date_deadline`, `stage_id`, `prorated_revenue`, `automated_probability`, `color`, `day_open`, `day_close`, `message_bounce`, `probability`, `recurring_revenue_monthly`, `recurring_revenue_monthly_prorated`, `recurring_revenue`, `recurring_revenue_prorated`, `stage_id_color` |  |  | `crm` |
| `crm.view_crm_case_opportunities_filter` | search |  | `name`, `partner_id`, `tag_ids`, `user_id`, `team_id`, `stage_id`, `country_id`, `city`, `phone_mobile_search`, `activity_state`, `lead_properties` |  | `My Pipeline`, `Unassigned`, `Open Opportunities`, `Unread Messages`, `Creation Date`, `Closed Date`, `Won`, `Ongoing`, `Rotting`, `Lost`, `Overdue Opportunities`, `Late Activities`, `Today Activities`, `Future Activities`, `Salesperson`, `Sales Team`, `Stage`, `City`, `Country`, `Lost Reason`, `Company`, `Campaign`, `Medium`, `Source`, `Creation Date`, `Creation Date`, `Conversion Date`, `Expected Closing`, `Closed Date`, `Properties` | `crm` |
| `crm.crm_lead_view_search_forecast` | xpath | `crm.view_crm_case_opportunities_filter` |  |  | `Upcoming Closings` | `crm` |
| `crm.crm_lead_view_tree_opportunity_reporting` | xpath | `crm.crm_case_tree_view_oppor` |  |  |  | `crm` |
| `crm.crm_opportunity_report_view_pivot` | pivot |  | `create_date`, `stage_id`, `prorated_revenue`, `color`, `message_bounce`, `probability`, `automated_probability`, `recurring_revenue_monthly`, `recurring_revenue_monthly_prorated`, `recurring_revenue`, `recurring_revenue_prorated` |  |  | `crm` |
| `crm.crm_opportunity_report_view_pivot_lead` | pivot |  | `create_date`, `team_id`, `color`, `message_bounce`, `probability`, `automated_probability`, `recurring_revenue_monthly`, `recurring_revenue_monthly_prorated`, `recurring_revenue`, `recurring_revenue_prorated` |  |  | `crm` |
| `crm.crm_opportunity_report_view_graph` | graph |  | `stage_id`, `date_deadline`, `prorated_revenue`, `color`, `message_bounce`, `probability`, `automated_probability`, `recurring_revenue_monthly`, `recurring_revenue_monthly_prorated`, `recurring_revenue`, `recurring_revenue_prorated` |  |  | `crm` |
| `crm.crm_opportunity_report_view_graph_lead` | graph |  | `create_date`, `team_id`, `color`, `automated_probability`, `message_bounce`, `probability`, `recurring_revenue_monthly`, `recurring_revenue_monthly_prorated`, `recurring_revenue`, `recurring_revenue_prorated` |  |  | `crm` |
| `crm.crm_opportunity_report_view_search` | search |  | `team_id`, `user_id`, `partner_id`, `stage_id`, `campaign_id`, `medium_id`, `source_id`, `company_id`, `create_date`, `date_open`, `date_closed` |  | `My Pipeline`, `Opportunities`, `Leads`, `Active`, `Inactive`, `Won`, `Lost`, `filter_create_date`, `Expected Closing`, `Date Closed`, `Archived`, `Salesperson`, `Sales Team`, `City`, `Country`, `Company`, `Stage`, `Campaign`, `Medium`, `Source`, `Creation Date`, `Conversion Date`, `Expected Closing`, `Closed Date`, `Lost Reason` | `crm` |
| `crm.crm_lead_view_tree_reporting` | xpath | `crm.crm_case_tree_view_leads` |  |  |  | `crm` |
| `crm_iap_enrich.crm_lead_view_form` | xpath | `crm.crm_lead_view_form` | `show_enrich_button` | `Enrich`, `Enrich` |  | `crm_iap_enrich` |
| `crm_iap_mine.crm_lead_view_tree_opportunity` | xpath | `crm.crm_case_tree_view_oppor` |  | `Generate Leads` |  | `crm_iap_mine` |
| `crm_iap_mine.crm_lead_view_tree_lead` | xpath | `crm.crm_case_tree_view_leads` |  | `Generate Leads` |  | `crm_iap_mine` |
| `crm_iap_mine.view_crm_lead_kanban` | xpath | `crm.view_crm_lead_kanban` |  | `Generate Leads` |  | `crm_iap_mine` |
| `crm_iap_mine.crm_case_kanban_view_leads` | xpath | `crm.crm_case_kanban_view_leads` |  | `Generate Leads` |  | `crm_iap_mine` |
| `crm_livechat.crm_lead_view_form` | xpath | `crm.crm_lead_view_form` |  | `action_open_livechat` |  | `crm_livechat` |
| `crm_sms.crm_case_tree_view_oppor` | xpath | `crm.crm_case_tree_view_oppor` |  | `SMS` |  | `crm_sms` |
| `crm_sms.crm_lead_view_tree_opportunity_reporting` | xpath | `crm.crm_lead_view_tree_opportunity_reporting` |  |  |  | `crm_sms` |
| `event_crm.crm_lead_view_form` | xpath | `crm.crm_lead_view_form` | `registration_ids`, `registration_count` | `%(event_registration_action_from_lead)d` |  | `event_crm` |
| `sale_crm.crm_case_form_view_oppor` | xpath | `crm.crm_lead_view_form` |  | `New Quotation` |  | `sale_crm` |
| `website_crm.crm_lead_view_form` | xpath | `crm.crm_lead_view_form` | `visitor_page_count` | `action_redirect_to_page_views` |  | `website_crm` |
| `website_crm_iap_reveal.crm_reveal_lead_opportunity_form` | xpath | `crm.crm_lead_view_form` | `reveal_ip`, `reveal_iap_credits`, `reveal_rule_id` |  |  | `website_crm_iap_reveal` |
| `website_crm_iap_reveal.crm_lead_view_pivot` | xpath | `crm.crm_lead_view_pivot` | `reveal_iap_credits` |  |  | `website_crm_iap_reveal` |
| `website_crm_iap_reveal.crm_lead_view_graph` | xpath | `crm.crm_lead_view_graph` | `reveal_iap_credits` |  |  | `website_crm_iap_reveal` |
| `website_crm_iap_reveal.crm_lead_view_graph_report_opportunity` | xpath | `crm.crm_opportunity_report_view_graph` | `reveal_iap_credits` |  |  | `website_crm_iap_reveal` |
| `website_crm_iap_reveal.crm_lead_view_graph_report_lead` | xpath | `crm.crm_opportunity_report_view_graph_lead` | `reveal_iap_credits` |  |  | `website_crm_iap_reveal` |
| `website_crm_iap_reveal.crm_lead_view_graph_report_forecast` | xpath | `crm.crm_lead_view_graph_forecast` | `reveal_iap_credits` |  |  | `website_crm_iap_reveal` |
| `website_crm_iap_reveal.crm_lead_view_pivot_forecast` | xpath | `crm.crm_lead_view_pivot_forecast` | `reveal_iap_credits` |  |  | `website_crm_iap_reveal` |
| `website_crm_iap_reveal.crm_opportunity_report_view_pivot_lead` | xpath | `crm.crm_opportunity_report_view_pivot_lead` | `reveal_iap_credits` |  |  | `website_crm_iap_reveal` |
| `website_crm_livechat.crm_lead_view_form` | xpath | `website_crm.crm_lead_view_form` | `visitor_sessions_count` | `action_redirect_to_livechat_sessions` |  | `website_crm_livechat` |
| `website_crm_partner_assign.view_crm_lead_opportunity_geo_assign_form` | xpath | `crm.crm_lead_view_form` | `partner_latitude`, `partner_longitude`, `partner_assigned_id` | `Automatic Assignment`, `Send Email` |  | `website_crm_partner_assign` |
| `website_crm_partner_assign.view_crm_opportunity_geo_assign_tree` | field | `crm.crm_case_tree_view_oppor` | `priority`, `partner_assigned_id`, `date_partner_assign` |  |  | `website_crm_partner_assign` |
| `website_crm_partner_assign.crm_opportunity_partner_filter` | filter | `crm.view_crm_case_opportunities_filter` |  |  | `stage`, `Assigned Partner` | `website_crm_partner_assign` |
| `website_crm_partner_assign.view_crm_lead_geo_assign_tree` | field | `crm.crm_case_tree_view_leads` | `partner_id`, `partner_assigned_id` |  |  | `website_crm_partner_assign` |
| `website_crm_partner_assign.crm_lead_partner_filter` | filter | `crm.view_crm_case_leads_filter` |  |  | `company`, `Assigned Partner` | `website_crm_partner_assign` |
| `website_crm_partner_assign.crm_lead_view_pivot` | xpath | `crm.crm_lead_view_pivot` | `partner_latitude`, `partner_longitude` |  |  | `website_crm_partner_assign` |
| `website_crm_partner_assign.crm_opportunity_report_view_pivot_lead` | xpath | `crm.crm_opportunity_report_view_pivot_lead` | `partner_latitude`, `partner_longitude` |  |  | `website_crm_partner_assign` |
| `website_crm_partner_assign.crm_lead_view_pivot_forecast` | xpath | `crm.crm_lead_view_pivot_forecast` | `partner_latitude`, `partner_longitude` |  |  | `website_crm_partner_assign` |
| `website_crm_partner_assign.crm_lead_view_graph` | xpath | `crm.crm_lead_view_graph` | `partner_latitude`, `partner_longitude` |  |  | `website_crm_partner_assign` |
| `website_crm_partner_assign.crm_lead_view_graph_forecast` | xpath | `crm.crm_lead_view_graph_forecast` | `partner_latitude`, `partner_longitude` |  |  | `website_crm_partner_assign` |
| `website_crm_partner_assign.crm_lead_view_graph_report_opportunity` | xpath | `crm.crm_opportunity_report_view_graph` | `partner_latitude`, `partner_longitude` |  |  | `website_crm_partner_assign` |
| `website_crm_partner_assign.crm_lead_view_graph_report_lead` | xpath | `crm.crm_opportunity_report_view_graph_lead` | `partner_latitude`, `partner_longitude` |  |  | `website_crm_partner_assign` |
| `website_crm_partner_assign.crm_lead_view_kanban` | xpath | `crm.crm_case_kanban_view_leads` | `partner_assigned_id` |  |  | `website_crm_partner_assign` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `crm.crm_lead_all_leads` | Leads | list,kanban,graph,pivot,calendar,form,activity | `['\|', ('type','=','lead'), ('type','=',False)]` | `{                     'default_type':'lead',                     'search_default_type': 'lead',                     'search_default_to_process':1,                 }` |  | `crm` |
| `crm.crm_lead_action_my_activities` | My Activities | list,kanban,graph,pivot,calendar,form,activity | `[("active", "in", [True, False]), ("activity_ids.active", "in", [True, False])]` | `{                 'default_type': 'opportunity',                 'force_search_count': 1,                 'search_default_assigned_to_me': 1,             }` |  | `crm` |
| `crm.crm_lead_opportunities` | Opportunities | kanban,list,graph,pivot,form,calendar,activity | `[('type','=','opportunity')]` | `{                     'default_type': 'opportunity',                 }` |  | `crm` |
| `crm.crm_lead_action_pipeline` | Pipeline | kanban,list,graph,pivot,form,calendar,activity | `[('type','=','opportunity')]` | `{                     'default_type': 'opportunity',                     'search_default_assigned_to_me': 1,                     'show_user_team_stages': 1,             }` |  | `crm` |
| `crm.crm_lead_action_forecast` | Forecast | kanban,graph,pivot,list,form | `[('type', '=', 'opportunity')]` | `{                 'default_type': 'opportunity',                 'search_default_assigned_to_me': 1,                 'search_default_forecast': 1,                 'search_default_date_deadline': 1,                 'forecast_field': 'date_deadline'             }` |  | `crm` |
| `crm.crm_lead_action_open_lead_form` | New Lead | form | `[('type','=','lead')]` | `{                 'search_default_team_id': [active_id],                 'default_team_id': active_id,                 'default_type': 'lead',             }` |  | `crm` |
| `crm.crm_opportunity_report_action` | Pipeline Analysis | graph,pivot,list,form |  | `{'search_default_filter_opportunity': True, 'search_default_current': True}` |  | `crm` |
| `crm.crm_opportunity_report_action_lead` | Leads Analysis | graph,pivot,list |  | `{                 'search_default_filter_active': 1,                 'search_default_filter_inactive': 1,                 'search_default_filter_create_date': 1,             }` |  | `crm` |
| `crm.crm_case_form_view_salesteams_lead` | Leads | list,kanban,form | `['\|', ('type','=','lead'), ('type','=',False)]` | `{                     'search_default_team_id': [active_id],                     'default_team_id': active_id,                     'default_type': 'lead',                 }` |  | `crm` |
| `crm.crm_case_form_view_salesteams_opportunity` | Opportunities | kanban,list,graph,form,calendar,pivot | `[('type','=','opportunity')]` | `{                     'search_default_team_id': [active_id],                     'default_team_id': active_id,                     'default_type': 'opportunity',                     'default_user_id': uid,                 }` |  | `crm` |
| `crm.crm_lead_action_team_overdue_opportunity` | Overdue Opportunities | kanban,list,graph,form,calendar,pivot | `[('type','=','opportunity')]` | `{                     'search_default_team_id': [active_id],                     'search_default_overdue_opp': 1,                     'default_team_id': active_id,                     'default_type': 'opportunity',                     'default_user_id': uid,                 }` |  | `crm` |
| `crm.action_report_crm_lead_salesteam` | Leads Analysis | graph,pivot,list,form | `[]` | `{'search_default_team_id': [active_id], 'search_default_filter_create_date': 1}` |  | `crm` |
| `crm.action_report_crm_opportunity_salesteam` | Pipeline Analysis | graph,pivot,list,form | `[]` | `{                 'search_default_team_id': [active_id],                 'list_view_ref': 'crm.crm_lead_view_tree_opportunity_reporting',                 'search_default_filter_opportunity': True,                 'search_default_filter_create_date': 1}` |  | `crm` |
| `crm.action_opportunity_form` | New Opportunity | form | `[('type','=','opportunity')]` | `{                     'search_default_team_id': [active_id],                     'default_team_id': active_id,                     'default_type': 'opportunity',                     'default_user_id': uid,             }` |  | `crm` |
| `crm_mail_plugin.crm_lead_action_form_edit` | Lead: redirect form in edit mode | form |  |  |  | `crm_mail_plugin` |
| `event_crm.crm_lead_action_from_registration` | Leads | list,kanban,graph,pivot,calendar,form,activity | `[('registration_ids', 'in', active_id)]` | `{'create': False}` |  | `event_crm` |
| `event_crm.crm_lead_action_from_event` | Leads | list,kanban,graph,pivot,calendar,form,activity | `[('event_id', '=', active_id)]` | `{'create': False}` |  | `event_crm` |
| `website_crm.crm_lead_action_from_visitor` | Leads | list,form | `[('visitor_ids', 'in', [active_id])]` |  |  | `website_crm` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `crm.crm_lead_menu_my_activities` | My Activities | `crm_menu_sales` | `crm.crm_lead_action_my_activities` | 2 | `sales_team.group_sale_salesman` |
| `crm.crm_menu_leads` | Leads | `crm_menu_root` | `crm.crm_lead_all_leads` | 5 | `crm.group_use_lead` |
| `crm.crm_opportunity_report_menu` | Pipeline | `crm_menu_report` | `crm.crm_opportunity_report_action` | 2 |  |
| `crm.crm_opportunity_report_menu_lead` | Leads | `crm_menu_report` | `crm.crm_opportunity_report_action_lead` | 3 |  |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `crm_iap_enrich.action_enrich_mail` | Enrich | code |  | yes |
| `crm_mail_plugin.lead_creation_prefilled_action` | Redirection to the lead creation form with prefilled info | code |  | yes |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `crm.website_crm_score_cron` | Predictive Lead Scoring: Recompute Automated Probabilities | 1 days | `_cron_update_automated_probabilities` |  |
| `crm_iap_enrich.ir_cron_lead_enrichment` | CRM: enrich leads (IAP) | 24 hours | `_iap_enrich_leads_cron` |  |

Machine-readable definition: `../../../schemas/data/entities/crm.lead.json`; views: `../../../schemas/interfaces/views/crm.lead.json`.
