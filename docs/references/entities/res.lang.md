# Languages (`res.lang`)

**Transport name:** `res.lang`  
**Storage name:** `res_lang`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`  
**Extended by packages:** `http_routing`, `spreadsheet`, `survey`, `website`, `point_of_sale`

Description: Languages

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Default ordering: `active desc,name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (14)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `code` | Locale Code | single line text |  | required; Help: This field is used to set/get locales for user |
| `iso_code` | ISO code | single line text |  | Help: This ISO code is the name of po files to use for translations |
| `url_code` | uniform resource locator Code | single line text |  | required; Help: The Lang Code displayed in the URL |
| `active` | Active | boolean |  |  |
| `direction` | Direction | selection |  | required; default `ltr` |
| `date_format` | Date Format | selection |  | required; default `%m/%d/%Y` |
| `time_format` | Time Format | selection |  | required; default `%H:%M:%S` |
| `week_start` | First Day of Week | selection |  | required; default `7` |
| `grouping` | Separator Format | selection |  | required; default `[3,0]`; Help: The International Grouping will represent 123456789 to be 123,456,789.00; The Indian Grouping will represent 123456789 to be 12,34,56,789.00 |
| `decimal_point` | Decimal Separator | single line text |  | required; default `.` |
| `thousands_sep` | Thousands Separator | single line text |  | default `,` |
| `flag_image` | Image | image |  |  |
| `flag_image_url` | Flag Image Uniform resource locator | single line text |  | computed by rule `fields.Char(compute=_compute_field_flag_image_url)` (not stored) |

## Selection values

### `direction` (Direction)

| Value | Label |
|---|---|
| `ltr` | Left-to-Right |
| `rtl` | Right-to-Left |

### `time_format` (Time Format)

| Value | Label |
|---|---|
| `%H:%M:%S` | 13:00:00 |
| `%I:%M:%S %p` | 1:00:00 PM |

### `week_start` (First Day of Week)

| Value | Label |
|---|---|
| `1` | Monday |
| `2` | Tuesday |
| `3` | Wednesday |
| `4` | Thursday |
| `5` | Friday |
| `6` | Saturday |
| `7` | Sunday |

### `grouping` (Separator Format)

| Value | Label |
|---|---|
| `[3,0]` | International Grouping |
| `[3,2,0]` | Indian Grouping |

## Database constraints and indexes (3)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_uniq` | Constraint | `unique(name)` | The name of the language must be unique! | `base` |
| `_code_uniq` | Constraint | `unique(code)` | The code of the language must be unique! | `base` |
| `_url_code_uniq` | Constraint | `unique(url_code)` | The URL code of the language must be unique! | `base` |

## Operations (29)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_date_format_selection` | preparation rule | self | `base` |  |  |
| `_compute_field_flag_image_url` | computation | self | `base` | depends: `code`, `flag_image` |  |
| `_check_active` | validation | self | `base` | constrains: `active` |  |
| `_check_format` | validation | self | `base` | constrains: `time_format`, `date_format` |  |
| `_onchange_format` | on change | self | `base` | onchange: `time_format`, `date_format` |  |
| `_register_hook` | internal rule | self | `base` |  |  |
| `_activate_lang` | internal rule | self, code | `base` |  | Activate languages :param code: code of the language to activate :return: the language matching 'code' activated |
| `_activate_and_install_lang` | internal rule | self, code | `base` |  | Activate languages and update their translations :param code: code of the language to activate :return: the language matching 'code' activated |
| `_create_lang` | internal rule | self, lang, lang_name | `base` |  | Create the given language and make it active. |
| `install_lang` | operation | self | `base` | model | This method is called from odoo/addons/base/data/res_lang_data.xml to load some language and set it as the default for every partners. The language is set via tools.config by the '_initialize_db' method on the 'db' object. This is a fragile solution and something else should be found. |
| `CACHED_FIELDS` | operation | self | `base` |  | Return fields to cache for the active languages Please promise all these fields don't depend on other models and context and are not translated. Warning: Don't add method names of ``dict`` to CACHED_FIELDS for sake of the implementation of LangData |
| `_get_data` | preparation rule | self, **kwargs | `base` |  | Get the language data for the given field value in kwargs For example, get_data(code='en_US') will return the LangData for the res.lang record whose 'code' field value is 'en_US'  :param dict kwargs: ``{field_name: field_value}``         field_name is the only key in kwargs and in ``self.CACHED_FIELDS``         Try to reuse the used ``field_name``: 'id', 'code', 'url_code' :return: Valid LangData if (field_name, field_value) pair is for an         **active** language. Otherwise, Dummy LangData which will return         ``False`` for all ``self.CACHED_FIELDS`` :raise: UserError if field_name is |
| `_lang_get` | internal rule | self, code | `base` |  | Return the language using this code if it is active |
| `_get_code` | preparation rule | self, code | `base` |  | Return the given language code if active, else return ``False`` |
| `get_installed` | operation | self | `base` | model; readonly | Return installed languages' (code, name) pairs sorted by name. |
| `_get_active_by` | preparation rule | self, field | `base` |  | Return a LangDataDict mapping active languages' **unique** **required** ``self.CACHED_FIELDS`` values to their LangData. Its items are ordered by languages' names Try to reuse the used ``field``: 'id', 'code', 'url_code' |
| `action_unarchive` | lifecycle override | self | `base` |  |  |
| `create` | lifecycle override | self, vals_list | `base` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `base`, `survey`, `website` |  | When languages are disabled, clear corresponding survey languages. |
| `_unlink_except_default_lang` | internal rule | self | `base` | ondelete |  |
| `unlink` | lifecycle override | self | `base` |  |  |
| `copy_data` | lifecycle override | self, default | `base` |  |  |
| `format` | operation | self, percent, value, grouping | `base` |  | Format() will return the language-specific output for float values |
| `action_activate_langs` | user action | self | `base`, `website` |  | Activate the selected languages |
| `_get_frontend` | preparation rule | self | `http_routing`, `website` |  | Return the available languages for current request :return: LangDataDict({code: LangData}) |
| `get_locales_for_spreadsheet` | operation | self | `spreadsheet` | readonly; model | Return the list of locales available for a spreadsheet. |
| `_get_user_spreadsheet_locale` | preparation rule | self | `spreadsheet` | model | Convert the odoo lang to a spreadsheet locale. |
| `_odoo_lang_to_spreadsheet_locale` | internal rule | self | `spreadsheet` |  | Convert an odoo lang to a spreadsheet locale. |
| `_load_pos_data_fields` | internal rule | self, config | `point_of_sale` | model |  |

## Validation and error messages (13)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_active` | ValidationError | At least one language must be active. | `base` |
| `_check_format` | ValidationError | Invalid date/time format directive specified. Please refer to the list of allowed directives, displayed when you edit a language. | `base` |
| `_get_active_by` | UserError | Field "%s" is not cached | `base` |
| `write` | UserError | Language code cannot be modified. | `base` |
| `write` | UserError | Cannot deactivate a language that is currently used by users. | `base` |
| `write` | UserError | Cannot deactivate a language that is currently used by contacts. | `base` |
| `write` | UserError | You cannot archive the language in which the system was setup as it is used by automated processes. | `base` |
| `_unlink_except_default_lang` | UserError | Base Language 'en_US' can not be deleted. | `base` |
| `_unlink_except_default_lang` | UserError | You cannot delete the language which is the user's preferred language. | `base` |
| `_unlink_except_default_lang` | UserError | You cannot delete the language which is Active! Please de-activate the language first. | `base` |
| `format` | UserError | The language %s is not installed. | `base` |
| `write` | UserError | error | `survey` |
| `write` | UserError | Cannot deactivate a language that is currently used on a website. | `website` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_public` | no | yes | no | no | `base` |
| `base.group_portal` | no | yes | no | no | `base` |
| `base.group_user` | no | yes | no | no | `base` |
| `group_system` | yes | yes | yes | yes | `base` |

## Views (5)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.res_lang_tree` | list |  | `name`, `code`, `iso_code`, `direction`, `active` | `Activate`, `Activate`, `Update`, `Disable` |  | `base` |
| `base.res_lang_form` | form |  | `flag_image`, `name`, `code`, `iso_code`, `active`, `direction`, `grouping`, `decimal_point`, `thousands_sep`, `date_format`, `time_format`, `week_start` | `%(base.action_view_base_language_install)d` |  | `base` |
| `base.res_lang_search` | search |  | `name`, `direction` |  | `Active` | `base` |
| `http_routing.res_lang_form_inherit_model` | field | `base.res_lang_form` | `iso_code`, `url_code` |  |  | `http_routing` |
| `http_routing.res_lang_tree_inherit_model` | field | `base.res_lang_tree` | `iso_code`, `url_code` |  |  | `http_routing` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.res_lang_act_window` | Languages |  |  | `{'active_test': False}` |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/res.lang.json`; views: `../../../schemas/interfaces/views/res.lang.json`.
