# Sample Mail Wizard (`mailing.mailing.test`)

**Transport name:** `mailing.mailing.test`  
**Storage name:** `mailing_mailing_test`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `mass_mailing`

Description: Sample Mail Wizard

## Identity and behavior

- Transient maximum hours: 10.0

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `email_to` | Recipients | multi line text |  | required; default computed dynamically (_default_email_to); Help: Carriage-return-separated list of email addresses. |
| `mass_mailing_id` | Mailing | many to one | `mailing.mailing` | required; on delete of the target: cascade |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_email_to` | preparation rule | self | `mass_mailing` |  | Fetch the last used 'email_to' to populate the email_to value, fallback to user email. This enables a user to do quick successive tests without having to type it every time. As this is a transient model, it will not always work, but is sufficient as just a default value. |
| `send_mail_test` | operation | self | `mass_mailing` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mass_mailing.group_mass_mailing_user` | yes | yes | yes | no | `mass_mailing` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mass_mailing.view_mail_mass_mailing_test_form` | form |  | `email_to` | `Send test`, `Cancel` |  | `mass_mailing` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mass_mailing.action_mail_mass_mailing_test` | Mailing Test | form |  |  | new | `mass_mailing` |

Machine-readable definition: `../../../schemas/data/entities/mailing.mailing.test.json`; views: `../../../schemas/interfaces/views/mailing.mailing.test.json`.
