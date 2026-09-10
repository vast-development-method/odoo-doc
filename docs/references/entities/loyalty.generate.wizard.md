# Generate Coupons (`loyalty.generate.wizard`)

**Transport name:** `loyalty.generate.wizard`  
**Storage name:** `loyalty_generate_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `loyalty`

Description: Generate Coupons

## Fields (12)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `program_id` | Program | many to one | `loyalty.program` | required; default computed dynamically (lambda self: self.env.context.get('active_id', False) or self.env.context.get('default_program_id', False)) |
| `program_type` | Program Type | selection |  | related through path `program_id.program_type` |
| `mode` | For | selection |  | required; default `anonymous` |
| `customer_ids` | Customers | many to many | `res.partner` |  |
| `customer_tag_ids` | Customer Tags | many to many | `res.partner.category` |  |
| `coupon_qty` | Quantity | integer |  | computed by rule `_compute_coupon_qty` and stored |
| `points_granted` | Grant | float |  | required; default `1` |
| `points_name` | Points Name | single line text |  | read only; related through path `program_id.portal_point_name` |
| `valid_until` | Valid Until | date |  |  |
| `will_send_mail` | Will Send Mail | boolean |  | computed by rule `_compute_will_send_mail` (not stored) |
| `confirmation_message` | Confirmation Message | single line text |  | computed by rule `_compute_confirmation_message` (not stored) |
| `description` | Description | multi line text |  |  |

## Selection values

### `mode` (For)

| Value | Label |
|---|---|
| `anonymous` | Anonymous Customers |
| `selected` | Selected Customers |

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_partners` | preparation rule | self | `loyalty` |  |  |
| `_compute_confirmation_message` | computation | self | `loyalty` | depends: `program_type`, `points_granted`, `coupon_qty` |  |
| `_compute_coupon_qty` | computation | self | `loyalty` | depends: `customer_ids`, `customer_tag_ids`, `mode` |  |
| `_compute_will_send_mail` | computation | self | `loyalty` | depends: `mode`, `program_id` |  |
| `_get_coupon_values` | preparation rule | self, partner | `loyalty` |  |  |
| `generate_coupons` | operation | self | `loyalty` |  |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `generate_coupons` | ValidationError | Can not generate coupon, no program is set. | `loyalty` |
| `generate_coupons` | ValidationError | Invalid quantity. | `loyalty` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | no | no | no | `loyalty` |
| `point_of_sale.group_pos_user` | yes | yes | yes | no | `pos_loyalty` |
| `sales_team.group_sale_salesman` | yes | yes | yes | no | `sale_loyalty` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `loyalty.loyalty_generate_wizard_view_form` | form |  | `program_id`, `will_send_mail`, `program_type`, `mode`, `customer_ids`, `customer_tag_ids`, `coupon_qty`, `points_granted`, `points_name`, `points_granted`, `points_name`, `points_granted`, `points_name`, `valid_until`, `description`, `confirmation_message`, `program_type` | `generate_coupons`, `Cancel` |  | `loyalty` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `loyalty.loyalty_generate_wizard_action` | Generate | form |  |  | new | `loyalty` |

Machine-readable definition: `../../../schemas/data/entities/loyalty.generate.wizard.json`; views: `../../../schemas/interfaces/views/loyalty.generate.wizard.json`.
