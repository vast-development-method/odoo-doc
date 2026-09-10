# Forum (`forum.forum`)

**Transport name:** `forum.forum`  
**Storage name:** `forum_forum`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_forum`  
**Extended by packages:** `website_slides_forum`

Description: Forum

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `image.mixin`, `website.seo.metadata`, `website.multi.mixin`, `website.searchable.mixin`
- Default ordering: `sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (64)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Forum Name | single line text |  | required; translatable |
| `sequence` | Sequence | integer |  | default `1` |
| `mode` | Mode | selection |  | required; default `questions`; Help: Questions mode: only one answer allowed  Discussions mode: multiple answers allowed |
| `privacy` | Privacy | selection |  | default `public`; Help: Public: Forum is public Signed In: Forum is visible for signed in users Some users: Forum and their content are hidden for non members of selected group |
| `authorized_group_id` | Authorized Group | many to one | `res.groups` |  |
| `active` | Active | boolean |  | default `True` |
| `faq` | Guidelines | rich text |  | translatable |
| `description` | Description | multi line text |  | translatable |
| `welcome_message` | Welcome Message | rich text |  | default computed dynamically (_get_default_welcome_message); translatable |
| `default_order` | Default | selection |  | required; default `last_activity_date desc` |
| `relevancy_post_vote` | First Relevance Parameter | float |  | default `0.8`; Help: This formula is used in order to sort by relevance. The variable 'votes' represents number of votes for a post, and 'days' is number of days since the post creation |
| `relevancy_time_decay` | Second Relevance Parameter | float |  | default `1.8` |
| `allow_share` | Sharing Options | boolean |  | default `True`; Help: After posting the user will be proposed to share its question or answer on social networks, enabling social network propagation of the forum content. |
| `post_ids` | Posts | one to many | `forum.post` | inverse field `forum_id` |
| `last_post_id` | Last Post | many to one | `forum.post` | computed by rule `_compute_last_post_id` (not stored) |
| `total_posts` | # Posts | integer |  | computed by rule `_compute_forum_statistics` (not stored) |
| `total_views` | # Views | integer |  | computed by rule `_compute_forum_statistics` (not stored) |
| `total_answers` | # Answers | integer |  | computed by rule `_compute_forum_statistics` (not stored) |
| `total_favorites` | # Favorites | integer |  | computed by rule `_compute_forum_statistics` (not stored) |
| `count_posts_waiting_validation` | Number of posts waiting for validation | integer |  | computed by rule `_compute_count_posts_waiting_validation` (not stored) |
| `count_flagged_posts` | Number of flagged posts | integer |  | computed by rule `_compute_count_flagged_posts` (not stored) |
| `karma_gen_question_new` | Asking a question | integer |  | default `2` |
| `karma_gen_question_upvote` | Question upvoted | integer |  | default `5` |
| `karma_gen_question_downvote` | Question downvoted | integer |  | default `-2` |
| `karma_gen_answer_upvote` | Answer upvoted | integer |  | default `10` |
| `karma_gen_answer_downvote` | Answer downvoted | integer |  | default `-2` |
| `karma_gen_answer_accept` | Accepting an answer | integer |  | default `2` |
| `karma_gen_answer_accepted` | Answer accepted | integer |  | default `15` |
| `karma_gen_answer_flagged` | Answer flagged | integer |  | default `-100` |
| `karma_ask` | Ask questions | integer |  | default `3` |
| `karma_answer` | Answer questions | integer |  | default `3` |
| `karma_edit_own` | Edit own posts | integer |  | default `1` |
| `karma_edit_all` | Edit all posts | integer |  | default `300` |
| `karma_edit_retag` | Change question tags | integer |  | default `75` |
| `karma_close_own` | Close own posts | integer |  | default `100` |
| `karma_close_all` | Close all posts | integer |  | default `500` |
| `karma_unlink_own` | Delete own posts | integer |  | default `500` |
| `karma_unlink_all` | Delete all posts | integer |  | default `1000` |
| `karma_tag_create` | Create new tags | integer |  | default `30` |
| `karma_upvote` | Upvote | integer |  | default `5` |
| `karma_downvote` | Downvote | integer |  | default `50` |
| `karma_answer_accept_own` | Accept an answer on own questions | integer |  | default `20` |
| `karma_answer_accept_all` | Accept an answer to all questions | integer |  | default `500` |
| `karma_comment_own` | Comment own posts | integer |  | default `1` |
| `karma_comment_all` | Comment all posts | integer |  | default `1` |
| `karma_comment_convert_own` | Convert own comments to answers | integer |  | default `50` |
| `karma_comment_convert_all` | Convert all comments to answers | integer |  | default `500` |
| `karma_comment_unlink_own` | Delete own comments | integer |  | default `50` |
| `karma_comment_unlink_all` | Delete all comments | integer |  | default `500` |
| `karma_flag` | Flag a post as offensive | integer |  | default `500` |
| `karma_dofollow` | Nofollow links | integer |  | default `500`; Help: If the author has not enough karma, a nofollow attribute is added to links |
| `karma_editor` | Editor Features: image and links | integer |  | default `30` |
| `karma_user_bio` | Display detailed user biography | integer |  | default `750` |
| `karma_post` | Ask questions without validation | integer |  | default `100` |
| `karma_moderate` | Moderate posts | integer |  | default `1000` |
| `has_pending_post` | Has pending post | boolean |  | computed by rule `_compute_has_pending_post` (not stored) |
| `can_moderate` | Is a moderator | boolean |  | computed by rule `_compute_can_moderate` (not stored) |
| `tag_ids` | Tags | one to many | `forum.tag` | inverse field `forum_id` |
| `tag_most_used_ids` | Most used tags | one to many | `forum.tag` | computed by rule `_compute_tag_ids_usage` (not stored) |
| `tag_unused_ids` | Unused tags | one to many | `forum.tag` | computed by rule `_compute_tag_ids_usage` (not stored) |
| `slide_channel_ids` | Courses | one to many | `slide.channel` | inverse field `forum_id`; Help: Edit the course linked to this forum on the course form. |
| `slide_channel_id` | Course | many to one | `slide.channel` | computed by rule `_compute_slide_channel_id` and stored |
| `visibility` | Visibility | selection |  | related through path `slide_channel_id.visibility`; Help: Forum linked to a Course, the visibility is the one applied on the course. |
| `image_1920` | Image | image |  | computed by rule `_compute_image_1920` and stored |

## Selection values

### `mode` (Mode)

| Value | Label |
|---|---|
| `questions` | Questions (1 answer) |
| `discussions` | Discussions (multiple answers) |

### `privacy` (Privacy)

| Value | Label |
|---|---|
| `public` | Public |
| `connected` | Signed In |
| `private` | Some users |

### `default_order` (Default)

| Value | Label |
|---|---|
| `create_date desc` | Newest |
| `last_activity_date desc` | Last Updated |
| `vote_count desc` | Most Voted |
| `relevancy desc` | Relevance |
| `child_count desc` | Answered |

## Operations (20)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_default_welcome_message` | preparation rule | self | `website_forum` | model |  |
| `_compute_has_pending_post` | computation | self | `website_forum` | depends_context: `uid` |  |
| `_compute_can_moderate` | computation | self | `website_forum` | depends_context: `uid`; depends: `karma_moderate` |  |
| `_compute_tag_ids_usage` | computation | self | `website_forum` | depends: `post_ids`, `post_ids.tag_ids`, `post_ids.tag_ids.posts_count`, `tag_ids` |  |
| `_compute_last_post_id` | computation | self | `website_forum` | depends: `post_ids` |  |
| `_compute_forum_statistics` | computation | self | `website_forum` | depends: `post_ids.state`, `post_ids.views`, `post_ids.child_count`, `post_ids.favourite_count` |  |
| `_compute_count_posts_waiting_validation` | computation | self | `website_forum` |  |  |
| `_compute_count_flagged_posts` | computation | self | `website_forum` |  |  |
| `_compute_website_url` | computation | self | `website_forum` |  |  |
| `create` | lifecycle override | self, vals_list | `website_forum` | model_create_multi |  |
| `unlink` | lifecycle override | self | `website_forum` |  |  |
| `write` | lifecycle override | self, vals | `website_forum` |  |  |
| `_set_default_faq` | internal rule | self | `website_forum` |  |  |
| `_tag_to_write_vals` | internal rule | self, tags | `website_forum` |  |  |
| `_get_tags_first_char` | preparation rule | self, tags | `website_forum` |  | Get set of first letter of forum tags.  :param tags: tags recordset to further filter forum's tags that are also in these tags. |
| `go_to_website` | operation | self | `website_forum` |  |  |
| `_search_get_detail` | search rule | self, website, order, options | `website_forum` | model |  |
| `_search_render_results` | search rule | self, fetch_fields, mapping, icon, limit | `website_forum` |  |  |
| `_compute_slide_channel_id` | computation | self | `website_slides_forum` | depends: `slide_channel_ids` |  |
| `_compute_image_1920` | computation | self | `website_slides_forum` | depends: `slide_channel_id`, `slide_channel_id.image_1920` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_public` | no | yes | no | no | `website_forum` |
| `base.group_portal` | no | yes | no | no | `website_forum` |
| `base.group_user` | no | yes | no | no | `website_forum` |
| `base.group_erp_manager` | yes | yes | yes | yes | `website_forum` |
| `website_slides.group_website_slides_officer` | yes | yes | yes | no | `website_slides_forum` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Website forum: Public user can only access to public forum | `[(4, ref('base.group_public'))]` | `[('privacy', '=', 'public')]` | True | True | True | True |
| Website forum: User can only access to public (or authorized) forum | `[(4, ref('base.group_portal')), (4, ref('base.group_user'))]` | `[             '\|',                 ('privacy', 'in', ['public', 'connected']),                 '&',                     ('privacy', '=', 'private'),                     ('authorized_group_id', 'in', user.all_group_ids.ids)]` | True | True | True | True |
| Website forum: Website designer can create private forum | `[(4, ref('website.group_website_designer'))]` | `[(1, '=', 1)]` | 0 | 0 | 1 | 0 |
| Website forum: All access for manager | `[(4, ref('base.group_erp_manager'))]` | `[(1, '=', 1)]` | True | True | True | True |
| Website forum: User can only access to forum related to public courses | `[(4, ref('base.group_public'))]` | `[('slide_channel_ids.website_published', '=', True), ('slide_channel_ids.visibility', '=', 'public')]` | True | True | True | True |
| Website forum: Signed In user can only access to forum related to courses | `[(4, ref('base.group_portal')), (4, ref('base.group_user'))]` | `[             '&',                 ('slide_channel_ids.website_published', '=', True),                 '\|',                     ('slide_channel_ids.visibility', 'in', ('public','connected')),                     ('slide_channel_ids.is_member', '=', True)             ]` | True | True | True | True |
| Website forum: website slides officer can access all forum | `[(4, ref('website_slides.group_website_slides_officer'))]` | `[(1, '=', 1)]` | True | True | True | True |

## Views (6)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website_forum.forum_forum_view_tree` | list |  | `sequence`, `name`, `website_id`, `total_posts`, `total_views`, `total_answers`, `total_favorites`, `active` |  |  | `website_forum` |
| `website_forum.forum_forum_view_form` | form |  | `total_posts`, `total_favorites`, `active`, `image_1920`, `name`, `mode`, `website_id`, `default_order`, `privacy`, `authorized_group_id`, `relevancy_post_vote`, `relevancy_time_decay`, `description`, `karma_gen_question_new`, `karma_gen_question_upvote`, `karma_gen_question_downvote`, `karma_gen_answer_upvote`, `karma_gen_answer_downvote`, `karma_gen_answer_accept`, `karma_gen_answer_accepted`, `karma_gen_answer_flagged`, `karma_ask`, `karma_answer`, `karma_upvote`, `karma_downvote`, `karma_edit_own`, `karma_edit_all`, `karma_close_own`, `karma_close_all`, `karma_unlink_own`, `karma_unlink_all`, `karma_dofollow`, `karma_answer_accept_own`, `karma_answer_accept_all`, `karma_editor`, `karma_comment_own`, `karma_comment_all`, `karma_comment_convert_own`, `karma_comment_convert_all`, `karma_comment_unlink_own`, `karma_comment_unlink_all`, `karma_post`, `karma_flag`, `karma_moderate`, `karma_edit_retag`, `karma_tag_create`, `karma_user_bio` | `%(forum_post_action_forum_main)d`, `%(forum_post_action_favorites)d`, `go_to_website` |  | `website_forum` |
| `website_forum.forum_forum_view_form_add` | form |  | `name`, `mode`, `privacy`, `authorized_group_id` |  |  | `website_forum` |
| `website_forum.forum_forum_view_search` | search |  | `name` |  | `Archived` | `website_forum` |
| `website_slides_forum.forum_forum_view_form` | xpath | `website_forum.forum_forum_view_form` | `slide_channel_id` |  |  | `website_slides_forum` |
| `website_slides_forum.forum_forum_view_tree_slides` | field | `website_forum.forum_forum_view_tree` | `website_id`, `slide_channel_id`, `visibility` |  |  | `website_slides_forum` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website_forum.forum_forum_action` | Forums | list,form |  |  |  | `website_forum` |
| `website_forum.forum_forum_action_add` | New Forum | form |  |  | new | `website_forum` |
| `website_slides_forum.forum_forum_action_channel` | Forums | list,form | `[('slide_channel_ids', '!=', 'False')]` |  |  | `website_slides_forum` |

Machine-readable definition: `../../../schemas/data/entities/forum.forum.json`; views: `../../../schemas/interfaces/views/forum.forum.json`.
