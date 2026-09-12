# Booths and exhibitors

Renting floor space at an event, and turning the renters into published sponsors and exhibitors. The field-by-field definition of every entity named here is in [entities.md](entities.md); this file carries the behaviour: the catalogue, the two-state availability model, the competition between several candidates for the same booth, the pricing, the sponsor creation and the public exhibitor pages.

## Contents

1. [The booth model](#1-the-booth-model)
2. [Building the booth catalogue](#2-building-the-booth-catalogue)
3. [Booth pricing](#3-booth-pricing)
4. [Availability and the two states](#4-availability-and-the-two-states)
5. [Booking from the back office](#5-booking-from-the-back-office)
6. [Booking through a sales order](#6-booking-through-a-sales-order)
7. [Booking from the public website](#7-booking-from-the-public-website)
8. [Paying for a booth](#8-paying-for-a-booth)
9. [Creating the sponsor of a booth](#9-creating-the-sponsor-of-a-booth)
10. [Sponsors, levels and exhibitor kinds](#10-sponsors-levels-and-exhibitor-kinds)
11. [Sponsor opening hours](#11-sponsor-opening-hours)
12. [The public exhibitor pages](#12-the-public-exhibitor-pages)
13. [Counters on the event](#13-counters-on-the-event)

---

## 1. The booth model

| Entity | Role |
|---|---|
| Event Booth Category | a class of booth: name, description, picture, product, price, and the sponsor options. |
| Event Booth | one physical booth of one event: name, category, renter, state. |
| Event Booth Template | a booth line of an event template, copied onto new events. |
| Event Booth Registration | a **pending reservation**: one candidate renter wanting one booth through one sales order line. |
| Event Sponsor | a sponsor or exhibitor of one event, with its own public page. |
| Event Sponsor Level | a sponsorship level with a ribbon style and a ranking sequence. |

The essential asymmetry of the model: a **booth** is a physical object that can be taken only once, while a **reservation** is a claim that several customers may hold at the same time. The claim becomes the booking when the order that carries it is confirmed, and every competing claim is then destroyed.

---

## 2. Building the booth catalogue

1. An Event Administrator creates the booth categories. A category requires a name and, with the sales bridge, a product whose service tracking is the booth value (`EV-RULE-042`). Three categories are shipped: Standard Booth, Premium Booth and Very Important Person Booth.
2. An Event User creates the booths of an event, either by hand or by copying them from an event template. A booth requires a name, a category and an event.
3. A booth template line requires a name and a category; when exactly one category exists in the whole system it is preselected. Applying the template to an event deletes every booth of that event that is still `available` and creates one booth per template line, copying `name` and `booth_category_id`, plus `product_id` and `price` when the sales bridge is installed.
4. The category picture defaults to the picture of its product while the category has none of its own.

---

## 3. Booth pricing

Without the sales bridge, booths carry no price at all: they are simply allocated. With the sales bridge:

```
category.price       = product sales price + product extra price,
                       whenever the product has a non-zero sales price;
                       otherwise the stored value, which stays editable
category.price_incl  = total including tax of ( price, quantity 1, category currency, product )
category.price_reduce        = (1 − contextual discount of the product) × price
category.price_reduce_taxinc = total including tax of ( price_reduce, quantity 1,
                                category currency, product )
```

The price is deliberately editable on the category, because one product may serve several categories at different prices. A booth template line reads the price and the product straight through from its category and stores the price, so that the value copied onto a new event is the one in force at that moment.

**Price written on a sales order line** carrying pending booths:

```
base = Σ over the pending booths of booth.booth_category.price_reduce
         when the matching pricelist rule may NOT show a discount
     = Σ over the pending booths of booth.price
         when the matching pricelist rule MAY show a discount
unit price = convert(base, from the currency of the company of the event, to the line currency)
```

A booth line therefore always carries quantity one and a price equal to the sum of the booths it reserves.

**Worked example.** A customer reserves two Premium booths at `500.00` each and one Standard booth at `100.00`, all in the same currency, with no discount. The line price is `500.00 + 500.00 + 100.00 = 1,100.00` for a quantity of one. With a 10 percent pricelist discount that the rule may not display, the line price is `450.00 + 450.00 + 90.00 = 990.00`.

---

## 4. Availability and the two states

A booth has exactly two states:

| State | Meaning |
|---|---|
| `available` | free, offered on the public page and in the configurator |
| `unavailable` | taken; it disappears from every offer |

`is_available` is the derived boolean `state = "available"`. Searching on it is rewritten into a search on the state: `is_available in true` becomes `state = available` and `is_available not in true` becomes `state = unavailable`; any other operator is refused.

There is **no** operation that returns a booth to `available`. Freeing a booth is a deliberate manual edit of the state, which is consistent with the fact that a booking is a commercial commitment.

Grouping booths by state always shows both columns, even when one of them is empty.

---

## 5. Booking from the back office

1. The user fills the renter block of the booth: the contact, and optionally the renter name, electronic mail address and telephone. Each of the three is filled from the contact **only while empty**.
2. The user presses Confirm, which writes `state = unavailable` together with the collected values in a **single** write.
3. The post-confirmation rule runs on exactly the booths that were `available` before the write, which is how a second confirmation of an already-booked booth is prevented from posting a second message:
   - a sponsor is created or reused when the category asks for one (section 9);
   - a message built from the booking layout is posted **on the event**, under the subtype "Booth Booked", naming the booth.
4. A booth created directly in state `unavailable`, for instance by an import, posts the booking message immediately on creation.

---

## 6. Booking through a sales order

### Putting booths on a line

1. The salesperson adds a line with a booth product; the Booth Configurator opens.
2. The configurator asks for the event, then the booth category, then the booths. Only the categories that still have at least one free booth are offered; changing the event clears the category, and changing the category clears the booths. Selecting nothing is refused with *"You have to select at least one booth."*
3. Writing the chosen booths on the line creates one Event Booth Registration per booth, each carrying the line, the booth and the customer of the order. De-selecting a booth deletes its reservation.
4. The line description becomes `<event display name> : ` followed by one line `- <booth name>` per booth. The product template description never overwrites it.
5. All reservations of one line must belong to a single event (`EV-RULE-071`).
6. Changing the product of the line clears the event when the product no longer belongs to the pending booths; changing the event clears the pending booths.

### Confirming the order

1. A booth line with no pending booth refuses the confirmation: *"Please make sure all your event-booth related lines are configured before confirming this order:"* followed by one line per offending line.
2. For each booth line that has pending booths **and no confirmed booth yet**:
   - every requested booth that is no longer available refuses the confirmation with *"The following booths are unavailable, please remove them to continue : "* followed by one indented line per booth;
   - otherwise every reservation of the line is confirmed.
3. Confirming a reservation writes on its booth, through the booth confirm operation, the values `sale_order_line_id`, `partner_id`, `contact_name`, `contact_email` and `contact_phone`; with the sponsor bridge installed, the six sponsor fields are appended. The booth becomes unavailable and the post-confirmation rule runs.
4. Then every **other** pending reservation on the same booths loses:
   - a message is posted on each losing order, addressed to the salesperson of that order, reading *"Your order has been cancelled because the following booths have been reserved"* followed by a bulleted list of the booth display names;
   - each losing order is cancelled;
   - the losing reservations are deleted.

**Worked example.** Booth `B12` is claimed by order A (salesperson Alice) and by order B (salesperson Bob). Alice confirms order A first: `B12` becomes unavailable and carries the contact of order A. Bob's reservation is deleted, order B is cancelled, and Bob receives the message on order B naming `B12`. If Bob had also reserved `B13` on the same line, `B13` is released as well, because the whole line is cancelled with the order.

### Guards

- A booth linked to a sales order cannot be deleted (`EV-RULE-072`).
- One reservation per pair of line and booth (`EV-RULE-073`).

---

## 7. Booking from the public website

The public flow has two shapes depending on whether online booth sales are installed.

### Common part

1. `/event/<event slug>/booth` shows the categories that still have a free booth, with the first of them preselected, the booths of the event and the exhibition map of the event. A reader who cannot read the event is forbidden.
2. The visitor selects a category and one or several booths, submits, and is redirected to the contact form carrying the chosen booths and the category in the address. A contact form opened without those two parameters gives "not found".
3. The contact form pre-fills the name, the electronic mail address and the telephone from the signed-in user, or from the identified visitor.
4. On submission, three validations run in order and answer a code rather than a sentence (`EV-RULE-076`): `boothError`, then `boothCategoryError`, then `existingPartnerError`.
5. The contact is resolved: for an anonymous visitor a contact is found or created from the normalised electronic mail address, carrying the posted name and telephone; for a signed-in visitor the contact of the user is used. The collected values are the contact, the contact name, the contact electronic mail address and the contact telephone, each falling back to the value of the contact when not posted.

### Without online booth sales

The booths are confirmed immediately with the collected values, which marks them unavailable, creates the sponsor when the category asks for one, and posts the booking message on the event. The answer carries `success`, the event name and the contact block.

### With online booth sales

Nothing is confirmed at this point, because the confirmation happens when the order is confirmed:

1. A cart is created or reused; an anonymous cart receives the resolved contact as its customer.
2. One cart line is added for the product of the category, with quantity one, the chosen booths as pending booths and the contact values.
3. When the cart total is non-zero the answer redirects the visitor to the cart, and the booths are confirmed only when the order is confirmed after payment.
4. When the cart total is zero the order is confirmed straight away, which confirms the booths, the cart is reset and the success answer is returned.

The contact form additionally tells the page whether a payment step will follow, which is true when the current cart already has an amount or the chosen category has a price.

### Cart rules for booths

- A cart line is matched to an existing line only when that line already contains one of the requested booths, which prevents two lines competing for the same booth.
- The quantity of a booth line can never exceed one (`EV-RULE-077`).
- Updating a booth line deletes its reservations and creates new ones from the new selection, with the same contact values.

### Helper endpoints

- one returns the identifiers of the requested booths that are no longer available, so that the page can grey them out live;
- one returns the free booths of a category as pairs of identifier and name, so that changing the category refreshes the list without reloading the page.

---

## 8. Paying for a booth

The booth itself creates no accounting document; see [accounting-effects.md](accounting-effects.md). The only accounting-driven write is the paid stamp:

1. An invoice built from the sales order reaches the state "paid".
2. The sales order lines behind its invoice lines are collected.
3. Among them, the lines whose product has the booth service tracking and which already have confirmed booths are kept.
4. `is_paid = true` is written on those booths, with elevated rights.

Reversing the payment does not clear the flag.

---

## 9. Creating the sponsor of a booth

A booth category may ask for a sponsor to be created when one of its booths is booked. Three fields drive it:

| Field | Meaning |
|---|---|
| `use_sponsor` | create a sponsor when a booth of this category is booked |
| `sponsor_type_id` | the sponsorship level given to that sponsor |
| `exhibitor_type` | the sponsor kind given to that sponsor: `sponsor`, `exhibitor` or `online` |

When the sponsor switch is turned on in the form and no level is set, the level with the **highest** sequence is proposed, that is the lowest level; when no kind is set, the first value of the list is proposed.

**Procedure**, run inside the post-confirmation rule of the booth, only when the category asks for a sponsor **and** the booth has a renter:

1. Search, with elevated rights, for an existing sponsor matching **all four** of: the same contact, the same sponsorship level, the same sponsor kind and the same event. The first match is reused, which prevents one renter from becoming several sponsors of the same event at the same level.
2. When no match exists, create a sponsor with:
   - the event, the sponsorship level and the sponsor kind of the category;
   - the contact of the booth;
   - every value passed to the confirmation whose name starts with `sponsor_`, with that prefix removed. The public booking form therefore fills the sponsor name, electronic mail address, telephone, slogan, description and logo in one pass;
   - the contact name as the sponsor name when no name was passed, which is the case for a confirmation made from the back office.
3. The created or reused sponsor is written on the booth, which makes the sponsor block of the booth readable through the read-through fields.

**Worked example.** The shipped Premium Booth category creates a Silver exhibitor; the Very Important Person Booth category creates a Gold online exhibitor. A renter who books one Premium booth and then a second Premium booth of the same event ends with **one** Silver sponsor linked to both booths. The same renter booking a Very Important Person booth additionally becomes a Gold online exhibitor, because the level and the kind differ.

---

## 10. Sponsors, levels and exhibitor kinds

### Levels

A level carries a name, a sequence and a ribbon style among `no_ribbon`, `Gold`, `Silver` and `Bronze`. **A lower sequence is a higher level**: the shipped data gives Gold sequence 1, Silver 2 and Bronze 3. A new level takes the highest existing sequence plus one, therefore new levels are created at the bottom. The default level of a new sponsor is the one with the **highest** sequence.

### Kinds

| Kind | Meaning | Appears in the exhibitor list |
|---|---|---|
| `sponsor` (Footer Logo Only) | the logo is shown in the page footer of the event | no |
| `exhibitor` (Exhibitor) | a physical exhibitor with a public page | yes |
| `online` (Online Exhibitor) | an exhibitor reachable online, with opening hours | yes |

### Synchronisation with the contact

`name`, `email`, `phone` and `image_512` are filled from the contact **only while empty**, and writing them never writes back on the contact, because the value may be specific to this event. Two fields behave differently:

- `url` is refreshed from the contact website whenever the contact has one, or when the field is empty;
- `website_description` is taken from the public description of the contact while the sponsor description is empty.

The logo address used on the public pages is derived as:

```
if the sponsor has a logo:            the 256-pixel rendering of the sponsor logo
else if the contact has an image:     the 256-pixel rendering of the contact image
else:                                 the shipped default sponsor picture
```

### Other fields

`show_on_ticket` decides whether the sponsor logo is printed on the event tickets; it defaults to true. `sequence` orders the sponsors inside their level. `country_id` and `country_flag_url` are read through the contact and are used by the country filter and the flag badge of the public list.

---

## 11. Sponsor opening hours

A sponsor holding an online booth publishes daily opening hours, `hour_from` and `hour_to`, expressed as fractional hours **in the display time zone of the event**, defaulting to `8.0` and `18.0`. The derived flag `is_in_opening_hours` answers "is this virtual booth open right now"; the algorithm and a worked example are in [calculations.md](calculations.md#10-sponsor-opening-hours). The three rules that matter operationally are:

1. an event that is not ongoing closes every sponsor, whatever the hours;
2. a sponsor with no hours at all is open for the whole event;
3. a closing hour of zero means midnight of the **following** day, and the opening window is always clipped to the event start and the event end.

---

## 12. The public exhibitor pages

### The exhibitor list

Reachable at `/event/<event slug>/exhibitors`. The base condition is:

```
event = this event AND exhibitor_type IN ("exhibitor", "online")
```

plus `is_published = true` for anyone below the Registration Desk group.

Three filters apply on top: a free text over the sponsor name and the sponsor description; a country filter; a sponsorship-level filter. The available countries and levels shown in the filter panel are computed from the **unfiltered** base condition, so that the panel does not collapse as the reader narrows the search.

The result is ordered by level sequence, then by sponsor sequence, and grouped per level. Inside a level the sponsors are **shuffled at random** at each request, so that no exhibitor gets a permanent advantage. For a Registration Desk reader the shuffle is done twice: the published sponsors are shuffled and placed first, then the unpublished ones are shuffled and placed after them.

### The sponsor page

Reachable at `/event/<event slug>/exhibitor/<sponsor slug>`; the event must have its exhibitor menu on and the sponsor must belong to the event. A reader who cannot read the sponsor is forbidden.

The page shows the logo, the name, the slogan, the description, the website, the electronic mail address, the telephone, the country flag, the opening hours and the "open now" marker. The sidebar lists at most 30 other exhibitors of the same event, ordered by the descending tuple:

1. published;
2. open right now;
3. same country as the sponsor being shown;
4. the negated level sequence, that is the higher level first;
5. a pseudo-random integer between 0 and 20.

A wide layout option is available, and an Event User additionally gets the editing affordances.

### The sponsor card

A small structured answer feeds the dialog shown when a visitor clicks a sponsor before the event starts or outside the opening hours. It carries: the name, the slogan, the website, the electronic mail address, the telephone, the description, the logo address, the two hours and their formatted texts, the "open now" flag, the display time zone of the event, the country flag address, the country name and identifier, the sponsorship level name and identifier, the event name, and the event flags "ongoing", "done", "starts today", "minutes before start" and the event start date.

---

## 13. Counters on the event

| Field | Rule |
|---|---|
| `event_booth_count` | the number of booths of the event, in any state |
| `event_booth_count_available` | the number of booths in state `available` |
| `event_booth_category_ids` | the distinct categories of the booths of the event |
| `event_booth_category_available_ids` | the distinct categories of the booths that are still available; this is the list offered on the public page and in the configurator |
| `event_booth_ids` (on a sales order) | the booths finally won by the order |
| `event_booth_count` (on a sales order) | how many |

The two counters are read in a single grouped query over the booths of the events being displayed; while an event is still being edited in a form and has no identifier yet, they are counted from the lines held in the form instead.

**Worked example.** An event has 10 booths: 4 Standard (2 booked), 4 Premium (all free) and 2 Very Important Person (1 booked). Then `event_booth_count = 10`, `event_booth_count_available = 7`, the category list holds the three categories, and the available category list holds Standard, Premium and Very Important Person, because each of them still has at least one free booth. Booking the last Standard booths would remove Standard from the second list and therefore from the public page.
