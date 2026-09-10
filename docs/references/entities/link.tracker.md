# Link Tracker (`link.tracker`)

**Transport name:** `link.tracker`  
**Storage name:** `link_tracker`  
**Kind:** persistent entity (one table)  
**Defined by package:** `link_tracker`  
**Extended by packages:** `mass_mailing`, `website_links`

Description: Link Tracker

## Identity and behavior

- Mixins (classical inheritance): `utm.mixin`
- Default ordering: `count DESC`
- Display name field: `short_url`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (15)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `url` | Target uniform resource locator | single line text |  | required |
| `absolute_url` | Absolute uniform resource locator | single line text |  | computed by rule `_compute_absolute_url` (not stored) |
| `short_url` | Tracked uniform resource locator | single line text |  | computed by rule `_compute_short_url` (not stored); searchable through a search rule |
| `redirected_url` | Redirected uniform resource locator | single line text |  | computed by rule `_compute_redirected_url` (not stored) |
| `short_url_host` | Host of the short uniform resource locator | single line text |  | computed by rule `_compute_short_url_host` (not stored) |
| `title` | Page Title | single line text |  |  |
| `label` | Button label | single line text |  |  |
| `link_code_ids` | Codes | one to many | `link.tracker.code` | inverse field `link_id` |
| `code` | Short uniform resource locator code | single line text |  | computed by rule `_compute_code` (not stored); writable through an inverse rule |
| `link_click_ids` | Clicks | one to many | `link.tracker.click` | inverse field `link_id` |
| `count` | Number of Clicks | integer |  | computed by rule `_compute_count` and stored |
| `campaign_id` | Campaign | many to one |  | on delete of the target: set null |
| `medium_id` | Medium | many to one |  | on delete of the target: set null |
| `source_id` | Source | many to one |  | on delete of the target: set null |
| `mass_mailing_id` | Mass Mailing | many to one | `mailing.mailing` |  |

## Operations (19)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_search_short_url` | search rule | self, operator, value | `link_tracker` |  | Remove the base url and the `/r/` from the given value, and only search for link trackers that have a code that matches the entered value. |
| `_compute_absolute_url` | computation | self | `link_tracker` | depends: `url` |  |
| `_compute_count` | computation | self | `link_tracker` | depends: `link_click_ids.link_id` |  |
| `_compute_short_url` | computation | self | `link_tracker` | depends: `code` |  |
| `_compute_short_url_host` | computation | self | `link_tracker`, `website_links` |  |  |
| `_compute_code` | computation | self | `link_tracker` |  |  |
| `_inverse_code` | inverse computation | self | `link_tracker` |  |  |
| `_compute_redirected_url` | computation | self | `link_tracker` | depends: `url` | Compute the URL to which we will redirect the user.  By default, add UTM values as GET parameters. But if the system parameter `link_tracker.no_external_tracking` is set, we add the UTM values in the URL *only* for URLs that redirect to the local website (base URL). |
| `_get_title_from_url` | computation | self, url | `link_tracker` | model; depends: `url` |  |
| `_check_unicity` | validation | self | `link_tracker` | constrains: | Check that the link trackers are unique. |
| `create` | lifecycle override | self, vals_list | `link_tracker` | model_create_multi |  |
| `search_or_create` | operation | self, vals_list | `link_tracker` | model | Get existing or newly created records matching vals_list items in preserved order supporting duplicates. |
| `convert_links` | operation | self, html, vals, blacklist | `link_tracker` | model |  |
| `_convert_links_text` | internal rule | self, body, vals, blacklist | `link_tracker` |  |  |
| `action_view_statistics` | user action | self | `link_tracker` |  |  |
| `action_visit_page` | user action | self | `link_tracker` |  |  |
| `recent_links` | operation | self, filter, limit | `link_tracker` | model |  |
| `get_url_from_code` | operation | self, code | `link_tracker` | model |  |
| `action_visit_page_statistics` | user action | self | `website_links` |  |  |

## Validation and error messages (4)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_compute_short_url` | UserError | Please enter valid short URL code. | `link_tracker` |
| `_check_unicity` | UserError | Combinations of Link Tracker values (URL, campaign, medium, source, and label) must be unique. The following combinations are already used:  - %(error_lines)s | `link_tracker` |
| `create` | UserError | “%s” is not a valid link, links cannot redirect to the current page. | `link_tracker` |
| `search_or_create` | UserError | '\n'.join(errors) | `link_tracker` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `link_tracker` |
| `base.group_public` | no | no | no | no | `link_tracker` |
| `base.group_system` | yes | yes | yes | yes | `link_tracker` |
| `mass_mailing.group_mass_mailing_user` | yes | yes | yes | yes | `mass_mailing` |
| `website.group_website_designer` | yes | yes | yes | yes | `website_links` |

## Views (8)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `link_tracker.link_tracker_view_search` | search |  | `url`, `label`, `campaign_id`, `medium_id`, `source_id` |  | `Campaign`, `Medium`, `Source` | `link_tracker` |
| `link_tracker.link_tracker_view_form` | form |  | `count`, `short_url_host`, `code`, `url`, `title`, `label`, `campaign_id`, `medium_id`, `source_id` | `Visit Page`, `action_view_statistics` |  | `link_tracker` |
| `link_tracker.link_tracker_view_tree` | list |  | `create_date`, `short_url`, `title`, `url`, `label`, `count`, `campaign_id`, `medium_id`, `source_id` |  |  | `link_tracker` |
| `link_tracker.link_tracker_view_graph` | graph |  | `url`, `count` |  |  | `link_tracker` |
| `mass_mailing.link_tracker_view_search` | xpath | `link_tracker.link_tracker_view_search` | `mass_mailing_id` |  |  | `mass_mailing` |
| `mass_mailing.link_tracker_view_form` | xpath | `link_tracker.link_tracker_view_form` | `mass_mailing_id` |  |  | `mass_mailing` |
| `mass_mailing.link_tracker_view_tree` | xpath | `link_tracker.link_tracker_view_tree` | `mass_mailing_id` |  |  | `mass_mailing` |
| `website_links.link_tracker_view_tree` | xpath | `link_tracker.link_tracker_view_tree` |  | `Statistics` |  | `website_links` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `link_tracker.link_tracker_action` | Link Tracker | list,form,graph |  |  |  | `link_tracker` |
| `link_tracker.link_tracker_action_campaign` | Statistics of Clicks | list,form,graph |  | `{'search_default_campaign_id': active_id}` |  | `link_tracker` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `mass_mailing.link_tracker_menu_mass_mailing` | Link Tracker | `mass_mailing_configuration` | `link_tracker.link_tracker_action` | 10 |  |
| `mass_mailing_sms.link_tracker_menu` | Link Tracker | `mass_mailing_sms_menu_configuration` | `link_tracker.link_tracker_action` | 2 | `mass_mailing.group_mass_mailing_user` |

Machine-readable definition: `../../../schemas/data/entities/link.tracker.json`; views: `../../../schemas/interfaces/views/link.tracker.json`.
