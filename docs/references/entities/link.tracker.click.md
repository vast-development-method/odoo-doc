# Link Tracker Click (`link.tracker.click`)

**Transport name:** `link.tracker.click`  
**Storage name:** `link_tracker_click`  
**Kind:** persistent entity (one table)  
**Defined by package:** `link_tracker`  
**Extended by packages:** `mass_mailing`

Description: Link Tracker Click

## Identity and behavior

- Display name field: `link_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `campaign_id` | campaign tracking parameter Campaign | many to one | `utm.campaign` | related through path `link_id.campaign_id` and stored; indexed (btree_not_null); on delete of the target: set null |
| `link_id` | Link | many to one | `link.tracker` | required; indexed; on delete of the target: cascade |
| `ip` | Internet Protocol | single line text |  |  |
| `country_id` | Country | many to one | `res.country` |  |
| `mailing_trace_id` | Mail Statistics | many to one | `mailing.trace` | indexed (btree_not_null) |
| `mass_mailing_id` | Mass Mailing | many to one | `mailing.mailing` | indexed (btree_not_null) |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_prepare_click_values_from_route` | preparation rule | self, **route_values | `link_tracker`, `mass_mailing` |  |  |
| `add_click` | operation | self, code, **route_values | `link_tracker`, `mass_mailing` | model | Main API to add a click on a link. |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `link_tracker` |
| `base.group_public` | no | no | no | no | `link_tracker` |
| `base.group_system` | yes | yes | yes | yes | `link_tracker` |
| `website.group_website_designer` | yes | yes | yes | yes | `website_links` |

## Views (8)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `link_tracker.link_tracker_click_view_search` | search |  | `link_id`, `country_id` |  | `Link`, `Country` | `link_tracker` |
| `link_tracker.link_tracker_click_view_form` | form |  | `link_id`, `ip`, `country_id` |  |  | `link_tracker` |
| `link_tracker.link_tracker_click_view_tree` | list |  | `link_id`, `ip`, `country_id` |  |  | `link_tracker` |
| `link_tracker.link_tracker_click_view_graph` | graph |  | `link_id`, `ip`, `country_id` |  |  | `link_tracker` |
| `mass_mailing.link_tracker_click_view_search` | xpath | `link_tracker.link_tracker_click_view_search` | `campaign_id`, `mass_mailing_id` |  |  | `mass_mailing` |
| `mass_mailing.link_tracker_click_view_form` | xpath | `link_tracker.link_tracker_click_view_form` | `campaign_id`, `mass_mailing_id`, `mailing_trace_id` |  |  | `mass_mailing` |
| `mass_mailing.link_tracker_click_view_tree` | xpath | `link_tracker.link_tracker_click_view_tree` | `campaign_id`, `mass_mailing_id` |  |  | `mass_mailing` |
| `mass_mailing.link_tracker_click_view_graph` | xpath | `link_tracker.link_tracker_click_view_graph` | `campaign_id`, `mass_mailing_id` |  |  | `mass_mailing` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `link_tracker.link_tracker_click_action_statistics` | Click Statistics | graph,list,form | `[]` | `{'search_default_groupby_country_id': 1}` |  | `link_tracker` |

Machine-readable definition: `../../../schemas/data/entities/link.tracker.click.json`; views: `../../../schemas/interfaces/views/link.tracker.click.json`.
