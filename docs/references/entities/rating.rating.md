# Rating (`rating.rating`)

**Transport name:** `rating.rating`  
**Storage name:** `rating_rating`  
**Kind:** persistent entity (one table)  
**Defined by package:** `rating`  
**Extended by packages:** `im_livechat`, `portal_rating`

Description: Rating

## Identity and behavior

- Default ordering: `write_date desc, id desc`
- Display name field: `res_name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (27)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `create_date` | Submitted on | date and time |  |  |
| `res_name` | Resource name | single line text |  | computed by rule `_compute_res_name` and stored |
| `res_model_id` | Related Document Model | many to one | `ir.model` | indexed; on delete of the target: cascade |
| `res_model` | Document Model | single line text |  | read only; related through path `res_model_id.model` and stored; indexed |
| `res_id` | Document | many to one by reference |  | required; indexed |
| `resource_ref` | Resource Ref | reference |  | read only; computed by rule `_compute_resource_ref` (not stored); values provided by rule `_selection_target_model` |
| `parent_res_name` | Parent Document Name | single line text |  | computed by rule `_compute_parent_res_name` and stored |
| `parent_res_model_id` | Parent Related Document Model | many to one | `ir.model` | indexed; on delete of the target: cascade |
| `parent_res_model` | Parent Document Model | single line text |  | related through path `parent_res_model_id.model` and stored; indexed |
| `parent_res_id` | Parent Document | integer |  | indexed |
| `parent_ref` | Parent Ref | reference |  | read only; computed by rule `_compute_parent_ref` (not stored); values provided by rule `_selection_target_model` |
| `rated_partner_id` | Rated Operator | many to one | `res.partner` |  |
| `rated_partner_name` | Rated Partner Name | single line text |  | related through path `rated_partner_id.name` |
| `partner_id` | Customer | many to one | `res.partner` |  |
| `rating` | Rating Value | float |  | default ; aggregated with avg |
| `rating_image` | Image | binary |  | computed by rule `_compute_rating_image` (not stored) |
| `rating_image_url` | Image uniform resource locator | single line text |  | computed by rule `_compute_rating_image` (not stored) |
| `rating_text` | Rating | selection |  | read only; computed by rule `_compute_rating_text` and stored |
| `feedback` | Comment | multi line text |  |  |
| `message_id` | Message | many to one | `mail.message` | indexed; on delete of the target: cascade |
| `is_internal` | Visible Internally Only | boolean |  | related through path `message_id.is_internal` and stored |
| `access_token` | Security Token | single line text |  | default computed dynamically (_default_access_token) |
| `consumed` | Filled Rating | boolean |  |  |
| `rated_on` | Rated On | date and time |  |  |
| `publisher_comment` | Publisher comment | multi line text |  |  |
| `publisher_id` | Commented by | many to one | `res.partner` | read only; indexed (btree_not_null); on delete of the target: set null |
| `publisher_datetime` | Commented on | date and time |  | read only |

## Database constraints and indexes (3)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_rating_range` | Constraint | `check(rating >= 0 and rating <= 5)` | Rating should be between 0 and 5 | `rating` |
| `_consumed_idx` | Index | `(res_model, res_id, write_date) WHERE consumed IS TRUE` |  | `rating` |
| `_parent_consumed_idx` | Index | `(parent_res_model, parent_res_id, write_date) WHERE consumed IS TRUE` |  | `rating` |

## Operations (19)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_access_token` | preparation rule | self | `rating` | model |  |
| `_selection_target_model` | internal rule | self | `rating` | model |  |
| `_compute_res_name` | computation | self | `im_livechat`, `rating` | depends: `res_model`, `res_id` |  |
| `_compute_resource_ref` | computation | self | `rating` | depends: `res_model`, `res_id` |  |
| `_compute_parent_ref` | computation | self | `rating` | depends: `parent_res_model`, `parent_res_id` |  |
| `_compute_parent_res_name` | computation | self | `rating` | depends: `parent_res_model`, `parent_res_id` |  |
| `_get_rating_image_filename` | preparation rule | self | `rating` |  |  |
| `_compute_rating_image` | computation | self | `rating` | depends: `rating` |  |
| `_compute_rating_text` | computation | self | `rating` | depends: `rating` |  |
| `create` | lifecycle override | self, vals_list | `portal_rating`, `rating` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `portal_rating`, `rating` |  |  |
| `unlink` | lifecycle override | self | `rating` |  |  |
| `_find_parent_data` | internal rule | self, values | `rating` |  | Determine the parent res_model/res_id, based on the values to create or write |
| `reset` | operation | self | `rating` |  |  |
| `action_open_rated_object` | user action | self | `im_livechat`, `rating` |  |  |
| `_classify_by_model` | internal rule | self | `rating` |  | To ease batch computation of various ratings related methods they are classified by model. Ratings not linked to a valid record through res_model / res_id are ignored.  :returns: for each model having at least one rating in self, have   a sub-dict containing     * ratings: ratings related to that model;     * record IDs: records linked to the ratings of that model, in same       order; :rtype: dict |
| `_to_store_defaults` | internal rule | self, target | `rating` |  |  |
| `_check_synchronize_publisher_values` | validation | self | `portal_rating` |  | Either current user is a member of website restricted editor group (done here by fetching the group record then using has_group, as it may not be defined and we do not want to make a complete bridge module just for that). Either write access on document is granted. |
| `_synchronize_publisher_values` | internal rule | self, values | `portal_rating` |  | Force publisher partner and date if not given in order to have coherent values. Those fields are readonly as they are not meant to be modified manually, behaving like a tracking. |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_synchronize_publisher_values` | AccessError | Updating rating comment require write access on related record | `portal_rating` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | no | `rating` |
| `base.group_public` | no | no | no | no | `rating` |
| `base.group_portal` | no | no | no | no | `rating` |
| `base.group_system` | yes | yes | yes | yes | `rating` |

## Views (19)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `portal_rating.rating_rating_view_form` | xpath | `rating.rating_rating_view_form` | `publisher_comment`, `publisher_id`, `publisher_datetime` |  |  | `portal_rating` |
| `project.rating_rating_view_tree_project` | field | `rating.rating_rating_view_tree` | `res_name` |  |  | `project` |
| `project.rating_rating_view_form_project` | xpath | `rating.rating_rating_view_form_text` |  |  |  | `project` |
| `project.rating_rating_view_pivot` | xpath | `rating.rating_rating_view_pivot` |  |  |  | `project` |
| `project.rating_rating_view_graph` | xpath | `rating.rating_rating_view_graph` |  |  |  | `project` |
| `project.rating_rating_view_search_project` | xpath | `rating.rating_rating_view_search` | `parent_res_name`, `res_name` |  |  | `project` |
| `rating.rating_rating_view_tree` | list |  | `create_date`, `rated_partner_id`, `partner_id`, `parent_res_name`, `res_name`, `feedback`, `rating_text` |  |  | `rating` |
| `rating.rating_rating_view_form` | form |  | `resource_ref`, `res_name`, `parent_ref`, `parent_res_name`, `rated_partner_id`, `rating`, `is_internal`, `consumed`, `partner_id`, `rating_image`, `rating_text`, `rated_on`, `feedback` |  |  | `rating` |
| `rating.rating_rating_view_form_text` | xpath | `rating.rating_rating_view_form` | `rating_text` |  |  | `rating` |
| `rating.rating_rating_view_kanban` | kanban |  | `rating_image`, `rated_partner_name`, `partner_id`, `res_name`, `rated_on`, `feedback` |  |  | `rating` |
| `rating.rating_rating_view_kanban_stars` | kanban |  | `rating`, `partner_id`, `res_name`, `rated_on`, `feedback` |  |  | `rating` |
| `rating.rating_rating_view_pivot` | pivot |  | `rated_partner_id`, `rated_on`, `rating`, `parent_res_id`, `res_id` |  |  | `rating` |
| `rating.rating_rating_view_graph` | graph |  | `rated_on`, `rating`, `parent_res_id`, `res_id` |  |  | `rating` |
| `rating.rating_rating_view_search` | search |  | `rated_partner_id`, `rating`, `partner_id`, `res_name`, `res_id`, `parent_res_name` |  | `My Ratings`, `Happy`, `Neutral`, `Unhappy`, `Rated On`, `Last 7 Days`, `Last 30 Days`, `Last 365 Days`, `Rated Operator`, `Customer`, `Rating`, `Resource`, `Submitted on` | `rating` |
| `website_slides.rating_rating_view_search_slide_channel` | xpath | `rating.rating_rating_view_search` |  |  | `Course` | `website_slides` |
| `website_slides.rating_rating_view_graph_slide_channel` | graph |  | `res_name`, `rating`, `res_id`, `parent_res_id` |  |  | `website_slides` |
| `website_slides.rating_rating_view_pivot_slide_channel` | pivot |  | `res_name`, `rating_text`, `rating` |  |  | `website_slides` |
| `website_slides.rating_rating_view_tree_slide_channel` | list |  | `rated_on`, `partner_id`, `res_name`, `rating`, `feedback` |  |  | `website_slides` |
| `website_slides.rating_rating_view_form_slides` | form |  | `partner_id`, `resource_ref`, `rated_on`, `rating`, `is_internal`, `feedback` |  |  | `website_slides` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `project.rating_rating_action_view_project_rating` | Ratings | kanban,list,graph,pivot,form | `[('consumed','=',True), ('parent_res_model','=','project.project'), ('parent_res_id', '=', active_id)]` |  |  | `project` |
| `project.rating_rating_action_task` | Ratings | kanban,list,pivot,graph,form | `[('res_model', '=', 'project.task'), ('res_id', '=', active_id), ('consumed', '=', True)]` |  |  | `project` |
| `project.rating_rating_action_project_report` | Customer Ratings | kanban,list,pivot,graph,form | `[('parent_res_model','=','project.project'), ('consumed', '=', True)]` | `{             'search_default_filter_rated_on': 1,             'graph_groupbys': ['rated_partner_id'],         }` |  | `project` |
| `rating.rating_rating_action` | Ratings | kanban,list,graph,pivot,form |  |  |  | `rating` |
| `website_slides.rating_rating_action_slide_channel` | Reviews | kanban,list,graph,pivot,form | `[('consumed', '=', True), ('res_model', '=', 'slide.channel')]` | `{}` |  | `website_slides` |

Machine-readable definition: `../../../schemas/data/entities/rating.rating.json`; views: `../../../schemas/interfaces/views/rating.rating.json`.
