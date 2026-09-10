# Mail Render Mixin (`mail.render.mixin`)

**Transport name:** `mail.render.mixin`  
**Storage name:** `mail_render_mixin`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `mail`  
**Extended by packages:** `link_tracker`, `mass_mailing`

Description: Mail Render Mixin

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `lang` | Language | single line text |  | Help: Optional translation language (ISO code) to select when sending out an email. If not set, the main partner's language will be used. This should usually be a placeholder expression that provides the appropriate language, e.g. {{ object.partner_id.lang }}. |
| `render_model` | Rendering Model | single line text |  | computed by rule `_compute_render_model` (not stored) |

## Operations (29)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_render_model` | computation | self | `mail` |  | Give the target model for rendering. Void by default as models inheriting from ``mail.render.mixin`` should define how to find this model. |
| `_build_expression` | internal rule | self, field_name, sub_field_name, null_value | `mail` | model | Returns a placeholder expression for use in a template field, based on the values provided in the placeholder assistant.  :param field_name: main field name :param sub_field_name: sub field name (M2O) :param null_value: default value if the target value is empty :return: final placeholder expression |
| `_valid_field_parameter` | lifecycle override | self, field, name | `mail` |  |  |
| `create` | lifecycle override | self, vals_list | `mail` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `mail` |  |  |
| `_update_field_translations` | internal rule | self, field_name, translations, digest, source_lang | `mail` |  |  |
| `_replace_local_links` | internal rule | self, html, base_url | `mail` |  | Replace local links by absolute links. It is required in various cases, for example when sending emails on chatter or sending mass mailings. It replaces   * href of links (mailto will not match the regex)  * src of images/v:fill/v:image (base64 hardcoded data will not match the regex)  * styling using url like background-image: url or background="url"  It is done using regex because it is shorter than using an html parser to create a potentially complex soupe and hope to have a result that has not been harmed. |
| `_render_encapsulate` | internal rule | self, layout_xmlid, html, add_context, context_record | `mail` | model | Encapsulate html content (i.e. an email body) in a layout containing more complex html. Used to generate a 'email friendly' content from simple html content.  Typical usage: encapsulate content in email layouts like 'mail_notification_layout' or 'mail_notification_light'. Also used for digest layouts. This leads to some default rendering values being computed here, often used in those templates. |
| `_prepend_preview` | internal rule | self, html, preview | `mail` | model | Prepare the email body before sending. Add the text preview at the beginning of the mail. The preview text is displayed bellow the mail subject of most mail client (gmail, outlook...).  :param html: html content for which we want to prepend a preview :param preview: the preview to add before the html content :return: html with preprended preview |
| `_is_restricted` | internal rule | self | `mail` |  |  |
| `_has_unsafe_expression` | internal rule | self | `mail` |  |  |
| `_has_unsafe_expression_template_qweb` | internal rule | self, template_src, model, fname | `mail` | model |  |
| `_has_unsafe_expression_template_inline_template` | internal rule | self, template_txt, model, fname | `mail` | model |  |
| `_check_access_right_dynamic_template` | validation | self | `mail` |  |  |
| `_render_eval_context` | internal rule | self | `mail` | model | Evaluation context used in all rendering engines. Contains  * ``user``: current user browse record; * ``ctx```: current context; * various formatting tools; |
| `_render_template_qweb` | internal rule | self, template_src, model, res_ids, add_context, options | `mail` | model | Render a raw QWeb template.  In addition to the generic evaluation context available, some other variables are added:   * ``object``: record based on which the template is rendered;  :param str template_src: raw QWeb template to render; :param str model: see ``MailRenderMixin._render_template()``; :param list res_ids: see ``MailRenderMixin._render_template()``;  :param dict add_context: additional context to give to renderer. It   allows to add or update values to base rendering context generated   by ``MailRenderMixin._render_eval_context()``; :param dict options: options for rendering propag |
| `_render_template_qweb_regex` | internal rule | self, template_src, model, res_ids | `mail` | model | Render the template with regex instead of qweb to avoid `eval` call.  Supporting only QWeb allowed expressions, no custom variable in that mode. |
| `_render_template_qweb_view` | internal rule | self, view_ref, model, res_ids, add_context, options | `mail` | model | Render a QWeb template based on an ir.ui.view content.  In addition to the generic evaluation context available, some other variables are added:   * ``object``: record based on which the template is rendered;  :param str/int/record view_ref: source QWeb template. It should be an   XmlID allowing to fetch an ``ir.ui.view``, or an ID of a view or   an ``ir.ui.view`` record; :param str model: see ``MailRenderMixin._render_template()``; :param list res_ids: see ``MailRenderMixin._render_template()``;  :param dict add_context: additional context to give to renderer. It   allows to add or update val |
| `_render_template_inline_template` | internal rule | self, template_txt, model, res_ids, add_context, options | `mail` | model | Render a string-based template on records given by a model and a list of IDs, using inline_template.  In addition to the generic evaluation context available, some other variables are added:   * ``object``: record based on which the template is rendered;  :param str template_txt: template text to render :param str model: see ``MailRenderMixin._render_template()``; :param list res_ids: see ``MailRenderMixin._render_template()``;  :param dict add_context: additional context to give to renderer. It   allows to add or update values to base rendering context generated   by ``MailRenderMixin._render |
| `_render_template_inline_template_regex` | internal rule | self, template_txt, model, res_ids | `mail` | model | Render the inline template in static mode, without calling safe eval. |
| `_render_template_postprocess` | internal rule | self, model, rendered | `mail`, `mass_mailing` | model | Tool method for post processing. In this method we ensure local links ('/shop/Basil-1') are replaced by global links ('https://www. mygarden.com/shop/Basil-1').  :param rendered: result of ``_render_template``;  :returns: updated version of rendered per record ID; :rtype: dict |
| `_process_scheduled_date` | background operation | self, scheduled_date | `mail` | model |  |
| `_render_template_get_valid_options` | internal rule | self | `mail` | model |  |
| `_render_template` | internal rule | self, template_src, model, res_ids, engine, add_context, options | `mail` | model | Render the given string on records designed by model / res_ids using the given rendering engine. Possible engine are small_web, qweb, or qweb_view.  :param str template_src: template text to render or xml id of a qweb view; :param str model: model name of records on which we want to perform   rendering (aka 'crm.lead'); :param list res_ids: list of ids of records. All should belong to the   Odoo model given by model; :param string engine: inline_template, qweb or qweb_view;  :param dict add_context: additional context to give to renderer. It   allows to add or update values to base rendering c |
| `_render_lang` | internal rule | self, res_ids, engine | `mail` |  | Given some record ids, return the lang for each record based on lang field of template or through specific context-based key. Lang is computed by performing a rendering on res_ids, based on self.render_model.  :param list res_ids: list of ids of records. All should belong to the   Odoo model given by model; :param string engine: inline_template or qweb_view;  :return: {res_id: lang code (i.e. en_US)} :rtype: dict |
| `_classify_per_lang` | internal rule | self, res_ids, engine | `mail` |  | Given some record ids, return for computed each lang a contextualized template and its subset of res_ids.  :param list res_ids: list of ids of records (all belonging to same model   defined by self.render_model) :param string engine: inline_template, qweb, or qweb_view; :return: {lang: (template with lang=lang_code if specific lang computed   or template, res_ids targeted by that language} :rtype: dict |
| `_render_field` | internal rule | self, field, res_ids, engine, compute_lang, res_ids_lang, set_lang, add_context, options | `mail` |  | Given some record ids, render a template located on field on all records. ``field`` should be a field of self (i.e. ``body_html`` on ``mail.template``). res_ids are record IDs linked to ``model`` field on self.  :param field: a field name existing on self; :param list res_ids: list of ids of records (all belonging to same model   defined by ``self.render_model``) :param string engine: inline_template, qweb, or qweb_view;  :param boolean compute_lang: compute language to render on translated   version of the template instead of default (probably english) one.   Language will be computed based o |
| `_shorten_links` | internal rule | self, html, link_tracker_vals, blacklist, base_url | `link_tracker` | model | Shorten links in an html content. It uses the '/r' short URL routing introduced in this module. Using the standard Odoo regex local links are found and replaced by global URLs (not including mailto, tel, sms).  TDE FIXME: could be great to have a record to enable website-based URLs  :param link_tracker_vals: values given to the created link.tracker, containing   for example: campaign_id, medium_id, source_id, and any other relevant fields   like mass_mailing_id in mass_mailing; :param list blacklist: list of (local) URLs to not shorten (e.g.   '/unsubscribe_from_list') :param str base_url: eit |
| `_shorten_links_text` | internal rule | self, content, link_tracker_vals, blacklist, base_url | `link_tracker` | model | Shorten links in a string content. Works like ``_shorten_links`` but targeting string content, not html.  :return: updated content |

## Validation and error messages (6)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_access_right_dynamic_template` | AccessError | Only members of %(group_name)s group are allowed to edit templates containing sensible placeholders | `mail` |
| `_render_template_qweb` | UserError | Failed to render QWeb template for %(template_label)s Target Model: %(model_name)s Language context: %(lang_context)s Error: %(error_details)s  Template Source Snippet: %(template_src)s | `mail` |
| `_render_template_qweb` | AccessError | Only members of %(group_name)s group are allowed to edit templates containing sensible placeholders | `mail` |
| `_render_template_qweb_view` | UserError | Failed to render template: %(view_ref)s | `mail` |
| `_render_template_inline_template` | AccessError | Only members of %(group_name)s group are allowed to edit templates containing sensible placeholders | `mail` |
| `_render_template_inline_template` | UserError | Failed to render inline_template template: %(template_txt)s Error details: %(error)s | `mail` |

Machine-readable definition: `../../../schemas/data/entities/mail.render.mixin.json`.
