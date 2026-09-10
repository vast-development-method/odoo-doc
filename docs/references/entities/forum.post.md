# Forum Post (`forum.post`)

**Transport name:** `forum.post`  
**Storage name:** `forum_post`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_forum`

Description: Forum Post

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `website.seo.metadata`, `website.searchable.mixin`
- Default ordering: `is_correct DESC, vote_count DESC, last_activity_date DESC`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (58)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Title | single line text |  |  |
| `forum_id` | Forum | many to one | `forum.forum` | required; indexed |
| `content` | Content | rich text |  |  |
| `plain_content` | Plain Content | multi line text |  | computed by rule `_compute_plain_content` and stored |
| `tag_ids` | Tags | many to many | `forum.tag` | association table `forum_tag_rel` |
| `state` | Status | selection |  | default `active` |
| `views` | Views | integer |  | read only; default ; not copied on duplication |
| `active` | Active | boolean |  | default `True` |
| `website_message_ids` | Website Message | one to many |  | restricted by domain `lambda self: [('model', '=', self._name), ('message_type', 'in', ['email', 'comment', 'email_outgoing'])]` |
| `website_url` | Website uniform resource locator | single line text |  | computed by rule `_compute_website_url` (not stored) |
| `website_id` | Website | many to one |  | read only; related through path `forum_id.website_id` |
| `create_date` | Asked on | date and time |  | read only; indexed |
| `create_uid` | Created by | many to one | `res.users` | read only; indexed |
| `write_date` | Updated on | date and time |  | read only; indexed |
| `last_activity_date` | Last activity on | date and time |  | required; read only; default computed dynamically (fields.Datetime.now); Help: Field to keep track of a post's last activity. Updated whenever it is replied to, or when a comment is added on the post or one of its replies. |
| `write_uid` | Updated by | many to one | `res.users` | read only; indexed |
| `relevancy` | Relevance | float |  | computed by rule `_compute_relevancy` and stored |
| `vote_ids` | Votes | one to many | `forum.post.vote` | inverse field `post_id` |
| `user_vote` | My Vote | integer |  | computed by rule `_compute_user_vote` (not stored) |
| `vote_count` | Total Votes | integer |  | computed by rule `_compute_vote_count` and stored |
| `favourite_ids` | Favourite | many to many | `res.users` |  |
| `user_favourite` | Is Favourite | boolean |  | computed by rule `_compute_user_favourite` (not stored) |
| `favourite_count` | Favorite | integer |  | computed by rule `_compute_favorite_count` and stored |
| `is_correct` | Correct | boolean |  | Help: Correct answer or answer accepted |
| `parent_id` | Question | many to one | `forum.post` | read only; indexed; on delete of the target: cascade |
| `self_reply` | Reply to own question | boolean |  | computed by rule `_compute_self_reply` and stored |
| `child_ids` | Post Answers | one to many | `forum.post` | restricted by domain `[('forum_id', '=', forum_id)]`; inverse field `parent_id` |
| `child_count` | Answers | integer |  | computed by rule `_compute_child_count` and stored |
| `uid_has_answered` | Has Answered | boolean |  | computed by rule `_compute_uid_has_answered` (not stored) |
| `has_validated_answer` | Is answered | boolean |  | computed by rule `_compute_has_validated_answer` and stored |
| `flag_user_id` | Flagged by | many to one | `res.users` |  |
| `moderator_id` | Reviewed by | many to one | `res.users` | read only |
| `closed_reason_id` | Reason | many to one | `forum.post.reason` | not copied on duplication |
| `closed_uid` | Closed by | many to one | `res.users` | read only; not copied on duplication |
| `closed_date` | Closed on | date and time |  | read only; not copied on duplication |
| `karma_accept` | Convert comment to answer | integer |  | computed by rule `_compute_post_karma_rights` (not stored) |
| `karma_edit` | Karma to edit | integer |  | computed by rule `_compute_post_karma_rights` (not stored) |
| `karma_close` | Karma to close | integer |  | computed by rule `_compute_post_karma_rights` (not stored) |
| `karma_unlink` | Karma to unlink | integer |  | computed by rule `_compute_post_karma_rights` (not stored) |
| `karma_comment` | Karma to comment | integer |  | computed by rule `_compute_post_karma_rights` (not stored) |
| `karma_comment_convert` | Karma to convert comment to answer | integer |  | computed by rule `_compute_post_karma_rights` (not stored) |
| `karma_flag` | Flag a post as offensive | integer |  | computed by rule `_compute_post_karma_rights` (not stored) |
| `can_ask` | Can Ask | boolean |  | computed by rule `_compute_post_karma_rights` (not stored) |
| `can_answer` | Can Answer | boolean |  | computed by rule `_compute_post_karma_rights` (not stored) |
| `can_accept` | Can Accept | boolean |  | computed by rule `_compute_post_karma_rights` (not stored) |
| `can_edit` | Can Edit | boolean |  | computed by rule `_compute_post_karma_rights` (not stored) |
| `can_close` | Can Close | boolean |  | computed by rule `_compute_post_karma_rights` (not stored) |
| `can_unlink` | Can Unlink | boolean |  | computed by rule `_compute_post_karma_rights` (not stored) |
| `can_upvote` | Can Upvote | boolean |  | computed by rule `_compute_post_karma_rights` (not stored) |
| `can_downvote` | Can Downvote | boolean |  | computed by rule `_compute_post_karma_rights` (not stored) |
| `can_comment` | Can Comment | boolean |  | computed by rule `_compute_post_karma_rights` (not stored) |
| `can_comment_convert` | Can Convert to Comment | boolean |  | computed by rule `_compute_post_karma_rights` (not stored) |
| `can_view` | Can View | boolean |  | computed by rule `_compute_post_karma_rights` (not stored); searchable through a search rule |
| `can_display_biography` | Is the author's biography visible from his post | boolean |  | computed by rule `_compute_post_karma_rights` (not stored) |
| `can_post` | Can Automatically be Validated | boolean |  | computed by rule `_compute_post_karma_rights` (not stored) |
| `can_flag` | Can Flag | boolean |  | computed by rule `_compute_post_karma_rights` (not stored) |
| `can_moderate` | Can Moderate | boolean |  | computed by rule `_compute_post_karma_rights` (not stored) |
| `can_use_full_editor` | Can Use Full Editor | boolean |  | computed by rule `_compute_post_karma_rights` (not stored) |

## Selection values

### `state` (Status)

| Value | Label |
|---|---|
| `active` | Active |
| `pending` | Waiting Validation |
| `close` | Closed |
| `offensive` | Offensive |
| `flagged` | Flagged |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Operations (45)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_parent_id` | validation | self | `website_forum` | constrains: `parent_id` |  |
| `_compute_plain_content` | computation | self | `website_forum` | depends: `content` |  |
| `_compute_website_url` | computation | self | `website_forum` | depends: `name` |  |
| `_compute_relevancy` | computation | self | `website_forum` | depends: `vote_count`, `forum_id.relevancy_post_vote`, `forum_id.relevancy_time_decay` |  |
| `_compute_user_vote` | computation | self | `website_forum` | depends_context: `uid` |  |
| `_compute_vote_count` | computation | self | `website_forum` | depends: `vote_ids.vote` |  |
| `_compute_user_favourite` | computation | self | `website_forum` | depends_context: `uid` |  |
| `_compute_favorite_count` | computation | self | `website_forum` | depends: `favourite_ids` |  |
| `_compute_self_reply` | computation | self | `website_forum` | depends: `create_uid`, `parent_id` |  |
| `_compute_child_count` | computation | self | `website_forum` | depends: `child_ids` |  |
| `_compute_uid_has_answered` | computation | self | `website_forum` | depends_context: `uid` |  |
| `_compute_has_validated_answer` | computation | self | `website_forum` | depends: `child_ids.is_correct` |  |
| `_compute_post_karma_rights` | computation | self | `website_forum` | depends_context: `uid` |  |
| `_search_can_view` | search rule | self, operator, value | `website_forum` |  |  |
| `_default_website_meta` | preparation rule | self | `website_forum` |  |  |
| `create` | lifecycle override | self, vals_list | `website_forum` | model_create_multi |  |
| `unlink` | lifecycle override | self | `website_forum` |  |  |
| `write` | lifecycle override | self, vals | `website_forum` |  |  |
| `_get_access_action` | preparation rule | self, access_uid, force_website | `website_forum` |  | Instead of the classic form view, redirect to the post on the website directly |
| `_unlink_if_enough_karma` | internal rule | self | `website_forum` | ondelete |  |
| `_update_content` | internal rule | self, content, forum_id | `website_forum` |  |  |
| `_notify_state_update` | internal rule | self | `website_forum` |  |  |
| `reopen` | operation | self | `website_forum` |  |  |
| `close` | operation | self, reason_id | `website_forum` |  |  |
| `validate` | operation | self | `website_forum` |  |  |
| `_refuse` | internal rule | self | `website_forum` |  |  |
| `_flag` | internal rule | self | `website_forum` |  |  |
| `_mark_as_offensive` | internal rule | self, reason_id | `website_forum` |  |  |
| `mark_as_offensive_batch` | operation | self, key, values | `website_forum` |  |  |
| `vote` | operation | self, upvote | `website_forum` |  |  |
| `convert_answer_to_comment` | operation | self | `website_forum` |  | Tools to convert an answer (forum.post) to a comment (mail.message). The original post is unlinked and a new comment is posted on the question using the post create_uid as the comment's author. |
| `convert_comment_to_answer` | operation | self, message_id | `website_forum` | model | Tool to convert a comment (mail.message) into an answer (forum.post). The original comment is unlinked and a new answer from the comment's author is created. Nothing is done if the comment's author already answered the question. |
| `unlink_comment` | operation | self, message_id | `website_forum` |  |  |
| `_set_viewed` | internal rule | self | `website_forum` |  |  |
| `_update_last_activity` | internal rule | self | `website_forum` |  |  |
| `_mail_get_operation_for_mail_message_operation` | messaging hook | self, message_operation | `website_forum` |  |  |
| `_notify_get_recipients_groups` | internal rule | self, message, model_description, msg_vals | `website_forum` |  |  |
| `message_post` | messaging hook | self, message_type, **kwargs | `website_forum` |  |  |
| `_notify_thread_by_inbox` | internal rule | self, message, recipients_data, msg_vals, **kwargs | `website_forum` |  |  |
| `_get_microdata` | preparation rule | self | `website_forum` |  | Generate structured data (microdata) for the post.  Returns:     str or None: Microdata in JSON format representing the post, or None     if not applicable. |
| `_get_structured_data` | preparation rule | self, post_type | `website_forum` |  | Generate structured data (microdata) for an answer or a question.  Returns:     dict: microdata. |
| `go_to_website` | operation | self | `website_forum` |  |  |
| `_search_get_detail` | search rule | self, website, order, options | `website_forum` | model |  |
| `_search_render_results` | search rule | self, fetch_fields, mapping, icon, limit | `website_forum` |  |  |
| `_get_related_posts` | preparation rule | self, limit | `website_forum` |  | Return at most a list of {limit} posts related to the main post, based on tag Jaccard similarity. It computes similarity of sets based on ratio of sets intersection divided by sets union (and thus varies from 0 to 1, 1 being identical sets). |

## Validation and error messages (21)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_parent_id` | ValidationError | You cannot create recursive forum posts. | `website_forum` |
| `create` | UserError | Posting answer on a [Deleted] or [Closed] question is not possible. | `website_forum` |
| `create` | AccessError | %d karma required to create a new question. | `website_forum` |
| `create` | AccessError | %d karma required to answer a question. | `website_forum` |
| `write` | AccessError | %d karma required to edit a post. | `website_forum` |
| `write` | AccessError | %d karma required to delete or reactivate a post. | `website_forum` |
| `write` | AccessError | %d karma required to accept or refuse an answer. | `website_forum` |
| `write` | AccessError | %d karma required to retag. | `website_forum` |
| `write` | AccessError | %d karma required to close or reopen a post. | `website_forum` |
| `write` | AccessError | %d karma required to flag a post. | `website_forum` |
| `_unlink_if_enough_karma` | AccessError | %d karma required to unlink a post. | `website_forum` |
| `_update_content` | AccessError | %d karma required to post an image or link. | `website_forum` |
| `validate` | AccessError | %d karma required to validate a post. | `website_forum` |
| `_refuse` | AccessError | %d karma required to refuse a post. | `website_forum` |
| `_flag` | AccessError | %d karma required to flag a post. | `website_forum` |
| `_mark_as_offensive` | AccessError | %d karma required to mark a post as offensive. | `website_forum` |
| `convert_answer_to_comment` | AccessError | %d karma required to convert an answer to a comment. | `website_forum` |
| `convert_comment_to_answer` | AccessError | %d karma required to convert your comment to an answer. | `website_forum` |
| `convert_comment_to_answer` | AccessError | %d karma required to convert a comment to an answer. | `website_forum` |
| `unlink_comment` | AccessError | %d karma required to delete a comment. | `website_forum` |
| `message_post` | AccessError | %d karma required to comment. | `website_forum` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_public` | no | yes | no | no | `website_forum` |
| `base.group_portal` | yes | yes | yes | yes | `website_forum` |
| `base.group_user` | yes | yes | yes | yes | `website_forum` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Website forum post: Public user can only access to public post | `[(4, ref('base.group_public'))]` | `[('forum_id.privacy', '=', 'public')]` | True | True | True | True |
| Website forum post: User can only access to public (or authorized) post | `[(4, ref('base.group_portal')), (4, ref('base.group_user'))]` | `['\|', ('forum_id.privacy', 'in', ['public', 'connected']), '&', ('forum_id.privacy', '=', 'private'), ('forum_id.authorized_group_id', 'in', user.all_group_ids.ids)]` | True | True | True | True |
| Website forum post : All access for manager | `[(4, ref('base.group_erp_manager'))]` | `[(1, '=', 1)]` | True | True | True | True |
| Website forum post: User can only access to post linked to forum related to followed courses | `[(4, ref('base.group_public'))]` | `[('forum_id.slide_channel_ids.website_published', '=', True), ('forum_id.slide_channel_ids.visibility', '=', 'public')]` | True | True | True | True |
| Website forum: Signed In user can only access to post linked to forum related to courses | `[(4, ref('base.group_portal')), (4, ref('base.group_user'))]` | `[             '&',                 ('forum_id.slide_channel_ids.website_published', '=', True),                 '\|',                     ('forum_id.slide_channel_ids.visibility', 'in', ('public','connected')),                     ('forum_id.slide_channel_ids.is_member', '=', True)             ]` | True | True | True | True |
| Website forum post: website slides officer can access all post | `[(4, ref('website_slides.group_website_slides_officer'))]` | `[(1, '=', 1)]` | True | True | True | True |

## Views (6)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website_forum.forum_post_view_form` | form |  | `name`, `active`, `forum_id`, `website_id`, `parent_id`, `tag_ids`, `state`, `closed_reason_id`, `closed_uid`, `closed_date`, `create_uid`, `create_date`, `write_uid`, `write_date`, `is_correct`, `views`, `vote_count`, `favourite_count`, `child_count`, `relevancy`, `child_ids`, `create_uid`, `vote_count`, `state`, `is_correct` | `go_to_website` |  | `website_forum` |
| `website_forum.forum_post_view_search` | search |  | `name`, `create_uid`, `forum_id`, `tag_ids` |  | `Posts`, `Answers`, `Accepted Answer`, `Answered Posts`, `filter_create_date`, `filter_write_date`, `Archived`, `Forum`, `Author`, `Post` | `website_forum` |
| `website_forum.forum_post_view_graph` | graph |  | `write_date`, `forum_id` |  |  | `website_forum` |
| `website_forum.forum_post_view_tree` | list |  | `active`, `name`, `website_url`, `forum_id`, `views`, `child_count`, `state`, `is_seo_optimized`, `website_id` |  |  | `website_forum` |
| `website_forum.forum_post_view_kanban` | kanban |  | `parent_id`, `name`, `website_id`, `forum_id`, `child_count`, `views`, `create_uid` |  |  | `website_forum` |
| `website_slides_forum.forum_post_view_graph_slides` | graph |  | `create_date`, `forum_id` |  |  | `website_slides_forum` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website_forum.forum_post_action` | Forum Post Pages | list,kanban,graph |  | `{'search_default_posts': 1, 'create_action': 'website_forum.forum_forum_action_add'}` |  | `website_forum` |
| `website_forum.forum_post_action_favorites` | Users favorite posts | list,form | `[('forum_id', '=', active_id), ('favourite_count', '>', 0), ('state', 'in', ('active', 'close'))]` |  |  | `website_forum` |
| `website_forum.forum_post_action_forum_main` | Posts | list,form | `[('forum_id', '=', active_id), ('parent_id', '=', False), ('state', 'in', ('active', 'close'))]` |  |  | `website_forum` |
| `website_slides_forum.forum_post_action_channel` | Forum Posts | list,graph,pivot,form | `[('forum_id.slide_channel_ids', '!=', 'False')]` | `{'search_default_questions': 1}` |  | `website_slides_forum` |

Machine-readable definition: `../../../schemas/data/entities/forum.post.json`; views: `../../../schemas/interfaces/views/forum.post.json`.
