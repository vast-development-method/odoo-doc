# Website Visitor (`website.visitor`)

**Transport name:** `website.visitor`  
**Storage name:** `website_visitor`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website`  
**Extended by packages:** `website_sale`, `website_event`, `website_event_track`, `website_crm`, `website_livechat`, `website_sms`, `website_crm_sms`

Description: Website Visitor

## Identity and behavior

- Default ordering: `id DESC`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (36)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | related through path `partner_id.name` |
| `access_token` | Access Token | single line text |  | required; default computed dynamically (_get_access_token); not copied on duplication |
| `website_id` | Website | many to one | `website` | read only |
| `partner_id` | Contact | many to one | `res.partner` | computed by rule `_compute_partner_id` and stored; indexed (btree_not_null); Help: Partner of the last logged in user. |
| `partner_image` | Partner Image | binary |  | related through path `partner_id.image_1920` |
| `country_id` | Country | many to one | `res.country` | read only |
| `country_flag` | Country Flag | single line text |  | related through path `country_id.image_url` |
| `lang_id` | Language | many to one | `res.lang` | Help: Language from the website when visitor has been created |
| `timezone` | Timezone | selection |  |  |
| `email` | Email | single line text |  | computed by rule `_compute_email_phone` (not stored) |
| `mobile` | Mobile | single line text |  | computed by rule `_compute_email_phone` (not stored) |
| `visit_count` | # Visits | integer |  | read only; default `1`; Help: A new visit is considered if last connection was more than 8 hours ago. |
| `website_track_ids` | Visited Pages History | one to many | `website.track` | read only; inverse field `visitor_id` |
| `visitor_page_count` | Page Views | integer |  | computed by rule `_compute_page_statistics` (not stored); Help: Total number of visits on tracked pages |
| `page_ids` | Visited Pages | many to many | `website.page` | computed by rule `_compute_page_statistics` (not stored); searchable through a search rule; visible only to groups `website.group_website_designer` |
| `page_count` | # Visited Pages | integer |  | computed by rule `_compute_page_statistics` (not stored); Help: Total number of tracked page visited |
| `last_visited_page_id` | Last Visited Page | many to one | `website.page` | computed by rule `_compute_last_visited_page_id` (not stored) |
| `create_date` | First Connection | date and time |  | read only |
| `last_connection_datetime` | Last Connection | date and time |  | read only; default computed dynamically (fields.Datetime.now); Help: Last page view date |
| `time_since_last_action` | Last action | single line text |  | computed by rule `_compute_time_statistics` (not stored); Help: Time since last page view. E.g.: 2 minutes ago |
| `is_connected` | Is connected? | boolean |  | computed by rule `_compute_time_statistics` (not stored); Help: A visitor is considered as connected if his last page view was within the last 5 minutes. |
| `visitor_product_count` | Product Views | integer |  | computed by rule `_compute_product_statistics` (not stored); Help: Total number of views on products |
| `product_ids` | Visited Products | many to many | `product.product` | computed by rule `_compute_product_statistics` (not stored) |
| `product_count` | Products Views | integer |  | computed by rule `_compute_product_statistics` (not stored); Help: Total number of product viewed |
| `event_registration_ids` | Event Registrations | one to many | `event.registration` | visible only to groups `event.group_event_registration_desk`; inverse field `visitor_id` |
| `event_registration_count` | # Registrations | integer |  | computed by rule `_compute_event_registration_count` (not stored); visible only to groups `event.group_event_registration_desk` |
| `event_registered_ids` | Registered Events | many to many | `event.event` | computed by rule `_compute_event_registered_ids` (not stored); searchable through a search rule; visible only to groups `event.group_event_registration_desk` |
| `event_track_visitor_ids` | Track Visitors | one to many | `event.track.visitor` | visible only to groups `event.group_event_user`; inverse field `visitor_id` |
| `event_track_wishlisted_ids` | Wishlisted Tracks | many to many | `event.track` | computed by rule `_compute_event_track_wishlisted_ids` (not stored); searchable through a search rule; visible only to groups `event.group_event_user` |
| `event_track_wishlisted_count` | # Wishlisted | integer |  | computed by rule `_compute_event_track_wishlisted_ids` (not stored); visible only to groups `event.group_event_user` |
| `lead_ids` | Leads | many to many | `crm.lead` | visible only to groups `sales_team.group_sale_salesman` |
| `lead_count` | # Leads | integer |  | computed by rule `_compute_lead_count` (not stored); visible only to groups `sales_team.group_sale_salesman` |
| `livechat_operator_id` | Speaking with | many to one | `res.partner` | computed by rule `_compute_livechat_operator_id` and stored; indexed (btree_not_null) |
| `livechat_operator_name` | Operator Name | single line text |  | related through path `livechat_operator_id.name` |
| `discuss_channel_ids` | Visitor's livechat channels | one to many | `discuss.channel` | read only; inverse field `livechat_visitor_id` |
| `session_count` | # Sessions | integer |  | computed by rule `_compute_session_count` (not stored) |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_access_token_unique` | Constraint | `unique(access_token)` | Access token should be unique. | `website` |

## Operations (38)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_access_token` | preparation rule | self | `website` |  | Either the user's partner.id or a hash. |
| `_compute_display_name` | computation | self | `website_event`, `website` | depends: `partner_id`; depends: `partner_id`, `event_registration_ids.name` | If there is an event registration for an anonymous visitor, use that registered attendee name as visitor name. |
| `_compute_partner_id` | computation | self | `website` | depends: `access_token` |  |
| `_compute_email_phone` | computation | self | `website_crm`, `website_event`, `website` | depends: `partner_id.email_normalized`, `partner_id.phone`; depends: `event_registration_ids.email`, `event_registration_ids.phone`; depends: `partner_id.email_normalized`, `partner_id.phone`, `lead_ids.email_normalized`, `lead_ids.phone` |  |
| `_compute_page_statistics` | computation | self | `website` | depends: `website_track_ids` |  |
| `_search_page_ids` | search rule | self, operator, value | `website` |  |  |
| `_compute_last_visited_page_id` | computation | self | `website` | depends: `website_track_ids.page_id` |  |
| `_compute_time_statistics` | computation | self | `website` | depends: `last_connection_datetime` |  |
| `_check_for_message_composer` | validation | self | `website_crm`, `website` |  | Purpose of this method is to actualize visitor model prior to contacting him. Used notably for inheritance purpose, when dealing with leads that could update the visitor model. |
| `_prepare_message_composer_context` | preparation rule | self | `website_crm`, `website` |  |  |
| `action_send_mail` | user action | self | `website` |  |  |
| `_upsert_visitor` | internal rule | self, access_token, force_track_values | `website_livechat`, `website` |  | Based on the given `access_token`, either create or return the related visitor if exists, through a single raw SQL UPSERT Query.  It will also create a tracking record if requested, in the same query.  :param access_token: token to be used to upsert the visitor :param force_track_values: an optional dict to create a track at the     same time. :return: a tuple containing the visitor id and the upsert result (either     `inserted` or `updated). |
| `_get_visitor_from_request` | preparation rule | self, force_create, force_track_values | `website` |  | Return the visitor as sudo from the request.  :param force_create: force a visitor creation if no visitor exists :param force_track_values: an optional dict to create a track at the     same time. :return: the website visitor if exists or forced, empty recordset     otherwise. |
| `_handle_webpage_dispatch` | internal rule | self, website_page | `website` |  | Create a website.visitor if the http request object is a tracked website.page or a tracked ir.ui.view. Since this method is only called on tracked elements, the last_connection_datetime might not be accurate as the visitor could have been visiting only untracked page during his last visit. |
| `_add_tracking` | internal rule | self, domain, website_track_values | `website` |  | Add the track and update the visitor |
| `_merge_visitor` | internal rule | self, target | `website_crm`, `website_event_track`, `website_event`, `website_livechat`, `website` |  | Merge an anonymous visitor data to a partner visitor then unlink that anonymous visitor. Purpose is to try to aggregate as much sub-records (tracked pages, leads, ...) as possible. It is especially useful to aggregate data from the same user on different devices.  This method is meant to be overridden for other modules to merge their own anonymous visitor data to the partner visitor before unlink.  This method is only called after the user logs in.  :param target: main visitor, target of link process; |
| `_cron_unlink_old_visitors` | background operation | self, batch_size | `website` |  | Unlink inactive visitors (see '_inactive_visitors_domain' for details).  Visitors were previously archived but we came to the conclusion that archived visitors have very little value and bloat the database for no reason. |
| `_inactive_visitors_domain` | internal rule | self | `website_crm`, `website_event_track`, `website_event`, `website` |  | This method defines the domain of visitors that can be cleaned. By default visitors not linked to any partner and not active for 'website.visitor.live.days' days (default being 60) are considered as inactive.  This method is meant to be overridden by sub-modules to further refine inactivity conditions. |
| `_update_visitor_timezone` | internal rule | self, timezone | `website` |  | We need to do this part here to avoid concurrent updates error. |
| `_update_visitor_last_visit` | internal rule | self | `website` |  |  |
| `_get_visitor_timezone` | preparation rule | self | `website` |  |  |
| `_compute_product_statistics` | computation | self | `website_sale` | depends: `website_track_ids` |  |
| `_add_viewed_product` | internal rule | self, product_id | `website_sale` |  | add a website_track with a page marked as viewed |
| `_compute_event_registration_count` | computation | self | `website_event` | depends: `event_registration_ids` |  |
| `_compute_event_registered_ids` | computation | self | `website_event` | depends: `event_registration_ids` |  |
| `_search_event_registered_ids` | search rule | self, operator, operand | `website_event` |  | Search visitors with terms on events within their event registrations. E.g. [('event_registered_ids', 'in', [1, 2])] should return visitors having a registration on events 1, 2 as well as their children for notification purpose. |
| `_compute_event_track_wishlisted_ids` | computation | self | `website_event_track` | depends: `event_track_visitor_ids.track_id`, `event_track_visitor_ids.is_wishlisted` |  |
| `_search_event_track_wishlisted_ids` | search rule | self, operator, operand | `website_event_track` |  | Search visitors with terms on wishlisted tracks. E.g. [('event_track_wishlisted_ids', 'in', [1, 2])] should return visitors having wishlisted tracks 1, 2. |
| `_compute_lead_count` | computation | self | `website_crm` | depends: `lead_ids` |  |
| `_auto_init` | lifecycle override | self | `website_livechat` |  |  |
| `_compute_livechat_operator_id` | computation | self | `website_livechat` | depends: `discuss_channel_ids.livechat_end_dt`, `discuss_channel_ids.livechat_operator_id` |  |
| `_compute_session_count` | computation | self | `website_livechat` | depends: `discuss_channel_ids` |  |
| `action_send_chat_request` | user action | self | `website_livechat` |  | Send a chat request to website_visitor(s). This creates a chat_request and a discuss_channel with livechat active flag. But for the visitor to get the chat request, the operator still has to speak to the visitor. The visitor will receive the chat request the next time he navigates to a website page. (see _handle_webpage_dispatch for next step) |
| `_field_store_repr` | internal rule | self, field_name | `website_livechat` |  |  |
| `_get_visitor_history` | preparation rule | self | `website_livechat` |  |  |
| `_check_for_sms_composer` | validation | self | `website_crm_sms`, `website_sms` |  | Purpose of this method is to actualize visitor model prior to contacting him. Used notably for inheritance purpose, when dealing with leads that could update the visitor model. |
| `_prepare_sms_composer_context` | preparation rule | self | `website_crm_sms`, `website_sms` |  |  |
| `action_send_sms` | user action | self | `website_sms` |  |  |

## Validation and error messages (6)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `action_send_mail` | UserError | There are no contact and/or no email linked to this visitor. | `website` |
| `_search_event_registered_ids` | UserError | Unsupported 'Not In' operation on visitors registrations | `website_event` |
| `_search_event_track_wishlisted_ids` | UserError | Unsupported 'Not In' operation on track wishlist visitors | `website_event_track` |
| `action_send_chat_request` | UserError | Recipients are not available. Please refresh the page to get latest visitors status. | `website_livechat` |
| `action_send_chat_request` | UserError | No Livechat Channel allows you to send a chat request for website %s. | `website_livechat` |
| `action_send_sms` | UserError | There are no contact and/or no phone or mobile numbers linked to this visitor. | `website_sms` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `website.group_website_designer` | no | yes | yes | yes | `website` |
| `base.group_system` | no | yes | yes | yes | `website` |
| `sales_team.group_sale_salesman` | no | yes | no | no | `website_crm` |
| `event.group_event_registration_desk` | no | yes | yes | no | `website_event` |
| `im_livechat.im_livechat_group_user` | no | yes | no | no | `website_livechat` |

## Views (23)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website.website_visitor_view_kanban` | kanban |  | `country_id`, `country_flag`, `email`, `is_connected`, `partner_image`, `partner_id`, `display_name`, `country_flag`, `time_since_last_action`, `visit_count`, `last_visited_page_id`, `page_count` | `action_send_mail` |  | `website` |
| `website.website_visitor_view_form` | form |  | `visit_count`, `visitor_page_count`, `country_flag`, `display_name`, `is_connected`, `partner_id`, `email`, `mobile`, `country_id`, `lang_id`, `website_id`, `create_date`, `last_connection_datetime`, `page_ids` | `Send Email`, , , , `%(website.website_visitor_page_action)d` |  | `website` |
| `website.website_visitor_view_tree` | list |  | `country_flag`, `display_name`, `create_date`, `last_connection_datetime`, `lang_id`, `country_id`, `visit_count`, `page_ids`, `last_visited_page_id`, `is_connected`, `email` | `Email` |  | `website` |
| `website.website_visitor_view_search` | search |  | `name`, `lang_id`, `country_id`, `visit_count`, `page_ids` |  | `Last 7 Days`, `Unregistered`, `Contacts`, `Connected`, `Country`, `Timezone`, `Language`, `# Visits`, `Website`, `First Connection`, `Last Connection` | `website` |
| `website.website_visitor_view_graph` | graph |  | `last_connection_datetime` |  |  | `website` |
| `website_crm.website_visitor_view_form` | xpath | `website.website_visitor_view_form` | `lead_count` | `%(website_crm.crm_lead_action_from_visitor)d` |  | `website_crm` |
| `website_crm.website_visitor_view_tree` | xpath | `website.website_visitor_view_tree` | `lead_count` |  |  | `website_crm` |
| `website_crm.website_visitor_view_search` | xpath | `website.website_visitor_view_search` |  |  |  | `website_crm` |
| `website_crm.website_visitor_view_kanban` | xpath | `website.website_visitor_view_kanban` | `lead_count` |  |  | `website_crm` |
| `website_event.website_visitor_view_tree` | xpath | `website.website_visitor_view_tree` | `event_registration_count` |  |  | `website_event` |
| `website_event.website_visitor_view_form` | xpath | `website.website_visitor_view_form` | `event_registration_count` | `%(website_event.event_registration_action_from_visitor)d` |  | `website_event` |
| `website_event_track.website_visitor_view_tree` | xpath | `website_event.website_visitor_view_tree` | `event_track_wishlisted_count` |  |  | `website_event_track` |
| `website_event_track.website_visitor_view_form` | xpath | `website_event.website_visitor_view_form` | `event_track_wishlisted_count` | `%(website_event_track.event_track_action_from_visitor)d` |  | `website_event_track` |
| `website_livechat.website_visitor_view_kanban` | field | `website.website_visitor_view_kanban` | `country_id`, `livechat_operator_id`, `session_count` |  |  | `website_livechat` |
| `website_livechat.website_visitor_view_form` | xpath | `website.website_visitor_view_form` |  | `Send chat request` |  | `website_livechat` |
| `website_livechat.website_visitor_view_tree` | xpath | `website.website_visitor_view_tree` | `livechat_operator_id` | `Chat` |  | `website_livechat` |
| `website_livechat.website_visitor_view_search` | xpath | `website.website_visitor_view_search` |  |  | `Available`, `In Conversation` | `website_livechat` |
| `website_sale.website_sale_visitor_view_form` | xpath | `website.website_visitor_view_form` | `visitor_product_count` | `%(website_sale.website_sale_visitor_product_action)d` |  | `website_sale` |
| `website_sale.website_sale_visitor_view_tree` | field | `website.website_visitor_view_tree` | `page_ids`, `product_ids` |  |  | `website_sale` |
| `website_sale.website_sale_visitor_view_kanban` | field | `website.website_visitor_view_kanban` | `country_id`, `product_ids` |  |  | `website_sale` |
| `website_sms.website_visitor_view_form` | xpath | `website.website_visitor_view_form` |  | `Send SMS` |  | `website_sms` |
| `website_sms.website_visitor_view_kanban` | field | `website.website_visitor_view_kanban` | `country_id`, `mobile` |  |  | `website_sms` |
| `website_sms.website_visitor_view_tree` | xpath | `website.website_visitor_view_tree` | `mobile` |  |  | `website_sms` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website.website_visitors_action` | Visitors | kanban,list,form,graph |  | `{'search_default_filter_last_7_days':1}` |  | `website` |
| `website_event_track.website_visitor_action_from_track` | Visitors Wishlist | kanban,list,form,graph | `[('event_track_wishlisted_ids', 'in', [active_id])]` |  |  | `website_event_track` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `website.website_visitor_menu` | Visitors | `website.menu_reporting` | `website.website_visitors_action` | 40 |  |
| `website_livechat.website_livechat_visitor_menu` | Visitors | `im_livechat.menu_livechat_root` | `website.website_visitors_action` | 15 | `im_livechat.im_livechat_group_user` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `website_livechat.website_livechat_send_chat_request_action_server` | Send Chat Requests | code |  | yes |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `website.website_visitor_cron` | Website Visitor : clean inactive visitors | 1 days | `_cron_unlink_old_visitors` |  |

Machine-readable definition: `../../../schemas/data/entities/website.visitor.json`; views: `../../../schemas/interfaces/views/website.visitor.json`.
