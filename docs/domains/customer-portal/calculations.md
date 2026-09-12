# Calculations of the Customer Portal

Every formula and algorithm of the Customer Portal, with its inputs, its output, its precision, its order of operations and at least one worked example with real numbers. Algorithms are written as numbered steps in language-neutral notation.

---

## 1. The page navigator

**Purpose**: turn a total number of rows, a page size and a requested page number into the navigator shown under a document list.

**Inputs**: `base_address` (text), `total` (integer, at least zero), `requested_page` (any value), `step` (integer page size, default thirty for the generic helper, eighty for the portal list pages, one hundred for the timesheet page), `address_parameters` (a set of name-value pairs, possibly empty).

**Outputs**: `page_count`, `offset`, and the addresses of the current, first, previous, next and last pages, plus the ordered `pages` list used to render the entries.

**Steps**

```
1.  page_count = ceiling(total ÷ step)
2.  candidate = requested_page when its textual form is a run of digits, otherwise 1
3.  page      = max(1, min(candidate, page_count))
4.  previous  = max(1, page − 1)
5.  next      = min(page_count, page + 1)
6.  offset    = (page − 1) × step
7.  page_list is chosen by the first matching case:
      page_count ≤ 5          →  [1, 2, …, page_count]                         (every page)
      page ≤ 3                →  [1, 2, 3, 4, ELLIPSIS, page_count]
      page ≥ page_count − 2   →  [1, ELLIPSIS, page_count−3, page_count−2, page_count−1, page_count]
      otherwise               →  [1, ELLIPSIS, page−1, page, page+1, ELLIPSIS, page_count]
8.  each entry of page_list becomes { number, address, is_current }, where the address is
    empty for an ELLIPSIS entry and address_of(number) otherwise, and is_current is true
    for the entry whose number equals page
```

`address_of(n)`:

```
address_of(n) = base_address                         when n = 1
              = base_address + "/page/" + n          when n > 1
and, when address_parameters is not empty,
              = that value + "?" + encoded(address_parameters)
```

The encoding drops every parameter whose value is empty, so a list page that passes an unset range start and range end produces an address with no query string at all.

**Precision**: `page_count` is an integer obtained by rounding the quotient up; `offset` is an exact integer product.

**Notes**
- When `total` is zero, `page_count` is zero, the clamp in step 3 yields `page = 1` and `offset = 0`; the navigator is not rendered because it is drawn only when `page_count` is greater than one.
- A requested page above the last one is clamped down to the last one rather than refused.
- A requested page that is not a run of digits (for example a word, or a negative sign) is treated as page one.
- The helper accepts a "scope" input that is not used by the algorithm above; it exists for callers that pass it and has no effect on the result.

**Worked examples** (page size thirty in every case, matching the shipped verification)

| Total rows | Requested page | `page_count` | `page` | `offset` | `pages` numbers |
|---|---|---|---|---|---|
| 20 | 1 | ceiling(20 ÷ 30) = 1 | 1 | 0 | 1 |
| 50 | 1 | ceiling(50 ÷ 30) = 2 | 1 | 0 | 1, 2 |
| 150 | 3 | ceiling(150 ÷ 30) = 5 | 3 | 60 | 1, 2, 3, 4, 5 |
| 300 | 5 | ceiling(300 ÷ 30) = 10 | 5 | 120 | 1, ellipsis, 4, 5, 6, ellipsis, 10 |
| 300 | 1 | 10 | 1 | 0 | 1, 2, 3, 4, ellipsis, 10 |
| 300 | 10 | 10 | 10 | 270 | 1, ellipsis, 7, 8, 9, 10 |
| 300 | 27 | 10 | 10 | 270 | 1, ellipsis, 7, 8, 9, 10 |
| 0 | 1 | 0 | 1 | 0 | navigator not rendered |

**Worked example with the portal page size**: an invoice list with 205 rows, page size eighty, requested page three.

```
page_count = ceiling(205 ÷ 80) = ceiling(2.5625) = 3
page       = max(1, min(3, 3)) = 3
previous   = 2,  next = 3
offset     = (3 − 1) × 80 = 160
page_count ≤ 5, so pages = [1, 2, 3]
```

The third page therefore reads rows 161 to 205, that is, forty-five rows.

---

## 2. The record navigator (previous and next on a record page)

**Purpose**: give a record page a link to the record before and after it, using the list the reader came from.

**Inputs**: `history` (the ordered list of identifiers stored in the session by the list page, at most one hundred entries), `current` (the record being displayed).

**Steps**

```
1. If current.identifier is not in history, or the record exposes neither a portal web
   address nor a website address, produce no links at all.
2. attribute = the portal web address when the record exposes one, otherwise the website address.
3. index = the position of current.identifier in history (zero based).
4. previous_record = history[index − 1] when index ≠ 0, otherwise none.
5. next_record     = history[index + 1] when index < length(history) − 1, otherwise none.
6. For each of the two, when the record exists and its chosen attribute is non-empty:
       when the attribute is the portal web address:
           link = attribute + "?access_token=" + _portal_ensure_token(record)
       otherwise:
           link = attribute
   When the record exists but its attribute is empty, the link is the record itself,
   which renders as a disabled control. When the record does not exist, there is no link.
```

**Worked example**: the session history of the quotation list is `[91, 87, 84, 80]`; the reader opens record 87.

```
index = 1
previous_record = 91  →  "/my/orders/91?access_token=<token of 91>"
next_record     = 84  →  "/my/orders/84?access_token=<token of 84>"
```

On record 91 the previous control is disabled; on record 80 the next control is disabled.

Note the side effect: computing the two links calls the token creation on both neighbours, so simply opening a record page may write tokens onto two other records.

---

## 3. Counter batching on the portal home

**Purpose**: fetch the card counters in as few round trips as possible without sending one enormous request.

**Inputs**: `needed`, the ordered list of counter names found on the rendered page.

**Steps**

```
1. n = length(needed)
2. number_of_calls = min( ceiling(n ÷ 5), 3 )
3. per_call        = ceiling(n ÷ number_of_calls)          (not evaluated when n = 0)
4. calls           = min(number_of_calls, n)
5. call i, for i from 0 to calls − 1, carries needed[i × per_call … (i + 1) × per_call − 1]
```

**Precision**: every quantity is an integer obtained by rounding up.

**Worked examples**

| Counters on the page | `number_of_calls` | `per_call` | Slices actually sent |
|---|---|---|---|
| 0 | min(0, 3) = 0 | not evaluated | none; the page issues no call |
| 1 | min(1, 3) = 1 | 1 | 1 |
| 5 | min(1, 3) = 1 | 5 | 5 |
| 7 | min(2, 3) = 2 | ceiling(7 ÷ 2) = 4 | 4 then 3 |
| 12 | min(3, 3) = 3 | 4 | 4, 4, 4 |
| 20 | min(4, 3) = 3 | ceiling(20 ÷ 3) = 7 | 7, 7, 6 |

**The server side of one call**

```
1. cache   = a copy of the session's counter cache
2. result  = the counter preparation run for the requested names only
3. cache[name] = (value ≠ 0), for each name in result whose name ends in "_count"
4. when cache ≠ the stored cache, store cache in the session
5. the answer is result
```

**Worked example**: a page carries `quotation_count`, `order_count`, `invoice_count`, `bill_count`, `project_count`, `task_count` and `timesheet_count`, that is seven names. Two calls are issued, the first with the first four names and the second with the last three. The server answers `{quotation_count: 0, order_count: 3, invoice_count: 12, bill_count: 0}` and `{project_count: 2, task_count: 0, timesheet_count: 45}`. The session cache becomes `{quotation_count: false, order_count: true, invoice_count: true, bill_count: false, project_count: true, task_count: false, timesheet_count: true}`. The browser reveals the order, invoice, project and timesheet cards and leaves the other three hidden.

---

## 4. Building the security token

**Purpose**: produce a value that a holder can present instead of a permission.

**Algorithm**: generate a version-4 random universally unique identifier, that is, one hundred and twenty-eight bits of which one hundred and twenty-two come from a cryptographically adequate random source, four are the version marker with the value four, and two are the variant marker with the value binary one-zero. Render it as thirty-two lowercase hexadecimal digits grouped as eight, four, four, four and twelve, separated by hyphens, for a total length of thirty-six characters.

**Example of the shape**: `3f2a7c18-5be4-4d0a-9f31-6c0b2e77a1d5` (the fifteenth character is the version marker `4`; the twentieth is one of `8`, `9`, `a` or `b`).

**Collision probability**: with one hundred and twenty-two random bits, the chance that any two of one hundred million issued tokens collide is below one in ten to the twentieth power. This is why the shipped behavior stores the value without a uniqueness constraint; an **industry-standard completion** is nevertheless to add a unique index per adopting model, because a collision would hand a reader the wrong document.

---

## 5. Signing a recipient identity

**Purpose**: prove, without a session, that the person following a link is a specific Contact.

**Inputs**: the database name `d`, the document's security token `t` (read from the field named by the model's external-posting token field name), the recipient Contact identifier `p`, and the platform's database secret `k`.

**Steps**

```
1. message = the canonical textual representation of the ordered triple (d, t, p)
2. digest  = keyed_hash_message_authentication_code(
                 key       = k,
                 message   = message,
                 hash      = the secure hash algorithm producing a 256-bit digest )
3. signature = digest rendered as sixty-four lowercase hexadecimal characters
```

**Verification** compares the supplied value with the recomputed one in constant time; when that fails, the same computation is repeated for the record's logical parent (when the model declares one) and compared again.

**Worked example of the shape**: for the database `acme_production`, the token `3f2a7c18-5be4-4d0a-9f31-6c0b2e77a1d5` and the Contact identifier `412`, the message is the three-element tuple written as text, and the signature is a sixty-four character hexadecimal string such as `9d41...c07e`. A replacement must render the tuple exactly as the reference does, because the signature is not portable across two different renderings of the same three values: the tuple representation is part of the specification.

**Invalidation matrix**

| Change | Existing signatures still valid |
|---|---|
| The document's token is cleared or replaced | No |
| The document is duplicated | The copy has no token, so no signature exists for it |
| The Contact is renamed or its address changes | Yes |
| The Contact is archived | Yes (the signature does not check the Contact's state) |
| The database is copied under another name | No |
| The database secret is rotated | No |

---

## 6. Assembling the portal addresses

### 6.1 The record page address with the token

```
portal_web_address(suffix, report_type, download, query_string, anchor) =
      access_url
    + suffix                                        when given, else ""
    + "?access_token=" + _portal_ensure_token()
    + "&report_type=" + report_type                 when report_type is given
    + "&download=true"                              when download is true
    + query_string                                  when given (the caller supplies the leading "&")
    + "#" + anchor                                  when given
```

**Worked example**: a sales order with identifier 42 and token `3f2a…a1d5`, asked for a downloadable portable document (the report kind travels on the wire as the three-letter value `pdf`, which abbreviates portable document):

```
/my/orders/42?access_token=3f2a7c18-5be4-4d0a-9f31-6c0b2e77a1d5&report_type=pdf&download=true
```

**Worked example** of the signature endpoint of the same order:

```
/my/orders/42/accept?access_token=3f2a7c18-5be4-4d0a-9f31-6c0b2e77a1d5
```

**Worked example** of the post-signature landing address with a flash message and a payment invitation:

```
/my/orders/42?access_token=3f2a7c18-5be4-4d0a-9f31-6c0b2e77a1d5&message=sign_ok&allow_payment=yes
```

### 6.2 The share address

```
share_address(redirect, signup_partner, recipient_contact, include_token) =
      ( "/mail/view"  when redirect is true, else access_url )
    + "?" + encoded(parameters)

parameters =   { model, res_id }                             when redirect is true
             ∪ { access_token }                              when include_token and the model has the field
             ∪ { pid, hash }                                 when a recipient contact is given
             ∪ { auth_signup_token } or { auth_login }       when signup_partner and the record has a customer
```

**Worked example**: the share dialog of sales order 42, sent to Contact 412 who already has an account:

```
https://acme.example/mail/view?access_token=3f2a7c18-5be4-4d0a-9f31-6c0b2e77a1d5&hash=9d41…c07e&model=<technical model name>&pid=412&res_id=42
```

**Worked example**: the same document sent to Contact 987 who has no account, with free sign-up enabled:

```
https://acme.example/web/login?database=acme_production&token=<sign-up token>&redirect=/mail/view%3Fmodel%3D…%26res_id%3D42
```

---

## 7. Detecting an address that is already used as a login

**Purpose**: decide the `email_state` of every invitation line in one pass.

**Inputs**: the set of lines `L`, each with an address `a(l)`, a linked user identifier `u(l)` (possibly none) and a Contact `p(l)`.

**Steps**

```
1. N  = { l ∈ L : normalize(a(l)) ≠ empty }          the lines with a usable address
2. every line in L \ N receives the state "ko"
3. F  = _get_similar_users_domain(N)                    the search condition
4. P  = _get_similar_users_fields()                 the fields to read
5. C  = users, archived ones included, matching F, each read as the fields P
6. for each l ∈ N:
       state(l) = "exist"  when ∃ c ∈ C : _is_portal_similar_than_user(c, l)
                = "ok"     otherwise
```

The three named operations are extension points. Their base definitions are:

```
_get_similar_users_domain(N)            = [ login IN { normalize(a(l)) : l ∈ N } ]
_get_similar_users_fields()         = [ id, login ]
_is_portal_similar_than_user(c,l) = c.login = normalize(a(l)) AND c.id ≠ u(l)
```

**Normalization** of an address: trim it; when it has the display form `Name <local@domain>`, keep the part between the angle brackets; lowercase the domain part; return nothing when the result is not a single syntactically valid address.

**Worked example**: an invitation session on a company with three people.

| Line | Typed address | Normalized | Linked user | Users with that login | State |
|---|---|---|---|---|---|
| Alice | `Alice <ALICE@Acme.COM>` | `alice@acme.com` | none | none | `ok` |
| Bob | `bob@acme.com` | `bob@acme.com` | user 55 | user 55 | `ok` (the only match is the line's own user) |
| Carol | `carol at acme dot com` | nothing | none | not searched | `ko` |
| Dave | `dave@acme.com` | `dave@acme.com` | none | user 71, an employee | `exist` |

Dave's line therefore refuses the grant with `The contact "Dave" has the same email as an existing user`, and Carol's line refuses with `The contact "Carol" does not have a valid email.`

### 7.1 The multi-website narrowing of the three extension points

When the Website and Storefront capability package is installed, a login is unique per website rather than globally, so the three operations are replaced as follows. `current_website` is the website serving the request in which the state is computed.

```
_get_similar_users_domain(N) = inherited(N) + [ website_id IN W ]

  where W is built by walking N in order, starting from the empty list:
      w = p(l).website_id
      when w ≠ none and w ∉ W:                append w
      when w = none  and "no website" ∉ W:    append "no website", then append current_website

_get_similar_users_fields() = inherited() + [ website_id ]

_is_portal_similar_than_user(c, l) =
      false                                                   when inherited(c, l) is false
      c.website_id ≠ none AND c.website_id = p(l).website_id    when p(l).website_id ≠ none
      c.website_id = none OR  c.website_id = current_website    when p(l).website_id = none
```

**Worked example five, two websites.** The installation serves two websites, `Shop Europe` (identifier 1) and `Shop Asia` (identifier 2). The request that opens the invitation dialog is served by `Shop Europe`, so `current_website = 1`. The dialog holds three lines.

| Line | Contact's website | Typed address | Normalized |
|---|---|---|---|
| Eve | `Shop Europe` (1) | `eve@acme.com` | `eve@acme.com` |
| Frank | `Shop Asia` (2) | `frank@acme.com` | `frank@acme.com` |
| Gina | none | `gina@acme.com` | `gina@acme.com` |

Building `W`: Eve contributes `1`; Frank contributes `2`; Gina, whose website is empty and "no website" is not yet in the list, contributes `no website` and then `1` (already present, harmless). So `W = [ 1, 2, no website, 1 ]`, and the search condition is:

```
login IN { eve@acme.com, frank@acme.com, gina@acme.com } AND website_id IN { 1, 2, no website }
```

Suppose the existing users are:

| Candidate | Login | Website |
|---|---|---|
| user 81 | `eve@acme.com` | `Shop Asia` (2) |
| user 82 | `frank@acme.com` | `Shop Asia` (2) |
| user 83 | `gina@acme.com` | `Shop Europe` (1) |

All three are returned by the search, because all three websites are in `W`. The per-line test then decides:

| Line | Inherited test | Contact's website | Candidate's website | `email_state` |
|---|---|---|---|---|
| Eve | passes (user 81 holds the login, and Eve has no linked user) | `Shop Europe` (1) | `Shop Asia` (2) | `ok`, because the websites differ |
| Frank | passes (user 82) | `Shop Asia` (2) | `Shop Asia` (2) | `exist` |
| Gina | passes (user 83) | none | `Shop Europe` (1) = `current_website` | `exist` |

Eve's line therefore grants normally and creates a second user with the login `eve@acme.com` on `Shop Europe`. Frank's line and Gina's line refuse the grant with `The contact "Frank" has the same email as an existing user` and `The contact "Gina" has the same email as an existing user`.

**Worked example six, the same data served by `Shop Asia`.** Only `current_website` changes, from 1 to 2. `W` becomes `[ 1, 2, no website, 2 ]`, which contains the same three distinct values, so the candidate set is unchanged. Eve is still `ok` and Frank is still `exist`; Gina, however, becomes `ok`, because her Contact has no website and user 83 belongs to `Shop Europe`, which is no longer the current website. The same dialog on the same data therefore produces a different state for Gina depending on the website that serves the request, and a replacement must reproduce that dependence.

**Without the Website and Storefront capability package** the base definitions apply unchanged, every line of the two examples above is `exist` whenever any user holds the login, and no website is ever consulted.

---

## 8. Deciding whether an address actually changed

**Purpose**: avoid writing an address that is identical to the stored one, which would create pointless tracking entries and fire side effects.

```
unchanged(submitted, stored):
1. take each key of submitted in turn
2.    s = internal_representation(stored[key])
3.    n = submitted[key]
4.    when n ≠ s and (s is non-empty or n is non-empty), the answer is false and the test stops
5. when no key produced a difference, the answer is true
```

Two empty values of different kinds (an empty text and an absent relation, for instance) count as equal, which is why the second condition is needed.

**Worked example**: the stored Contact has `city = "Ramillies"`, `zip = "1367"`, `street2` empty. The form submits `city = "Ramillies"`, `zip = "1367"`, `street2 = ""`. Every comparison either matches or compares two empty values, so the address is unchanged and no write is issued; the response is nevertheless the success response.

**Worked example of the name exception**: the stored name is `" Partner A "` and the form submits `"Partner A"`. The comparison in step 6 of the update workflow trims both and finds them equal, so the name is removed from the values before the write. The holder name of the person's bank account is therefore untouched, which is the point of the rule.

---

## 9. Building the mandatory address field set

**Inputs**: the country `c`, the requested address kind `k` (billing or delivery), the "use the delivery address as the billing address" flag `u`, the form's own base required list `B`, the submitted values `V`, whether the edited address is the commercial address `m`, and whether the page needs a postal address `A`.

**Steps**

```
1. common(c) = { street, city, country }
             ∪ { state }  when c.state_required
             ∪ { zip }    when c.zip_required
2. billing(c)  = { name, email } ∪ ( { phone } ∪ common(c)  when A )
3. delivery(c) = { name, email } ∪ ( { phone } ∪ common(c)  when A )
4. R = B
5. when k = delivery or u:  R = R ∪ delivery(c)
6. when k = billing  or u:  R = R ∪ billing(c)
                            and when not m:  R = R \ { f ∈ commercial_fields : f ∉ V }
7. when ∃ f ∈ common(c) : V[f] is non-empty:  R = R ∪ common(c)
8. missing = { f ∈ R : V[f] is empty }
```

**Worked example one**: the account form of a person whose country requires neither a subdivision nor a postal code, submitting a complete address.

```
c.state_required = false, c.zip_required = false, A = true, k = billing, u = true, m = true
B          = { name, email }
common(c)  = { street, city, country }
delivery(c)= { name, email, phone, street, city, country }
billing(c) = { name, email, phone, street, city, country }
R after 5  = { name, email, phone, street, city, country }
R after 6  = the same set (m is true, so nothing is removed)
R after 7  = the same set
missing    = empty  →  the address is written
```

**Worked example two**: the same form for a country that requires both a subdivision and a postal code, with the city left empty.

```
common(c)  = { street, city, country, state, zip }
R          = { name, email, phone, street, city, country, state, zip }
V          = { name: "Acme Farm 3", email: "o@d.oo", phone: "+32…", street: "Rue de Ramillies 1",
               city: "", zip: "1367", country: 21 }
missing    = { city }
messages   = [ "Some required fields are empty." ]
response   = { invalid_fields: [ "city" ], messages: [ "Some required fields are empty." ] }
```

**Worked example three**: a child address of a company, kind billing, with no commercial field submitted.

```
m = false
commercial_fields = { vat, company_registry_number, industry,
                      payable_account, receivable_account, fiscal_position,
                      customer_payment_terms, vendor_payment_terms, credit_limit }
                    (the complete set as defined by PORT-RULE-102: the first three are always
                     present, the six accounting ones are present when the accounting capability
                     is installed, and a country package adds its own identification fields)
R after 6 = billing(c) ∪ B, minus every commercial field that is not in V
          = { name, email, phone, street, city, country }        (no commercial field survives)
```

The child therefore never has to supply the company's tax identification number in order to save its own address.

**Worked example four**: a page that does not need a postal address (a registration for an event with no shipping), with the customer typing a street anyway.

```
A = false  →  billing(c) = { name, email }
R after 6  = { name, email }
V[street]  = "Rue de Ramillies 1"  →  step 7 fires
R after 7  = { name, email, street, city, country }  (plus state and zip when the country requires them)
missing    = { city, country }
```

That is the purpose of step 7: a half-typed address is completed rather than stored incomplete.

---

## 10. Testing whether an address is complete

**Purpose**: decide whether the commercial entity of a company may be offered as a reusable address to its children.

```
billing_complete(p)  = every field of mandatory_billing_fields(p.country)  has a non-empty value on p
delivery_complete(p) = every field of mandatory_delivery_fields(p.country) has a non-empty value on p
```

**Worked example**: the company Contact has a name, an electronic mail address, a street, a city, a postal code and a country, but no telephone number. With the page needing an address, `mandatory_billing_fields` contains the telephone number, so `billing_complete` is false and the company address is removed from the billing list offered to its children.

---

## 11. Star display

**Purpose**: turn an average rating into a row of full, half and empty stars.

**Inputs**: `average` (a decimal between zero and five), `count` (an integer).

**Steps**

```
1. average   = round(average × 100) ÷ 100                       two decimal places
2. fraction  = round(average modulo 1, 1)                       one decimal place
3. whole     = the integer part of average
4. empty     = 5 − (whole + 1)   when fraction ≠ 0
             = 5 − whole         when fraction = 0
5. render: `whole` full stars, then one half star when fraction ≠ 0, then `empty` empty stars,
   then the count in parentheses
```

The title attribute of the whole widget is the rounded average.

**Worked examples**

| Raw average | Rounded | `fraction` | `whole` | `empty` | Rendered |
|---|---|---|---|---|---|
| 4.2837 | 4.28 | round(0.28, 1) = 0.3 | 4 | 5 − 5 = 0 | 4 full, 1 half, 0 empty |
| 3.0 | 3.00 | 0.0 | 3 | 5 − 3 = 2 | 3 full, 0 half, 2 empty |
| 2.04 | 2.04 | round(0.04, 1) = 0.0 | 2 | 5 − 2 = 3 | 2 full, 0 half, 3 empty |
| 4.96 | 4.96 | round(0.96, 1) = 1.0 | 4 | 5 − 5 = 0 | 4 full, 1 half, 0 empty |
| 0.0 | 0.00 | 0.0 | 0 | 5 | 0 full, 0 half, 5 empty |
| 5.0 | 5.00 | 0.0 | 5 | 0 | 5 full, 0 half, 0 empty |

Note the two consequences a replacement must reproduce: an average of 4.96 shows four and a half stars rather than five, and an average of 2.04 shows two stars rather than two and a half, because the fraction is rounded to one decimal place before it is tested.

**The compressed variant** shows one single icon plus the numeric average:

```
average ≤ 2.0                     →  an empty star
2.1 ≤ average ≤ 3.5               →  a half star
otherwise                         →  a full star
```

Worked examples: 1.8 shows an empty star; 2.05 falls in none of the first two cases and therefore shows a **full** star; 3.5 shows a half star; 3.51 shows a full star.

---

## 12. Rating statistics of a record

**Purpose**: feed the distribution bars shown under a set of ratings.

**Inputs**: the ratings of the record with a value of at least one.

**Steps**

```
1. total       = the number of ratings
2. average     = the arithmetic mean of the values
3. repartition = for each integer level from 1 to 5, the number of ratings at that level
4. percent[l]  = repartition[l] × 100 ÷ total     when total > 0
               = 0                                when total = 0
```

**Worked example**: the ratings of a course are 5, 5, 4, 3 and 1.

```
total       = 5
average     = (5 + 5 + 4 + 3 + 1) ÷ 5 = 18 ÷ 5 = 3.6
repartition = { 1: 1, 2: 0, 3: 1, 4: 1, 5: 2 }
percent     = { 1: 1 × 100 ÷ 5 = 20, 2: 0, 3: 20, 4: 20, 5: 2 × 100 ÷ 5 = 40 }
```

The star widget for the same record renders `average = 3.6`, `fraction = round(0.6, 1) = 0.6`, `whole = 3`, `empty = 5 − 4 = 1`, that is three full stars, one half star and one empty star, followed by `(5)`.

---

## 13. The due-date label in a record sidebar

**Purpose**: turn a due date into a sentence.

**Inputs**: the due date carried by the element, the reader's current date.

**Steps**

```
1. due   = the due date at the start of its day, in the reader's time zone
2. today = the current date at the start of its day, in the reader's time zone
3. difference = (due − today) expressed in whole days
4. difference = 0  →  "Due today"
   difference > 0  →  "Due in %s days"   with the absolute difference rendered with zero decimals
   difference < 0  →  "%s days overdue"  with the absolute difference rendered with zero decimals
```

**Worked examples** with a current date of the eleventh of September 2026:

| Due date | Difference | Label |
|---|---|---|
| eleventh of September 2026 | 0 | `Due today` |
| fifteenth of September 2026 | +4 | `Due in 4 days` |
| twelfth of September 2026 | +1 | `Due in 1 days` (the wording is not adjusted for the singular; reproduce it as written) |
| ninth of September 2026 | −2 | `2 days overdue` |

---

## 14. The author picture address in a portal message

**Purpose**: let a reader without a session fetch the picture of a message author.

```
when the call carried a security token:
    /mail/avatar/<message entity name>/<message identifier>/author_avatar/50x50?access_token=<token>
else when the call carried a signed identity and a recipient identifier:
    /mail/avatar/<message entity name>/<message identifier>/author_avatar/50x50?_hash=<signature>&pid=<contact>
else:
    /web/image/<message entity name>/<message identifier>/author_avatar/50x50
```

The serving endpoint resolves the message with elevated rights, then resolves the thread of that message with the supplied proof; when the thread does not resolve, it serves the shipped placeholder image instead of the author's picture, at the requested dimensions. When neither a token nor a signed identity was supplied, it serves the placeholder without even looking at the message.

**Worked example**: message 3175 on a sales order opened with a signed link for Contact 412 produces `/mail/avatar/<message entity name>/3175/author_avatar/50x50?_hash=9d41…c07e&pid=412`, which returns the author's picture scaled to fifty by fifty pixels.

---

## 15. The report file name

```
sanitized_name = replace_every_run_of_non_word_characters_with_one_underscore(report_base_name)
file_name      = sanitized_name + "." + extension
```

A "word character" is a letter, a digit or the underscore. The extension is always `pdf`, because only a portable document is served with a file name.

**Worked examples of the sanitized name**

| Report base name | Sanitized name |
|---|---|
| `S00042 - Acme (draft)` | `S00042_Acme_draft_` |
| `Invoice INV/2026/00042` | `Invoice_INV_2026_00042` |
| `Bon de commande n°7` | `Bon_de_commande_n_7` |

Note that a trailing run of non-word characters produces a trailing underscore before the dot; reproduce that.

---

## 16. The deleted-account login

```
new_login = "__deleted_user_" + user_identifier + "_" + current_time_in_seconds_with_fraction
```

The current time is the number of seconds since the start of the first of January 1970 in coordinated universal time, with a fractional part.

**Worked example**: user 57 deleted at the moment whose value is `1789123456.789012` receives the login `__deleted_user_57_1789123456.789012`. The fractional part is what makes two deletions of two different accounts in the same second produce different logins, and the identifier is what makes the same second for the same account impossible to collide with anything.

The password is set to the empty string at the same time. A credential check of an empty stored password always fails, so the account is unusable from that instant, before the archiving even runs.

---

## 17. The user menu name

```
displayed_name = first 23 characters of name + "..."   when length(name) > 25
               = name                                   otherwise
```

**Worked examples**

| Name | Length | Displayed |
|---|---|---|
| `Willis Barnett` | 14 | `Willis Barnett` |
| `Jean-Baptiste Poquelin ` | 23 | `Jean-Baptiste Poquelin ` |
| `Marie-Christine Delacroix` | 25 | `Marie-Christine Delacroix` |
| `Marie-Christine Delacroix-Fontaine` | 34 | `Marie-Christine Delacro...` |

Note the gap between 23 and 25: a name of exactly twenty-four or twenty-five characters is shown in full, and a name of twenty-six characters is cut at twenty-three, so the shortened form is twenty-six characters long. Reproduce this as written.

---

## 18. The language selector labels

```
full_label    = the segment of the language name after the last slash
compact_label = the segment of the language address code before the first underscore, uppercased
```

**Worked examples**

| Language name | Address code | Full label | Compact label |
|---|---|---|---|
| `English (US)` | `en` | `English (US)` | `EN` |
| `French (BE) / Français (BE)` | `fr_BE` | ` Français (BE)` | `FR` |
| `Spanish / Español` | `es` | ` Español` | `ES` |

The leading space of a label taken from a name that contains spaces around the slash is collapsed by the rendering.

---

## 19. Ordering and grouping of a list page

**Purpose**: turn a chosen grouping and a chosen sort into one ordering expression.

```
group_field = none                     when the chosen grouping is "none"
            = "priority desc"          when the chosen grouping is the priority (highest first)
            = the grouping field       otherwise

ordering = group_field + ", " + sort_expression    when group_field is set
         = sort_expression                          otherwise
```

The rows are then read with that ordering, the page size as the limit and the navigator's offset, and are cut into consecutive runs that share the same value of the grouping field. Because the grouping is applied **after** the paging, a group can be split across two pages; that is the observed behavior and a replacement must reproduce it rather than paging by group.

**Sorting by a selection field**: when the chosen sort is a selection field whose natural ordering is its stored value rather than its meaning (the task status is the shipped case), the groups are re-sorted after the read by the position of the group's first row's value in the selection list, and an ungrouped page is re-sorted row by row the same way.

**Worked example** (task list, grouping by stage, sorting by deadline ascending, page size eighty, page two):

```
ordering = "stage_id, date_deadline asc"
offset   = 80
rows     = the eighty-first to the one-hundred-and-sixtieth task in that ordering
groups   = consecutive runs of equal stage among those eighty rows
```

**Timesheet totals**: the timesheet page additionally computes, per group, the sum of the recorded durations over the **whole** filtered set rather than over the displayed page, so the total shown next to a group heading is the group's real total even when the page shows only part of it. When the grouping is by day, the groups themselves are read from the whole filtered set ordered by day descending.

---

## 20. Deciding the payment amount offered on a record page

This computation belongs to [Sales](../sales/README.md); it is reproduced here because the portal record page is where it becomes visible.

```
is_down_payment =
      true                                    when the link says "down payment"
      false                                   when the link says "full amount"
      required_prepayment_share < 1           when no amount was suggested in the link
      suggested_amount < total                otherwise

amount =
      suggested_amount                        when is_down_payment and 0 < suggested_amount < total
      required_prepayment_amount              when is_down_payment otherwise
      suggested_amount or total               when the order is confirmed
      total                                   otherwise
```

A suggested amount below the required prepayment on an order that is not yet confirmed is refused before the page renders, with the message `The amount is lower than the prepayment amount.`

**Worked example**: an order of 1,200.00 in the reporting currency with a required prepayment share of 30 percent, opened from a payment link with no suggested amount and no explicit choice.

```
required_prepayment_share = 0.30 < 1          →  is_down_payment = true
required_prepayment_amount = round(1200.00 × 0.30, 2) = 360.00
amount = 360.00
```

The sidebar then shows the heading `Down payment`, the amount 360.00 and the share `30%`, and the button reads `Sign & Pay` when a signature is still required and `Accept & Pay` otherwise.
