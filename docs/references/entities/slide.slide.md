# Slides (`slide.slide`)

**Transport name:** `slide.slide`  
**Storage name:** `slide_slide`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_slides`  
**Extended by packages:** `website_slides_survey`

Description: Slides

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `image.mixin`, `website.seo.metadata`, `website.published.mixin`, `website.searchable.mixin`
- Default ordering: `sequence asc, is_category asc, id asc`
- Posting a message requires access: read
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (71)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Title | single line text |  | required; computed by rule `_compute_name` and stored; translatable; extended by packages `website_slides_survey` |
| `image_1920` | Image 1920 | image |  | computed by rule `_compute_image_1920` and stored |
| `active` | Active | boolean |  | default `True`; changes are tracked in the message thread |
| `sequence` | Sequence | integer |  | default  |
| `user_id` | Uploaded by | many to one | `res.users` | default computed dynamically (lambda self: self.env.uid) |
| `description` | Description | rich text |  | translatable |
| `channel_id` | Course | many to one | `slide.channel` | required; indexed; on delete of the target: cascade |
| `tag_ids` | Tags | many to many | `slide.tag` | association table `rel_slide_tag` |
| `is_preview` | Allow Preview | boolean |  | computed by rule `_compute_is_preview` and stored; default ; Help: The course is accessible by anyone : the users don't need to join the channel to access the content of the course.; extended by packages `website_slides_survey` |
| `is_new_slide` | Is New Slide | boolean |  | computed by rule `_compute_is_new_slide` (not stored) |
| `completion_time` | Duration | float |  | computed by rule `_compute_category_completion_time` and stored; precision `[10, 4]`; recursive dependency |
| `is_category` | Is a category | boolean |  | default  |
| `category_id` | Section | many to one | `slide.slide` | computed by rule `_compute_category_id` and stored; indexed (btree_not_null) |
| `slide_ids` | Content | one to many | `slide.slide` | inverse field `category_id` |
| `partner_ids` | Subscribers | many to many | `res.partner` | not copied on duplication; visible only to groups `website_slides.group_website_slides_officer`; association table `slide_slide_partner` |
| `slide_partner_ids` | Subscribers information | one to many | `slide.slide.partner` | not copied on duplication; visible only to groups `website_slides.group_website_slides_officer`; inverse field `slide_id` |
| `user_membership_id` | Subscriber information | many to one | `slide.slide.partner` | computed by rule `_compute_user_membership_id` (not stored); Help: Subscriber information for the current logged in user |
| `user_vote` | User vote | integer |  | computed by rule `_compute_user_membership_id` (not stored) |
| `user_has_completed` | Is Member | boolean |  | computed by rule `_compute_user_membership_id` (not stored) |
| `user_has_completed_category` | Is Category Completed | boolean |  | computed by rule `_compute_category_completed` (not stored) |
| `question_ids` | Questions | one to many | `slide.question` | inverse field `slide_id` |
| `questions_count` | Numbers of Questions | integer |  | computed by rule `_compute_questions_count` (not stored) |
| `quiz_first_attempt_reward` | Reward: first attempt | integer |  | default `10` |
| `quiz_second_attempt_reward` | Reward: second attempt | integer |  | default `7` |
| `quiz_third_attempt_reward` | Reward: third attempt | integer |  | default `5` |
| `quiz_fourth_attempt_reward` | Reward: every attempt after the third try | integer |  | default `2` |
| `can_self_mark_completed` | Can Mark Completed | boolean |  | computed by rule `_compute_mark_complete_actions` (not stored); Help: The slide can be marked as completed even without opening it |
| `can_self_mark_uncompleted` | Can Mark Uncompleted | boolean |  | computed by rule `_compute_mark_complete_actions` (not stored); Help: The slide can be marked as not completed and the progression |
| `slide_category` | Category | selection |  | required; default `document`; on delete of the target: {"certification": "set default"}; extended by packages `website_slides_survey` |
| `source_type` | Source Type | selection |  | required; default `local_file` |
| `url` | External uniform resource locator | single line text |  | Help: URL of the Google Drive file or URL of the YouTube video |
| `binary_content` | File | binary |  |  |
| `slide_resource_ids` | Additional Resource for this slide | one to many | `slide.slide.resource` | inverse field `slide_id` |
| `slide_resource_downloadable` | Allow Download | boolean |  | default ; Help: Allow the user to download the content of the slide. |
| `google_drive_id` | Google Drive identifier of the external uniform resource locator | single line text |  | computed by rule `_compute_google_drive_id` (not stored) |
| `html_content` | hypertext markup language Content | rich text |  | translatable; Help: Custom HTML content for slides of category 'Article'. |
| `image_binary_content` | Image Content | binary |  | related through path `binary_content` |
| `image_google_url` | Image Link | single line text |  | related through path `url`; Help: Link of the image (we currently only support Google Drive as source) |
| `slide_icon_class` | Slide Icon fa-class | single line text |  | computed by rule `_compute_slide_icon_class` (not stored) |
| `slide_type` | Slide Type | selection |  | computed by rule `_compute_slide_type` and stored; on delete of the target: {"certification": "set null"}; Help: Subtype of the slide category, allows more precision on the actual file type / source type.; extended by packages `website_slides_survey` |
| `document_google_url` | Document Link | single line text |  | related through path `url`; Help: Link of the document (we currently only support Google Drive as source) |
| `document_binary_content` | Portable Document Format Content | binary |  | related through path `binary_content` |
| `video_url` | Video Link | single line text |  | related through path `url`; Help: Link of the video (we support YouTube, Google Drive and Vimeo as sources) |
| `video_source_type` | Video Source | selection |  | computed by rule `_compute_video_source_type` (not stored) |
| `youtube_id` | Video YouTube identifier | single line text |  | computed by rule `_compute_youtube_id` (not stored) |
| `vimeo_id` | Video Vimeo identifier | single line text |  | computed by rule `_compute_vimeo_id` (not stored) |
| `website_id` | Website | many to one |  | read only; related through path `channel_id.website_id` |
| `date_published` | Publish Date | date and time |  | read only; not copied on duplication |
| `likes` | Likes | integer |  | computed by rule `_compute_like_info` and stored |
| `dislikes` | Dislikes | integer |  | computed by rule `_compute_like_info` and stored |
| `embed_code` | Embed Code | rich text |  | read only; computed by rule `_compute_embed_code` (not stored) |
| `embed_code_external` | External Embed Code | rich text |  | read only; computed by rule `_compute_embed_code` (not stored); Help: Same as 'Embed Code' but used to embed the content on an external website. |
| `website_share_url` | Share uniform resource locator | single line text |  | computed by rule `_compute_website_share_url` (not stored) |
| `embed_ids` | External Slide Embeds | one to many | `slide.embed` | inverse field `slide_id` |
| `embed_count` | # of Embeds | integer |  | computed by rule `_compute_embed_counts` (not stored) |
| `slide_views` | # of Website Views | integer |  | computed by rule `_compute_slide_views` and stored |
| `public_views` | # of Public Views | integer |  | read only; default ; not copied on duplication |
| `total_views` | # Total Views | integer |  | computed by rule `_compute_total` and stored; default `0` |
| `comments_count` | Number of comments | integer |  | computed by rule `_compute_comments_count` (not stored) |
| `channel_type` | Channel type | selection |  | related through path `channel_id.channel_type` |
| `channel_allow_comment` | Allows comment | boolean |  | related through path `channel_id.allow_comment` |
| `nbr_document` | Number of Documents | integer |  | computed by rule `_compute_slides_statistics` and stored |
| `nbr_video` | Number of Videos | integer |  | computed by rule `_compute_slides_statistics` and stored |
| `nbr_infographic` | Number of Images | integer |  | computed by rule `_compute_slides_statistics` and stored |
| `nbr_article` | Number of Articles | integer |  | computed by rule `_compute_slides_statistics` and stored |
| `nbr_quiz` | Number of Quizs | integer |  | computed by rule `_compute_slides_statistics` and stored |
| `total_slides` | Total Slides | integer |  | computed by rule `_compute_slides_statistics` and stored |
| `is_published` | Is Published | boolean |  | changes are tracked in the message thread |
| `website_published` | Website Published | boolean |  |  |
| `survey_id` | Certification | many to one | `survey.survey` | indexed (btree_not_null) |
| `nbr_certification` | Number of Certifications | integer |  | computed by rule `_compute_slides_statistics` and stored |

## Selection values

### `slide_category` (Category)

| Value | Label |
|---|---|
| `infographic` | Image |
| `article` | Article |
| `document` | Document |
| `video` | Video |
| `quiz` | Quiz |
| `certification` | Certification |

### `source_type` (Source Type)

| Value | Label |
|---|---|
| `local_file` | Upload from Device |
| `external` | Retrieve from Google Drive |

### `slide_type` (Slide Type)

| Value | Label |
|---|---|
| `image` | Image |
| `article` | Article |
| `quiz` | Quiz |
| `pdf` | PDF |
| `sheet` | Sheet (Excel, Google Sheet, ...) |
| `doc` | Document (Word, Google Doc, ...) |
| `slides` | Slides (PowerPoint, Google Slides, ...) |
| `youtube_video` | YouTube Video |
| `google_drive_video` | Google Drive Video |
| `vimeo_video` | Vimeo Video |
| `certification` | Certification |

### `video_source_type` (Video Source)

| Value | Label |
|---|---|
| `youtube` | YouTube |
| `google_drive` | Google Drive |
| `vimeo` | Vimeo |

## Database constraints and indexes (3)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_exclusion_html_content_and_url` | Constraint | `CHECK(html_content IS NULL OR url IS NULL)` | A slide is either filled with a url or HTML content. Not both. | `website_slides` |
| `_check_survey_id` | Constraint | `CHECK(slide_category != 'certification' OR survey_id IS NOT NULL)` | A slide of type 'certification' requires a certification. | `website_slides_survey` |
| `_check_certification_preview` | Constraint | `CHECK(slide_category != 'certification' OR is_preview = False)` | A slide of type certification cannot be previewed. | `website_slides_survey` |

## Operations (68)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_image_1920` | computation | self | `website_slides` | depends: `slide_category`, `source_type`, `image_binary_content` |  |
| `_compute_is_new_slide` | computation | self | `website_slides` | depends: `date_published`, `is_published` |  |
| `_get_placeholder_filename` | preparation rule | self, field | `website_slides` |  |  |
| `_compute_category_id` | computation | self | `website_slides` | depends: `channel_id.slide_ids.is_category`, `channel_id.slide_ids.sequence`, `channel_id.slide_ids.slide_ids` | Will take all the slides of the channel for which the index is higher than the index of this category and lower than the index of the next category.  Lists are manually sorted because when adding a new browse record order will not be correct as the added slide would actually end up at the first place no matter its sequence. |
| `_compute_mark_complete_actions` | computation | self | `website_slides_survey`, `website_slides` | depends: `slide_category`, `question_ids`, `channel_id.is_member`; depends_context: `uid` | Determine if the slide can be marked as (un)completed.  We can't mark a slide with questions as completed manually because we need to complete the quiz first. But we can mark as uncompleted a slide with questions, and the answers will be reset, the karma removed, etc (see mark_uncompleted). |
| `_compute_questions_count` | computation | self | `website_slides` | depends: `question_ids` |  |
| `_compute_comments_count` | computation | self | `website_slides` | depends: `website_message_ids` |  |
| `_compute_total` | computation | self | `website_slides` | depends: `slide_views`, `public_views` |  |
| `_compute_like_info` | computation | self | `website_slides` | depends: `slide_partner_ids.vote` |  |
| `_compute_slide_views` | computation | self | `website_slides` | depends: `slide_partner_ids.slide_id` |  |
| `_compute_embed_counts` | computation | self | `website_slides` | depends: `embed_ids.slide_id` |  |
| `_compute_slides_statistics` | computation | self | `website_slides` | depends: `slide_ids.sequence`, `slide_ids.active`, `slide_ids.slide_category`, `slide_ids.is_published`, `slide_ids.is_category` |  |
| `_compute_category_completed` | computation | self | `website_slides` | depends: `category_id`, `category_id.slide_ids`, `category_id.slide_ids.user_has_completed` |  |
| `_compute_category_completion_time` | computation | self | `website_slides` | depends: `slide_ids.sequence`, `slide_ids.active`, `slide_ids.completion_time`, `slide_ids.is_published`, `slide_ids.is_category` |  |
| `_compute_slide_icon_class` | computation | self | `website_slides_survey`, `website_slides` | depends: `slide_type` |  |
| `_compute_slide_type` | computation | self | `website_slides_survey`, `website_slides` | depends: `slide_category`, `source_type`, `video_source_type`; depends: `slide_category`, `source_type` | For 'local content' or specific slide categories, the slide type is directly derived from the slide category.  For external content, the slide type is determined from the metadata and the mime_type. (See #_fetch_google_drive_metadata() for more details). |
| `_compute_user_membership_id` | computation | self | `website_slides` | depends: `slide_partner_ids.partner_id`, `slide_partner_ids.vote`, `slide_partner_ids.completed`; depends: `uid` |  |
| `_compute_embed_code` | computation | self | `website_slides` | depends: `slide_category`, `google_drive_id`, `video_source_type`, `youtube_id` |  |
| `_compute_video_source_type` | computation | self | `website_slides` | depends: `video_url` |  |
| `_compute_youtube_id` | computation | self | `website_slides` | depends: `video_url`, `video_source_type` |  |
| `_compute_vimeo_id` | computation | self | `website_slides` | depends: `video_url`, `video_source_type` |  |
| `_compute_google_drive_id` | computation | self | `website_slides` | depends: `url`, `document_google_url`, `image_google_url`, `video_url` | Extracts the Google Drive ID from the url based on the slide category. |
| `_on_change_url` | on change | self | `website_slides` | onchange: `url`, `document_google_url`, `image_google_url`, `video_url` | Keeping a 'onchange' because we want this behavior for the frontend. Changing the document / video external URL will populate some metadata on the form view. We only populate the field that are empty to avoid overriding user assigned values. The slide metadata are also fetched in create / write overrides to ensure consistency. |
| `_on_change_document_binary_content` | on change | self | `website_slides` | onchange: `document_binary_content` |  |
| `_on_change_slide_category` | on change | self | `website_slides` | onchange: `slide_category` | Prevents mis-match when ones uploads an image and then a pdf without saving the form. |
| `_compute_website_url` | computation | self | `website_slides` | depends: `name`, `channel_id.website_id.domain` |  |
| `_compute_website_absolute_url` | computation | self | `website_slides` | depends: `channel_id.website_id.domain` |  |
| `_compute_website_share_url` | computation | self | `website_slides` | depends: `is_published` |  |
| `_compute_can_publish` | computation | self | `website_slides` | depends: `channel_id.can_publish` |  |
| `_get_can_publish_error_message` | preparation rule | self | `website_slides` | model |  |
| `create` | lifecycle override | self, vals_list | `website_slides_survey`, `website_slides` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `website_slides_survey`, `website_slides` |  |  |
| `copy_data` | lifecycle override | self, default | `website_slides` |  | Sets the sequence to zero so that it always lands at the beginning of the newly selected course as an uncategorized slide |
| `unlink` | lifecycle override | self | `website_slides_survey`, `website_slides` |  |  |
| `_can_return_content` | internal rule | self, field_name, access_token | `website_slides` |  |  |
| `message_post` | messaging hook | self, message_type, **kwargs | `website_slides` |  |  |
| `_get_access_action` | preparation rule | self, access_uid, force_website | `website_slides` |  | Instead of the classic form view, redirect to website if it is published. |
| `_notify_get_recipients_groups` | internal rule | self, message, model_description, msg_vals | `website_slides` |  |  |
| `_embed_increment` | internal rule | self, url | `website_slides` |  | Increment the view count of the record we have based on the passed url. If the url is empty, which typically happens if the browser does not pass the 'referer' header properly, then we increment the entry that has 'False' as url value. |
| `_post_publication` | internal rule | self | `website_slides` |  |  |
| `_send_share_email` | internal rule | self, email, fullscreen | `website_slides` |  |  |
| `action_like` | user action | self | `website_slides` |  |  |
| `action_dislike` | user action | self | `website_slides` |  |  |
| `_action_vote` | internal rule | self, upvote | `website_slides` |  | Private implementation of voting. It does not check for any real access rights; public methods should grant access before calling this method.    :param upvote: if True, is a like; if False, is a dislike |
| `action_set_viewed` | user action | self, quiz_attempts_inc | `website_slides` |  |  |
| `_action_set_viewed` | internal rule | self, target_partner, quiz_attempts_inc | `website_slides` |  |  |
| `action_mark_completed` | user action | self | `website_slides` |  |  |
| `_action_mark_completed` | internal rule | self | `website_slides` |  |  |
| `action_mark_uncompleted` | user action | self | `website_slides` |  |  |
| `_action_set_quiz_done` | internal rule | self, completed | `website_slides` |  | Add or remove karma point related to the quiz.  :param completed:     True if the quiz will be marked as completed (karma will be increased)     If set to False, we will remove the karma instead of increasing it,     so that the user can take the quiz multiple times but not gain karma infinitely |
| `action_view_embeds` | user action | self | `website_slides` |  |  |
| `_compute_quiz_info` | computation | self, target_partner, quiz_done | `website_slides` |  |  |
| `_fetch_external_metadata` | internal rule | self, image_url_only | `website_slides` |  |  |
| `_fetch_youtube_metadata` | internal rule | self, image_url_only | `website_slides` |  | Fetches video metadata from the YouTube API.  Returns a dict containing video metadata with the following keys (matching slide.slide fields): - 'name' matching the video title - 'description' matching the video description - 'image_1920' binary data of the video thumbnail   OR 'image_url' containing an external link to the thumbnail when 'image_url_only' param is True - 'completion_time' matching the video duration   The received duration is under a special format (e.g: PT1M21S15, meaning 1h 21m 15s).  :param image_url_only: if True, will return 'image_url' instead of binary data   Typically u |
| `_fetch_google_drive_metadata` | internal rule | self, image_url_only | `website_slides` |  | Fetches document / video metadata from the Google Drive API.  Returns a dict containing metadata with the following keys (matching slide.slide fields): - 'name' matching the external file title - 'image_1920' binary data of the file thumbnail   OR 'image_url' containing an external link to the thumbnail when 'image_url_only' param is True - 'completion_time' which is computed for 2 types of files:   - pdf files where we download the content and then use slide.slide#_get_completion_time_pdf()   - videos where we use the 'videoMediaMetadata' to extract the 'durationMillis'  :param image_url_only |
| `_fetch_vimeo_metadata` | internal rule | self, image_url_only | `website_slides` |  | Fetches video metadata from the Vimeo API. See https://developer.vimeo.com/api/oembed/showcases for more information.  Returns a dict containing video metadata with the following keys (matching slide.slide fields): - 'name' matching the video title - 'description' matching the video description - 'image_1920' binary data of the video thumbnail   OR 'image_url' containing an external link to the thumbnail when 'fetch_image' param is False - 'completion_time' matching the video duration  :param image_url_only: if False, will return 'image_url' instead of binary data   Typically used when display |
| `_default_website_meta` | preparation rule | self | `website_slides` |  |  |
| `_get_completion_time_pdf` | preparation rule | self, data_bytes | `website_slides` |  | For PDFs, we assume that it takes 5 minutes to read a page. This method receives the data of the PDF as bytes. |
| `_get_next_category` | preparation rule | self | `website_slides` |  |  |
| `get_backend_menu_id` | operation | self | `website_slides` |  |  |
| `_search_get_detail` | search rule | self, website, order, options | `website_slides` | model |  |
| `_search_render_results` | search rule | self, fetch_fields, mapping, icon, limit | `website_slides` |  |  |
| `get_base_url` | operation | self | `website_slides` |  | As website_id is not defined on this record, we rely on channel website_id for base URL. |
| `_mail_get_partner_fields` | messaging hook | self, introspect_fields | `website_slides` |  |  |
| `_compute_name` | computation | self | `website_slides_survey` | depends: `survey_id` |  |
| `_compute_is_preview` | computation | self | `website_slides_survey` | depends: `slide_category` |  |
| `_ensure_challenge_category` | internal rule | self, old_surveys, unlink | `website_slides_survey` |  | If a slide is linked to a survey that gives a badge, the challenge category of this badge must be set to 'slides' in order to appear under the certification badge list on ranks_badges page. If the survey is unlinked from the slide, the challenge category must be reset to 'certification' |
| `_generate_certification_url` | internal rule | self | `website_slides_survey` |  | get a map of certification url for certification slide from `self`. The url will come from the survey user input:     1/ existing and not done user_input for member of the course     2/ create a new user_input for member     3/ for no member, a test user_input is created and the url is returned Note: the slide.slides.partner should already exist  We have to generate a new invite_token to differentiate pools of attempts since the course can be enrolled multiple times. |

## Validation and error messages (6)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `message_post` | AccessError | Not enough karma to comment | `website_slides` |
| `_send_share_email` | UserError | Impossible to send emails. Select a "Share Template" for courses %(course_names)s first | `website_slides` |
| `action_set_viewed` | UserError | You cannot mark a slide as viewed if you are not among its members. | `website_slides` |
| `action_mark_completed` | UserError | You cannot mark a slide as completed if you are not among its members. | `website_slides` |
| `action_mark_uncompleted` | UserError | You cannot mark a slide as uncompleted if you are not among its members. | `website_slides` |
| `_action_set_quiz_done` | UserError | _('You cannot mark a slide quiz as completed if you are not among its members or it is unpublished.') if completed else _('You cannot mark a slide quiz as not completed if you are not among its members or it is unpublished.') | `website_slides` |

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
| Slide: always visible (sub rules exist) | global (all users) | `[(1, '=', 1)]` | True | True | True | True |
| Slide: public: restricted to published or public/link-based channel & (category or previewable) | `[(4, ref('base.group_public'))]` | `[                     ('channel_id.website_published', '=', True),                     ('website_published', '=', True),                     ('channel_id.visibility', 'in', ['public', 'link']),                     '\|',                         ('is_category','=', True),                         ('is_preview', '=', True),                 ]` | 1 | 0 | 0 | 0 |
| Slide: portal/user: restricted to published and connected user, (invited) attendee or link-based, if course visible to attendees only | `[(4, ref('base.group_portal')), (4, ref('base.group_user'))]` | `[                 '&',                     '\|',                         ('user_id', '=', user.id),                         '&',                             ('website_published', '=', True),                             ('channel_id.website_published', '=', True),                     '\|',                         '&',                             '\|',                                 ('channel_id.visibility', 'in', ('public', 'connected',  'link')),                                 ('channel_id.is_member_invited', '=', True),                             '\|',                                 ('is_category', '=', True),                                 ('is_preview', '=', True),                         ('channel_id.is_member', '=', True),                 ]` | 1 | 0 | 0 | 0 |
| Slide: officer: read all | `[(4, ref('group_website_slides_officer'))]` | `[(1, '=', 1)]` | 1 | 0 | 0 | 0 |
| Slide: officer: create/write own only | `[(4, ref('group_website_slides_officer'))]` | `[('channel_id.user_id', '=', user.id)]` | 0 | 1 | 1 | 0 |
| Slide: manager: crud all | `[(4, ref('group_website_slides_manager'))]` | `[(1, '=', 1)]` | 1 | 1 | 1 | 1 |

## Views (9)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website_slides.view_slide_slide_form` | form |  | `channel_type`, `channel_allow_comment`, `slide_views`, `likes`, `dislikes`, `comments_count`, `embed_count`, `is_published`, `image_1920`, `name`, `is_category`, `tag_ids`, `active`, `channel_id`, `slide_category`, `slide_type`, `source_type`, `video_url`, `document_google_url`, `image_google_url`, `document_binary_content`, `image_binary_content`, `user_id`, `completion_time`, `slide_resource_downloadable`, `date_published`, `is_preview`, `public_views`, `total_views`, `description`, `slide_resource_ids`, `sequence`, `resource_type`, `name`, `file_name`, `data`, `link`, `quiz_first_attempt_reward`, `quiz_second_attempt_reward`, `quiz_third_attempt_reward`, `quiz_fourth_attempt_reward`, `question_ids`, `sequence`, `question`, `answer_ids` | `%(slide_slide_partner_action_from_slide)d`, , , , `action_view_embeds` |  | `website_slides` |
| `website_slides.view_slide_slide_form_wo_channel_id` | field | `view_slide_slide_form` | `channel_id` |  |  | `website_slides` |
| `website_slides.slide_slide_view_kanban` | kanban |  | `image_128`, `channel_id`, `name`, `channel_id`, `tag_ids`, `slide_category`, `completion_time`, `questions_count`, `total_views`, `user_id` |  |  | `website_slides` |
| `website_slides.view_slide_slide_tree` | list |  | `name`, `channel_id`, `category_id`, `user_id`, `is_published`, `date_published`, `completion_time` |  |  | `website_slides` |
| `website_slides.slide_slide_view_tree_report` | list |  | `name`, `user_id`, `channel_id`, `category_id`, `date_published`, `total_views`, `questions_count`, `completion_time` |  |  | `website_slides` |
| `website_slides.view_slide_slide_search` | search |  | `name`, `channel_id`, `user_id`, `tag_ids` |  | `My Content`, `Published`, `Waiting for validation`, `Archived`, `Course`, `Category`, `Type` | `website_slides` |
| `website_slides.slide_slide_view_graph` | graph |  | `channel_id`, `slide_category`, `total_views`, `quiz_first_attempt_reward`, `quiz_second_attempt_reward`, `quiz_third_attempt_reward`, `quiz_fourth_attempt_reward`, `sequence` |  |  | `website_slides` |
| `website_slides.slide_slide_view_pivot` | pivot |  | `channel_id`, `total_views` |  |  | `website_slides` |
| `website_slides_survey.slide_slide_view_form` | xpath | `website_slides.view_slide_slide_form` | `survey_id` |  |  | `website_slides_survey` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website_slides.slide_slide_action` | Contents | kanban,list,form | `[('is_category', '=', False)]` | `{'search_default_own_publications':True}` |  | `website_slides` |
| `website_slides.slide_slide_action_report` | Contents | graph,list,form,pivot | `[('is_category', '=', False)]` | `{"search_default_published": 1}` |  | `website_slides` |
| `website_slides_survey.slide_slide_action_certification` | Certifications | list,form,graph | `[('slide_category', '=', 'certification')]` | `{'default_slide_category': 'certification'}` |  | `website_slides_survey` |

## Email templates

| Template | Name | Subject |
|---|---|---|
| `website_slides.slide_template_published` | Elearning: New Course Content Notification | New {{ object.slide_category }} published on {{ object.channel_id.name }} |
| `website_slides.slide_template_shared` | Elearning: Course Share | {{ user.name }} shared a {{ object.slide_category }} with you! |

Machine-readable definition: `../../../schemas/data/entities/slide.slide.json`; views: `../../../schemas/interfaces/views/slide.slide.json`.
