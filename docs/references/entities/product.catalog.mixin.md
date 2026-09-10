# Product Catalog Mixin (`product.catalog.mixin`)

**Transport name:** `product.catalog.mixin`  
**Storage name:** `product_catalog_mixin`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `product`  
**Extended by packages:** `account`, `stock`

Description: Product Catalog Mixin

## Operations (17)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `action_add_from_catalog` | user action | self | `product` | readonly |  |
| `_default_order_line_values` | preparation rule | self, child_field | `product` |  |  |
| `_get_product_catalog_domain` | preparation rule | self | `product` |  | Get the domain to search for products in the catalog.  For a model that uses products that has to be hidden in the catalog, it must override this method and extend the appropriate domain. :returns: A domain. |
| `_get_product_catalog_record_lines` | preparation rule | self, product_ids, **kwargs | `product` |  | Returns the record's lines grouped by product. Must be overrided by each model using this mixin.  :param list product_ids: The ids of the products currently displayed in the product catalog. :rtype: dict |
| `_get_product_catalog_order_data` | preparation rule | self, products, **kwargs | `product` |  | Returns a dict containing the products' data. Those data are for products who aren't in the record yet. For products already in the record, see `_get_product_catalog_lines_data`.  For each product, its id is the key and the value is another dict with all needed data. By default, the price is the only needed data but each model is free to add more data. Must be overrided by each model using this mixin.  :param products: Recordset of `product.product`. :param dict kwargs: additional values given for inherited models. :rtype: dict :return: A dict with the following structure:     {         'produ |
| `_get_product_catalog_order_line_info` | preparation rule | self, product_ids, child_field, **kwargs | `product` |  | Returns products information to be shown in the catalog. :param list product_ids: The products currently displayed in the product catalog, as a list                          of `product.product` ids. :param dict kwargs: additional values given for inherited models. :rtype: dict :return: A dict with the following structure:     {         'productId': int         'quantity': float (optional)         'productType': string         'price': float         'uomDisplayName': string         'code': string (optional)         'readOnly': bool (optional)     } |
| `_get_action_add_from_catalog_extra_context` | preparation rule | self | `product`, `stock` |  |  |
| `_is_readonly` | internal rule | self | `product` |  | Must be overrided by each model using this mixin. :return: Whether the record is read-only or not. :rtype: bool |
| `_update_order_line_info` | internal rule | self, product_id, quantity, **kwargs | `product` |  | Update the line information for a given product or create a new one if none exists yet. Must be overrided by each model using this mixin. :param int product_id: The product, as a `product.product` id. :param int quantity: The product's quantity. :param dict kwargs: additional values given for inherited models. :return: The unit price of the product, based on the pricelist of the          purchase order and the quantity selected. :rtype: float |
| `_create_section` | internal rule | self, child_field, name, position, **kwargs | `account` |  | Create a new section in order.  :param str child_field: Field name of the order's lines (e.g., 'order_line'). :param str name: The name of the section to create. :param str position: The position of the section where it should be created, either 'top'                       or 'bottom'. :param dict kwargs: Additional values given for inherited models.  :return: A dictionary with newly created section's 'id' and 'sequence'. :rtype: dict |
| `_get_new_line_sequence` | preparation rule | self, child_field, section_id | `account` |  | Compute the sequence number for inserting a new line into the order.  :param str child_field: Field name of the order's lines (e.g., 'order_line'). :param int section_id: ID of the section line to insert after. :rtype: int :return: Computed sequence number. |
| `_get_sections` | preparation rule | self, child_field, **kwargs | `account` |  | Return section data for the product catalog display.  :param str child_field: Field name of the order's lines (e.g., 'order_line'). :param dict kwargs: Additional values given for inherited models. :rtype: list :return: List of section dicts with 'id', 'name', 'sequence', and 'line_count'. |
| `_get_default_create_section_values` | preparation rule | self | `account` |  | Return default values for creating a new section in order through catalog.  :return: A dictionary with default values for creating a new section. :rtype: dict |
| `_get_parent_field_on_child_model` | preparation rule | self | `account` |  | Return the parent field for the order lines.  :return: parent field :rtype: str |
| `_is_line_valid_for_section_line_count` | internal rule | self, line | `account` |  | Check if a line is valid for inclusion in the section's line count.  :param recordset line: A record of an order line. :return: whether this line should be considered in the section lines count. :rtype: bool |
| `_resequence_sections` | internal rule | self, sections, child_field, **kwargs | `account` |  | Resequence the order content based on the new sequence order.  :param list sections: A list of dictionaries containing move and target sections. :param str child_field: Field name of the order's lines (e.g., 'order_line'). :param dict kwargs: Additional values given for inherited models. :return: A dictonary containing the new sequences of all the sections of order. :rtype: dict |
| `_is_display_stock_in_catalog` | internal rule | self | `stock` |  |  |

Machine-readable definition: `../../../schemas/data/entities/product.catalog.mixin.json`.
