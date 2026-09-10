# Followers edit wizard (`mail.followers.edit`)

**Transport name:** `mail.followers.edit`  
**Storage name:** `mail_followers_edit`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `mail`

Description: Followers edit wizard

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `res_model` | Related Document Model | single line text |  | required; Help: Model of the followed resource |
| `res_ids` | Related Document identifiers | single line text |  | Help: Ids of the followed resources |
| `operation` | Operation | selection |  | required; default `add` |
| `partner_ids` | Followers | many to many | `res.partner` | required |
| `message` | Message | rich text |  |  |
| `notify` | Notify Recipients | boolean |  | default  |

## Selection values

### `operation` (Operation)

| Value | Label |
|---|---|
| `add` | Add |
| `remove` | Remove |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `edit_followers` | operation | self | `mail` |  |  |
| `_prepare_message_values` | preparation rule | self, documents, model_name | `mail` |  |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `edit_followers` | UserError | No documents found for the selected records. | `mail` |
| `edit_followers` | UserError | Unable to post message, please configure the sender's email address. | `mail` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | no | `mail` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mail.mail_followers_edit_form` | form |  | `res_model`, `res_ids`, `operation`, `partner_ids`, `notify`, `message` | `Update Followers`, `Discard` |  | `mail` |
| `mail.mail_followers_list_edit_form` | data | `mail_followers_edit_form` | `operation` | `edit_followers` |  | `mail` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `crm.mail_followers_edit_action_from_lead` | Add/Remove Followers | form |  | `{'default_res_model': 'crm.lead', 'default_res_ids': active_ids}` | new | `crm` |
| `hr_recruitment.mail_followers_edit_action_from_hr_recruitment` | Add/Remove Followers | form |  | `{'default_res_model': 'hr.applicant', 'default_res_ids': active_ids}` | new | `hr_recruitment` |
| `project.mail_followers_edit_action_from_task` | Add/Remove Followers | form |  | `{'default_res_model': 'project.task', 'default_res_ids': active_ids}` | new | `project` |
| `purchase.mail_followers_edit_action_from_purchase` | Add/Remove Followers | form |  | `{'default_res_model': 'purchase.order', 'default_res_ids': active_ids}` | new | `purchase` |
| `sale.mail_followers_edit_action_from_sale` | Add/Remove Followers | form |  | `{'default_res_model': 'sale.order', 'default_res_ids': active_ids}` | new | `sale` |

Machine-readable definition: `../../../schemas/data/entities/mail.followers.edit.json`; views: `../../../schemas/interfaces/views/mail.followers.edit.json`.
