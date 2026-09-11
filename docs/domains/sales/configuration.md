# Sales — Configuration

Settings, system parameters, sequences, numbering formats, default records, security groups,
access-rights matrix, record rules and scheduled jobs of the sales domain.

---

## 1. Company-level settings

These are stored on the Company (`res.company`, table `res_company`) and are therefore per company.

| Setting (storage name) | Type | Default | Effect |
|---|---|---|---|
| Online Signature (`portal_confirmation_sign`) | Boolean | true | Default value of the per-order signature requirement. |
| Online Payment (`portal_confirmation_pay`) | Boolean | false | Default value of the per-order payment requirement. |
| Prepayment percentage (`prepayment_percent`) | Decimal fraction | 1.0 | Default fraction of the total the customer must pay to confirm the order from the portal. Must be greater than zero and not greater than one whenever online payment is on. |
| Default Quotation Validity (`quotation_validity_days`) | Integer, days | 30 | Number of days added to today to produce the expiration date of a new quotation. Zero disables automatic expiration. Negative values are refused by a database check. |
| Discount Product (`sale_discount_product_id`) | Link to Product Variant | empty | The product carried by the lines the global-discount dialogue creates. Restricted to service products invoiced on ordered quantities. Created on demand. |
| Downpayment Account (`downpayment_account_id`) | Link to Account | empty | Overrides the account used on advance-invoice lines. Restricted to accounts of the income, other-income and current-liability types. Tracked. |
| Default Sale Template (`sale_order_template_id`) | Link to Quotation Template | empty | Applied to every new quotation of the company. Cleared automatically when the quotation-template feature is switched off or when the template is archived. |
| Sale onboarding payment method (`sale_onboarding_payment_method`) | Selection | empty | Records the guided-setup choice: `digital_signature` "Sign online", `paypal`, `stripe`, `other` "Pay with another payment provider", `manual` "Manual Payment". |

---

## 2. Settings screen

The settings screen of the sales application exposes the following controls. Three kinds exist:
*defaults* (which write a default value for a field of another entity), *feature switches* (which
grant or revoke a group), and *parameters* (which write a system parameter).

### 2.1 Defaults

| Control | Writes |
|---|---|
| Invoicing Policy (`default_invoice_policy`) — values `order` "Invoice what is ordered" (default) and `delivery` "Invoice what is delivered" | The default invoicing policy of new product templates. |

### 2.2 Feature switches

| Control | Group granted | Effect |
|---|---|---|
| Lock Confirmed Sales (`group_auto_done_setting`) | "Lock Confirmed Sales" | Confirmation locks the order automatically. |
| Discounts (`group_discount_per_so_line`) | "Discount on lines" | Enables the per-line discount column and makes price-list rules derive a discount instead of a reduced price. Switching it on also switches on the price-list feature. |
| Pro-Forma Invoice (`group_proforma_sales`) | "Pro-forma Invoices" | Allows sending a pro-forma document. |
| Sale Order Warnings (`group_warning_sale`) | "A warning can be set on a product or a customer (Sale)" | Makes the customer and product warnings visible. |
| Quotation Templates (`group_sale_order_template`) | "Quotation Templates" | Enables templates. Switching it off clears the company default template on every company. |

### 2.3 Parameters

| Control | Parameter | Effect |
|---|---|---|
| Automatic Invoice (`automatic_invoice`) | `sale.automatic_invoice` | The invoice is generated automatically and made available in the customer portal when the payment provider confirms the transaction; the invoice is marked paid and the payment is registered in the payment journal configured on the provider. Advised when the final invoice is issued at the order rather than after the delivery. Forced to false whenever the default invoicing policy is not *ordered quantities*. Switching it on also activates the scheduled job "automatic invoicing: send ready invoice". |
| Email Template (`invoice_mail_template_id`) | `sale.default_invoice_email_template` | The message template used to send the automatically generated invoice. Restricted to templates addressed to the invoice entity. |

### 2.4 Company-related controls

Quotation validity, online signature, online payment, prepayment percentage, the advance-invoice
account and the company default template are shown here as direct mirrors of the company fields of
section 1.

### 2.5 Optional capability switches

Each of these installs or removes a capability package: delivery methods and the individual carrier
connectors, product-specific messages, external marketplace synchronisation, commissions,
print-on-demand production, coupons and loyalty, margins, the document-based quotation builder, the
variant grid entry, and the second marketplace connector. Switching the variant feature off also
switches the grid entry off.

---

## 3. System parameters

| Parameter key | Shipped value | Meaning |
|---|---|---|
| `sale.default_confirmation_template` | the shipped order-confirmation message template | The template used for the confirmation message when the quotation template does not name its own. |
| `sale.default_invoice_email_template` | the shipped invoice message template | The template used to send automatically generated invoices. |
| `sale.async_emails` | `False` | When true, order status messages are queued on the order and sent by the scheduled job instead of being sent inline. |
| `sale.automatic_invoice` | absent by default | When true, a completed payment produces and posts the invoices automatically. |
| `sales_team.membership_multi` | absent (false) | When true, a user may belong to several sales teams at once. |

### 3.1 Parameter-to-job synchronisation

Two parameters directly control the active state of a scheduled job. Creating, writing or deleting
the parameter rewrites the job's active flag:

| Parameter | Job |
|---|---|
| `sale.async_emails` | "Sales: Send pending emails" |
| `sale.automatic_invoice` | "automatic invoicing: send ready invoice" |

Writing the parameter sets the job's active flag to the boolean interpretation of the value;
deleting the parameter deactivates the job.

---

## 4. Sequences and numbering

| Sequence | Code | Prefix | Padding | Company | Produces |
|---|---|---|---|---|---|
| Sales Order | `sale.order` | `S` | 5 | company-neutral | `S00001`, `S00002`, … |

Rules:

1. A number is drawn at creation, and only when the supplied name is absent or equal to the literal
   word `New`.
2. The draw is made in the company of the record being created.
3. The draw is dated with the record's order date, converted from storage into the reader's time
   zone. This matters only when the sequence is configured with a date-dependent prefix.
4. The sequence is shipped as company-neutral, so all companies share one counter unless a
   per-company sequence is added.
5. The number is not copied when an order is duplicated; the duplicate draws a fresh one.
6. Invoices produced from orders are numbered by the receivables domain, from the sale journal's
   own sequence; the sales domain never numbers an invoice.

---

## 5. Security groups

### 5.1 Sales organisation groups

These form a chain: each implies the previous one.

| Group | Implies | Meaning |
|---|---|---|
| User: Own Documents Only | the internal-user group | Access to the user's own data in the sales application. |
| User: All Documents | User: Own Documents Only | Access to every record of everyone in the sales application. |
| Administrator | User: All Documents, plus the canned-response administration group | Access to the sales configuration and to the statistical reports. Granted by default to the system user and to the administrator user. |

All three belong to the privilege "Sales" (sequence 1) of the sales application category.

### 5.2 Feature groups

These are not hierarchical; they switch behaviour on and off.

| Group | Switched by | Effect |
|---|---|---|
| Lock Confirmed Sales | the "Lock Confirmed Sales" setting | Confirmation locks the order. |
| Discount on lines | the "Discounts" setting | The discount column and rule-derived discounts. |
| A warning can be set on a product or a customer (Sale) | the "Sale Order Warnings" setting | Customer and product warnings. |
| Pro-forma Invoices | the "Pro-Forma Invoice" setting | Sending the pro-forma document. |
| Quotation Templates | the "Quotation Templates" setting | Templates and the company default template. |

---

## 6. Access-rights matrix

Read / Write / Create / Delete per group. A blank cell means the group has no rule of its own for
that entity.

### 6.1 Entities of the sales domain

| Entity | Group | R | W | C | D |
|---|---|---|---|---|---|
| Sales Order | Portal user | ✓ | | | |
| Sales Order | Accounting read-only | ✓ | | | |
| Sales Order | Invoicing | ✓ | ✓ | | |
| Sales Order | Accounting | ✓ | ✓ | | |
| Sales Order | User: Own Documents Only | ✓ | ✓ | ✓ | |
| Sales Order | Administrator | ✓ | ✓ | ✓ | ✓ |
| Sales Order Line | Portal user | ✓ | | | |
| Sales Order Line | Accounting read-only | ✓ | | | |
| Sales Order Line | Invoicing | ✓ | ✓ | | |
| Sales Order Line | Accounting | ✓ | ✓ | | |
| Sales Order Line | User: Own Documents Only | ✓ | ✓ | ✓ | ✓ |
| Sales Analysis | User: Own Documents Only | ✓ | | | |
| Quotation Template | User: Own Documents Only | ✓ | | | |
| Quotation Template | Administrator | ✓ | ✓ | ✓ | ✓ |
| Quotation Template | System administrator | ✓ | | | |
| Quotation Template Line | User: Own Documents Only | ✓ | | | |
| Quotation Template Line | Administrator | ✓ | ✓ | ✓ | ✓ |
| Sales Team | Internal user | ✓ | | | |
| Sales Team | User: Own Documents Only | ✓ | | | |
| Sales Team | Administrator | ✓ | ✓ | ✓ | ✓ |
| Sales Team Member | everyone (no group) | | | | |
| Sales Team Member | Internal user | ✓ | | | |
| Sales Team Member | Administrator | ✓ | ✓ | ✓ | ✓ |
| Sales Tag | Internal user | | | | |
| Sales Tag | User: Own Documents Only | ✓ | ✓ | ✓ | |
| Sales Tag | Administrator | ✓ | ✓ | ✓ | ✓ |

Note the deliberate pattern on the Sales Order: a salesperson may create and modify orders but may
**not delete** them; only an administrator may. Deleting is additionally restricted by the status
rule of [business-rules.md](business-rules.md), section 3.4.

### 6.2 Transient dialogues

| Dialogue | Group | R | W | C | D |
|---|---|---|---|---|---|
| Advance Payment Invoice | User: Own Documents Only | ✓ | ✓ | ✓ | |
| Mass Cancel Orders | User: Own Documents Only | ✓ | ✓ | ✓ | |
| Discount | User: Own Documents Only | ✓ | ✓ | ✓ | |
| Payment Link | User: Own Documents Only | ✓ | ✓ | ✓ | |

### 6.3 Access granted to salespeople on other domains' entities

The sales domain grants a salesperson the minimum read access needed to build and invoice an order.

| Entity | Group | R | W | C | D |
|---|---|---|---|---|---|
| Account | User: Own Documents Only | ✓ | | | |
| Account Tag | User: Own Documents Only | ✓ | | | |
| Analytic Account | User: Own Documents Only | ✓ | ✓ | ✓ | |
| Journal | User: Own Documents Only | ✓ | | | |
| Journal Entry | User: Own Documents Only | ✓ | | | |
| Journal Item | User: Own Documents Only | ✓ | | | |
| Partial Reconciliation | User: Own Documents Only | ✓ | | | |
| Payment Term | User: Own Documents Only | ✓ | | | |
| Tax | User: Own Documents Only | ✓ | | | |
| Tax Group | User: Own Documents Only | ✓ | | | |
| Invoice send dialogue (single) | User: Own Documents Only | ✓ | ✓ | ✓ | |
| Invoice send dialogue (batch) | User: Own Documents Only | ✓ | ✓ | ✓ | |
| Customer | User: Own Documents Only | ✓ | | | |
| Customer | Administrator | ✓ | ✓ | ✓ | |
| Activity Type | Administrator | ✓ | ✓ | ✓ | ✓ |
| Activity Plan | Administrator | ✓ | ✓ | ✓ | ✓ |
| Activity Plan Template | Administrator | ✓ | ✓ | ✓ | ✓ |
| Price list | User: Own Documents Only | ✓ | | | |
| Price list | Administrator | ✓ | ✓ | ✓ | ✓ |
| Price list Item | Administrator | ✓ | ✓ | ✓ | ✓ |
| Custom Attribute Value | User: Own Documents Only | ✓ | ✓ | ✓ | ✓ |
| Product Document | Administrator | ✓ | ✓ | ✓ | ✓ |
| Unit of Measure | User: Own Documents Only | ✓ | | | |

---

## 7. Record rules

A record rule restricts which records a group may reach. Rules of the same group are combined with
a logical *or*; rules that name no group apply to everyone and are combined with a logical *and*.

### 7.1 Multi-company rules (apply to everyone)

| Entity | Condition |
|---|---|
| Sales Order | the order's company is among the reader's enabled companies |
| Sales Order Line | the line's company is among the reader's enabled companies |
| Sales Analysis | the row's company is among the reader's enabled companies |
| Sales Team | the team's company is among the reader's enabled companies, or the team has no company |
| Quotation Template | the template's company is among the reader's enabled companies, or the template has no company |

### 7.2 Ownership rules for salespeople

| Entity | Group | Condition |
|---|---|---|
| Sales Order | User: Own Documents Only | the salesperson is the reader, or the order has no salesperson |
| Sales Order | User: All Documents | always true |
| Sales Order Line | User: Own Documents Only | the line's salesperson is the reader, or the line has no salesperson |
| Sales Order Line | User: All Documents | always true |
| Sales Analysis | User: Own Documents Only | the row's salesperson is the reader, or the row has no salesperson |
| Sales Analysis | User: All Documents | always true |
| Sales Team | User: All Documents | always true |

### 7.3 Rules granting salespeople sight of customer invoices

| Entity | Group | Condition |
|---|---|---|
| Journal Entry | User: Own Documents Only | the document is a customer invoice or credit note, **and** its salesperson is the reader or is unset |
| Journal Entry | User: All Documents | the document is a customer invoice or credit note |
| Journal Item | User: Own Documents Only | its document is a customer invoice or credit note, **and** that document's salesperson is the reader or is unset |
| Journal Item | User: All Documents | its document is a customer invoice or credit note |
| Invoice Analysis | User: Own Documents Only | the row's salesperson is the reader or is unset |
| Invoice Analysis | User: All Documents | always true |
| Invoice send dialogue (single) | User: Own Documents Only | the document is a customer invoice or credit note whose salesperson is the reader or is unset |
| Invoice send dialogue (single) | User: All Documents | the document is a customer invoice or credit note |
| Invoice send dialogue (batch) | User: Own Documents Only | every document is a customer invoice or credit note whose salesperson is the reader or is unset |
| Invoice send dialogue (batch) | User: All Documents | every document is a customer invoice or credit note |

### 7.4 Rules granting salespeople sight of payment data

| Entity | Group | Condition |
|---|---|---|
| Payment Transaction | User: Own Documents Only | always true — this deliberately resets the narrower rule of the payment domain |
| Payment Token | User: Own Documents Only | always true — same reset |

### 7.5 Portal rules

| Entity | Group | Condition | Permissions |
|---|---|---|---|
| Sales Order | Portal user | the order's customer is the reader's commercial customer or one of its descendants | read, write and delete allowed by the rule; create denied. Actual write and delete are still blocked by the access matrix, which grants portal users read only. |
| Sales Order Line | Portal user | the line's order belongs to the same customer tree | read |

### 7.6 Dialogue ownership rules

| Entity | Condition |
|---|---|
| Advance Payment Invoice | the record's creator is the reader |
| Mass Cancel Orders | the record's creator is the reader |
| Discount | the record's creator is the reader |

### 7.7 Configuration rules for the sales administrator

| Entity | Group | Condition | Permissions |
|---|---|---|---|
| Activity Plan | Administrator | the plan addresses the order entity | write, create and delete (read is deliberately excluded from the rule so that reading stays governed by the generic rules) |
| Activity Plan Template | Administrator | the plan it belongs to addresses the order entity | same |

---

## 8. Scheduled jobs

| Job | Entity it runs on | Frequency | Active by default | What it does |
|---|---|---|---|---|
| Sales: Send pending emails | Sales Order | every 1 day | no | Finds the orders whose pending-template field is set, sends the stored message synchronously, clears the field, and commits progress order by order so that a long run can be resumed. |
| automatic invoicing: send ready invoice | Payment Transaction | every 1 day | no | Stops immediately when automatic invoicing is off; otherwise finds the completed, post-processed transactions of the last two days whose orders are confirmed and which own a posted but unsent invoice, and sends those invoices. |

Both run as the system user. Both are switched on and off by their system parameter (section 3.1).
Both can also be triggered on demand: the message job is triggered when an order queues a message;
the invoice job is triggered when an invoice becomes ready to be sent, and after an automatic
invoicing run when asynchronous sending is enabled.

---

## 9. Message subtypes

| Subtype | Entity | Internal only | Subscribed by default | Description |
|---|---|---|---|---|
| Quotation sent | Sales Order | no | no | "Quotation sent" |
| Sales Order Confirmed | Sales Order | no | no | "Quotation confirmed" |
| Quotation Viewed | Sales Order | yes | yes | — |
| Quotation sent | Sales Team | no | yes | Parent: the order subtype of the same name; related through the team field. Sequence 20. |
| Quotation Viewed | Sales Team | no | yes | Parent: the order subtype. Sequence 25. |
| Sales Order Confirmed | Sales Team | no | yes | Parent: the order subtype. Sequence 30. |
| Invoice Paid | Sales Team | no | yes | Parent: the receivables domain's invoice-paid subtype. Sequence 35. |
| Invoice Posted | Sales Team | no | no | Parent: the receivables domain's invoice-validated subtype. Sequence 40. |

The team-level subtypes mean that following a sales team makes the follower receive the quotation
and order events of every order assigned to that team.

---

## 10. Message templates shipped

| Template | Addressed to | Subject |
|---|---|---|
| Sales: Send Quotation | Sales Order | "*company name* *Quotation or Order* (Ref *order reference or "n/a"*)" — the word is "Quotation" while the status is `draft` or `sent`, otherwise "Order". |
| Sales: Send Proforma | Sales Order | "*company name* *Proforma or Order* (Ref *order reference or "n/a"*)" — same rule. |
| Sales: Order Confirmation | Sales Order | "*company name* *Pending Order or Order* (Ref *order reference or "n/a"*)" — the words "Pending Order" are used when the order's last transaction is pending. |
| Sales: Payment Done | Sales Order | Same subject rule as the confirmation template. |

The confirmation template is the shipped value of the parameter
`sale.default_confirmation_template`, so replacing that parameter replaces the confirmation message
for the whole database without touching any order.

---

## 11. Default and demonstration records

### 11.1 Shipped default records

| Record | Notes |
|---|---|
| Sales team "Sales" | Sequence 0, no company, led by the administrator user; a membership for the administrator is created unless one already exists. |
| Sales team "Website" | No company, **inactive** by default; activated when the storefront capability is installed. Protected against deletion. |
| Sales team "Point of Sale" | Protected against deletion. |
| Sales tags | Eight tags shipped as reference data: Product, Software, Services, Information, Design, Training, Consulting, Other, with colours 1 to 8. |
| Order sequence | Section 4. |
| Message templates | Section 10. |
| Message subtypes | Section 9. |
| Scheduled jobs | Section 8. |
| System parameters | Section 3. |

### 11.2 Demonstration data

A demonstration quotation template is configured lazily: when it exists and has no lines yet, it is
filled with four ordinary lines and one optional section containing three further lines with a
quantity of zero. This is the shipped illustration of the optional-products mechanism.

---

## 12. Decimal precisions used

| Precision name | Used for | Shipped decimal places |
|---|---|---|
| Product Unit | ordered, delivered, invoiced and to-invoice quantities; zero tests and comparisons in the invoicing and status algorithms | 2 |
| Product Price | the minimum display precision of unit prices and of the margin cost | 2 |
| Discount | the discount percentage on lines, in the analysis and in discount-line descriptions | 2 |

---

## 13. Report definitions

| Report | Produces | Notes |
|---|---|---|
| Quotation / Order | A printable document of the order | Its file name is "*type name* *order reference*", where the type name is "Quotation" or "Sales Order" according to the status. |
| Pro-forma Invoice | A pro-forma rendering of the order | Only available when the pro-forma feature group is enabled. |

When at least one structured-document builder is available and exactly one order is rendered, the
structured document is attached inside the produced file, named by the builder and typed as
extensible-markup-language content.

---

## 14. Digest indicator

| Indicator | Computation | Drill-down |
|---|---|---|
| All Sales (`kpi_all_sale_total`) | The sum of the tax-inclusive totals of the analysis rows whose status is neither `draft`, nor `sent`, nor `cancel`, over the digest period, per company. | The all-channels sales analysis, opened under the sales application menu. |

Reading the indicator without the "User: All Documents" group raises an access error, which the
digest treats as "omit this indicator for this reader".

---

## 15. Menu structure

The sales application root menu is inactive until the sales application is installed. Its tree:

```
Sales
├── Orders
│   ├── Quotations                       (User: Own Documents Only)
│   ├── Orders                           (User: Own Documents Only)
│   ├── Sales Teams                      (Administrator)
│   └── Customers                        (User: Own Documents Only)
├── To Invoice                           (User: Own Documents Only)
│   ├── Orders to Invoice
│   └── Orders to Upsell
├── Products                             (User: Own Documents Only)
│   ├── Products
│   ├── Product Variants                 (product-variant feature)
│   └── Pricelists                       (price-list feature)
├── Reporting                            (Administrator)
│   ├── Sales
│   ├── Salespersons
│   ├── Products
│   └── Customers
└── Configuration                        (Administrator)
    ├── Settings                         (system administrator)
    ├── Sales Teams
    ├── Sales Orders
    │   └── Tags
    ├── Products
    │   ├── Attributes                   (product-variant feature)
    │   ├── Combo Choices
    │   ├── Product Categories
    │   ├── Product Tags
    │   └── Units & Packagings           (multiple-units feature)
    ├── Online Payments                  (system administrator)
    │   ├── Providers
    │   ├── Payment Methods
    │   ├── Tokens                       (developer visibility)
    │   └── Transactions                 (developer visibility)
    └── Activities
        ├── Activity Types               (developer visibility)
        └── Activity Plans               (Administrator)
```

With the quotation-template capability installed, a "Quotation Templates" entry is added under
Configuration, visible when the quotation-template feature group is enabled.

---

## 16. Import templates

| Entity | Template offered |
|---|---|
| Sales Order | "Import Template for Quotations", a spreadsheet at the path `/sale/static/xls/quotations_import_template.xlsx`. |
| Product Template | "Import Template for Products", a spreadsheet at the path `/product/static/xls/product_template.xls`, offered only when the reading context marks the list as the price-list-aware product list and the reader has the price-list feature. |

On import of an order line, the combo-item field is deliberately written **after** the line is
created rather than as part of the creation values, because the integrity rule of
[business-rules.md](business-rules.md), section 4.1, needs the linked line to exist first.

---

## 17. Configuration checklist for a new installation

1. Choose the invoicing policy default: ordered or delivered quantities.
2. Decide whether confirmed orders are locked.
3. Decide whether discounts are shown per line.
4. Set the quotation validity in days, or zero to disable expiration.
5. Decide whether the customer signs online, pays online, or both, and set the prepayment
   percentage.
6. If advance invoices should be treated as a liability, configure the advance-invoice account.
7. If global discounts will be used, configure or let the system create the discount product.
8. Decide whether invoices are produced automatically after an online payment, and pick the message
   template used to send them.
9. Decide whether status messages are sent inline or queued, and enable the corresponding job.
10. Create the sales teams, assign their leaders, members and monthly invoicing targets.
11. Create the quotation templates and, if wanted, set one as the company default.
12. Grant each user one of the three sales-organisation groups.
