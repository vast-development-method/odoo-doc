# Create new or use existing Customer on new Quotation (`crm.quotation.partner`)

**Transport name:** `crm.quotation.partner`  
**Storage name:** `crm_quotation_partner`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `sale_crm`

Description: Create new or use existing Customer on new Quotation

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `action` | Quotation Customer | selection |  | required |
| `lead_id` | Associated Lead | many to one | `crm.lead` | required |
| `partner_id` | Customer | many to one | `res.partner` |  |

## Selection values

### `action` (Quotation Customer)

| Value | Label |
|---|---|
| `create` | Create a new customer |
| `exist` | Link to an existing customer |
| `nothing` | Do not link to a customer |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `sale_crm` | model |  |
| `action_apply` | user action | self | `sale_crm` |  | Convert lead to opportunity or merge lead and opportunity and open the freshly created opportunity view. |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `default_get` | UserError | You can only apply this action from a lead. | `sale_crm` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_salesman` | yes | yes | yes | no | `sale_crm` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `sale_crm.crm_quotation_partner_view_form` | form |  | `action`, `lead_id`, `partner_id` | `Confirm`, `Cancel` |  | `sale_crm` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `sale_crm.crm_quotation_partner_action` | New Quotation | form |  |  | new | `sale_crm` |

Machine-readable definition: `../../../schemas/data/entities/crm.quotation.partner.json`; views: `../../../schemas/interfaces/views/crm.quotation.partner.json`.
