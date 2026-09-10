# Portal User Config (`portal.wizard.user`)

**Transport name:** `portal.wizard.user`  
**Storage name:** `portal_wizard_user`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `portal`  
**Extended by packages:** `website`

Description: Portal User Config

## Fields (8)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `wizard_id` | Wizard | many to one | `portal.wizard` | required; on delete of the target: cascade |
| `partner_id` | Contact | many to one | `res.partner` | required; read only; on delete of the target: cascade |
| `email` | Email | single line text |  |  |
| `user_id` | User | many to one | `res.users` | computed by rule `_compute_user_id` (not stored) |
| `login_date` | Latest Authentication | date and time |  | related through path `user_id.login_date` |
| `is_portal` | Is Portal | boolean |  | computed by rule `_compute_group_details` (not stored) |
| `is_internal` | Is Internal | boolean |  | computed by rule `_compute_group_details` (not stored) |
| `email_state` | Status | selection |  | computed by rule `_compute_email_state` (not stored); default `ok` |

## Selection values

### `email_state` (Status)

| Value | Label |
|---|---|
| `ok` | Valid |
| `ko` | Invalid |
| `exist` | Already Registered |

## State fields

State machine fields of this entity: `email_state`. Transitions are specified in the domain documents.

## Operations (14)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_email_state` | computation | self | `portal` | depends: `email` |  |
| `_compute_user_id` | computation | self | `portal` | depends: `partner_id` |  |
| `_compute_group_details` | computation | self | `portal` | depends: `user_id`, `user_id.active`, `user_id.group_ids` |  |
| `action_grant_access` | user action | self | `portal` |  | Grant the portal access to the partner.  If the partner has no linked user, we will create a new one in the same company as the partner (or in the current company if not set).  An invitation email will be sent to the partner. |
| `action_revoke_access` | user action | self | `portal` |  | Archive the portal user of the partner.  User is kept in `group_portal` as `group_public` should only be used for automated tasks and guest interactions. |
| `action_invite_again` | user action | self | `portal` |  | Re-send the invitation email to the partner. |
| `action_refresh_modal` | user action | self | `portal` |  | Refresh the portal wizard modal and keep it open. Used as fallback action of email state icon buttons, required as they must be non-disabled buttons to fire mouse events to show tooltips on email state. |
| `_create_user` | internal rule | self | `portal` |  | create a new user for wizard_user.partner_id :returns record of res.users |
| `_send_email` | internal rule | self | `portal` |  | send notification email to a new portal user |
| `_assert_user_email_uniqueness` | internal rule | self | `portal` |  | Check that the email can be used to create a new user. |
| `_update_partner_email` | internal rule | self | `portal` |  | Update partner email on portal action, if a new one was introduced and is valid. |
| `_get_similar_users_domain` | preparation rule | self, portal_users_with_email | `portal`, `website` |  | Returns the domain needed to find the users that have the same email as portal users. :param portal_users_with_email: portal users that have an email address. |
| `_get_similar_users_fields` | preparation rule | self | `portal`, `website` |  | Returns a list of field elements to extract from users. |
| `_is_portal_similar_than_user` | internal rule | self, user, portal_user | `portal`, `website` |  | Checks if the credentials of a portal user and a user are the same (users are distinct and their emails are similar). |

## Validation and error messages (6)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `action_grant_access` | UserError | The partner "%s" already has the portal access. | `portal` |
| `action_revoke_access` | UserError | The partner "%s" has no portal access or is internal. | `portal` |
| `action_invite_again` | UserError | You should first grant the portal access to the partner "%s". | `portal` |
| `_send_email` | UserError | The template "Portal: new user" not found for sending email to the portal user. | `portal` |
| `_assert_user_email_uniqueness` | UserError | The contact "%s" does not have a valid email. | `portal` |
| `_assert_user_email_uniqueness` | UserError | The contact "%s" has the same email as an existing user | `portal` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_partner_manager` | yes | yes | yes | no | `portal` |

Machine-readable definition: `../../../schemas/data/entities/portal.wizard.user.json`.
