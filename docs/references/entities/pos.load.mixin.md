# PoS data loading mixin (`pos.load.mixin`)

**Transport name:** `pos.load.mixin`  
**Storage name:** `pos_load_mixin`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `point_of_sale`  
**Extended by packages:** `pos_self_order`

Description: PoS data loading mixin

## Operations (12)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_load_pos_data_search_read` | internal rule | self, data, config | `point_of_sale` | model | Search and return records to be loaded in the pos |
| `_load_pos_data_domain` | internal rule | self, data, config | `point_of_sale` | model | Return the domain used to filter records |
| `_server_date_to_domain` | internal rule | self, domain | `point_of_sale` | model | Optionally restrict the domain to records modified after the last server sync |
| `_last_server_date_to_load` | internal rule | self | `point_of_sale` |  |  |
| `_load_pos_data_read` | internal rule | self, records, config | `point_of_sale` | model | Read specific fields from the given records |
| `_unrelevant_records` | internal rule | self, config | `point_of_sale` |  |  |
| `_load_pos_data_fields` | internal rule | self, config | `point_of_sale` | model | Return the list of fields to be loaded |
| `_convert_pos_data_currency` | internal rule | self, records, config, price_field, currency_field | `point_of_sale` | model | Convert ``price_field`` of each loaded record to the POS currency.  ``records`` is the list of dicts returned by ``_load_pos_data_read`` and is updated in place. The source currency of each record is read from ``currency_field`` (an ``id``, as fields are read with ``load=False``); records already expressed in the ``config`` currency are left untouched.  ``currency_field`` matters because a product stores its sale price and its cost in two potentially different currencies (``currency_id`` and ``cost_currency_id``): each price must be converted from its own currency. |
| `_load_pos_self_data_search_read` | internal rule | self, data, config | `pos_self_order` | model | Search and return records to be loaded in the self |
| `_load_pos_self_data_domain` | internal rule | self, data, config | `pos_self_order` | model | Return the domain used to filter records |
| `_load_pos_self_data_read` | internal rule | self, records, config | `pos_self_order` | model | Read specific fields from the given records |
| `_load_pos_self_data_fields` | internal rule | self, config | `pos_self_order` | model | Return the list of fields to be loaded |

Machine-readable definition: `../../../schemas/data/entities/pos.load.mixin.json`.
