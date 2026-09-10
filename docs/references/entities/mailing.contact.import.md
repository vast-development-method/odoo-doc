# Mailing Contact Import (`mailing.contact.import`)

**Transport name:** `mailing.contact.import`  
**Storage name:** `mailing_contact_import`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `mass_mailing`

Description: Mailing Contact Import

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `mailing_list_ids` | Lists | many to many | `mailing.list` |  |
| `contact_list` | Contact List | multi line text |  | Help: Contact list that will be imported, one contact per line |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `action_import` | user action | self | `mass_mailing` |  | Import each lines of "contact_list" as a new contact. |
| `action_open_base_import` | user action | self | `mass_mailing` |  | Open the base import wizard to import mailing list contacts with a xlsx file. |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mass_mailing.group_mass_mailing_user` | yes | yes | yes | yes | `mass_mailing` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mass_mailing.mailing_contact_import_view_form` | form |  | `mailing_list_ids`, `contact_list` | `action_open_base_import`, `Import`, `Discard` |  | `mass_mailing` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mass_mailing.mailing_contact_import_action` | Import Mailing Contacts | form |  |  | new | `mass_mailing` |

Machine-readable definition: `../../../schemas/data/entities/mailing.contact.import.json`; views: `../../../schemas/interfaces/views/mailing.contact.import.json`.
