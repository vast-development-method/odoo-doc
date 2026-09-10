# Website Snippet Filter (`website.snippet.filter`)

**Transport name:** `website.snippet.filter`  
**Storage name:** `website_snippet_filter`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website`  
**Extended by packages:** `website_sale`, `website_event`, `website_blog`

Description: Website Snippet Filter

## Identity and behavior

- Mixins (classical inheritance): `website.published.multi.mixin`
- Default ordering: `name ASC`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (9)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `action_server_id` | Server Action | many to one | `ir.actions.server` | on delete of the target: cascade |
| `field_names` | Field Names | single line text |  | required; default ; Help: A list of comma-separated field names |
| `filter_id` | Filter | many to one | `ir.filters` | on delete of the target: cascade |
| `limit` | Limit | integer |  | required; Help: The limit is the maximum number of records retrieved |
| `website_id` | Website | many to one | `website` | on delete of the target: cascade |
| `model_name` | Model name | single line text |  | computed by rule `_compute_model_name` (not stored) |
| `help` | Description | multi line text |  | translatable; Help: Optional help text describing the filter usage and/or purpose. |
| `product_cross_selling` | About cross selling products | boolean |  | Help: True only for product filters that require a product_id because they relate to cross selling |

## Operations (22)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_model_name` | computation | self | `website` | depends: `filter_id`, `action_server_id` |  |
| `_check_data_source_is_provided` | validation | self | `website` | constrains: `action_server_id`, `filter_id` |  |
| `_check_limit` | validation | self | `website` | constrains: `limit` | Limit must be between 1 and 16. |
| `_check_field_names` | validation | self | `website` | constrains: `field_names` |  |
| `_render` | internal rule | self, template_key, limit, search_domain, with_sample, res_model, res_id, **custom_template_data | `website` |  | Renders the website dynamic snippet items |
| `_prepare_values` | preparation rule | self, limit, search_domain, **options | `website_sale`, `website` |  | Gets the data and returns it the right format for render. |
| `_get_field_name_and_type` | preparation rule | self, model, field_name | `website` |  | Separates the name and the widget type  @param model: Model to which the field belongs, without it type is deduced from field_name @param field_name: Name of the field possibly followed by a colon and a forced field type  @return Tuple containing the field name and the field type |
| `_get_filter_meta_data` | preparation rule | self, model | `website` |  | Extracts the meta data of each field  @return OrderedDict containing the widget type for each field name |
| `_prepare_sample` | preparation rule | self, length, **options | `website` |  | Generates sample data and returns it the right format for render.  @param length: Number of sample records to generate @param options: Additional options: - res_model (str): The name of the targeted model.  @return Array of objets with a value associated to each name in field_names |
| `_prepare_sample_records` | preparation rule | self, length, **options | `website` |  | Generates sample records.  @param length: Number of sample records to generate @param options: Additional options: - res_model (str): The name of the targeted model.  @return List of of sample records |
| `_fill_sample` | internal rule | self, model, sample, index | `website` |  | Fills the missing fields of a sample  @param sample: Data structure to fill with values for each name in field_names @param index: Index of the sample within the dataset |
| `_get_hardcoded_sample` | preparation rule | self, model | `website_blog`, `website_event`, `website_sale`, `website` |  | Returns a hard-coded sample  @param model: Model of the currently rendered view  @return Sample data records with field values |
| `_filter_records_to_values` | internal rule | self, records, **options | `website_sale`, `website` |  | Extract the fields from the data source 'records' and put them into a dictionary of values  @param records: Model records returned by the filter @param options: Additional options: - res_model (str): The name of the targeted model. - is_sample (bool): True if conversion is for sample records.  @return List of dict associating the field value to each field name |
| `_get_website_currency` | preparation rule | self | `website_sale`, `website` | model |  |
| `_prepare_category_list_data` | preparation rule | self, parent_id | `website_sale` | model | Return a list of categories to be displayed in the category list snippet. If `parent_id` is provided, return it with its children, otherwise top-level categories.  :param int parent_id: ID of the parent category, if any. :return: List of dictionaries containing category ID, name, and cover image URL. :rtype: list[dict] |
| `_get_products` | preparation rule | self, mode, **kwargs | `website_sale` | model |  |
| `_get_products_latest_sold` | preparation rule | self, website, limit, domain, **kwargs | `website_sale` |  |  |
| `_get_products_latest_viewed` | preparation rule | self, website, limit, domain, **kwargs | `website_sale` |  |  |
| `_get_products_recently_sold_with` | preparation rule | self, website, limit, domain, product_template_id, **kwargs | `website_sale` |  |  |
| `_get_products_accessories` | preparation rule | self, website, limit, domain, product_template_id, **kwargs | `website_sale` |  |  |
| `_get_products_alternative_products` | preparation rule | self, website, limit, domain, product_template_id, **kwargs | `website_sale` |  |  |
| `default_get` | lifecycle override | self, fields | `website_blog`, `website_event`, `website_sale` | model |  |

## Validation and error messages (3)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_data_source_is_provided` | ValidationError | Either action_server_id or filter_id must be provided. | `website` |
| `_check_limit` | ValidationError | The limit must be between 1 and 16. | `website` |
| `_check_field_names` | ValidationError | Empty field name in “%s” | `website` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| all internal users | no | no | no | no | `website` |
| `base.group_system` | yes | yes | yes | yes | `website` |

Machine-readable definition: `../../../schemas/data/entities/website.snippet.filter.json`.
