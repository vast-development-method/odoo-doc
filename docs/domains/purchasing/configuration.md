# Purchasing — Configuration

Everything an administrator can set, everything the platform ships as data, every numbering
series, every privilege, the full access-rights matrix, every record rule and every scheduled
job of the purchasing domain.

> **Reproduced text.** Status labels, button labels, subtype names and message bodies are
> reproduced exactly as the system produces them, because a rebuilt implementation must
> produce the same text. Some shipped strings contain the short form of *request for
> quotation*; that short form appears only inside such reproduced strings and never in this
> specification's own prose. See the conventions in [`README.md`](README.md).

---

## 1. Company-level settings

These are stored on the company record, so each company in a multi-company installation has
its own values. The settings screen presents them; the stored values are what the behaviour
reads.

| Setting as presented | Stored field | Type | Shipped default | Effect |
|---|---|---|---|---|
| Lock Confirmed Orders | `po_lock` (purchase order lock) | Selection: `edit` = "Allow to edit purchase orders", `lock` = "Confirmed purchase orders are not editable" | `edit` | When `lock`, approving an order also sets its locked flag, which makes the document read-only and refuses cancellation. |
| Purchase Order Approval | `po_double_validation` (purchase order double validation) | Selection: `one_step` = "Confirm purchase orders in one step", `two_step` = "Get 2 levels of approvals to confirm a purchase order" | `one_step` | When `two_step`, confirming an order at or above the amount below stops it at the "To Approve" status unless the confirming user is a purchase administrator. |
| Minimum Amount | `po_double_validation_amount` (purchase order double validation amount) | Monetary in the company currency | 5000 | The threshold used by the approval test. |
| Days to Purchase | `days_to_purchase` | Decimal | 0 (the field ships with no value, which behaves as zero) | Added by a buy rule to the cumulative lead time, so the scheduler starts the purchase this many days earlier. Present when inventory is installed. |

The settings screen exposes two of these through booleans that are translated into the stored
selections when the screen is saved:

- *Lock Confirmed Orders* writes `lock` when ticked and `edit` when not.
- *Purchase Order Approval* writes `two_step` when ticked and `one_step` when not.

The translation is one-way: ticking the boolean sets the selection, and the boolean itself is
initialised from the selection when the screen opens.

## 2. Capability switches on the settings screen

These do not store a value of their own; they install or uninstall an optional capability, or
grant a privilege to the reference user group.

| Switch | What it does |
|---|---|
| Purchase Warnings | Grants or revokes the purchase-warnings privilege. Without it every warning text computes to empty. |
| Receipt Reminder | Grants or revokes the receipt-reminder privilege. Ticked by default. Without it neither the automatic nor the manual reminder does anything. |
| 3-way matching: purchases, receptions and bills | Installs the optional three-way-matching capability, which adds a "should be paid" indicator to vendor bills. |
| Purchase Agreements | Installs the purchase agreements capability, which adds blanket orders, purchase templates and alternative requests for quotation. |
| Purchase Grid Entry | Installs the product grid capability. Ticking it also ticks the product variants switch, because a grid is meaningless without variants; and un-ticking the product variants switch un-ticks this one. |
| Dropshipping | Installs the drop-shipping capability. Present when inventory is installed. |
| Purchase Alternatives | Grants or revokes the purchase-alternatives privilege. Present when purchase agreements are installed. |

## 3. Partner-level settings

Set on each vendor. Those marked *per company* store a different value per company.

| Setting | Stored field | Type | Default | Effect |
|---|---|---|---|---|
| Supplier Currency | `property_purchase_currency_id` | Many-to-one to Currency, per company | none | The currency used for orders placed with this vendor, and the default currency of a vendor pricelist entry created for it. |
| Message for Purchase Order | `purchase_warn_msg` | Long text | none | Surfaced on orders and bills for this partner and for its child contacts through the parent. |
| Receipt Reminder | `receipt_reminder_email` | Boolean, per company | false | Enables the automatic reminder for orders placed with this vendor. |
| Days Before Receipt | `reminder_date_before_receipt` | Integer, per company | 1, shipped as a platform-wide fallback | How many days before the expected arrival the reminder is sent. |
| Buyer | `buyer_id` | Many-to-one to User | none | The default buyer of orders placed with this vendor, and part of the grouping key the buy rule uses. |
| Group Requests for Quotation | `group_rfq` | Selection: `default` = "On Order", `day` = "Daily", `week` = "Weekly", `all` = "Always" | `default` | How procurement needs for this vendor are merged onto one request for quotation. Present when inventory is installed. |
| Week Day | `group_on` | Selection: `default` = "Expected Date", `1` to `7` = Monday to Sunday | `default` | The target week day when weekly grouping is used. Present when inventory is installed. |
| Suggestion Basis | `suggest_based_on` | Text | `30_days` | Remembered from the last use of the catalog suggestion panel. Present when inventory is installed. |
| Suggestion Days | `suggest_days` | Integer | 7 | Same. |
| Suggestion Percentage | `suggest_percent` | Integer | 100 | Same. |

## 4. Product-level settings

| Setting | Stored field | Where | Default | Effect |
|---|---|---|---|---|
| Purchasable | `purchase_ok` | Product template | true | A product that is not purchasable cannot be chosen on a purchase order line. |
| Control Policy | `purchase_method` | Product template | Computed: `purchase` for a service, otherwise the configured field default, which the platform ships as `receive` | Decides whether the quantity to bill is derived from the ordered or from the received quantity. |
| Purchase Description | `description_purchase` | Product template | none | Appended to the description of every purchase order line for the product. |
| Message for Purchase Order Line | `purchase_line_warn_msg` | Product template | none | Surfaced on order lines, bill lines and the alternative-creation assistant. |
| Subcontract Service | `service_to_purchase` | Product template, per company | false | When ticked, confirming a sales order line for this service creates or extends a request for quotation. Only valid on a service that has at least one vendor. Present when the sales-purchase bridge is installed. |
| Print Variant Grids | `report_grids` | Purchase order | true | Whether the variant grid is printed on the order document. Present when the product grid capability is installed. |

## 5. System parameters

| Parameter key | Full name | Shipped value | Effect |
|---|---|---|---|
| `purchase_stock.on_time_delivery_days` | The purchase-stock on-time-delivery-days parameter | not shipped; the code defaults to `365` | The length in days of the window over which a vendor's on-time delivery rate is computed. |

## 6. Numbering series

Three series are shipped. Each is company-independent as shipped, which means one shared
counter across companies; an administrator may create per-company variants.

| Series code | Name | Prefix | Padding | Example | Used for |
|---|---|---|---|---|---|
| `purchase.order` | Purchase Order | `P` | 5 | `P00007` | The reference of every request for quotation and purchase order. |
| `purchase.requisition.blanket.order` | Blanket Order | `BO` | 5 | `BO00003` | The name of a blanket-order agreement. |
| `purchase.requisition.purchase.template` | Purchase Template | `PT` | 5 | `PT00002` | The name of a purchase-template agreement. |

### 6.1 How the order reference is drawn

1. When a record is created without an explicit reference, or with the literal word `New`, the
   creation routine asks the series for the next value.
2. The request is made **in the record's company**, so a per-company series is honoured.
3. When the values carry an order deadline, that deadline — converted to the user's time zone —
   is passed as the sequence date, so a series with a date-based prefix produces a number in
   the right period.
4. If the series yields nothing at all, the reference becomes the single character `/`.

### 6.2 How an agreement name is drawn

1. On creation, the type (defaulting to blanket order) and the company (defaulting to the
   active company) are read from the values.
2. The series of that type is asked, in that company, for the next value. The result always
   replaces whatever name was supplied.
3. When the type or the company of a **draft** agreement changes, the name is drawn again from
   the series of the new type in the new company. Changing either on a non-draft agreement is
   refused.

## 7. Shipped default records

| Record | Value |
|---|---|
| Platform-wide fallback for the partner field *Days Before Receipt* | 1 |
| Buy route | A route named *Buy*, with no company, sequence 10, not selectable on a product, selectable on a warehouse, and linked to the reference warehouse. Present when inventory is installed. |
| Buy-to-resupply on the reference warehouse | Enabled. Present when inventory is installed. |
| Message subtypes on the purchase order thread | *RFQ Confirmed*, *RFQ Approved*, *RFQ Sent* — all three not subscribed by default. |
| Share action | A bound action named *Share* on the purchase order form, which produces a portal share link. |
| Digest tips | Two tips shown to purchase users: *Tip: How to keep late receipts under control?* (sequence 100) and *Tip: Never miss a purchase order* (sequence 2000). |
| One-time data fix when inventory is installed | Every purchase order that is not in the purchase status has its received-quantity method recomputed, so that pre-existing lines adopt the stock-moves method. |

### 7.1 The buy rule on a warehouse

When inventory is installed, every warehouse gains a *Buy to Resupply* flag, true by default,
and a reference to its own buy rule. Enabling the flag links the warehouse to the shared *Buy*
route; disabling it unlinks it. The rule itself is generated with:

| Rule property | Value |
|---|---|
| Action | Buy |
| Operation type | The warehouse's incoming operation type |
| Company | The warehouse's company |
| Route | The shared *Buy* route |
| Destination location | The warehouse's stock location |
| Name | The warehouse's rule-naming pattern applied to the stock location and the word *Buy* |
| Active | The warehouse's buy-to-resupply flag |
| Propagate cancellation | True unless the warehouse receives goods in one step |

The rule is regenerated whenever the warehouse's reception steps or its buy-to-resupply flag
change. When the purchasing-with-inventory capability is installed after warehouses already
exist, an installation hook enables buy-to-resupply on every warehouse that has no buy rule
yet.

A buy rule additionally declares that its operation type must be an incoming one, and clears
its source location when the action is set to buy, because goods come from outside.

## 8. Privileges

| Privilege | Internal name | Belongs to | Implies |
|---|---|---|---|
| User | `group_purchase_user` | The *Purchase* privilege family, sequence 10, in the supply-chain category | The reference internal-user group |
| Administrator | `group_purchase_manager` | The same family, sequence 20 | The purchase user privilege |
| A warning can be set on a product or a customer (Purchase) | `group_warning_purchase` | Standalone | — |
| Send an automatic reminder email to confirm delivery | `group_send_reminder` | Standalone | — |
| Manage Purchase Alternatives | `group_purchase_alternatives` | Standalone, shipped with the agreements capability | — |

The reference internal-user group is shipped implying the reminder privilege, so every internal
user may send reminders unless an administrator removes it. The administrator privilege is
shipped assigned to the system user and to the default administrator account.

The *Purchase* privilege family is shipped with sequence 8 inside the supply-chain application
category.

## 9. Access-rights matrix

Rows are entity and privilege; the four columns are create, read, update and delete. A blank
cell means the operation is not granted by that row. Access is the union of every row that
applies to the reader.

### 9.1 Entities owned by purchasing

| Entity | Privilege | Create | Read | Update | Delete |
|---|---|---|---|---|---|
| Purchase Order | Purchase user | ✔ | ✔ | ✔ | ✔ |
| Purchase Order | Purchase administrator | ✔ | ✔ | ✔ | ✔ |
| Purchase Order | Accounting read-only | | ✔ | | |
| Purchase Order | Accounting invoicing | | ✔ | ✔ | |
| Purchase Order | Portal | | ✔ | | |
| Purchase Order | Inventory user | | ✔ | | |
| Purchase Order Line | Purchase user | ✔ | ✔ | ✔ | ✔ |
| Purchase Order Line | Purchase administrator | ✔ | ✔ | ✔ | ✔ |
| Purchase Order Line | Accounting read-only | | ✔ | | |
| Purchase Order Line | Accounting invoicing | | ✔ | ✔ | |
| Purchase Order Line | Portal | | ✔ | | |
| Purchase Order Line | Inventory user | | ✔ | | |
| Purchase Agreement | Purchase user | ✔ | ✔ | ✔ | ✔ |
| Purchase Agreement | Purchase administrator | | ✔ | | |
| Purchase Agreement Line | Purchase user | ✔ | ✔ | ✔ | ✔ |
| Purchase Agreement Line | Purchase administrator | | ✔ | | |
| Alternative Order Group | Purchase user | ✔ | ✔ | ✔ | ✔ |
| Alternative Order Creation Assistant | Purchase user | ✔ | ✔ | ✔ | ✔ |
| Alternative Order Warning Assistant | Purchase user | ✔ | ✔ | ✔ | ✔ |
| Bill To Purchase Order Assistant | Purchase user | ✔ | ✔ | ✔ | |
| Purchase and Bill Line Match Entry | Purchase user | | ✔ | | |
| Purchase and Bill Line Match Entry | Accounting read-only | | ✔ | | |
| Purchase and Bill Line Match Entry | Accounting invoicing | | ✔ | ✔ | |
| Purchases and Bills Union Entry | Purchase user | | ✔ | | |
| Purchase Analysis Entry | Purchase user | | ✔ | | |
| Purchase Analysis Entry | Purchase administrator | | ✔ | | |
| Vendor Delay Entry | Purchase user | | ✔ | | |
| Vendor Delay Entry | Purchase administrator | | ✔ | | |

Note the asymmetry on agreements: the **user** privilege grants full access while the
**administrator** privilege grants only read. Because the administrator privilege implies the
user privilege, an administrator still has full access; the read-only row exists so that the
matrix is complete for an installation that assigns the administrator privilege alone.

### 9.2 Entities of other domains that purchasing grants access to

| Entity | Privilege | Create | Read | Update | Delete |
|---|---|---|---|---|---|
| Tax | Purchase user | | ✔ | | |
| Tax | Purchase administrator | | ✔ | | |
| Account Tag | Purchase user | | ✔ | | |
| Product Variant | Purchase user | | ✔ | | |
| Product Template | Purchase user | | ✔ | | |
| Fiscal Position | Purchase user | | ✔ | | |
| Partner | Purchase user | | ✔ | | |
| Partner | Purchase administrator | ✔ | ✔ | ✔ | |
| Journal | Purchase user | | ✔ | | |
| Journal | Purchase administrator | | ✔ | | |
| Journal Entry | Purchase user | ✔ | ✔ | ✔ | ✔ |
| Journal Item | Purchase user | ✔ | ✔ | ✔ | |
| Journal Item | Purchase administrator | ✔ | ✔ | ✔ | ✔ |
| Analytic Line | Purchase user | | ✔ | | |
| Partial Reconciliation | Purchase user | | ✔ | | |
| Vendor Pricelist Entry | Purchase administrator | ✔ | ✔ | ✔ | ✔ |
| Pricelist Rule | Purchase administrator | ✔ | ✔ | ✔ | ✔ |
| Account | Purchase administrator | | ✔ | | |

### 9.3 Additional rows when inventory is installed

| Entity | Privilege | Create | Read | Update | Delete |
|---|---|---|---|---|---|
| Location | Purchase user | | ✔ | | |
| Location | Purchase administrator | | ✔ | | |
| Warehouse | Purchase user | | ✔ | | |
| Warehouse | Purchase administrator | | ✔ | | |
| Transfer | Purchase user | ✔ | ✔ | ✔ | ✔ |
| Transfer | Purchase administrator | ✔ | ✔ | ✔ | ✔ |
| Stock Move | Purchase user | ✔ | ✔ | ✔ | |
| Stock Move | Purchase administrator | ✔ | ✔ | ✔ | ✔ |
| Reordering Rule | Purchase user | | ✔ | | |
| Reordering Rule | Purchase administrator | | ✔ | | |

The two rows that grant an inventory user read access to purchase orders and purchase order
lines also belong to this group.

## 10. Record rules

| Rule | Entity | Domain | Applies to | Operations |
|---|---|---|---|---|
| Purchase Order multi-company | Purchase Order | The order's company is among the reader's allowed companies | Everyone | All |
| Purchase Order Line multi-company | Purchase Order Line | The line's stored company is among the reader's allowed companies | Everyone | All |
| Purchase Requisition multi-company | Purchase Agreement | Same shape | Everyone | All |
| Purchase requisition Line multi-company | Purchase Agreement Line | Same shape | Everyone | All |
| Purchases and Bills Union multi-company | Purchases and Bills Union Entry | The row's company is among the reader's allowed companies, or is empty | Everyone | All |
| Purchase Order Report multi-company | Purchase Analysis Entry | The row's company is among the reader's allowed companies | Everyone | All |
| Portal Purchase Orders | Purchase Order | The vendor is the reader's commercial partner or one of its descendants | Portal | Read, update, delete — **not** create |
| Portal Purchase Order Lines | Purchase Order Line | The order's vendor is the reader's commercial partner or one of its descendants | Portal | All |
| Purchase User Account Move | Journal Entry | The document type is a vendor bill, a vendor refund or a purchase receipt | Purchase user | All |
| Purchase User Account Move Line | Journal Item | The parent document's type is a vendor bill, a vendor refund or a purchase receipt | Purchase user | All |

All of these are shipped as non-updatable data, so an upgrade never rewrites an administrator's
changes to them.

## 11. Scheduled jobs

| Job | Name | Runs | As | What it does |
|---|---|---|---|---|
| Purchase reminder | *Purchase reminder* | Every 1 day | The system user | Selects the purchase orders eligible for a vendor reminder and posts the reminder message on those whose expected arrival minus their days-before-receipt falls exactly on today. The full algorithm is in [`workflows.md`](workflows.md). |

The job is shipped as non-updatable data with forced creation, so it exists from the moment the
capability is installed and an administrator's changes to its schedule survive upgrades.

No other scheduled job belongs to this domain. The procurement scheduler that may create
requests for quotation belongs to
[`../replenishment-and-procurement/configuration.md`](../replenishment-and-procurement/configuration.md).

## 12. Printable documents

| Document | Entity | Produced name | Bound to the entity's print menu |
|---|---|---|---|
| Purchase Order | Purchase Order | *Request for Quotation - the order reference* while the status is draft or sent; *Purchase Order - the order reference* otherwise | Yes |
| Request for Quotation | Purchase Order | *Request for Quotation - the order reference* | Yes |
| Purchase Agreement | Purchase Agreement | The agreement name | Yes, when the agreements capability is installed |

The download file name of the purchase order document is *Purchase Order-the order reference*.

When the structured electronic order capability is installed, producing either purchase order
document additionally embeds one machine-readable attachment per configured builder inside the
produced file, but only when a single record is being printed.

## 13. Message templates

| Template | Entity | Subject | Attaches | Purpose |
|---|---|---|---|---|
| Purchase: Request For Quotation | Purchase Order | *the company name Order (Ref the order reference or n/a)* | The quotation document | Sent manually to a vendor to request a quotation. |
| Purchase: Purchase Order | Purchase Order | The same subject | The purchase order document | Sent to a vendor with the confirmed order. Carries an *Acknowledge* button when the order has an expected arrival. |
| Purchase: Vendor Reminder | Purchase Order | The same subject | The purchase order document | Sent before the expected arrival, automatically or by hand, asking the vendor to confirm the date. Carries an *Acknowledge* button. Its sender address is the buyer's formatted address, falling back to the acting user's. |

All three address the recipient by the template's own default-recipient rule rather than by a
fixed partner, and all three delete the outgoing mail once it has been sent.

Three further rendered snippets are used as notes rather than emails:

| Snippet | Used when |
|---|---|
| Order-line quantity change note | A line's ordered quantity changes on a confirmed order. |
| Received-quantity change note | A line's received quantity changes on a confirmed order. |
| Exception on purchase order | A line's ordered quantity is decreased on a confirmed order, producing a document exception on the impacted transfers. |
| Exception on purchase, quantity decreased on the sales order | A sold quantity of a re-purchased service is decreased. |
| Exception on sale, purchase cancelled | A purchase order carrying re-purchased service lines is cancelled. |

## 14. Optional capabilities and what each adds

| Capability | Adds |
|---|---|
| Purchasing (the base) | Purchase orders, order lines, agreements-free flows, billing, reminders, the portal, the analysis entity. Depends on the accounting capability. |
| Purchasing with inventory | Receipts, the buy rule and route, the operation type on orders, received quantities from stock moves, the on-time delivery rate, the vendor delay entity, replenishment suggestions, the drop-ship address. Installs itself automatically when both purchasing and inventory valuation are present. |
| Purchase agreements | Blanket orders, purchase templates, alternative requests for quotation, the comparison flow, the company subtotal on lines. |
| Purchase agreements with inventory | An operation type and a warehouse on the agreement, a downstream move on the agreement line, agreement-aware procurement grouping. Installs itself automatically. |
| Purchase agreements with sales | Carries the originating sales line onto an alternative request. Installs itself automatically. |
| Product grid entry | The variant grid on a purchase order and its printing. |
| Structured electronic orders | Exporting a purchase order as a machine-readable order document, embedding it in the printed file, offering it for download from the portal, and importing a received order document. Installs itself automatically when both purchasing and the structured-document capability are present. |
| Purchasing with manufacturing | Kits on purchase orders, component cost shares, the manufacturing-order counters. |
| Purchasing with repairs | The repair-order and purchase-order counters in both directions. |
| Purchasing from sales | Services that generate a request for quotation when sold, and the two-way counters. |
| Purchasing with projects | A project on a purchase order, the project's purchase order counter, and the purchasing contribution to project profitability. |

## 15. Multi-company behaviour summary

| Aspect | Behaviour |
|---|---|
| Order company | Required, defaults to the active company, and governs the numbering series, the approval policy, the lock policy, the currency fallback, the fiscal position, the operation type default, the accounts of the resulting bill and the record-rule visibility. |
| Product company | A product belonging to a company outside the order company's accessible branch tree is refused on a line. |
| Grouping bills | Bills are grouped by company as well as by vendor and currency, so one bill never spans two companies. |
| Branches | When an order's company is a branch of the bill's company, auto-completing the bill from the order adopts the order's company. |
| Agreements | One company per agreement; changing it is only allowed while the agreement is a draft and renumbers it. |
| Numbering series | Shipped without a company, so the counter is shared. An administrator may add per-company series with the same code. |
| Approval threshold | Stored in the company's currency; converted into the order's currency using the **active** company's currency as the source, which matters when an order of a branch is confirmed from a parent company context. |
| Accrual entries | Refused across more than one company and across more than one currency. |
