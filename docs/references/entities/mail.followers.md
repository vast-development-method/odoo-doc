# Document Followers (`mail.followers`)

**Transport name:** `mail.followers`  
**Storage name:** `mail_followers`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mail`  
**Extended by packages:** `sms`

Description: Document Followers

## Fields (7)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `res_model` | Related Document Model Name | single line text |  | required; indexed |
| `res_id` | Related Document identifier | many to one by reference |  | indexed; Help: Id of the followed resource |
| `partner_id` | Related Partner | many to one | `res.partner` | required; indexed; on delete of the target: cascade |
| `subtype_ids` | Subtype | many to many | `mail.message.subtype` | Help: Message subtypes followed, meaning subtypes that will be pushed onto the user's Wall. |
| `name` | Name | single line text |  | related through path `partner_id.name` |
| `email` | Email | single line text |  | related through path `partner_id.email` |
| `is_active` | Is Active | boolean |  | related through path `partner_id.active` |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_mail_followers_res_partner_res_model_id_uniq` | Constraint | `unique(res_model,res_id,partner_id)` | Error, a partner cannot follow twice the same object. | `mail` |

## Operations (12)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_invalidate_documents` | internal rule | self, vals_list | `mail` |  | Invalidate the cache of the documents followed by ``self``.  Modifying followers change access rights to individual documents. As the cache may contain accessible/inaccessible data, one has to refresh it. |
| `create` | lifecycle override | self, vals_list | `mail` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `mail` |  |  |
| `unlink` | lifecycle override | self | `mail` |  |  |
| `_compute_display_name` | computation | self | `mail` | depends: `partner_id` |  |
| `_get_mail_doc_to_followers` | preparation rule | self, mail_ids | `mail` | model | Get partner mail recipients that follows the related record of the mails.  :param list mail_ids: mail_mail ids  :return: for each (model, document_id): list of partner ids that are followers :rtype: dict |
| `_get_recipient_data` | preparation rule | self, records, message_type, subtype_id, pids | `mail`, `sms` |  | Private method allowing to fetch recipients data based on a subtype. Purpose of this method is to fetch all data necessary to notify recipients in a single query. It fetches data from   * followers of records that follow the given subtype if records and    subtype are set;  * partners if pids is given;  :param records: fetch data from followers of ``records`` that follow   ``subtype_id``; :param str message_type: mail.message.message_type in order to allow custom   behavior depending on it (SMS for example); :param int subtype_id: mail.message.subtype to check against followers; :param pids: a |
| `_get_subscription_data` | preparation rule | self, doc_data, pids, include_pshare, include_active | `mail` |  | Private method allowing to fetch follower data from several documents of a given model. MailFollowers can be filtered given partner IDs and channel IDs.  :param doc_data: list of pair (res_model, res_ids) that are the documents from which we   want to have subscription data; :param pids: optional partner to filter; if None take all, otherwise limitate to pids :param include_pshare: optional join in partner to fetch their share status :param include_active: optional join in partner to fetch their active flag  :return: list of followers data which is a list of tuples containing   follower ID,    |
| `_insert_followers` | internal rule | self, res_model, res_ids, partner_ids, subtypes, customer_ids, check_existing, existing_policy | `mail` |  | Main internal method allowing to create or update followers for documents, given a res_model and the document res_ids. This method does not handle access rights. This is the role of the caller to ensure there is no security breach.  :param subtypes: see ``_add_followers``. If not given, default ones are computed. :param customer_ids: see ``_add_default_followers`` :param check_existing: see ``_add_followers``; :param existing_policy: see ``_add_followers``; |
| `_add_default_followers` | internal rule | self, res_model, res_ids, partner_ids, customer_ids, check_existing, existing_policy | `mail` |  | Shortcut to ``_add_followers`` that computes default subtypes. Existing followers are skipped as their subscription is considered as more important compared to new default subscription.  :param customer_ids: optional list of partner ids that are customers. It is used if computing  default subtype is necessary and allow to avoid the check of partners being customers (no  user or share user). It is just a matter of saving queries if the info is already known; :param check_existing: see ``_add_followers``; :param existing_policy: see ``_add_followers``;  :return: see ``_add_followers`` |
| `_add_followers` | internal rule | self, res_model, res_ids, partner_ids, subtypes, check_existing, existing_policy | `mail` |  | Internal method that generates values to insert or update followers. Callers have to handle the result, for example by making a valid ORM command, inserting or updating directly follower records, ... This method returns two main data   * first one is a dict which keys are res_ids. Value is a list of dict of values valid for    creating new followers for the related res_id;  * second one is a dict which keys are follower ids. Value is a dict of values valid for    updating the related follower record;  :param subtypes: optional subtypes for new partner followers. This   is a dict whose keys are |
| `_to_store_defaults` | internal rule | self, target | `mail` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `mail` |
| `base.group_system` | yes | yes | yes | yes | `mail` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mail.view_followers_tree` | list |  | `res_model`, `res_id`, `partner_id` |  |  | `mail` |
| `mail.view_mail_subscription_form` | form |  | `res_model`, `partner_id`, `res_id`, `subtype_ids` |  |  | `mail` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mail.action_view_followers` | Followers | list,form |  |  |  | `mail` |

Machine-readable definition: `../../../schemas/data/entities/mail.followers.json`; views: `../../../schemas/interfaces/views/mail.followers.json`.
