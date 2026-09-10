# Create links that apply a coupon and redirect to a specific page (`coupon.share`)

**Transport name:** `coupon.share`  
**Storage name:** `coupon_share`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `website_sale_loyalty`

Description: Create links that apply a coupon and redirect to a specific page

## Fields (7)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `website_id` | Website | many to one | `website` | required; default computed dynamically (_get_default_website_id) |
| `coupon_id` | Coupon | many to one | `loyalty.card` | restricted by domain `[('program_id', '=', program_id)]` |
| `program_id` | Program | many to one | `loyalty.program` | required; restricted by domain `["\|", ["program_type", "=", "coupons"], "\|", ["trigger", "=", "with_code"], ["rule_ids.code", "!=", false]]` |
| `program_website_id` | Program Website | many to one | `website` | related through path `program_id.website_id` |
| `promo_code` | Promo Code | single line text |  | computed by rule `_compute_promo_code` (not stored) |
| `share_link` | Share Link | single line text |  | computed by rule `_compute_share_link` (not stored) |
| `redirect` | Redirect | single line text |  | required; default `/shop` |

## Operations (7)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_default_website_id` | preparation rule | self | `website_sale_loyalty` |  |  |
| `_check_program` | validation | self | `website_sale_loyalty` | constrains: `coupon_id`, `program_id` |  |
| `_check_website` | validation | self | `website_sale_loyalty` | constrains: `website_id`, `program_id` |  |
| `_compute_promo_code` | computation | self | `website_sale_loyalty` | depends: `coupon_id.code`, `program_id.rule_ids.code` |  |
| `_compute_share_link` | computation | self | `website_sale_loyalty` | depends: `website_id`, `redirect`; depends_context: `use_short_link` |  |
| `action_generate_short_link` | user action | self | `website_sale_loyalty` |  |  |
| `create_share_action` | operation | self, coupon, program | `website_sale_loyalty` | model |  |

## Validation and error messages (3)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_program` | ValidationError | A coupon is needed for coupon programs. | `website_sale_loyalty` |
| `_check_website` | ValidationError | The shared website should correspond to the website of the program. | `website_sale_loyalty` |
| `create_share_action` | UserError | Provide either a coupon or a program. | `website_sale_loyalty` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_manager` | yes | yes | yes | no | `website_sale_loyalty` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website_sale_loyalty.coupon_share_view_form` | form |  | `website_id`, `program_website_id`, `share_link`, `website_id`, `redirect` | `Done`, `Generate Short Link` |  | `website_sale_loyalty` |

Machine-readable definition: `../../../schemas/data/entities/coupon.share.json`; views: `../../../schemas/interfaces/views/coupon.share.json`.
