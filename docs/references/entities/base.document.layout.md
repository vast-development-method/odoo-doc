# Company Document Layout (`base.document.layout`)

**Transport name:** `base.document.layout`  
**Storage name:** `base_document_layout`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `web`  
**Extended by packages:** `account`, `sale`, `l10n_din5008`, `l10n_ca`, `l10n_cz`, `l10n_fr_account`, `l10n_ma`, `l10n_mu_account`, `l10n_my_ubl_pint`, `l10n_sk`

Description: Company Document Layout

## Fields (43)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `company_id` | Company | many to one | `res.company` | required; default computed dynamically (lambda self: self.env.company) |
| `logo` | Logo | binary |  | related through path `company_id.logo` |
| `preview_logo` | Preview logo | binary |  | related through path `logo` |
| `report_header` | Report Header | rich text |  | related through path `company_id.report_header` |
| `report_footer` | Report Footer | rich text |  | related through path `company_id.report_footer`; default computed dynamically (_default_report_footer); extended by packages `l10n_din5008`, `l10n_fr_account` |
| `company_details` | Company Details | rich text |  | related through path `company_id.company_details`; default computed dynamically (_default_company_details); extended by packages `l10n_din5008`, `l10n_ma`, `l10n_mu_account` |
| `is_company_details_empty` | Is Company Details Empty | boolean |  | computed by rule `_compute_empty_company_details` (not stored) |
| `paperformat_id` | Paperformat | many to one |  | related through path `company_id.paperformat_id` |
| `external_report_layout_id` | External Report Layout | many to one |  | related through path `company_id.external_report_layout_id` |
| `font` | Font | selection |  | related through path `company_id.font` |
| `primary_color` | Primary Color | single line text |  | related through path `company_id.primary_color` |
| `secondary_color` | Secondary Color | single line text |  | related through path `company_id.secondary_color` |
| `custom_colors` | Custom Colors | boolean |  | computed by rule `_compute_custom_colors` (not stored) |
| `logo_primary_color` | Logo Primary Color | single line text |  | computed by rule `_compute_logo_colors` (not stored) |
| `logo_secondary_color` | Logo Secondary Color | single line text |  | computed by rule `_compute_logo_colors` (not stored) |
| `layout_background` | Layout Background | selection |  | related through path `company_id.layout_background` |
| `layout_background_image` | Layout Background Image | binary |  | related through path `company_id.layout_background_image` |
| `report_layout_id` | Report Layout | many to one | `report.layout` |  |
| `preview` | Preview | rich text |  | computed by rule `_compute_preview` (not stored) |
| `partner_id` | Partner | many to one |  | read only; related through path `company_id.partner_id` |
| `phone` | Phone | single line text |  | read only; related through path `company_id.phone` |
| `email` | Email | single line text |  | read only; related through path `company_id.email` |
| `website` | Website | single line text |  | read only; related through path `company_id.website` |
| `vat` | Value-added tax | single line text |  | related through path `company_id.vat`; extended by packages `account` |
| `name` | Name | single line text |  | read only; related through path `company_id.name` |
| `country_id` | Country | many to one |  | read only; related through path `company_id.country_id` |
| `from_invoice` | From Invoice | boolean |  |  |
| `qr_code` | Quick response Code | boolean |  | related through path `company_id.qr_code` |
| `account_number` | Account Number | single line text |  | computed by rule `_compute_account_number` (not stored); writable through an inverse rule |
| `street` | Street | single line text |  | read only; related through path `company_id.street` |
| `street2` | Street2 | single line text |  | read only; related through path `company_id.street2` |
| `zip` | Zip | single line text |  | read only; related through path `company_id.zip` |
| `city` | City | single line text |  | read only; related through path `company_id.city` |
| `company_registry` | Company Registry | single line text |  | read only; related through path `company_id.company_registry`; extended by packages `l10n_cz`, `l10n_sk` |
| `bank_ids` | Bank | one to many |  | read only; related through path `company_id.partner_id.bank_ids` |
| `account_fiscal_country_id` | Account Fiscal Country | many to one |  | read only; related through path `company_id.account_fiscal_country_id`; extended by packages `l10n_ca`, `l10n_cz`, `l10n_my_ubl_pint`, `l10n_sk` |
| `l10n_din5008_invoice_date` | Localization Din5008 Invoice Date | date |  | default computed dynamically (fields.Date.today) |
| `l10n_din5008_due_date` | Localization Din5008 Due Date | date |  | default computed dynamically (fields.Date.today() + relativedelta(day=7)) |
| `l10n_din5008_delivery_date` | Localization Din5008 Delivery Date | date |  | default computed dynamically (fields.Date.today() + relativedelta(day=7)) |
| `l10n_ca_pst` | Localization Ca Pst | single line text |  | read only; related through path `company_id.l10n_ca_pst` |
| `sst_registration_number` | Sst Registration Number | single line text |  | related through path `company_id.sst_registration_number` |
| `ttx_registration_number` | Ttx Registration Number | single line text |  | related through path `company_id.ttx_registration_number` |
| `income_tax_id` | Income Tax | single line text |  | related through path `company_id.income_tax_id` |

## Operations (19)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_report_footer` | preparation rule | self | `l10n_din5008`, `l10n_fr_account`, `web` | model |  |
| `_default_company_details` | preparation rule | self | `l10n_din5008`, `l10n_ma`, `l10n_mu_account`, `web` | model |  |
| `_clean_address_format` | internal rule | self, address_format, company_data | `web` |  |  |
| `_compute_custom_colors` | computation | self | `web` | depends: `logo_primary_color`, `logo_secondary_color`, `primary_color`, `secondary_color` |  |
| `_compute_logo_colors` | computation | self | `web` | depends: `logo` |  |
| `_compute_preview` | computation | self | `account`, `web` | depends: `report_layout_id`, `logo`, `font`, `primary_color`, `secondary_color`, `report_header`, `report_footer`, `layout_background`, `layout_background_image`, `company_details`; depends: `qr_code`, `account_number` | compute a qweb based preview to display on the wizard |
| `_get_preview_template` | preparation rule | self | `account`, `sale`, `web` |  |  |
| `_get_render_information` | preparation rule | self, styles | `account`, `sale`, `web` |  |  |
| `_onchange_company_id` | on change | self | `web` | onchange: `company_id` |  |
| `_onchange_custom_colors` | on change | self | `web` | onchange: `custom_colors` |  |
| `_onchange_report_layout_id` | on change | self | `web` | onchange: `report_layout_id` |  |
| `_onchange_logo` | on change | self | `web` | onchange: `logo` |  |
| `extract_image_primary_secondary_colors` | operation | self, logo, white_threshold, mitigate | `web` | model | Identifies dominant colors  First resizes the original image to improve performance, then discards transparent colors and white-ish colors, then calls the averaging method twice to evaluate both primary and secondary colors.  :param logo: logo to process :param white_threshold: arbitrary value defining the maximum value a color can reach :param mitigate: arbitrary value defining the maximum value a band can reach  :return: a 2-value tuple with hex values of primary and secondary colors |
| `document_layout_save` | operation | self | `account`, `web` |  | Save layout and onboarding step progress, return super() result |
| `_get_asset_style` | preparation rule | self | `web` |  | Compile the style template. It is a qweb template expecting company ids to generate all the code in one batch. We give a useless company_ids arg, but provide the PREVIEW_ID arg that will prepare the template for '_get_css_for_preview' processing later. :return: |
| `_get_css_for_preview` | preparation rule | self, scss, new_id | `web` | model | Compile the scss into css. |
| `_compute_empty_company_details` | computation | self | `web` | depends: `company_details` |  |
| `_compute_account_number` | computation | self | `account` | depends: `partner_id`, `account_number` |  |
| `_inverse_account_number` | inverse computation | self | `account` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | no | `web` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.view_base_document_layout` | xpath | `web.view_base_document_layout` | `from_invoice`, `vat`, `account_number`, `qr_code` |  |  | `account` |
| `web.view_base_document_layout` | form |  | `company_id`, `external_report_layout_id`, `logo_primary_color`, `logo_secondary_color`, `report_layout_id`, `layout_background`, `layout_background_image`, `font`, `logo`, `primary_color`, `secondary_color`, `custom_colors`, `company_details`, `report_header`, `report_footer`, `paperformat_id`, `preview` | `Continue`, `Discard` |  | `web` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `account.action_base_document_layout_configurator` | Configure your document layout | form |  | `{"dialog_size": "extra-large"}` | new | `account` |
| `web.action_base_document_layout_configurator` | Configure your document layout | form |  | `{"dialog_size": "extra-large"}` | new | `web` |

Machine-readable definition: `../../../schemas/data/entities/base.document.layout.json`; views: `../../../schemas/interfaces/views/base.document.layout.json`.
