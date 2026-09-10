# Assets Utils (`website.assets`)

**Transport name:** `website.assets`  
**Storage name:** `website_assets`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `website`

Description: Assets Utils

## Operations (9)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `reset_asset` | operation | self, url, bundle | `website` | model | Delete the potential customizations made to a given (original) asset.  Params:     url (str): the URL of the original asset (scss / js) file      bundle (str):         the name of the bundle in which the customizations to delete         were made |
| `save_asset` | operation | self, url, bundle, content, file_type | `website` | model | Customize the content of a given asset (scss / js).  Params:     url (src):         the URL of the original asset to customize (whether or not the         asset was already customized)      bundle (src):         the name of the bundle in which the customizations will take         effect      content (src): the new content of the asset (scss / js)      file_type (src):         either 'scss' or 'js' according to the file being customized |
| `_get_content_from_url` | preparation rule | self, url, url_info, custom_attachments | `website` | model | Fetch the content of an asset (scss / js) file. That content is either the one of the related file on the disk or the one of the corresponding custom ir.attachment record.  Params:     url (str): the URL of the asset (scss / js) file/ir.attachment      url_info (dict, optional):         the related url info (see _get_data_from_url) (allows to optimize         some code which already have the info and do not want this         function to re-get it)      custom_attachments (ir.attachment(), optional):         the related custom ir.attachment records the function might need         to search into |
| `_get_data_from_url` | preparation rule | self, url | `website` | model | Return information about an asset (scss / js) file/ir.attachment just by looking at its URL.  Params:     url (str): the url of the asset (scss / js) file/ir.attachment  Returns:     dict:         module (str): the original asset's related app          resource_path (str):             the relative path to the original asset from the related app          customized (bool): whether the asset is a customized one or not          bundle (str):             the name of the bundle the asset customizes (False if this             is not a customized asset) |
| `_make_custom_asset_url` | internal rule | self, url, bundle_xmlid | `website` | model | Return the customized version of an asset URL, that is the URL the asset would have if it was customized.  Params:     url (str): the original asset's url     bundle_xmlid (str): the name of the bundle the asset would customize  Returns:     str: the URL the given asset would have if it was customized in the          given bundle |
| `make_scss_customization` | operation | self, url, values | `website` | model | Makes a scss customization of the given file. That file must contain a scss map including a line comment containing the word 'hook', to indicate the location where to write the new key,value pairs.  Params:     url (str):         the URL of the scss file to customize (supposed to be a variable         file which will appear in the assets_frontend bundle)      values (dict):         key,value mapping to integrate in the file's map (containing the         word hook). If a key is already in the file's map, its value is         overridden. |
| `_get_custom_attachment` | preparation rule | self, custom_url, op | `website` | model | Fetch the ir.attachment record related to the given customized asset.  Params:     custom_url (str): the URL of the customized asset     op (str, default: '='): the operator to use to search the records  Returns:     ir.attachment() Only return the attachments related to the current website. |
| `_get_custom_asset` | preparation rule | self, custom_url | `website` | model | Fetch the ir.asset record related to the given customized asset (the inheriting view which replace the original asset by the customized one).  Params:     custom_url (str): the URL of the customized asset  Returns:     ir.asset() Return the views related to the current website. |
| `_add_website_id` | internal rule | self, values | `website` | model |  |

Machine-readable definition: `../../../schemas/data/entities/website.assets.json`.
