# Accounting effects

The Replenishment and Procurement domain creates no journal entry, no journal item, no analytic item and no valuation layer of its own. Every posting that eventually results from a replenishment is created by another domain, at a later moment, from a document that this domain produced. This file states that boundary precisely, names the documents this domain hands over, specifies the valuation characteristics that this domain stamps on those documents (because the receiving domains read them), and specifies the two flows where this domain changes how another domain values a movement: drop shipping, and drop shipping to a subcontractor.

---

## 1. The boundary

| Event in this domain | Record written here | Accounting consequence, and where it is specified |
|---|---|---|
| A pull rule creates a stock move | Stock Move, state `confirmed` or `waiting` | None. A move that is not completed is never valued and never posted. When the move is later completed, the valuation and the journal entry are produced by `../inventory-valuation-and-costing/accounting-effects.md`. |
| A buy rule creates or extends a draft purchase order | Purchase Order in state `draft`, Purchase Order Line | None. A draft purchase order has no accounting effect. Confirmation, receipt and billing are specified in `../purchasing/accounting-effects.md` and `../accounts-payable/accounting-effects.md`. |
| A manufacture rule creates a manufacturing order | Manufacturing Order in state `draft` or `confirmed` | None. Component consumption and finished-product production are valued by `../manufacturing/accounting-effects.md`. |
| A push rule creates the next move of a chain | Stock Move | None at creation. The chain's internal moves are transfers between internal locations and, under the standard valuation rules, produce no journal entry; only the first move into stock and the last move out of stock are valued. |
| A reordering rule orders | see the three rows above | None of its own. |
| The scheduler runs | Stock Moves, Purchase Orders, Manufacturing Orders, warning activities | None of its own. |
| A route, a rule or a reordering rule is created, changed, archived or deleted | configuration records | None. Configuration never posts. |
| A drop shipment is validated | Stock Move completed, with the drop-shipment characteristics of section 3 | The move is valued as both an entry into and an exit out of the company's books in the same step. The entries are produced by `../inventory-valuation-and-costing/` using the inputs specified in section 3 below. |

**Industry-standard completion.** The separation above follows the standard rule of perpetual inventory accounting: value is recognized when control of the goods changes, not when the intention to buy or make them is recorded. A replacement must not post anything when a procurement request is run; posting at that moment would recognize inventory that does not exist and would double-count it at receipt.

---

## 2. What this domain stamps on the documents it creates, and why accounting reads it

| Field written here | Entity | Why the accounting domains read it |
|---|---|---|
| `rule` | Stock Move | Identifies the route step, which tells the valuation domain whether the move is an internal step of a chain (not valued) or the first or last step (valued). |
| `procure_method` | Stock Move | A `make_to_order` move is bound to an origin move; the valuation domain uses that link to trace the cost from the receipt to the delivery in a chained flow. |
| `move_destinations` and `move_origins` | Stock Move | The chain links that let the valuation domain walk from a delivery back to the receipt that supplied it, and from a receipt forward to the vendor bill that prices it. |
| `location_final` | Stock Move | Distinguishes an intermediate destination from the ultimate one; a move whose destination is intermediate is an internal step. |
| `purchase_line` | Stock Move | The link from a receipt move to the purchase order line, which carries the purchase price and the vendor bill lines. It is the single most important input of the purchase valuation: the value of a received unit comes from that line, and, once billed, from the bill lines of that line. |
| `created_purchase_lines` | Stock Move | The link from a make-to-order move to the purchase order line that was created for it before the purchase order was confirmed. It lets cancellation and quantity changes reach the right documents. |
| `orderpoint` | Stock Move, Purchase Order Line | Traceability only; no accounting effect. |
| `propagate_cancel` | Stock Move, Purchase Order Line | Governs whether cancelling one document cancels the next, which in turn governs whether the corresponding value is ever recognized. |
| `references` | Purchase Order, Manufacturing Order, Stock Move | Traceability between the originating document and the created documents; used by the reporting of `../purchasing/` and `../sales/`, not by the ledger. |
| `operation_type` | Purchase Order | Determines the destination location of the receipt, which determines whether the flow is an ordinary receipt or a drop shipment (section 3). |
| `destination_address` | Purchase Order | For a drop shipment, the customer; the delivery address the vendor ships to. It does not change the accounts used. |
| `currency` | Purchase Order | The currency in which the purchase is valued until the bill is posted (see `RP-RULE-299` and `RP-RULE-300`). |

---

## 3. Drop shipping

A drop shipment is a single stock move from the vendor location straight to the customer location. The goods never enter a location of the company, yet ownership passes through the company: the company buys from the vendor and sells to the customer in the same movement.

### 3.1 How the flow is recognized

A stock move is a drop shipment when, at the moment it is completed:

```
is_dropship = state = "done"
              AND (   source location usage = "supplier"
                   OR (source location usage = "transit" AND the source location has no company) )
              AND (   destination location usage = "customer"
                   OR (destination location usage = "transit" AND the destination location has no company) )
```

The reverse movement, a customer returning drop-shipped goods to the vendor, is recognized as a drop-shipment return and is treated as the mirror image.

When the Dropship and Subcontracting Management capability package is installed, the test is widened: a move is also a drop shipment when the partner of the move has a subcontracting location and either the move goes from that subcontracting location to a customer location, or the move goes from a vendor location to that subcontracting location. A move is also a drop-shipment return when it goes from a customer location into that subcontracting location.

### 3.2 Valuation consequences

| Characteristic | Value | Consequence |
|---|---|---|
| Counts as an entry into the company's books | yes | The purchase is recognized: the cost of the goods is taken from the purchase order line, or, once a vendor bill is posted for that line, from the bill lines. |
| Counts as an exit out of the company's books | yes | The sale is recognized in the same step: under a cost-of-sales valuation policy, the expense of the goods sold is recognized at the same moment as the purchase. |
| Effect on the product's average cost | none | A drop shipment does not change the average cost of the product, because no unit ever enters stock. The valuation domain excludes drop shipments from the running average. |
| Effect on the on-hand quantity | none | The quantity at every internal location is unchanged. |
| Effect on the stock valuation account balance | net zero for the drop-shipped quantity | The entry and the exit offset each other for that quantity; only the difference between the purchase price and the sale-side cost, when the two differ, remains. |

The exact debit and credit lines, the account selection precedence (product, product category, fiscal position, journal, company default) and the reconciliation are owned by `../inventory-valuation-and-costing/accounting-effects.md` and `../accounts-payable/accounting-effects.md`. This domain contributes only the recognition rule above and the two inputs of section 3.3.

### 3.3 Where the value of a drop-shipped move comes from

1. When the purchase order line of the move has at least one posted vendor bill line, the value is taken from those bill lines: for every posted bill line of the line whose date is not later than the valuation date, an invoice line adds its subtotal converted into the company currency at the bill's own rate, and a credit-note line subtracts it; the quantities are accumulated the same way, converted into the product's reference unit. The accumulated value is used only when the accumulated quantity exceeds the quantity already valued by the earlier moves of the same purchase order line (earlier meaning an earlier scheduled date, or the same date and a lower surrogate key); movements into the company and drop shipments add to that already-valued quantity and movements out of the company subtract from it.
2. Otherwise the value is taken from the purchase order line's own unit price, converted at the rate of the move's date, multiplied by the quantity actually moved, capped at the move's own quantity. The justification text shown to the user is `<formatted value> for <quantity> <unit> from <purchase order display name> (not billed)`.
3. **Dropship to a subcontractor.** When the move is a subcontracting receipt, is a drop shipment, and has a purchase order line, the value is taken instead from the finished-product move of the subcontracting production order that fed it. This is what makes a product that is both subcontracted and drop-shipped carry the production cost (components plus subcontracting service) rather than only the purchase price of the service.

### 3.4 Reconciliation and reversal

- A drop shipment is reversed by returning it. A return of a drop shipment moves from the customer location back to the vendor location and is recognized as a drop-shipment return; it reverses both the entry and the exit.
- A return of a drop shipment into an internal location of the company, rather than to the vendor, is **not** a drop-shipment return: it is an ordinary entry into stock and is valued as such, which increases the on-hand quantity and the stock valuation.
- Cancelling a drop shipping purchase order before the transfer is validated cancels the transfer and leaves no accounting trace at all (see `RP-RULE-245` and `RP-RULE-246`).

---

## 4. Goods received and not yet invoiced

This domain creates the receipts that make a purchase "received but not invoiced". The measurement itself belongs to the valuation report of `../inventory-valuation-and-costing/`, and is computed purchase order by purchase order:

```
for every purchase order line whose quantity still to invoice is not zero
    (optionally restricted to a product category, and to lines whose order approval date is not later than a given date):
        invoiced_value      = sum of the amounts in the order currency of its posted vendor bill lines
        not_invoiced_value  = the line's subtotal excluding tax − invoiced_value
group by purchase order and sum
```

The resulting total is the liability for goods that are in the company's books but for which no vendor bill has been posted. It is a reporting figure; it posts nothing.

---

## 5. Analytic accounting

This domain sets no analytic distribution. The analytic distribution of a purchase order line created by a `buy` rule comes from the shared purchase-line preparation of `../purchasing/`, which resolves it from the analytic distribution model rules. The analytic items generated when a transfer is validated are owned by `../analytic-accounting/`.

---

## 6. Currency

| Situation | Rule |
|---|---|
| The currency of a purchase order created by a `buy` rule | The currency of the chosen Vendor Price; when that is empty, the vendor's purchase currency for the order company; when that is empty too, the currency of the order company. |
| A Vendor Price in another currency than the order | Converted into the order currency at the rate of today, both when a line is created and when a line is updated. |
| Two needs that resolve to different currencies | Never merged into the same purchase order: the currency is part of the grouping key. |
| Revaluation | This domain never revalues. A purchase billed in a foreign currency is revalued by `../accounts-payable/` at the bill rate. |

---

## 7. What a replacement must not do

1. Do not post anything when a procurement request is run, when a reordering rule orders, or when the scheduler runs.
2. Do not value an internal step of a multi-step route as an entry or an exit. Only the first move that brings goods into the company's locations and the last move that takes them out are valued; drop shipments are both at once.
3. Do not let a drop shipment change the average cost of the product.
4. Do not recognize the cost of a purchase twice when a receipt is both linked to a purchase order line and fed by a chained move: the purchase order line is the single source of the purchase value.
5. Do not treat a return of a drop shipment into an internal location as a drop-shipment return; it is a genuine entry into stock.
