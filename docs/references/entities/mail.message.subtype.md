# Message subtypes (`mail.message.subtype`)

**Transport name:** `mail.message.subtype`  
**Storage name:** `mail_message_subtype`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mail`  
**Extended by packages:** `hr_holidays`

Description: Message subtypes

## Identity and behavior

- Default ordering: `sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (10)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Message Type | single line text |  | required; translatable; Help: Message subtype gives a more precise type on the message, especially for system notifications. For example, it can be a notification related to a new record (New), or to a stage change in a process (Stage change). Message subtypes allow to precisely tune the notifications the user want to receive on its wall. |
| `description` | Description | multi line text |  | translatable; Help: Description that will be added in the message posted for this subtype. If void, the name will be added instead. |
| `internal` | Internal Only | boolean |  | Help: Messages with internal subtypes will be visible only by employees, aka members of base_user group |
| `parent_id` | Parent | many to one | `mail.message.subtype` | on delete of the target: set null; Help: Parent subtype, used for automatic subscription. This field is not correctly named. For example on a project, the parent_id of project subtypes refers to task-related subtypes. |
| `relation_field` | Relation field | single line text |  | Help: Field used to link the related model to the subtype model when using automatic subscription on a related document. The field is used to compute getattr(related_document.relation_field). |
| `res_model` | Model | single line text |  | Help: Model the subtype applies to. If False, this subtype applies to all models. |
| `default` | Default | boolean |  | default `True`; Help: Activated by default when subscribing. |
| `sequence` | Sequence | integer |  | default `1`; Help: Used to order subtypes. |
| `hidden` | Hidden | boolean |  | Help: Hide the subtype in the follower options |
| `track_recipients` | Track Recipients | boolean |  | Help: Whether to display all the recipients or only the important ones. |

## Operations (8)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `create` | lifecycle override | self, vals_list | `hr_holidays`, `mail` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `hr_holidays`, `mail` |  |  |
| `unlink` | lifecycle override | self | `mail` |  |  |
| `_get_auto_subscription_subtypes` | preparation rule | self, model_name | `mail` |  | Return data related to auto subscription based on subtype matching. Here model_name indicates child model (like a task) on which we want to make subtype matching based on its parents (like a project).  Example with tasks and project :   * generic: discussion, res_model = False  * task: new, res_model = project.task  * project: task_new, parent_id = new, res_model = project.project, field = project_id  Returned data    * child_ids: all subtypes that are generic or related to task (res_model = False or model_name)   * def_ids: default subtypes ids (either generic or task specific)   * all_int_id |
| `default_subtypes` | operation | self, model_name | `mail` | model | Retrieve the default subtypes (all, internal, external) for the given model. |
| `_default_subtypes` | preparation rule | self, model_name | `mail` |  |  |
| `_get_department_subtype` | preparation rule | self | `hr_holidays` |  |  |
| `_update_department_subtype` | internal rule | self | `hr_holidays` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_public` | no | yes | no | no | `mail` |
| `base.group_portal` | no | yes | no | no | `mail` |
| `base.group_user` | no | yes | no | no | `mail` |
| `base.group_system` | yes | yes | yes | yes | `mail` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| mail.message.subtype: portal/public: read public subtypes | `[Command.link(ref('base.group_portal')), Command.link(ref('base.group_public'))]` | `[('internal', '=', False)]` | True | True | True | True |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mail.view_message_subtype_tree` | list |  | `sequence`, `name`, `res_model`, `default` |  |  | `mail` |
| `mail.view_mail_message_subtype_form` | form |  | `name`, `sequence`, `res_model`, `description`, `default`, `internal`, `hidden`, `track_recipients`, `parent_id`, `relation_field` |  |  | `mail` |
| `mail.mail_message_subtype_view_search` | search |  | `name`, `res_model`, `description` |  | `Default` | `mail` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mail.action_view_message_subtype` | Subtypes | list,form |  |  |  | `mail` |

Machine-readable definition: `../../../schemas/data/entities/mail.message.subtype.json`; views: `../../../schemas/interfaces/views/mail.message.subtype.json`.
