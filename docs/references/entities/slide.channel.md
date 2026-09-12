# Course (`slide.channel`)

**Transport name:** `slide.channel`  
**Storage name:** `slide_channel`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_slides`  
**Extended by packages:** `hr_skills_slides`, `mass_mailing_slides`, `website_sale_slides`, `website_slides_forum`, `website_slides_survey`

Description: Course

## Identity and behavior

- Mixins (classical inheritance): `rating.mixin`, `mail.activity.mixin`, `image.mixin`, `website.cover_properties.mixin`, `website.seo.metadata`, `website.published.multi.mixin`, `website.searchable.mixin`
- Default ordering: `sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (73)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `active` | Active | boolean |  | default `True`; changes are tracked in the message thread |
| `description` | Description | rich text |  | translatable; Help: The description that is displayed on top of the course page, just below the title |
| `description_short` | Short Description | rich text |  | translatable; Help: The description that is displayed on the course card |
| `description_html` | Detailed Description | rich text |  | translatable |
| `channel_type` | Course type | selection |  | required; default `training`; Help: Defines the course type (e.g., "Training" for interactive learning, or "Documentation" for resources and guides). |
| `sequence` | Sequence | integer |  | default `10` |
| `user_id` | Responsible | many to one | `res.users` | default computed dynamically (lambda self: self.env.uid) |
| `color` | Color Index | integer |  | default ; Help: Used to decorate kanban view |
| `tag_ids` | Tags | many to many | `slide.channel.tag` | association table `slide_channel_tag_rel`; Help: Used to categorize and filter displayed channels/courses |
| `slide_ids` | Slides and categories | one to many | `slide.slide` | inverse field `channel_id` |
| `slide_content_ids` | Content | one to many | `slide.slide` | computed by rule `_compute_category_and_slide_ids` (not stored) |
| `slide_category_ids` | Categories | one to many | `slide.slide` | computed by rule `_compute_category_and_slide_ids` (not stored) |
| `slide_last_update` | Last Update | date |  | computed by rule `_compute_slide_last_update` and stored |
| `slide_partner_ids` | Slide User Data | one to many | `slide.slide.partner` | not copied on duplication; visible only to groups `website_slides.group_website_slides_officer`; inverse field `channel_id` |
| `promote_strategy` | Featured Content | selection |  | default `latest`; not copied on duplication; Help: Defines the content that will be promoted on the course home page |
| `promoted_slide_id` | Promoted Slide | many to one | `slide.slide` | not copied on duplication |
| `access_token` | Security Token | single line text |  | default computed dynamically (_default_access_token); not copied on duplication |
| `nbr_document` | Documents | integer |  | computed by rule `_compute_slides_statistics` and stored |
| `nbr_video` | Videos | integer |  | computed by rule `_compute_slides_statistics` and stored |
| `nbr_infographic` | Infographics | integer |  | computed by rule `_compute_slides_statistics` and stored |
| `nbr_article` | Articles | integer |  | computed by rule `_compute_slides_statistics` and stored |
| `nbr_quiz` | Number of Quizs | integer |  | computed by rule `_compute_slides_statistics` and stored |
| `total_slides` | Number of Contents | integer |  | computed by rule `_compute_slides_statistics` and stored |
| `total_views` | Visits | integer |  | computed by rule `_compute_slides_statistics` and stored |
| `total_votes` | Votes | integer |  | computed by rule `_compute_slides_statistics` and stored |
| `total_time` | Duration | float |  | computed by rule `_compute_slides_statistics` and stored; precision `[10, 2]` |
| `rating_avg_stars` | Rating Average (Stars) | float |  | computed by rule `_compute_rating_stats` (not stored); precision `[16, 1]` |
| `allow_comment` | Allow rating on Course | boolean |  | computed by rule `_compute_allow_comment` and stored; precomputed before insertion; Help: Allow Attendees to like and comment your content and to submit reviews on your course. |
| `publish_template_id` | New Content Notification | many to one | `mail.template` | default computed dynamically (lambda self: self.env['ir.model.data']._xmlid_to_res_id('website_slides.slide_template_published')); restricted by domain `[["model", "=", "slide.slide"]]`; Help: Defines the email your Attendees will receive each time you upload new content. |
| `share_channel_template_id` | Channel Share Template | many to one | `mail.template` | default computed dynamically (lambda self: self.env['ir.model.data']._xmlid_to_res_id('website_slides.mail_template_channel_shared')); Help: Email template used when sharing a channel |
| `share_slide_template_id` | Share Template | many to one | `mail.template` | default computed dynamically (lambda self: self.env['ir.model.data']._xmlid_to_res_id('website_slides.slide_template_shared')); Help: Email template used when sharing a slide |
| `completed_template_id` | Completion Notification | many to one | `mail.template` | default computed dynamically (lambda self: self.env['ir.model.data']._xmlid_to_res_id('website_slides.mail_template_channel_completed')); restricted by domain `[["model", "=", "slide.channel.partner"]]`; Help: Defines the email your Attendees will receive once they reach the end of your course. |
| `enroll` | Enroll Policy | selection |  | required; computed by rule `_compute_enroll` and stored; default `public`; not copied on duplication; on delete of the target: {"expression": "{'payment': lambda recs: recs.write({'enroll': 'invite'})}"}; Help: Defines how people can enroll to your Course.; extended by packages `website_sale_slides` |
| `enroll_msg` | Enroll Message | rich text |  | default computed dynamically (_get_default_enroll_msg); translatable; Help: Message explaining the enroll process |
| `enroll_group_ids` | Auto Enroll Groups | many to many | `res.groups` | Help: Members of those groups are automatically added as members of the channel. |
| `visibility` | Show Course To | selection |  | required; default `public`; Help: Defines who can access your courses and their content. |
| `upload_group_ids` | Upload Groups | many to many | `res.groups` | visible only to groups `base.group_user`; association table `rel_upload_groups`; Help: Group of users allowed to publish contents on a documentation course. |
| `website_default_background_image_url` | Background image uniform resource locator | single line text |  | computed by rule `_compute_website_default_background_image_url` (not stored) |
| `channel_partner_ids` | Enrolled Attendees Information | one to many | `slide.channel.partner` | visible only to groups `website_slides.group_website_slides_officer`; restricted by domain `[["member_status", "!=", "invited"]]`; inverse field `channel_id` |
| `channel_partner_all_ids` | All Attendees Information | one to many | `slide.channel.partner` | visible only to groups `website_slides.group_website_slides_officer`; inverse field `channel_id` |
| `members_count` | # Enrolled Attendees | integer |  | computed by rule `_compute_members_counts` (not stored) |
| `members_all_count` | # Enrolled or Invited Attendees | integer |  | computed by rule `_compute_members_counts` (not stored) |
| `members_engaged_count` | # Active Attendees | integer |  | computed by rule `_compute_members_counts` (not stored); Help: Active attendees include both 'joined' and 'ongoing' attendees. |
| `members_completed_count` | # Completed Attendees | integer |  | computed by rule `_compute_members_counts` (not stored) |
| `members_invited_count` | # Invited Attendees | integer |  | computed by rule `_compute_members_counts` (not stored) |
| `partner_ids` | Attendees | many to many | `res.partner` | computed by rule `_compute_partners` (not stored); searchable through a search rule; Help: Enrolled partners in the course |
| `completed` | Done | boolean |  | computed by rule `_compute_user_statistics` (not stored) |
| `completion` | Completion | integer |  | computed by rule `_compute_user_statistics` (not stored) |
| `can_upload` | Can Upload | boolean |  | computed by rule `_compute_can_upload` (not stored) |
| `has_requested_access` | Access Requested | boolean |  | computed by rule `_compute_has_requested_access` (not stored) |
| `is_member` | Is Enrolled Attendee | boolean |  | computed by rule `_compute_membership_values` (not stored); searchable through a search rule; Help: Is the attendee actively enrolled. |
| `is_member_invited` | Is Invited Attendee | boolean |  | computed by rule `_compute_membership_values` (not stored); searchable through a search rule; Help: Is the invitation for this attendee pending. |
| `is_visible` | Is Visible On Website | boolean |  | computed by rule `_compute_is_visible` (not stored); searchable through a search rule |
| `partner_has_new_content` | Partner Has New Content | boolean |  | computed by rule `_compute_partner_has_new_content` (not stored) |
| `karma_gen_channel_rank` | Course ranked | integer |  | default `5` |
| `karma_gen_channel_finish` | Course finished | integer |  | default `10` |
| `karma_review` | Add Review | integer |  | default `10`; Help: Karma needed to add a review on the course |
| `karma_slide_comment` | Add Comment | integer |  | default `3`; Help: Karma needed to add a comment on a slide of this course |
| `karma_slide_vote` | Vote | integer |  | default `3`; Help: Karma needed to like/dislike a slide of this course. |
| `can_review` | Can Review | boolean |  | computed by rule `_compute_action_rights` (not stored) |
| `can_comment` | Can Comment | boolean |  | computed by rule `_compute_action_rights` (not stored) |
| `can_vote` | Can Vote | boolean |  | computed by rule `_compute_action_rights` (not stored) |
| `prerequisite_channel_ids` | Prerequisites | many to many | `slide.channel` | restricted by domain `[('id', '!=', id), ('visibility', '=', visibility), ('website_published', '=', website_published)]`; association table `slide_channel_prerequisite_slide_channel_rel`; Help: Prerequisite courses to complete before accessing this one. |
| `prerequisite_of_channel_ids` | Prerequisite Of | many to many | `slide.channel` | association table `slide_channel_prerequisite_slide_channel_rel`; Help: Courses that have this course as prerequisite. |
| `prerequisite_user_has_completed` | Has Completed Prerequisite | boolean |  | computed by rule `_compute_prerequisite_user_has_completed` (not stored) |
| `product_id` | Product | many to one | `product.product` | default computed dynamically (_get_default_product_id); indexed (btree_not_null); restricted by domain `[["service_tracking", "=", "course"]]` |
| `product_sale_revenues` | Total revenues | monetary |  | computed by rule `_compute_product_sale_revenues` (not stored); visible only to groups `sales_team.group_sale_salesman` |
| `currency_id` | Currency | many to one |  | related through path `product_id.currency_id` |
| `forum_id` | Course Forum | many to one | `forum.forum` | indexed (btree_not_null); not copied on duplication |
| `forum_total_posts` | Number of active forum posts | integer |  | related through path `forum_id.total_posts` |
| `members_certified_count` | # Certified Attendees | integer |  | computed by rule `_compute_members_certified_count` (not stored) |
| `nbr_certification` | Number of Certifications | integer |  | computed by rule `_compute_slides_statistics` and stored |

## Selection values

### `channel_type` (Course type)

| Value | Label |
|---|---|
| `training` | Training |
| `documentation` | Documentation |

### `promote_strategy` (Featured Content)

| Value | Label |
|---|---|
| `latest` | Latest Created |
| `most_voted` | Most Voted |
| `most_viewed` | Most Viewed |
| `specific` | Select Manually |
| `none` | None |

### `enroll` (Enroll Policy)

| Value | Label |
|---|---|
| `public` | Open |
| `invite` | On Invitation |
| `payment` | On payment |

### `visibility` (Show Course To)

| Value | Label |
|---|---|
| `public` | Everyone |
| `connected` | Signed In |
| `members` | Course Attendees |
| `link` | Anyone with the link |

## Database constraints and indexes (3)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_check_enroll` | Constraint | `CHECK(visibility != 'members' OR enroll = 'invite')` | The Enroll Policy should be set to 'On Invitation' when visibility is set to 'Course Attendees' | `website_slides` |
| `_product_id_check` | Constraint | `CHECK( enroll!='payment' OR product_id IS NOT NULL )` | Product is required for on payment channels. | `website_sale_slides` |
| `_forum_uniq` | Constraint | `unique (forum_id)` | Only one course per forum! | `website_slides_forum` |

## Operations (75)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_cover_properties` | preparation rule | self | `website_slides` |  | Cover properties defaults are overridden to keep a consistent look for the slides channels headers across the system versions (pre-customization, with purple gradient fitting the homepage images, etc). Furthermore, as adding padding to the cover would not look great, its height is set to fit to content (snippet option to change this also disabled on the view). |
| `_default_access_token` | preparation rule | self | `website_slides` |  |  |
| `_get_default_enroll_msg` | preparation rule | self | `website_slides` |  |  |
| `_compute_enroll` | computation | self | `website_slides` | depends: `visibility` |  |
| `_compute_is_visible` | computation | self | `website_slides` | depends: `visibility`, `is_member`; depends_context: `uid` |  |
| `_search_is_visible` | search rule | self, operator, value | `website_slides` | model |  |
| `_compute_partners` | computation | self | `website_slides` | depends: `channel_partner_all_ids`, `channel_partner_all_ids.member_status`, `channel_partner_all_ids.active` |  |
| `_search_partner_ids` | search rule | self, operator, value | `website_slides` |  |  |
| `_compute_slide_last_update` | computation | self | `website_slides` | depends: `slide_ids.is_published` |  |
| `_compute_members_counts` | computation | self | `website_slides` | depends: `channel_partner_all_ids.channel_id`, `channel_partner_all_ids.member_status` |  |
| `_compute_has_requested_access` | computation | self | `website_slides` | depends: `activity_ids.request_partner_id`; depends_context: `uid`; model |  |
| `_compute_membership_values` | computation | self | `website_slides` | depends: `channel_partner_all_ids.partner_id`, `channel_partner_all_ids.member_status`, `channel_partner_all_ids.active`; depends_context: `uid` |  |
| `_search_is_member` | search rule | self, operator, value | `website_slides` |  |  |
| `_search_is_member_invited` | search rule | self, operator, value | `website_slides` |  |  |
| `_search_is_member_channel_ids` | search rule | self, invited | `website_slides` |  |  |
| `_compute_category_and_slide_ids` | computation | self | `website_slides` | depends: `slide_ids.is_category` |  |
| `_compute_slides_statistics` | computation | self | `website_slides` | depends: `slide_ids.slide_category`, `slide_ids.is_published`, `slide_ids.completion_time`, `slide_ids.likes`, `slide_ids.dislikes`, `slide_ids.total_views`, `slide_ids.is_category`, `slide_ids.active` |  |
| `_compute_rating_stats` | computation | self | `website_slides` |  |  |
| `_compute_allow_comment` | computation | self | `website_slides` | depends: `channel_type` | Comment allowed by default except for documentation channels. |
| `_compute_user_statistics` | computation | self | `website_slides` | depends: `slide_partner_ids`, `slide_partner_ids.completed`, `total_slides`; depends_context: `uid` |  |
| `_compute_can_upload` | computation | self | `website_slides` | depends: `upload_group_ids`, `user_id`; depends_context: `uid` |  |
| `_compute_can_publish` | computation | self | `website_slides` | depends: `channel_type`, `user_id`, `can_upload`; depends_context: `uid` | For channels of type 'training', only the responsible (see user_id field) can publish slides. The 'sudo' user needs to be handled because they are the one used for uploads done on the front-end when the logged in user is not publisher but fulfills the upload_group_ids condition. Invited attendees can preview the course as public and sudo. Prevent them from uploading. |
| `_get_can_publish_error_message` | preparation rule | self | `website_slides` | model |  |
| `_compute_partner_has_new_content` | computation | self | `website_slides` | depends: `slide_partner_ids`; depends_context: `uid` |  |
| `_compute_website_default_background_image_url` | computation | self | `website_slides` | depends: `channel_type` |  |
| `_compute_website_url` | computation | self | `website_slides` | depends: `name` |  |
| `_compute_website_absolute_url` | computation | self | `website_slides` | depends: `website_id.domain` |  |
| `_compute_action_rights` | computation | self | `website_slides` | depends: `can_publish`, `is_member`, `karma_review`, `karma_slide_comment`, `karma_slide_vote`; depends_context: `uid` |  |
| `_compute_prerequisite_user_has_completed` | computation | self | `website_slides` | depends: `prerequisite_channel_ids`, `channel_partner_ids.member_status`; depends_context: `uid` |  |
| `_init_column` | internal rule | self, column_name | `website_slides` |  | Initialize the value of the given column for existing rows. Overridden here because we need to generate different access tokens and by default _init_column calls the default method once and applies it for every record. |
| `create` | lifecycle override | self, vals_list | `website_sale_slides`, `website_slides_forum`, `website_slides` | model_create_multi |  |
| `copy_data` | lifecycle override | self, default | `website_slides` |  |  |
| `write` | lifecycle override | self, vals | `website_sale_slides`, `website_slides_forum`, `website_slides` |  |  |
| `unlink` | lifecycle override | self | `website_slides` |  | " Necessary override to avoid cache issues in the ORM. This signals the ORM to remove slides first to avoid having the SQL cascade the deletion, which attempts to recompute slide statistics of removed slides and creates a cache failure.  Indeed, slides statistics are computed using a read_group which will try to flush the records first and fail with a "Could not find all values of slide.slide.category_id to flush them". (Fix suggested by the ORM team).  (See '_compute_slides_statistics' and '_compute_category_completion_time'). |
| `action_archive` | lifecycle override | self | `website_slides` |  | Archiving a channel does it on its slides, too.  We want to be archiving the channel FIRST. So that when slides are archived and the recompute is triggered, it does not try to mark the channel as "completed". That happens because it counts slide_done / slide_total, but slide_total will be 0 since all the slides for the course have been archived as well. |
| `action_unarchive` | lifecycle override | self | `website_slides` |  | Unarchiving a channel does it on its slides, too.  We want to archive the channel LAST. So that when it recomputes stats for the channel and completion, it correctly counts the slides_total by counting slides that are already un-archived. |
| `message_post` | messaging hook | self, parent_id, subtype_id, **kwargs | `website_slides` |  | Temporary workaround to avoid spam. If someone replies on a channel through the 'Presentation Published' email, it should be considered as a note as we don't want all channel followers to be notified of this answer. Also make sure that only one review can be posted per course. |
| `_mail_get_partner_fields` | messaging hook | self, introspect_fields | `website_slides` |  |  |
| `action_redirect_to_members` | user action | self, status_filter | `website_slides` |  | Redirects to attendees of the course. If status_filter is set to 'invited' / 'engaged' ('joined' + 'ongoing') / 'completed', attendees are filtered accordingly. |
| `action_redirect_to_engaged_members` | user action | self | `website_slides` |  |  |
| `action_redirect_to_completed_members` | user action | self | `website_slides` |  |  |
| `action_redirect_to_invited_members` | user action | self | `website_slides` |  |  |
| `action_channel_enroll` | user action | self | `website_slides` |  |  |
| `action_channel_invite` | user action | self | `website_slides` |  |  |
| `_action_channel_open_invite_wizard` | internal rule | self, mail_template, enroll_mode | `website_slides` |  | Open the invitation wizard to invite and add attendees to the course(s) in self.  :param mail_template: mail.template used in the invite wizard. :param enroll_mode: true if we want to enroll the attendees invited through the wizard.     False otherwise, adding them as 'invited', e.g. when using "Invite" action. |
| `_action_add_members` | internal rule | self, target_partners, member_status, raise_on_access | `hr_skills_slides`, `website_slides` |  | Adds the target_partners as attendees of the channel(s). Partners are added as follows, depending on the value of member_status: 1) (Default) 'joined'. The partners will be added as enrolled attendees. This will make the content     (slides) of the channel available to that partner. This can also happen when an invited attendee     enrolls themself. The attendees are also subscribed to the chatter of the channel.     :return: the union of previous partners re-enrolling, new attendees and invited ones enrolling. 2) 'invited' : This is used when inviting partners. The partners are added as invit |
| `_filter_add_members` | internal rule | self, raise_on_access | `website_slides` |  |  |
| `_add_groups_members` | internal rule | self | `website_slides` |  |  |
| `_get_earned_karma` | preparation rule | self, partner_ids | `website_slides` |  | Compute the number of karma earned by partners on a channel Warning: this count will not be accurate if the configuration has been modified after the completion of a course! |
| `_remove_membership` | internal rule | self, partner_ids | `hr_skills_slides`, `website_slides_survey`, `website_slides` |  | Karma earned during course progress is kept upon membership removal. This is done because re-joining the course will not allow you to gain the karma again, as we keep your progress |
| `_send_share_email` | internal rule | self, emails | `website_slides` |  | Share channel through emails. |
| `action_view_slides` | user action | self | `website_slides` |  |  |
| `action_view_ratings` | user action | self | `website_slides` |  |  |
| `action_request_access` | user action | self | `website_slides` |  | Request access to the channel. Returns a dict with keys being either 'error' (specific error raised) or 'done' (request done or not). |
| `action_grant_access` | user action | self, partner_id | `website_slides` |  |  |
| `action_refuse_access` | user action | self, partner_id | `website_slides` |  |  |
| `_rating_domain` | internal rule | self | `website_slides` |  | Only take the published rating into account to compute avg and count |
| `_action_request_access` | internal rule | self, partner | `website_slides` |  |  |
| `_get_access_action` | preparation rule | self, access_uid, force_website | `website_slides` |  | Instead of the classic form view, redirect non-internal users to website if it is published. |
| `_get_categorized_slides` | preparation rule | self, base_domain, order, force_void, limit, offset | `website_slides` |  | Return an ordered structure of slides by categories within a given base_domain that must fulfill slides. As a course structure is based on its slides sequences, uncategorized slides must have the lowest sequences.  Example   * category 1 (sequence 1), category 2 (sequence 3)   * slide 1 (sequence 0), slide 2 (sequence 2)   * course structure is: slide 1, category 1, slide 2, category 2     * slide 1 is uncategorized,     * category 1 has one slide : Slide 2     * category 2 is empty.  Backend and frontend ordering is the same, uncategorized first. It eases resequencing based on DOM / displayed |
| `_move_category_slides` | internal rule | self, category, new_category | `website_slides` |  |  |
| `_resequence_slides` | internal rule | self, slide, force_category | `website_slides` |  |  |
| `get_backend_menu_id` | operation | self | `website_slides` |  |  |
| `_search_get_detail` | search rule | self, website, order, options | `website_slides` | model |  |
| `_get_placeholder_filename` | preparation rule | self, field | `website_slides` |  |  |
| `_allow_publish_rating_stats` | internal rule | self | `website_slides` | model |  |
| `_message_employee_chatter` | messaging hook | self, msg, partners | `hr_skills_slides` |  |  |
| `action_mass_mailing_attendees` | user action | self | `mass_mailing_slides` |  |  |
| `_get_default_product_id` | preparation rule | self | `website_sale_slides` |  |  |
| `_compute_product_sale_revenues` | computation | self | `website_sale_slides` | depends: `product_id` |  |
| `_synchronize_product_publish` | internal rule | self | `website_sale_slides` |  | Ensure that when publishing a course that its linked product is also published If all courses linked to a product are unpublished, we also unpublished the product |
| `action_view_sales` | user action | self | `website_sale_slides` |  |  |
| `action_redirect_to_forum` | user action | self | `website_slides_forum` |  |  |
| `_compute_members_certified_count` | computation | self | `website_slides_survey` | depends: `channel_partner_ids` |  |
| `action_redirect_to_certified_members` | user action | self | `website_slides_survey` |  |  |

## Validation and error messages (4)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `message_post` | AccessError | Not enough karma to review | `website_slides` |
| `message_post` | ValidationError | Only a single review can be posted per course. | `website_slides` |
| `_filter_add_members` | AccessError | You are not allowed to add members to this course. Please contact the course responsible or an administrator. | `website_slides` |
| `_send_share_email` | UserError | Impossible to send emails. Select a "Channel Share Template" for courses %(course_names)s first | `website_slides` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_public` | no | yes | no | no | `website_slides` |
| `base.group_portal` | no | yes | no | no | `website_slides` |
| `base.group_user` | no | yes | no | no | `website_slides` |
| `website_slides.group_website_slides_officer` | yes | yes | yes | no | `website_slides` |
| `website_slides.group_website_slides_manager` | yes | yes | yes | yes | `website_slides` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Channel: always visible (sub rules exist) | global (all users) | `[(1, '=', 1)]` | True | True | True | True |
| Channel: public: restricted to public/link-based and published | `[(4, ref('base.group_public'))]` | `[('website_published', '=', True), ('visibility', 'in', ['public', 'link'])]` | 1 | 0 | 0 | 0 |
| Channel: portal/user: restricted to published, public or (invited) attendee or link-based, connected user | `[(4, ref('base.group_portal')), (4, ref('base.group_user'))]` | `[                 '&',                     ('website_published', '=', True),                     '\|',                         ('visibility', 'in', ('public', 'connected', 'link')),                         '\|',                             ('is_member_invited', '=', True),                             ('is_member', '=', True),                 ]` | 1 | 0 | 0 | 0 |
| Channel: officer: read all | `[(4, ref('group_website_slides_officer'))]` | `[(1, '=', 1)]` | 1 | 0 | 0 | 0 |
| Channel: officer: create/write own only | `[(4, ref('group_website_slides_officer'))]` | `[('user_id', '=', user.id)]` | 0 | 1 | 1 | 0 |
| Channel: manager: crud all | `[(4, ref('group_website_slides_manager'))]` | `[(1, '=', 1)]` | 1 | 1 | 1 | 1 |

## Views (19)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_skills_slides.slide_channel_view_list` | list |  | `sequence`, `name`, `description`, `tag_ids`, `user_id`, `prerequisite_channel_ids`, `prerequisite_of_channel_ids`, `website_id`, `channel_type`, `visibility`, `enroll`, `is_published`, `active` |  |  | `hr_skills_slides` |
| `mass_mailing_slides.slide_channel_view_form` | button | `website_slides.view_slide_channel_form` | `members_count` | `action_channel_enroll`, `Contact Attendees` |  | `mass_mailing_slides` |
| `mass_mailing_slides.slide_channel_view_kanban` | xpath | `website_slides.slide_channel_view_kanban` |  |  |  | `mass_mailing_slides` |
| `website_sale_slides.slide_channel_view_form` | xpath | `website_slides.view_slide_channel_form` | `product_id` |  |  | `website_sale_slides` |
| `website_sale_slides.slide_channel_view_tree_report` | field | `website_slides.slide_channel_view_tree_report` | `members_completed_count`, `currency_id`, `product_sale_revenues` |  |  | `website_sale_slides` |
| `website_sale_slides.slide_channel_view_kanban` | xpath | `website_slides.slide_channel_view_kanban` | `product_sale_revenues`, `currency_id` |  |  | `website_sale_slides` |
| `website_sale_slides.slide_channel_view_form_add_inherit_sale_slides` | xpath | `website_slides.slide_channel_view_form_add` | `enroll`, `product_id` |  |  | `website_sale_slides` |
| `website_slides.view_slide_channel_form` | form |  | `total_views`, `total_slides`, `members_completed_count`, `members_all_count`, `rating_avg_stars`, `rating_count`, `is_published`, `image_1920`, `name`, `active`, `tag_ids`, `slide_ids`, `sequence`, `name`, `slide_category`, `completion_time`, `total_views`, `is_preview`, `is_published`, `is_category`, `description`, `user_id`, `website_id`, `website_published`, `prerequisite_channel_ids`, `prerequisite_of_channel_ids`, `visibility`, `website_absolute_url`, `enroll`, `upload_group_ids`, `enroll_group_ids`, `enroll_msg`, `allow_comment`, `share_slide_template_id`, `share_channel_template_id`, `publish_template_id`, `completed_template_id`, `channel_type`, `promote_strategy`, `promoted_slide_id`, `karma_gen_channel_rank`, `karma_gen_channel_finish`, `karma_review`, `karma_slide_comment`, `karma_slide_vote` | `Add Attendees`, `Invite`, , `action_view_slides`, `action_redirect_to_completed_members`, `action_redirect_to_members`, `action_view_ratings` |  | `website_slides` |
| `website_slides.slide_channel_view_tree` | list |  | `sequence`, `name`, `user_id`, `website_id`, `channel_type`, `visibility`, `enroll`, `is_published`, `active` |  |  | `website_slides` |
| `website_slides.slide_channel_view_tree_report` | list |  | `name`, `user_id`, `total_views`, `rating_avg_stars`, `total_time`, `members_count`, `members_completed_count` |  |  | `website_slides` |
| `website_slides.slide_channel_view_search` | search |  | `name`, `user_id`, `tag_ids`, `slide_ids` |  | `Published`, `Archived` | `website_slides` |
| `website_slides.slide_channel_view_graph` | graph |  | `name`, `total_views`, `karma_slide_comment`, `karma_review`, `color`, `karma_gen_channel_finish`, `karma_gen_channel_rank`, `sequence` |  |  | `website_slides` |
| `website_slides.slide_channel_view_pivot` | pivot |  | `name`, `total_views`, `karma_slide_comment`, `karma_review`, `color`, `karma_gen_channel_finish`, `karma_gen_channel_rank`, `sequence` |  |  | `website_slides` |
| `website_slides.slide_channel_view_kanban` | kanban |  | `website_published`, `color`, `name`, `tag_ids`, `total_views`, `total_slides`, `total_time`, `rating_count`, `rating_avg_stars`, `members_invited_count`, `members_engaged_count`, `members_completed_count`, `members_all_count` | `open_website_url` |  | `website_slides` |
| `website_slides.slide_channel_pages_tree_view` | xpath | `slide_channel_view_tree` |  |  |  | `website_slides` |
| `website_slides.slide_channel_pages_kanban_view` | xpath | `slide_channel_view_kanban` |  |  |  | `website_slides` |
| `website_slides.slide_channel_view_form_add` | form |  | `name`, `website_url`, `tag_ids`, `channel_type`, `description`, `allow_comment` |  |  | `website_slides` |
| `website_slides_forum.website_slides_forum_channel_inherit_view_form` | xpath | `website_slides.view_slide_channel_form` | `forum_total_posts` | `action_redirect_to_forum` |  | `website_slides_forum` |
| `website_slides_survey.slide_channel_view_form` | xpath | `website_slides.view_slide_channel_form` | `nbr_certification`, `members_certified_count` | `action_redirect_to_certified_members` |  | `website_slides_survey` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_skills_slides.slide_channel_training_elearning_action` | eLearning Courses | list,kanban,form |  |  |  | `hr_skills_slides` |
| `website_slides.slide_channel_action_overview` | All Courses | kanban,list,form |  |  |  | `website_slides` |
| `website_slides.slide_channel_action_report` | Courses | list,graph,pivot,form |  | `{"search_default_filter_published":1}` |  | `website_slides` |
| `website_slides.action_slide_channel_pages_list` | Course Pages | list,kanban,form |  | `{'create_action': 'website_slides.slide_channel_action_add'}` |  | `website_slides` |
| `website_slides.slide_channel_action_add` | New Course | form |  |  | new | `website_slides` |

## Email templates

| Template | Name | Subject |
|---|---|---|
| `website_slides.mail_template_channel_shared` | Channel Shared | {{ user.name }} shared a Course |

Machine-readable definition: `../../../schemas/data/entities/slide.channel.json`; views: `../../../schemas/interfaces/views/slide.channel.json`.
