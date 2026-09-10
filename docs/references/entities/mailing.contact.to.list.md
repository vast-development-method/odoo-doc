# Add Contacts to Mailing List (`mailing.contact.to.list`)

**Transport name:** `mailing.contact.to.list`  
**Storage name:** `mailing_contact_to_list`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `mass_mailing`

Description: Add Contacts to Mailing List

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `contact_ids` | Contacts | many to many | `mailing.contact` |  |
| `mailing_list_id` | Mailing List | many to one | `mailing.list` | required |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `action_add_contacts` | user action | self | `mass_mailing` |  | Simply add contacts to the mailing list and close wizard. |
| `action_add_contacts_and_send_mailing` | user action | self | `mass_mailing` |  | Add contacts to the mailing list and redirect to a new mailing on this list. |
| `_add_contacts_to_mailing_list` | internal rule | self, action | `mass_mailing` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mass_mailing.group_mass_mailing_user` | yes | yes | yes | yes | `mass_mailing` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mass_mailing.mailing_contact_to_list_view_form` | form |  | `mailing_list_id`, `contact_ids` | `Add`, `Add and Send Mailing`, `Cancel` |  | `mass_mailing` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mass_mailing.mailing_contact_to_list_action` | Add Selected Contacts to a Mailing List | form |  |  | new | `mass_mailing` |

Machine-readable definition: `../../../schemas/data/entities/mailing.contact.to.list.json`; views: `../../../schemas/interfaces/views/mailing.contact.to.list.json`.
