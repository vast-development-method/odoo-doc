# Website Checkout Step (`website.checkout.step`)

**Transport name:** `website.checkout.step`  
**Storage name:** `website_checkout_step`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_sale`

Description: Website Checkout Step

## Identity and behavior

- Mixins (classical inheritance): `website.published.multi.mixin`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `sequence` | Sequence | integer |  |  |
| `step_href` | Href | single line text |  | required |
| `main_button_label` | Main Button Label | single line text |  | translatable |
| `back_button_label` | Back Button Label | single line text |  | translatable |
| `website_id` | Website | many to one | `website` | on delete of the target: cascade |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_next_checkout_step` | preparation rule | self, allowed_steps_domain | `website_sale` |  | Get the next step in the checkout flow based on the sequence. |
| `_get_previous_checkout_step` | preparation rule | self, allowed_steps_domain | `website_sale` |  | Get the previous step in the checkout flow based on the sequence. |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `website.group_website_designer` | yes | yes | yes | yes | `website_sale` |

Machine-readable definition: `../../../schemas/data/entities/website.checkout.step.json`.
