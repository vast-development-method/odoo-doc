# Product Margin (`product.margin`)

**Transport name:** `product.margin`  
**Storage name:** `product_margin`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `product_margin`

Description: Product Margin

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `from_date` | From | date |  | default computed dynamically (time.strftime('%Y-01-01')) |
| `to_date` | To | date |  | default computed dynamically (time.strftime('%Y-12-31')) |
| `invoice_state` | Invoice State | selection |  | required; default `open_paid` |

## Selection values

### `invoice_state` (Invoice State)

| Value | Label |
|---|---|
| `paid` | Paid |
| `open_paid` | Open and Paid |
| `draft_open_paid` | Draft, Open and Paid |

## State fields

State machine fields of this entity: `invoice_state`. Transitions are specified in the domain documents.

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `action_open_window` | user action | self | `product_margin` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_user` | yes | yes | yes | no | `product_margin` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `product_margin.product_margin_form_view` | form |  | `from_date`, `to_date`, `invoice_state` | `Open Margins`, `Cancel` |  | `product_margin` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `product_margin.product_margin_act_window` | Product Margins | form |  |  | new | `product_margin` |

Machine-readable definition: `../../../schemas/data/entities/product.margin.json`; views: `../../../schemas/interfaces/views/product.margin.json`.
