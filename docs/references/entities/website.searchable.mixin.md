# Website Searchable Mixin (`website.searchable.mixin`)

**Transport name:** `website.searchable.mixin`  
**Storage name:** `website_searchable_mixin`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `website`

Description: Website Searchable Mixin

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_search_build_domain` | search rule | self, domain_list, search, fields, extra | `website` | model | Builds a search domain AND-combining a base domain with partial matches of each term in the search expression in any of the fields.  :param domain_list: base domain list combined in the search expression :param search: search expression string :param fields: list of field names to match the terms of the search expression with :param extra: function that returns an additional subdomain for a search term  :return: domain limited to the matches of the search expression |
| `_search_get_detail` | search rule | self, website, order, options | `website` | model | Returns indications on how to perform the searches  :param website: website within which the search is done :param order: order in which the results are to be returned :param options: search options  :return: search detail as expected in elements of the result of website._search_get_details()     These elements contain the following fields:     - model: name of the searched model     - base_domain: list of domains within which to perform the search     - search_fields: fields within which the search term must be found     - fetch_fields: fields from which data must be fetched     - mapping: ma |
| `_search_fetch` | search rule | self, search_detail, search, limit, order | `website` | model |  |
| `_search_render_results` | search rule | self, fetch_fields, mapping, icon, limit | `website` |  |  |

Machine-readable definition: `../../../schemas/data/entities/website.searchable.mixin.json`.
