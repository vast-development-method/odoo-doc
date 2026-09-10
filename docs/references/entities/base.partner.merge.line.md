# Merge Partner Line (`base.partner.merge.line`)

**Transport name:** `base.partner.merge.line`  
**Storage name:** `base_partner_merge_line`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `base`

Description: Merge Partner Line

## Identity and behavior

- Default ordering: `min_id asc`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `wizard_id` | Wizard | many to one | `base.partner.merge.automatic.wizard` |  |
| `min_id` | MinID | integer |  |  |
| `aggr_ids` | Ids | single line text |  | required |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_partner_manager` | yes | yes | yes | yes | `base` |

Machine-readable definition: `../../../schemas/data/entities/base.partner.merge.line.json`.
