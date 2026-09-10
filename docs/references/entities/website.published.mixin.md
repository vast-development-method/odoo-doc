# Website Published Mixin (`website.published.mixin`)

**Transport name:** `website.published.mixin`  
**Storage name:** `website_published_mixin`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `website`

Description: Website Published Mixin

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `website_published` | Visible on current website | boolean |  | related through path `is_published` |
| `is_published` | Is Published | boolean |  | default computed dynamically (lambda self: self._default_is_published()); indexed; not copied on duplication |
| `can_publish` | Can Publish | boolean |  | computed by rule `_compute_can_publish` (not stored) |
| `website_url` | Website uniform resource locator | single line text |  | computed by rule `_compute_website_url` (not stored); Help: The full relative URL to access the document through the website. |
| `website_absolute_url` | Website Absolute uniform resource locator | single line text |  | computed by rule `_compute_website_absolute_url` (not stored); Help: The full absolute URL to access the document through the website. |

## Operations (10)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_website_url` | computation | self | `website` | depends_context: `lang` |  |
| `_compute_website_absolute_url` | computation | self | `website` | depends: `website_url` |  |
| `_default_is_published` | preparation rule | self | `website` |  |  |
| `website_publish_button` | operation | self | `website` |  |  |
| `open_website_url` | operation | self | `website` |  |  |
| `create` | lifecycle override | self, vals_list | `website` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `website` |  |  |
| `create_and_get_website_url` | operation | self, **kwargs | `website` |  |  |
| `_compute_can_publish` | computation | self | `website` | depends_context: `uid` | This method can be overridden if you need more complex rights management than just write access to the model. The publish widget will be hidden and the user won't be able to change the 'website_published' value if this method sets can_publish False |
| `_get_can_publish_error_message` | preparation rule | self | `website` | model | Override this method to customize the error message shown when the user doesn't have the rights to publish/unpublish. |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `create` | AccessError | self._get_can_publish_error_message() | `website` |
| `write` | AccessError | self._get_can_publish_error_message() | `website` |

Machine-readable definition: `../../../schemas/data/entities/website.published.mixin.json`.
