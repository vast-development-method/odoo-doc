# Partner Activation (`res.partner.activation`)

**Transport name:** `res.partner.activation`  
**Storage name:** `res_partner_activation`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_crm_partner_assign`

Description: Partner Activation

## Identity and behavior

- Default ordering: `sequence`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `sequence` | Sequence | integer |  |  |
| `name` | Name | single line text |  | required |
| `active` | Active | boolean |  | default `True` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `website_crm_partner_assign` |
| `base.group_partner_manager` | yes | yes | yes | yes | `website_crm_partner_assign` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website_crm_partner_assign.res_partner_activation_form` | form |  | `name`, `sequence` |  |  | `website_crm_partner_assign` |
| `website_crm_partner_assign.res_partner_activation_tree` | list |  | `sequence`, `name`, `active` |  |  | `website_crm_partner_assign` |
| `website_crm_partner_assign.res_partner_activation_view_search` | search |  | `name` |  | `Archived` | `website_crm_partner_assign` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website_crm_partner_assign.res_partner_activation_act` | Partner Activations | list,form |  |  |  | `website_crm_partner_assign` |

Machine-readable definition: `../../../schemas/data/entities/res.partner.activation.json`; views: `../../../schemas/interfaces/views/res.partner.activation.json`.
