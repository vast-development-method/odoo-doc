# Mail Composer Mixin (`mail.composer.mixin`)

**Transport name:** `mail.composer.mixin`  
**Storage name:** `mail_composer_mixin`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `mail`

Description: Mail Composer Mixin

## Identity and behavior

- Mixins (classical inheritance): `mail.render.mixin`

## Fields (7)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `subject` | Subject | single line text |  | computed by rule `_compute_subject` and stored |
| `body` | Contents | rich text |  | computed by rule `_compute_body` and stored |
| `body_has_template_value` | Body content is the same as the template | boolean |  | computed by rule `_compute_body_has_template_value` (not stored) |
| `template_id` | Mail Template | many to one | `mail.template` | restricted by domain `[('model', '=', render_model)]` |
| `lang` | Lang | single line text |  | computed by rule `_compute_lang` and stored; precomputed before insertion |
| `is_mail_template_editor` | Is Editor | boolean |  | computed by rule `_compute_is_mail_template_editor` (not stored) |
| `can_edit_body` | Can Edit Body | boolean |  | computed by rule `_compute_can_edit_body` (not stored) |

## Operations (8)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_subject` | computation | self | `mail` | depends: `template_id` | Computation is coming either from template, either reset. When having a template with a value set, copy it. When removing the template, reset it. |
| `_compute_body` | computation | self | `mail` | depends: `template_id` | Computation is coming either from template, either reset. When having a template with a value set, copy it. When removing the template, reset it. |
| `_compute_body_has_template_value` | computation | self | `mail` | depends: `body`, `template_id` | Computes if the current body is the same as the one from template. Both real and sanitized values are considered, to avoid editor issues as much as possible. |
| `_compute_lang` | computation | self | `mail` | depends: `template_id` | Computation is coming either from template, either reset. When having a template with a value set, copy it. When removing the template, reset it. |
| `_compute_is_mail_template_editor` | computation | self | `mail` | depends_context: `uid` |  |
| `_compute_can_edit_body` | computation | self | `mail` | depends: `template_id`, `is_mail_template_editor` |  |
| `_render_lang` | internal rule | self, res_ids, engine | `mail` |  | Given some record ids, return the lang for each record based on lang field of template or through specific context-based key. This method enters sudo mode to allow qweb rendering (which is otherwise reserved for the 'mail template editor' group') if we consider it safe. Safe means content comes from the template which is a validated master data. As a summary the heuristic is :    * if no template, do not bypass the check;   * if record lang and template lang are the same, bypass the check; |
| `_render_field` | internal rule | self, field, res_ids, *args, **kwargs | `mail` |  | Render the given field on the given records. This method enters sudo mode to allow qweb rendering (which is otherwise reserved for the 'mail template editor' group') if we consider it safe. Safe means content comes from the template which is a validated master data. As a summary the heuristic is :    * if no template, do not bypass the check;   * if current user is a template editor, do not bypass the check;   * if record value and template value are the same (or equals the     sanitized value in case of an HTML field), bypass the check;   * for body: if current user cannot edit it, force temp |

Machine-readable definition: `../../../schemas/data/entities/mail.composer.mixin.json`.
