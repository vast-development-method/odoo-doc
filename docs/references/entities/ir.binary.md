# File streaming helper model for controllers (`ir.binary`)

**Transport name:** `ir.binary`  
**Storage name:** `ir_binary`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `base`  
**Extended by packages:** `website`

Description: File streaming helper model for controllers

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_find_record` | internal rule | self, xmlid, res_model, res_id, access_token, field | `base`, `website` |  | Find and return a record either using an xmlid either a model+id pair. This method is an helper for the ``/web/content`` and ``/web/image`` controllers and should not be used in other contextes.  :param Optional[str] xmlid: xmlid of the record :param Optional[str] res_model: model of the record,     ir.attachment by default. :param Optional[id] res_id: id of the record :param Optional[str] access_token: access token to use instead     of the access rights and access rules. :param Optional[str] field: image field name to check the access to :returns: single record :raises MissingError: when no  |
| `_record_to_stream` | internal rule | self, record, field_name | `base` |  | Low level method responsible for the actual conversion from a model record to a stream. This method is an extensible hook for other modules. It is not meant to be directly called from outside or the ir.binary model.  :param record: the record where to load the data from. :param str field_name: the binary field where to load the data     from. :rtype: odoo.http.Stream |
| `_get_stream_from` | preparation rule | self, record, field_name, filename, filename_field, mimetype, default_mimetype | `base` |  | Create a :class:odoo.http.Stream: from a record's binary field.  :param record: the record where to load the data from. :param str field_name: the binary field where to load the data     from. :param Optional[str] filename: when the stream is downloaded by     a browser, what filename it should have on disk. By default     it is ``{model}-{id}-{field}.{extension}``, the extension is     determined thanks to mimetype. :param Optional[str] filename_field: like ``filename`` but use     one of the record's char field as filename. :param Optional[str] mimetype: the data mimetype to use instead      |
| `_get_image_stream_from` | preparation rule | self, record, field_name, filename, filename_field, mimetype, default_mimetype, placeholder, width, height, crop, quality | `base` |  | Create a :class:odoo.http.Stream: from a record's binary field, equivalent of :meth:`~get_stream_from` but for images.  In case the record does not exist or is not accessible, the alternative ``placeholder`` path is used instead. If not set, a path is determined via :meth:`~odoo.models.BaseModel._get_placeholder_filename` which ultimately fallbacks on ``web/static/img/placeholder.png``.  In case the arguments ``width``, ``height``, ``crop`` or ``quality`` are given, the image will be post-processed and the ETags (the unique cache http header) will be updated accordingly. See also :func:`odoo.t |
| `_get_placeholder_stream` | preparation rule | self, path | `base` |  |  |
| `_placeholder` | internal rule | self, path | `base` |  |  |

## Validation and error messages (4)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_find_record` | MissingError | f'No record found for xmlid={xmlid}, res_model={res_model}, id={res_id}' | `base` |
| `_record_to_stream` | MissingError | The related attachment does not exist. | `base` |
| `_get_stream_from` | UserError | f'Field {field_def!r} is type {field_def.type!r} but it is only possible to stream Binary or Image fields.' | `base` |
| `_get_stream_from` | UserError | f'Record has no field {field_name!r}.' | `base` |

Machine-readable definition: `../../../schemas/data/entities/ir.binary.json`.
