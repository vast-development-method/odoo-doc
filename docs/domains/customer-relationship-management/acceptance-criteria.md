# Acceptance criteria

Given / When / Then scenarios that a replacement must pass. Each criterion is independently verifiable and uses concrete values. The scenarios cover every rule of [business-rules.md](business-rules.md), every state transition of [workflows.md](workflows.md), every formula of [calculations.md](calculations.md), the scoring model of [predictive-lead-scoring.md](predictive-lead-scoring.md) and the assignment algorithm of [lead-assignment.md](lead-assignment.md).

Unless a scenario says otherwise, the shipped master data of [configuration.md](configuration.md) is in place: the four stages `New` (sequence 1), `Qualified` (sequence 2), `Proposition` (sequence 3) and `Won` (sequence 70, won flag set), none restricted to a team; the team `Sales`; the three lost reasons; the four recurring plans. The acting user is a salesperson of the group **User: All Documents** unless stated otherwise.

## 1. Creating a Lead

### LEAD-AC-001 A Lead needs a title

**Given** a user creating a Lead
**When** the user saves it with no title and no customer
**Then** the record is refused by the storage layer and nothing is created.

### LEAD-AC-002 The title is derived from the customer

**Given** a contact named `Nibbler`
**When** a Lead is created with that contact as customer and no title
**Then** the title becomes `Nibbler's opportunity`.

### LEAD-AC-003 The default type follows the access group

**Given** a user who belongs to the access group **Show Lead Menu**
**When** that user creates a record with no explicit type
**Then** the type is `lead`.
**And given** a user outside that group
**When** that user creates a record with no explicit type
**Then** the type is `opportunity`.

### LEAD-AC-004 The default salesperson is the creator

**Given** the user Lucy
**When** Lucy creates a Lead with no explicit salesperson
**Then** the salesperson is Lucy and the assignment date is the instant of the creation.

### LEAD-AC-005 The default priority is the lowest

**Given** any creation with no explicit priority
**When** the record is saved
**Then** the priority is `0`.

### LEAD-AC-006 The stage is the first non-folded stage of the team

**Given** the team `Sales`, and a stage `Backlog` with sequence 0 whose folded flag is set
**When** a Lead of that team is created with no explicit stage
**Then** the stage is `New`, not `Backlog`, because the folded stage is excluded.

### LEAD-AC-007 A record created directly in a won stage is closed immediately

**Given** the stage `Won`
**When** a Lead is created with that stage and no closing date
**Then** the closing date is the instant of the creation, the probability is one hundred and the won status is `won`.

### LEAD-AC-008 Creation posts the team-aware creation message

**Given** the team `Sales`
**When** a Lead of that team is created
**Then** a message with the subtype Opportunity Created is posted whose body is `A new lead has been created for the team "Sales".`
**And when** a Lead with no team is created
**Then** the body is `A new lead has been created and is not assigned to any team.`

### LEAD-AC-009 The salesperson follows the record

**Given** a Lead created with the salesperson Lucy
**When** the record is saved
**Then** Lucy is a follower of the record.

### LEAD-AC-010 A commercial entity with matching contact data becomes the customer

**Given** a company contact `Nibbler` with the telephone `+1 202-555-0888` and the email `contact@nibbler.example.com`
**When** a Lead is created with that company as commercial entity, the same telephone, the same email and no customer
**Then** the customer of the Lead becomes `Nibbler`.

### LEAD-AC-011 A commercial entity with different contact data only fills the company name

**Given** the same company
**When** a Lead is created with that company as commercial entity and the telephone `+1 202-555-7799`
**Then** the customer stays empty and the company name of the Lead becomes `Nibbler`.

### LEAD-AC-012 The web address is normalized

**Given** a user typing `www.nibbler.example.com` in the web address of a new Lead
**When** the record is saved
**Then** the stored value is the canonical form produced by the contact domain's cleaning rule, and typing `http://www.nibbler.example.com` on another record produces the same stored value.

## 2. Contact synchronization

### LEAD-AC-020 Choosing a customer copies the contact block

**Given** a contact `Robert Poilvert`, child of the company `Nibbler`, with the job position `Chief Buyer`, the web address `https://nibbler.example.com`, the language `English`, the telephone `+1 202-555-0888` and the email `robert@nibbler.example.com`
**When** that contact is set as customer of a Lead whose contact fields are all empty
**Then** the contact name becomes `Robert Poilvert`, the company name becomes `Nibbler`, the job position becomes `Chief Buyer`, the web address becomes `https://nibbler.example.com`, the language becomes `English`, the email becomes `robert@nibbler.example.com` and the telephone becomes `+1 202-555-0888`.

### LEAD-AC-021 The address travels as a whole

**Given** a contact with the street `Rue du Test 5`, the city `Namur`, the postal code `5000`, the country Belgium and no second address line and no subdivision
**And** a Lead carrying the street `Old street`, the second address line `Box 9`, the city `Brussels`, the postal code `1000` and the country Belgium
**When** that contact is set as customer
**Then** the six address fields of the Lead become `Rue du Test 5`, empty, `Namur`, `5000`, empty, Belgium: the second address line is cleared even though the Lead had one, because the whole block is replaced.

### LEAD-AC-022 An address-less customer does not erase the address of the Lead

**Given** a contact with none of the six address fields filled
**And** a Lead carrying a complete address
**When** that contact is set as customer
**Then** the six address fields of the Lead are unchanged.

### LEAD-AC-023 The contact name of a company customer is not copied

**Given** a company contact `Nibbler`
**And** a Lead whose contact name is `Robert Poilvert`
**When** that company is set as customer
**Then** the contact name of the Lead stays `Robert Poilvert`, because the candidate computed from a company customer is empty and the previous value is kept.

### LEAD-AC-024 The company name is taken from the parent first

**Given** a contact `Robert Poilvert` whose parent is the company `Nibbler` and whose own stored company name is `Planet Express`
**When** that contact is set as customer
**Then** the company name of the Lead becomes `Nibbler`.
**And given** a contact with no parent, not a company, whose stored company name is `Planet Express`
**When** that contact is set as customer
**Then** the company name of the Lead becomes `Planet Express`.

### LEAD-AC-025 A customer without a language clears the language of the Lead

**Given** a Lead whose language is `French` and a contact whose language is empty
**When** that contact is set as customer
**Then** the language of the Lead becomes empty.

### LEAD-AC-026 A formatting-only change of the email does not touch the customer

**Given** a Lead whose customer has the email `hermes@test.example.com`, with the same value on the Lead
**When** the user types `"Hermes Conrad" <hermes@test.example.com>` in the email of the Lead and saves
**Then** the warning flag stays false and the email of the customer is still `hermes@test.example.com`.

### LEAD-AC-027 A formatting-only change of the telephone does not touch the customer

**Given** a Lead whose customer has the telephone `+1 (202) 555-0888`, formatted on the Lead as `+1 202-555-0888`
**When** the user types the strictly digit form `+12025550888` in the telephone of the Lead and saves
**Then** the warning flag stays false and the telephone of the customer is unchanged.

### LEAD-AC-028 A real change of the email updates the customer

**Given** the same Lead
**When** the user types `"John Zoidberg" <john.zoidberg@test.example.com>` and saves
**Then** the warning flag is true before saving, and after saving the email of the customer is `"John Zoidberg" <john.zoidberg@test.example.com>` and its normalized address is `john.zoidberg@test.example.com`.

### LEAD-AC-029 Clearing the email of the Lead never clears the email of the customer

**Given** the same Lead after the previous scenario
**When** the user clears the email and the telephone of the Lead and saves
**Then** the email and the telephone of the customer are unchanged, and the email, the telephone and the sanitized telephone of the Lead are empty.

### LEAD-AC-030 A portal user does not overwrite the email of a customer with a salesperson

**Given** a Lead assigned to a reseller, whose customer already has a salesperson
**When** a portal user of that reseller changes the email of the Lead through the portal
**Then** the email of the customer is unchanged.

### LEAD-AC-031 Telephone quality

**Given** the country Belgium on a Lead
**When** the telephone `+32 2 290 34 90` is written
**Then** the telephone quality is `correct`.
**When** the telephone `not a number` is written
**Then** the telephone quality is `incorrect`.
**When** the telephone is cleared
**Then** the telephone quality is empty.

### LEAD-AC-032 Email quality

**When** the email `robert@nibbler.example.com` is written
**Then** the email quality is `correct`.
**When** the email `robert@@nibbler` is written
**Then** the email quality is `incorrect`.
**When** the email `invalid, robert@nibbler.example.com` is written
**Then** the email quality is `correct`, because at least one contained address is valid.

### LEAD-AC-033 Telephone reformatting in a form

**Given** a form with the country Belgium
**When** the user types `022903490` in the telephone field
**Then** the field shows `+32 2 290 34 90`.
**When** the user types `not a number`
**Then** the field keeps `not a number` unchanged.

### LEAD-AC-034 Email domain criterion

**When** the email of a Lead is `robert.poilvert@mycompany.example`
**Then** the email domain criterion is `@mycompany.example`.
**When** the email is `robert.poilvert@freemail.example`, a generic provider domain
**Then** the email domain criterion is `robert.poilvert@freemail.example`.
**When** the email is cleared
**Then** the email domain criterion is empty.

### LEAD-AC-035 Choosing a contradicting commercial entity clears the customer

**Given** a Lead whose customer is `Robert Poilvert`, child of `Nibbler`, with the email and telephone of that contact
**When** the user sets the commercial entity to the unrelated company `Planet Express`
**Then** the customer, the email and the telephone of the Lead are cleared and the commercial entity stays `Planet Express`.

### LEAD-AC-036 Posting a message with a recipient attaches the customer

**Given** two Leads with no customer, both carrying the email `robert@nibbler.example.com`, one in the stage `New` and one in a folded stage
**When** a message naming the contact `Robert Poilvert`, whose email is that address, is posted on the first one
**Then** the first Lead receives that contact as customer and the Lead in the folded stage does not.

## 3. Team, company and stage derivation

### LEAD-AC-040 The team follows the salesperson

**Given** the user Lucy, member of the team `Direct Sales` only
**When** a Lead of type `opportunity` is created with Lucy as salesperson and no team
**Then** the team becomes `Direct Sales`.

### LEAD-AC-041 The team is kept when the salesperson belongs to it

**Given** a Lead of the team `Europe` with the salesperson Lucy, who is a member of both `Europe` and `Direct Sales`
**When** the salesperson is written again with the same value
**Then** the team stays `Europe`.

### LEAD-AC-042 Clearing the salesperson keeps the team

**Given** a Lead of the team `Europe` with the salesperson Lucy
**When** the salesperson is cleared
**Then** the team stays `Europe` and the assignment date becomes empty.

### LEAD-AC-043 The type restricts the team lookup

**Given** the team `Lead Qualification` that uses leads but not opportunities, and the team `Direct Sales` that uses opportunities but not leads, both having Lucy as member, with `Lead Qualification` having the lower sequence
**When** a record of type `lead` is created with Lucy as salesperson and no team
**Then** the team becomes `Lead Qualification`.
**When** a record of type `opportunity` is created the same way
**Then** the team becomes `Direct Sales`.

### LEAD-AC-044 The company follows the team when the team has one

**Given** the team `Europe` whose company is `Company A`
**When** a Lead of that team is created
**Then** the company of the Lead is `Company A`.

### LEAD-AC-045 The company is invalidated when it contradicts the salesperson

**Given** a Lead whose company is `Company A` and a user allowed only in `Company B`
**When** that user is written as salesperson and the team has no company
**Then** the company of the Lead is recomputed and becomes `Company B`.

### LEAD-AC-046 A Lead with nothing set has no company

**Given** a Lead with no salesperson, no team and no customer
**When** it is saved
**Then** its company is empty, and it is visible from every company.

### LEAD-AC-047 The stage is replaced when it does not belong to the team

**Given** the stage `Europe qualified` restricted to the team `Europe`, and a Lead of the team `Europe` in that stage
**When** the team of the Lead is changed to `Asia`
**Then** the stage is replaced by the first non-folded stage available to `Asia`, which is `New`.

### LEAD-AC-048 The stage is kept when it is shared

**Given** a Lead of the team `Europe` in the shared stage `Qualified`
**When** the team is changed to `Asia`
**Then** the stage stays `Qualified`.

### LEAD-AC-049 A Lead without a team only sees the shared stages

**Given** the stage `Europe qualified` restricted to the team `Europe`
**When** a Lead with no team looks for its first stage
**Then** `Europe qualified` is not a candidate and the result is `New`.

### LEAD-AC-050 Changing the team changes the available properties

**Given** the team `Europe` defining the property `Contract length` and the team `Asia` defining the property `Local partner`
**And** a Lead of `Europe` whose property `Contract length` holds `24 months`
**When** the team is changed to `Asia`
**Then** the property `Contract length` no longer exists on the record and the property `Local partner` does.

## 4. Probability, won, lost and restore

### LEAD-AC-060 The probability bounds are enforced

**When** a probability of `-1` or of `101` is written on a Lead
**Then** the write is refused with `The probability of closing the deal should be between 0% and 100%!`

### LEAD-AC-061 A probability of one hundred alone is not won

**Given** a Lead in the stage `New` with a probability of fifty
**When** the probability is written to one hundred
**Then** the won status stays `pending`, because the stage is not a won stage.

### LEAD-AC-062 A won stage forces the record won

**Given** a lost Lead, that is, archived with a probability of zero, in the stage `New`
**When** the stage is written to `Won`
**Then** the record is active again, its probability is one hundred, its automated probability is one hundred, its closing date is the instant of the write and its won status is `won`.

### LEAD-AC-063 A won record cannot be marked lost

**Given** a Lead whose won status is `won`
**When** the Mark Lost operation is applied
**Then** the operation is refused with `A lead in a Won stage cannot be lost. Move it to another stage first.` and nothing changes.

### LEAD-AC-064 A won record cannot have a probability below one hundred

**Given** a Lead in the stage `Won` with a probability of one hundred
**When** a probability of seventy-five is written
**Then** the write is refused with `A lead in a Won stage cannot be lost. Move it to another stage first.`

### LEAD-AC-065 A won record stays won when archived

**Given** a Lead whose won status is `won`
**When** it is archived
**Then** its probability is still one hundred and its won status is still `won`.

### LEAD-AC-066 Leaving a won stage makes the record pending

**Given** a Lead in the stage `Won` with a probability of one hundred
**When** the stage is written to `Qualified`
**Then** the won status becomes `pending`, and the record is not lost because its probability is not zero.

### LEAD-AC-067 Mark Lost archives and zeroes

**Given** a pending Lead with a probability of thirty-two
**When** the Mark Lost operation is applied with the reason `Too expensive`
**Then** the record is archived, its probability is zero, its automated probability is zero, its lost reason is `Too expensive`, its closing date is the instant of the operation and its won status is `lost`.

### LEAD-AC-068 The closing note is embedded in the tracking message

**Given** two Leads selected together
**When** the Mark Lost operation is applied with the reason `Too expensive` and the closing note `Competitor was 20 % cheaper`
**Then** each of the two records carries a message with the subtype Opportunity Lost whose body begins with `Lost Comment:` followed by `Competitor was 20 % cheaper`, and which reports the changes of the active flag, of the lost reason and of the won status.

### LEAD-AC-069 An empty rich text closing note produces no note

**Given** a Lead
**When** the Mark Lost operation is applied with a closing note that contains only empty rich text markup
**Then** the tracking message carries no closing note at all.

### LEAD-AC-070 Restore realigns the probability

**Given** a lost Lead in the stage `Qualified` whose automated probability recomputes to `41.23`
**When** the Restore operation is applied
**Then** the record is active, its lost reason is empty, its automated probability is `41.23`, its probability is `41.23` and it is automatic again, and a message with the subtype Opportunity Restored is posted.

### LEAD-AC-071 Unarchiving keeps a manual probability

**Given** a lost Lead whose probability had been typed manually as `60` before it was lost
**When** the record is unarchived without restoring
**Then** the record is active, its lost reason is empty, its automated probability has been recomputed, and its probability is whatever the unarchive left, that is, not realigned on the automated value.

### LEAD-AC-072 Mark Won selects the next won stage

**Given** the stages `New` (1), `Won small` (2, won), `Negotiation` (3), `Won large` (4, won)
**And** a Lead in `Negotiation`
**When** the Mark Won operation is applied
**Then** the stage becomes `Won large` and the probability one hundred.

### LEAD-AC-073 Mark Won falls back on the previous won stage

**Given** the same pipeline
**And** a Lead in `Won large`
**When** the Mark Won operation is applied
**Then** the stage stays `Won large` and the closing date is not rewritten.

### LEAD-AC-074 A probability typed by a salesperson is not overwritten

**Given** a Lead whose automated probability is `20.87` and whose probability was typed as `40`
**When** any scored value of the record changes and the model recomputes
**Then** the automated probability changes and the probability stays `40`.

### LEAD-AC-075 An automatic probability follows the model

**Given** a Lead whose probability and automated probability are both `2.43`
**When** its country changes and the model recomputes the automated value to `34.38`
**Then** both the automated probability and the probability become `34.38`.

### LEAD-AC-076 Using the automated value makes the record automatic again

**Given** the Lead of scenario LEAD-AC-074
**When** the operation "use the automated value" is invoked
**Then** the probability becomes equal to the freshly computed automated probability and the record is automatic.

### LEAD-AC-077 Flagging a stage as won rewrites its records

**Given** three Leads in the stage `Negotiation`, with probabilities of ten, fifty and a manually typed ninety
**When** an administrator sets the won flag on `Negotiation`
**Then** the three records have a probability of one hundred and an automated probability of one hundred.

### LEAD-AC-078 Unflagging a stage recomputes its records

**Given** the same three records
**When** the administrator clears the won flag on `Negotiation`
**Then** the automated probability of the three records is recomputed and the probability of the records that were still automatic follows it; the manually typed ninety is lost.

### LEAD-AC-079 The stage toggle warns the user

**Given** an existing stage
**When** the user changes its won flag in a form
**Then** a warning is shown, titled `Do you really want to update this stage?` with the body `Changing the value of 'Is Won Stage' may induce a large number of operations, as the probabilities of opportunities in this stage will be recomputed on saving.`
**And when** the user sets the flag while creating a new stage
**Then** no warning is shown.

## 5. Dates

### LEAD-AC-090 The closing date is set when the probability reaches one hundred

**Given** a Lead with no closing date
**When** a probability of one hundred is written
**Then** the closing date becomes the instant of the write.

### LEAD-AC-091 The closing date is cleared when the probability becomes positive again

**Given** a closed Lead
**When** a probability of forty is written
**Then** the closing date becomes empty.

### LEAD-AC-092 A stage change without a probability clears the closing date

**Given** a closed Lead in a non-won stage
**When** its stage is changed to another non-won stage without writing a probability
**Then** the closing date becomes empty.

### LEAD-AC-093 A stage change between two won stages keeps the closing date

**Given** a Lead closed on the tenth of March in the won stage `Won small`
**When** its stage is changed to the won stage `Won large` on the twelfth of March
**Then** the closing date is still the tenth of March.

### LEAD-AC-094 The assignment date

**Given** a Lead with no salesperson
**When** the salesperson Lucy is written
**Then** the assignment date becomes the instant of the write.
**When** Lucy is written again
**Then** the assignment date does not move.
**When** the salesperson Bob is written
**Then** the assignment date becomes the instant of that write.
**When** the salesperson is cleared
**Then** the assignment date becomes empty.

### LEAD-AC-095 The last stage update date

**Given** a Lead in the stage `New`
**When** the stage `New` is written again
**Then** the last stage update date does not move.
**When** the stage `Qualified` is written
**Then** the last stage update date becomes the instant of that write.

### LEAD-AC-096 Days to assign and days to close

**Given** a Lead created on the third of March at 09:30, assigned on the fifth of March at 08:00 and closed on the fourteenth of March at 17:45
**When** the derived fields are read
**Then** the days to assign is `1` and the days to close is `11`.

### LEAD-AC-097 Rotting

**Given** the stage `Negotiation` with a rotting threshold of seven days
**And** an opportunity in that stage whose last stage change is eight days old, with a pending won status
**When** the rotting flag is read
**Then** it is true and the rotting days reads `8`.
**And given** a record of type `lead` in the same situation
**Then** the rotting flag is false.
**And given** an opportunity in a stage whose threshold is zero
**Then** the rotting flag is false.

### LEAD-AC-098 Duplicating a Lead

**Given** an opportunity in the stage `Proposition` with a probability of sixty, a closing date, the salesperson Lucy (active), the team `Europe`, a recurring revenue of 1 200.00 and the plan `Yearly`
**When** it is duplicated by a user who belongs to the recurring revenue group
**Then** the copy has no stage taken from the source but the computed first stage, no closing date, a recomputed probability, the type `opportunity`, the team `Europe`, the assignment date set to the instant of the copy, a recurring revenue of 1 200.00 and the plan `Yearly`.
**And when** it is duplicated by a user outside that group
**Then** the copy has a recurring revenue of zero and no plan.

### LEAD-AC-099 Duplicating a Lead whose salesperson is archived

**Given** an opportunity whose salesperson has been archived
**When** it is duplicated
**Then** the copy has no salesperson and an empty assignment date.

## 6. Revenue calculations

### LEAD-AC-110 Prorated revenue

**Given** an expected revenue of `12 345.67` and a probability of `37.5`
**When** the prorated revenue is read
**Then** it is `4 629.63`.

### LEAD-AC-111 Monthly recurring revenue

**Given** a recurring revenue of `36 000.00` and the plan `Over 3 years`
**Then** the monthly recurring revenue is `1 000.00`.
**And given** no plan at all
**Then** the monthly recurring revenue is `36 000.00`.

### LEAD-AC-112 Prorated recurring figures

**Given** a recurring revenue of `36 000.00`, the plan `Over 3 years` and a probability of forty-two
**Then** the prorated recurring revenue is `15 120.00` and the prorated monthly recurring revenue is `420.00`.

### LEAD-AC-113 The recurring plan becomes required

**Given** a form open on an opportunity, for a user in the recurring revenue group
**When** the user types a recurring revenue different from zero and leaves the plan empty
**Then** the record cannot be saved until a plan is chosen.

### LEAD-AC-114 The currency follows the company

**Given** a Lead whose company uses the euro, denoted `EUR`
**When** the currency of the record is read
**Then** it is `EUR`.
**And given** a Lead with no company, in a session whose active company uses the United States dollar, denoted `USD`
**Then** the currency of the record reads `USD`.

### LEAD-AC-115 The sum of confirmed orders

**Given** an opportunity of a company whose currency is `EUR`, linked to a draft quotation of `1 000.00 EUR`, a confirmed order of `2 500.00 EUR` and a confirmed order of `3 000.00 USD` dated a day whose rate is one `USD` for `0.92 EUR`
**When** the counters are read
**Then** the quotation count is `1`, the order count is `2` and the sum of orders is `5 260.00`.

### LEAD-AC-116 Confirming an order raises the expected revenue

**Given** an opportunity with an expected revenue of `2 000.00 EUR` and a probability of sixty, linked to a quotation of `2 500.00 EUR`
**When** the quotation is confirmed
**Then** the expected revenue becomes `2 500.00`, the prorated revenue becomes `1 500.00` and the change is tracked with the message `Expected revenue has been updated based on the linked Sales Orders.`

### LEAD-AC-117 A smaller order does not lower the expected revenue

**Given** the opportunity after the previous scenario
**When** an order of `1 800.00 EUR` is confirmed on it
**Then** the expected revenue stays `2 500.00`.

### LEAD-AC-118 An order in another currency never changes the expected revenue

**Given** the same opportunity, whose company currency is `EUR`
**When** an order of `9 000.00 USD` is confirmed on it
**Then** the expected revenue stays `2 500.00`.

## 7. Duplicates

### LEAD-AC-130 Duplicates by email domain

**Given** three Leads carrying the emails `robert@mycompany.example`, `accounting@mycompany.example` and `robert@freemail.example`
**When** the potential duplicates of the first are read
**Then** the second is among them and the third is not.

### LEAD-AC-131 A non-discriminating criterion is dropped

**Given** twenty-two Leads carrying an email of the same company domain
**When** the potential duplicates of one of them are read through the email domain criterion
**Then** that criterion contributes nothing, because it matches twenty-one records or more.
**And given** twenty Leads carrying that domain
**Then** the criterion contributes the nineteen other records.

### LEAD-AC-132 Duplicates by commercial entity

**Given** a company `Nibbler` with the child contacts `Robert` and `Alice`
**And** one Lead whose customer is `Robert` and one whose customer is `Alice`
**When** the potential duplicates of the first are read
**Then** the second is among them, with no cap applied to this criterion.

### LEAD-AC-133 Duplicates by sanitized telephone number

**Given** two Leads whose telephone numbers are written `+1 202-555-0888` and `+12025550888`, which sanitize to the same value
**When** the potential duplicates of the first are read
**Then** the second is among them.

### LEAD-AC-134 The duplicate set includes archived records and crosses companies

**Given** a Lead and an archived Lead of another company sharing the same email domain criterion
**When** the potential duplicates are read by a user who may not see the other company
**Then** the archived record of the other company is counted.

### LEAD-AC-135 The conversion duplicate search includes lost records

**Given** a Lead with the email `robert@nibbler.example.com`
**And** a lost opportunity with the same email
**When** the conversion dialogue computes the similar records
**Then** the lost opportunity is among them.
**And when** the mass conversion dialogue computes them
**Then** the lost opportunity is not among them.

## 8. Conversion

### LEAD-AC-140 Conversion is refused on a closed record

**Given** a Lead whose probability is one hundred
**When** the conversion dialogue is opened on it
**Then** the operation is refused with `Closed/Dead leads cannot be converted into opportunities.`

### LEAD-AC-141 The dialogue proposes merging when duplicates exist

**Given** a Lead with two similar records
**When** the conversion dialogue is opened
**Then** the proposed action is `Merge with existing opportunities`.
**And given** a Lead with none or one
**Then** the proposed action is `Convert to opportunity`.

### LEAD-AC-142 The dialogue proposes linking when a contact matches

**Given** a Lead whose email matches the contact `Robert Poilvert`
**When** the conversion dialogue is opened
**Then** the customer handling is `Link to an existing customer` and the customer is `Robert Poilvert`.
**And given** a Lead whose email matches nothing
**Then** the customer handling is `Create a new customer` and the customer is empty.

### LEAD-AC-143 Converting creates a company and a contact

**Given** a Lead titled `Spacecraft fleet` with the contact name `Robert Poilvert`, the company name `Nibbler`, the email `"Robert Poilvert" <robert@nibbler.example.com>`, the telephone `+32 2 290 34 90`, the address `Rue du Test 5`, `Namur`, `5000`, Belgium, and the salesperson Lucy
**When** it is converted with the action `Create a new customer`
**Then** a company contact `Nibbler` is created with that address, that telephone, the email `robert@nibbler.example.com`, the salesperson Lucy and the company flag set
**And** a contact `Robert Poilvert` is created as a child of `Nibbler`, not a company, with the same values and the contact kind `contact`
**And** the Lead is linked to `Robert Poilvert`, its type becomes `opportunity` and its conversion date is the instant of the operation.

### LEAD-AC-144 Converting with no contact name and no company name

**Given** a Lead titled `Website request` with no contact name, no company name and no email
**When** it is converted with the action `Create a new customer`
**Then** a single contact named `Website request`, not a company, is created and linked.

### LEAD-AC-145 Converting parses the contact name from the email

**Given** a Lead with no contact name and the email `"Robert Poilvert" <robert@nibbler.example.com>`
**When** it is converted with the action `Create a new customer`
**Then** the created contact is named `Robert Poilvert`.

### LEAD-AC-146 Converting skips archived and won records

**Given** a selection of three records: one active pending, one archived, one whose won status is `won`
**When** the conversion is applied to the three
**Then** only the first becomes an opportunity; the two others are unchanged and no error is raised.

### LEAD-AC-147 The salespeople are distributed round robin

**Given** six Leads selected in the order `L1` to `L6` and four salespeople `S1` to `S4`
**When** they are converted with those four salespeople and the force flag set
**Then** `L1` and `L5` go to `S1`, `L2` and `L6` to `S2`, `L3` to `S3` and `L4` to `S4`.

### LEAD-AC-148 The force flag protects the existing salesperson

**Given** three Leads, the second of which already has the salesperson Bob
**When** they are converted with the salesperson Lucy and the force flag cleared
**Then** the first and the third receive Lucy and the second keeps Bob.
**And when** the same is done with the force flag set
**Then** the three receive Lucy.

### LEAD-AC-149 Mass conversion deduplicates first

**Given** four Leads, two of which share the customer `Nibbler`
**When** the mass conversion is applied with deduplication enabled
**Then** the two records sharing the customer are merged into one before conversion, and three opportunities exist afterwards.
**And when** the same is done with deduplication disabled
**Then** four opportunities exist afterwards.

### LEAD-AC-150 Mass conversion links or creates per record

**Given** two Leads, the first matching the contact `Robert Poilvert` by email and the second matching nothing
**When** the mass conversion is applied
**Then** the first is linked to `Robert Poilvert` and a new contact is created for the second.

### LEAD-AC-151 Conversion in merge mode re-points the dialogue

**Given** a Lead `A` whose similar records include an opportunity `B` of higher confidence
**When** the conversion dialogue is confirmed in merge mode
**Then** `B` survives, `A` is deleted, and the dialogue points at `B` at the moment the deletion happens, so that the deletion does not cascade onto the dialogue.

### LEAD-AC-152 Properties survive a conversion inside the same team

**Given** a Lead of the team `Europe` whose property `Contract length` holds `24 months`
**When** it is converted without changing the team
**Then** the property still holds `24 months`.
**And when** it is converted with the team changed to `Asia`, which does not define that property
**Then** the property no longer exists on the record.

### LEAD-AC-153 An archived language is not carried onto the created customer

**Given** the language `Inactive Language` which exists but is archived
**And given** a Lead whose language is `Inactive Language` and which has no customer
**When** the Lead is converted with the customer handling `Create a new customer`
**Then** a contact is created and its language is the default customer language of the platform, `en_US`, not the archived one.
**And given** a second Lead whose language is the active language `en_US`
**When** the same conversion runs
**Then** the created contact carries the language `en_US` because that language is active (see [entities.md](entities.md) section 8.7).

## 9. Merging

### LEAD-AC-160 Merging fewer than two records is refused

**When** a merge is requested on a single record
**Then** it is refused with `Select at least two Leads/Opportunities from the list to merge them.`

### LEAD-AC-161 Merging more than five records manually is refused

**When** a user requests a merge on six records
**Then** it is refused with `To prevent data loss, Leads and Opportunities can only be merged by groups of 5.`
**And when** the assignment job merges six duplicates
**Then** the merge succeeds.

### LEAD-AC-162 The merge dialogue excludes won records

**Given** a selection of three records, one of which has the won status `won`
**When** the merge dialogue is opened
**Then** it proposes only the two others.

### LEAD-AC-163 Survivor selection

**Given** four records: `P` an archived lead in a stage of sequence three with a probability of twenty-five and identifier 91; `Q` an active lead in a stage of sequence one with a probability of ten and identifier 45; `R` an archived opportunity in a stage of sequence two with a probability of sixty and identifier 12; `S` an active opportunity in a stage of sequence two with a probability of sixty and identifier 77
**When** they are merged
**Then** `S` survives, and the confidence order is `S`, `R`, `Q`, `P`.

### LEAD-AC-164 Merging two records with conflicting fields

**Given** the two records of [calculations.md](calculations.md) section 7.6.1
**When** they are merged
**Then** the survivor is the opportunity `Nibbler Spacecraft Request`, which keeps its own probability of fifty, receives the type `opportunity`, the salesperson Lucy, the team `Direct Sales`, the description `Wants a quotation for 3 units.` followed by a blank line and `Asked for a demonstration.`, the priority `2`, the tags Service and Training, an expected revenue of `1 500.00`, the customer `Nibbler`, the email `contact@nibbler.example.com`, the whole address `Test street`, empty, `Test City`, `5000`, empty, Belgium, and no lost reason
**And** the merged-away record is deleted.

### LEAD-AC-165 The merged priority is the highest

**Given** three records with priorities `1`, empty and `3`
**When** they are merged
**Then** the merged priority is `3`.
**And given** three records all with an empty priority
**Then** the merged priority is empty.

### LEAD-AC-166 The merged address block

**Given** three records with two, six and four non-empty address fields respectively, the second one being the middle of the confidence order
**When** they are merged
**Then** the six address fields of the survivor are those of the record with six, copied as a whole.

### LEAD-AC-167 A tie in the address score is broken by confidence

**Given** two records with four non-empty address fields each
**When** they are merged
**Then** the address of the record that comes first in the confidence order is used.

### LEAD-AC-168 The lost reason is propagated only onto a lost survivor

**Given** two lost records, the survivor having a probability of zero and no lost reason, the other having the reason `Too expensive`
**When** they are merged
**Then** the survivor receives `Too expensive`.
**And given** a survivor with a strictly positive probability
**Then** the merged lost reason is empty.

### LEAD-AC-169 The survivor keeps its automatic or manual probability

**Given** a survivor whose probability equals its automated probability
**When** it is merged with another record whose probability is different
**Then** the survivor is still automatic and its probability is unchanged.
**And given** a survivor whose probability was typed manually
**Then** it is still manual and its probability is unchanged.

### LEAD-AC-170 History moves with a prefixed subject

**Given** a merged-away record named `Website request` carrying a message with the subject `Demo request` and a message with no subject
**When** the merge runs
**Then** the survivor carries a message with the subject `From Website request: Demo request` and a message with the subject `From Website request`.

### LEAD-AC-171 Attachments move and are renamed

**Given** a merged-away record named `Website request` carrying the attachment `offer.txt`
**When** the merge runs
**Then** the survivor carries the attachment named `offer.txt (from Website request)`.

### LEAD-AC-172 Activities and meetings move

**Given** a merged-away record carrying one open activity and one meeting
**When** the merge runs
**Then** both point at the survivor, and the meeting points at it through the opportunity link and through the generic document reference.

### LEAD-AC-173 Only recently active followers move

**Given** a merged-away record followed by Robert, who posted a message on it eleven days ago, and by Alice, who never posted
**And** a survivor already followed by Lucy
**When** the merge runs
**Then** Robert follows the survivor, Alice does not, and Lucy is not duplicated.

### LEAD-AC-174 A merge summary is logged

**When** any merge runs
**Then** an internal note is posted on the survivor listing every merged-away record, its main fields, its properties formatted for reading, and the followers that were carried over.

### LEAD-AC-175 The stage is repaired after a forced team

**Given** a survivor whose merged stage is `Europe qualified`, restricted to the team `Europe`
**When** the merge forces the team `Asia`
**Then** the stage becomes the first stage available to `Asia` ordered by sequence then identifier.

### LEAD-AC-176 Merging a lead with an opportunity produces an opportunity

**Given** two leads and one opportunity
**When** they are merged
**Then** the result is an opportunity.
**And given** three leads
**Then** the result is a lead.

## 10. Automatic assignment

### LEAD-AC-190 Assignment is refused to a plain salesperson

**Given** a user who belongs only to the group **User: Own Documents Only**
**When** that user runs the assignment
**Then** it is refused with `Lead/Opportunities automatic assignment is limited to managers or administrators`.

### LEAD-AC-191 The daily quota

**Given** a member with a capacity of forty-five and no lead received in the last twenty-four hours
**Then** the quota of a scheduled run is `2` and the quota of a manual run is `2`.
**And given** the same member having received thirty leads in the last twenty-four hours
**Then** the quota of a scheduled run is `-28` and the quota of a manual run is `2`.

### LEAD-AC-192 A capacity of zero excludes the member

**Given** a member whose capacity is zero
**When** the assignment runs
**Then** that member receives nothing.

### LEAD-AC-193 A paused member receives nothing

**Given** a member whose capacity is forty-five and whose pause flag is set
**When** the assignment runs
**Then** that member receives nothing.

### LEAD-AC-194 An archived membership receives nothing

**Given** a membership that has been archived
**When** the assignment runs
**Then** it receives nothing and is not counted in the team capacity.

### LEAD-AC-195 Team capacity

**Given** a team with three active members of capacities forty-five, fifteen and fifteen
**Then** the team capacity is `75`.
**And when** the member of capacity forty-five is archived
**Then** the team capacity becomes `30`.

### LEAD-AC-196 The team is skipped when its capacity is zero

**Given** a team whose members all have a capacity of zero
**When** the assignment runs
**Then** no Lead is allocated to it and, for a manual run on that single team, the message is `No allocated leads to <team name> team because it has no capacity. Add capacity to its salespersons.`

### LEAD-AC-197 Distribution of ten Leads over three members

**Given** the team of [lead-assignment.md](lead-assignment.md) section 7, that is, three members with capacities ninety, sixty and thirty, no restricting condition, and ten unassigned Leads of that team with probabilities 82, 75, 70, 64, 58, 51, 45, 30, 22 and 9
**When** a manual assignment runs
**Then** the member of capacity ninety receives the Leads of probability 82, 64 and 51; the member of capacity sixty receives those of probability 75 and 58; the member of capacity thirty receives the one of probability 70; and the four Leads of probability 45, 30, 22 and 9 stay unassigned
**And** the six assigned Leads are opportunities.

### LEAD-AC-198 A restricting condition is respected

**Given** the same set-up with the condition a probability of at least seventy-five on the member of capacity thirty
**When** the assignment runs
**Then** that member receives nothing, the member of capacity ninety receives the Leads of probability 82, 70 and 58, and the member of capacity sixty receives those of probability 75 and 64.

### LEAD-AC-199 A preference condition is served first

**Given** the same set-up, with an empty assignment condition on the member of capacity thirty but the preference condition the tag list contains the tag Strategic, and the two Leads of probability 30 and 22 carrying that tag
**When** the assignment runs
**Then** the member of capacity thirty receives the Lead of probability 30 even though five Leads of higher probability were available, and the Lead of probability 22 stays unassigned because that member's quota is exhausted.

### LEAD-AC-200 Fairness between equal members

**Given** two members of the same team, both with a capacity of one hundred and fifty, the first membership created before the second
**And** thirty daily runs, each with exactly one Lead to distribute, spaced twenty-five hours apart
**When** the thirty runs complete
**Then** each member holds fifteen Leads.

### LEAD-AC-201 A won Lead is never allocated

**Given** an unassigned Lead whose stage is `Won`
**When** the assignment runs
**Then** it keeps no team and no salesperson.

### LEAD-AC-202 A lost Lead is never allocated

**Given** an unassigned Lead that is archived with a probability of zero
**When** the assignment runs
**Then** it keeps no team and no salesperson.

### LEAD-AC-203 A Lead with no stage is allocated

**Given** an unassigned Lead with no stage and a probability of zero
**When** the assignment runs
**Then** it receives a team and a member, because it is neither won nor lost.

### LEAD-AC-204 An already assigned Lead is not reassigned

**Given** a Lead that already has the team `Conversion` and the salesperson Bob
**When** the assignment runs over every team
**Then** its team and its salesperson are unchanged.

### LEAD-AC-205 A Lead whose salesperson was removed is not redistributed

**Given** a Lead of a team that once had a salesperson, therefore has an assignment date, and whose salesperson has since been cleared
**When** the member distribution runs
**Then** the Lead is not a candidate and receives nobody.

### LEAD-AC-206 A copy of a Lead is allocated

**Given** a copy of an opportunity, therefore a record with no assignment date, no team and no salesperson
**When** the assignment runs on a team whose condition is the salesperson is empty
**Then** the copy receives that team and a member.

### LEAD-AC-207 Deduplication during allocation keeps the master

**Given** an opportunity `Master` of the team `Direct Sales` with the salesperson Alice and the email `contact@nibbler.example.com`
**And** a new Lead `Duplicate` with no team, no salesperson and the email `Duplicate Email <contact@nibbler.example.com>`
**When** the assignment runs on a different team `Overflow`
**Then** `Duplicate` no longer exists, `Master` keeps the team `Direct Sales` and the salesperson Alice, and the notification reports `1 duplicates leads have been merged.`

### LEAD-AC-208 The allocation respects the team condition

**Given** the team `Conversion` whose assignment condition is the priority is `1`, `2` or `3`
**And** a pool of unassigned Leads of which some have the priority `0`
**When** the assignment runs on that team alone
**Then** no Lead of priority `0` receives that team.

### LEAD-AC-209 An invalid team condition is refused at save time

**When** an administrator types a condition naming a field that does not exist in the assignment condition of the team `Europe`
**Then** the save is refused with `Assignment domain for team Europe is incorrectly formatted`.

### LEAD-AC-210 An invalid member condition is refused at save time

**When** an administrator types an unparsable condition in the assignment condition of the membership of Lucy in `Europe`
**Then** the save is refused with `Member assignment domain for user Lucy and team Europe is incorrectly formatted`
**And when** the same is typed in the preference condition
**Then** the save is refused with `Member preferred assignment domain for user Lucy and team Europe is incorrectly formatted`.

### LEAD-AC-211 The assignment schedule is written from the settings

**Given** the current instant being the second of November at 10:00
**When** an administrator enables rule-based assignment, chooses the mode `Repeatedly`, the unit `Hours` and the number nineteen, and saves
**Then** the scheduled action is active and its next run is the third of November at 05:00.
**And when** the administrator then chooses the unit `Days` and the number two and saves
**Then** the next run is the fourth of November at 10:00.
**And when** the administrator then types the first of November at 10:00 directly and saves
**Then** the next run is the first of November at 10:00 and the scheduled action is still active.
**And when** the administrator then chooses the mode `Manually` and saves
**Then** the scheduled action is inactive and the next run is still the first of November at 10:00.
**And when** the administrator then disables rule-based assignment while leaving the mode at `Repeatedly` and saves
**Then** the scheduled action is still inactive.

### LEAD-AC-212 A repetition number out of range is refused

**When** an administrator types zero, or a negative number, in the repetition number
**Then** the change is refused with `Repeat frequency should be positive.`
**And when** the administrator types one hundred or more
**Then** the change is refused with `Invalid repeat frequency. Consider changing frequency type instead of using large numbers.`

### LEAD-AC-213 Leads created without a salesperson go to the team leader

**Given** rule-based assignment disabled and the team `Sales` whose leader is Lucy
**When** an incoming email creates a Lead for that team
**Then** the salesperson becomes Lucy and a note is logged reading `This new lead created by incoming email was automatically assigned to team leader Lucy`.

## 11. Predictive lead scoring

### LEAD-AC-230 A Lead with no stage scores zero

**Given** a Lead whose stage is empty
**When** the model runs
**Then** its automated probability is zero.

### LEAD-AC-231 A Lead in a won stage scores one hundred

**Given** a Lead in the stage `Won`
**When** the model runs
**Then** its automated probability is one hundred.

### LEAD-AC-232 A pending Lead never scores exactly zero or one hundred

**Given** a frequency table in which the stage row and the only tag row both hold a won count of ten million and a lost count of one
**When** the model runs on a pending Lead carrying that stage and that tag
**Then** its probability is `99.99`.
**And given** the counts reversed
**Then** its probability is `0.01`
**And in both cases** the won record of that universe reads one hundred and the lost record reads zero.

### LEAD-AC-233 The probability of a Lead with three known field values

**Given** the data set of [predictive-lead-scoring.md](predictive-lead-scoring.md) section 6, that is, five closed records and one open record in one team, with the scoring variables country, state and source
**When** a full rebuild and a recomputation run
**Then** the frequency table holds `stage_id` `New` won `2.1` lost `3.1`, `stage_id` `Qualified` won `2.1` lost `1.1`, `stage_id` `Proposition` won `2.1` lost `1.1`, `stage_id` `Won` won `2.1` lost `0.1`, `country_id` `C1` won `1.1` lost `1.1`, `country_id` `C2` won `1.1` lost `0.1`, `state` `S1` won `2.1` lost `0.1`, `state` `S2` won `0.1` lost `1.1`, `source_id` `Src1` won `0.1` lost `2.1` and `source_id` `Src2` won `1.1` lost `0.1`
**And** the automated probability of the open record is `51.01`.

### LEAD-AC-234 The probability with five variables

**Given** the same data set extended as in [predictive-lead-scoring.md](predictive-lead-scoring.md) section 7, with the variables country, state, email quality, telephone quality and source
**When** a full rebuild and a recomputation run
**Then** the automated probability of the open record is `74.30`.

### LEAD-AC-235 The explanation panel ranks the factors

**Given** the situation of the previous scenario
**When** the explanation panel is opened on the open record
**Then** the negative factors, lowest first, are the source and the country; the positive factors, highest first, are the state, the telephone quality and the stage; the email quality appears in neither list; the reported probability is `74.30`; and the reported team name is the name of the team of the record.

### LEAD-AC-236 Nonsense quality factors are dropped

**Given** a data set in which every won record carries an empty telephone quality and every lost record carries `correct`
**When** the explanation panel is opened on a record whose telephone quality is empty
**Then** the telephone quality does not appear among the positive factors even though its score exceeds one half.

### LEAD-AC-237 Tag frequencies and their effect

**Given** the data set of [predictive-lead-scoring.md](predictive-lead-scoring.md) section 8, that is, one hundred and fifty records in one team, forty-two won and one hundred and five lost, with two tags
**When** a full rebuild and a recomputation run
**Then** the tag rows hold `Tag one` won `33.1` lost `65.1` and `Tag two` won `23.1` lost `75.1`
**And** the open record with `Tag one` alone scores `33.69`, the one with `Tag two` alone scores `23.51`, and the one with both scores `28.05`.

### LEAD-AC-238 Removing every tag falls back on the team base rate

**Given** the open record with both tags of the previous scenario
**When** its tags are removed
**Then** its automated probability becomes `28.60`.

### LEAD-AC-239 A rare tag is ignored

**Given** a tag whose won and lost counts sum to forty-nine
**When** the model runs on a record carrying that tag
**Then** the tag contributes nothing, neither to the totals nor to the product.

### LEAD-AC-240 The stage contribution when winning

**Given** a record in the stage `Qualified` (sequence 2) of a team whose four stages are `New`, `Qualified`, `Proposition` and `Won`
**When** the record is marked won
**Then** the won count of all four stage rows increases by one.

### LEAD-AC-241 The stage contribution when losing

**Given** a record in the stage `Proposition` (sequence 3)
**When** the record is marked lost
**Then** the lost count of `New`, `Qualified` and `Proposition` increases by one and the lost count of `Won` does not move.

### LEAD-AC-242 Restoring decrements the lost counts

**Given** the record of the previous scenario
**When** it is unarchived
**Then** the lost count of `New`, `Qualified` and `Proposition` decreases by one.

### LEAD-AC-243 Counters never go below the floor

**Given** a frequency row whose lost count is `1.1`
**When** a decrement of one and then another decrement of one are applied
**Then** the stored value after the first is `0.1` and after the second is still `0.1`.

### LEAD-AC-244 Archiving a won record does not move any counter

**Given** a won record
**When** it is archived
**Then** no frequency row changes.

### LEAD-AC-245 Forcing a probability of zero on an archived record makes it lost

**Given** an archived record in the stage `New` whose probability is one hundred
**When** the probability is written to zero
**Then** the won status becomes `lost` and the lost counts of the stage `New`, of the country and of the email quality increase by one.

### LEAD-AC-246 Team-restricted stages disable the model

**Given** every stage restricted to a team, so that no stage has an empty team list
**And** a Lead with a probability typed manually as `41.23`
**When** the recomputation runs
**Then** the probability stays `41.23` and the automated probability stays zero.

### LEAD-AC-247 A Lead with no team uses the cross-team statistics

**Given** three teams each with their own statistics
**And** a Lead with no team
**When** the model runs
**Then** the probability uses the sums over the three teams, and it differs from the probability the same Lead would have if it belonged to a fourth team holding the same records.

### LEAD-AC-248 Deleting a team folds its statistics

**Given** the cross-team rows `stage_id` `1` won 20 lost 10, `stage_id` `2` won 0.1 lost 0.1, `stage_id` `3` won 10 lost 0, `country_id` `1` won 10 lost 0.1
**And** a team holding `stage_id` `1` won 20 lost 10, `country_id` `1` won 0.1 lost 10, `country_id` `2` won 0.1 lost 0, `country_id` `3` won 30 lost 30
**When** the team is deleted
**Then** the cross-team rows are `stage_id` `1` won 40 lost 20, `stage_id` `2` won 0.1 lost 0.1, `stage_id` `3` won 10 lost 0, `country_id` `1` won 10 lost 10 and `country_id` `3` won 30 lost 30, and no row exists for `country_id` `2`
**And** the rows of the deleted team no longer exist.

### LEAD-AC-249 The probability update dialogue writes the configuration

**Given** the scoring variable list holding seven names and the start date `2021-01-01`
**When** an administrator opens the probability update dialogue
**Then** it shows that date and those seven variables
**And when** the administrator changes the date to `2021-02-02`, removes the source and the language and confirms
**Then** the stored variable list holds the five remaining names in their original order, the stored start date is `2021-02-02`, and every open record has been recomputed.

### LEAD-AC-250 A non-administrator changes nothing

**Given** a sales administrator who is not a platform administrator
**When** that user confirms the probability update dialogue
**Then** neither system parameter changes, no rebuild runs, and no message is shown.

### LEAD-AC-251 An unparsable start date falls back for display

**When** the stored start date holds an empty text, or a text that is not a date
**Then** the settings screen shows the date eight days before today, and the model computes nothing.

### LEAD-AC-252 An unknown variable name is ignored

**Given** the stored variable list holding `country_id` followed by a name that is not a field
**When** the model runs
**Then** only the country is used and no error is raised.

### LEAD-AC-253 Removing every variable leaves the stage alone

**Given** the stored variable list holding an empty text
**When** a recomputation runs
**Then** the probability of each record depends on the stage alone, and the settings screen label reads `Stage`.

## 12. Teams and memberships

### LEAD-AC-270 Single-membership mode archives the other memberships

**Given** single-membership mode and the user Lucy, member of `Europe`
**When** a membership of Lucy in `Asia` is created
**Then** the membership in `Europe` is archived and the one in `Asia` is active.

### LEAD-AC-271 Multi-membership mode keeps both

**Given** multi-membership mode and the same situation
**When** the membership in `Asia` is created
**Then** both memberships are active.

### LEAD-AC-272 Duplicate active memberships are refused

**Given** single-membership mode and an active membership of Lucy in `Europe`
**When** a second active membership of Lucy in `Europe` is created
**Then** it is refused with `You are trying to create duplicate membership(s). We found that Lucy (Europe) already exist(s).`

### LEAD-AC-273 An archived duplicate is tolerated

**Given** an archived membership of Lucy in `Europe`
**When** a new active membership of Lucy in `Europe` is created
**Then** it is accepted and the archived one is left alone.

### LEAD-AC-274 A member must be allowed in the company of the team

**Given** the team `Europe` whose company is `Company A` and the user Bob, allowed only in `Company B`
**When** a membership of Bob in `Europe` is created
**Then** it is refused with `User 'Bob' is not allowed in the company 'Company A' of the Sales Team 'Europe'.`

### LEAD-AC-275 Changing the company of a team validates its members

**Given** the team `Europe` with no company and the members Lucy (allowed in `Company A`) and Bob (allowed in `Company B`)
**When** the company `Company A` is written on the team
**Then** the write is refused with `The following team members are not allowed in company 'Company A' of the Sales Team 'Europe': Bob`.

### LEAD-AC-276 Writing the member list synchronizes the memberships

**Given** the team `Europe` whose members are Lucy and Bob
**When** the member list is written as Lucy and Carol
**Then** the membership of Lucy stays active, the membership of Bob is archived, a membership of Carol is created, and Carol is added to the favourite users of the team.

### LEAD-AC-277 Archiving a user archives the memberships

**Given** the user Bob, member of `Europe` and `Asia`
**When** Bob is archived
**Then** both memberships are archived.

### LEAD-AC-278 The main team of a user is the oldest active membership

**Given** the user Lucy with an active membership in `Europe` created first and an active membership in `Asia` created later
**Then** the main team of Lucy is `Europe`.
**And when** the membership in `Europe` is archived
**Then** the main team of Lucy becomes `Asia`.

### LEAD-AC-279 The default team lookup

**Given** the user Lucy, member of `Europe` (sequence 5) and `Asia` (sequence 10), both using opportunities
**When** a document asks for the default team of Lucy with no extra condition
**Then** the result is `Europe`.
**And given** a condition restricting to teams that use leads, which only `Asia` satisfies
**Then** the result is `Asia`.
**And given** a user who is a member of nothing
**Then** the result is the first team of the database that satisfies the condition, ordered by the team ordering.

### LEAD-AC-280 Clearing both usage flags clears the alias

**Given** the team `Sales` with the alias name `info`
**When** both usage flags are cleared in a form
**Then** the alias name becomes empty.

### LEAD-AC-281 Writing a usage flag rewrites the alias defaults

**Given** the team `Sales` using opportunities only
**When** the leads usage flag is set by a user who belongs to the group **Show Lead Menu**
**Then** the alias defaults become the type `lead` and the team `Sales`.

### LEAD-AC-282 The shipped teams cannot be deleted

**When** the shipped website team is deleted
**Then** the deletion is refused with `Cannot delete default team "Website"`.

### LEAD-AC-283 A team with many sales orders cannot be deleted

**Given** a team with six active Sales Orders
**When** it is deleted
**Then** the deletion is refused with `Team Europe has 6 active sale orders. Consider cancelling them or archiving the team instead.`

### LEAD-AC-284 Member counters

**Given** a member that received two leads yesterday at 14:00, one today at 08:00 and twenty-seven more over the previous four weeks
**When** the counters are read today at 10:00
**Then** the twenty-four hour counter reads `3` and the thirty-day counter reads `30`.

### LEAD-AC-285 The counters include lost records

**Given** a member whose fourteen leads of the last thirty days include one that has since been marked lost
**Then** the thirty-day counter still reads `14`.

### LEAD-AC-286 Reassigning a lead moves it between counters

**Given** a member with fourteen leads of the last thirty days
**When** two of those leads are reassigned to another salesperson
**Then** the counter of the first member reads `12`.

### LEAD-AC-287 The monthly saturation flag

**Given** a team of capacity seventy-five whose members received forty, twenty and sixteen leads in the last thirty days
**Then** the monthly assigned count reads `76` and the saturation flag is true.

## 13. Acquisition channels

### LEAD-AC-300 Incoming email creates a Lead for the team of the alias

**Given** the team `Sales` with the alias `info`
**When** an email with the subject `Need a quotation` is received at that address from `robert@nibbler.example.com`
**Then** a record is created with the title `Need a quotation`, the email `robert@nibbler.example.com`, the team `Sales`, the type imposed by the alias defaults, and no salesperson taken from the gateway identity.

### LEAD-AC-301 An email with no subject

**When** an email with an empty subject is received at the alias
**Then** the created record is titled `No Subject`.

### LEAD-AC-302 A recognized sender becomes the customer

**Given** the contact `Robert Poilvert` whose email is `robert@nibbler.example.com`
**When** an email from that address is received at the alias
**Then** the created record has that contact as customer.

### LEAD-AC-303 The message priority is carried over

**When** an email carrying the priority `2` is received at the alias
**Then** the created record has the priority `2`.
**And when** an email carrying a priority outside the four allowed values is received
**Then** the created record has the default priority `0`.

### LEAD-AC-304 Replies go back to the team alias

**Given** a Lead of the team `Sales` whose alias is `info`
**When** a notification about that record is sent to a follower
**Then** the reply address of the notification is the address of that alias.

### LEAD-AC-305 The website contact form

**Given** a website whose default team is `Web Sales`, which uses leads, and whose default salesperson is Lucy
**When** a visitor submits the contact form with a description and an email, carrying tracking cookies for a campaign, a source and a medium
**Then** a record is created with the team `Web Sales`, the salesperson Lucy, the type `lead`, the description, the email, the campaign, the source and the medium of the cookies, and the language of the request.

### LEAD-AC-306 The form medium falls back on the website medium

**Given** a submission carrying no medium
**When** the record is created
**Then** the medium is the default medium of the entity when one exists, and otherwise the medium named `website`, which is created if it does not exist.

### LEAD-AC-307 The form drops the email and telephone when a customer is imposed

**Given** a form submission carrying a customer identifier, an email and a telephone
**When** the record is created
**Then** the submitted email and telephone are ignored and the values come from the customer.

### LEAD-AC-308 The visitor is linked and protected

**Given** a website visitor with no previous lead
**When** that visitor submits the contact form
**Then** the created record is linked to the visitor, the visitor's name becomes the contact name of the record, and the visitor is never removed by the inactive-visitor cleanup while the record exists.

### LEAD-AC-309 The live chat command

**Given** an operator in a conversation with an external contact
**When** the operator types `/lead Spacecraft fleet`
**Then** a record is created with the title `Spacecraft fleet`, the conversation as origin channel, the conversation history as description, the operator name as referrer, the source `Livechat`, no salesperson and no team, and that external contact as customer
**And** the operator receives the transient message `Created a new lead: <link to the record>`.

### LEAD-AC-310 The live chat command with no title

**When** the operator types `/lead` with no title
**Then** no record is created and the operator receives `Create a new lead with: /lead <lead title>`.

### LEAD-AC-311 A public visitor produces a record with no customer

**Given** a conversation whose only external member is the public visitor contact
**When** the command creates a record
**Then** the record has no customer.

### LEAD-AC-312 The chatbot lead step

**Given** the shipped lead generation script, whose fifth step creates a record, and a team configured on that step
**When** a public visitor answers the first question with `We need pricing for 3 units` and gives the email `visitor@example.com`
**Then** a record is created whose title is `We need pricing for 3 units` truncated to one hundred characters, whose description holds the collected answers followed by the conversation history, whose origin channel is the conversation, whose source is the source of the script, whose team is the configured one, and whose email is `visitor@example.com`.

### LEAD-AC-313 The chatbot lead step for a signed-in visitor

**Given** the same script and a signed-in visitor whose contact has a different email
**When** the step creates the record
**Then** the record is linked to that contact instead of carrying the email, and the contact's email and telephone are updated with the collected values when they differ.

### LEAD-AC-314 The chatbot lead step with no free answer

**Given** the same script and a visitor who gives no free answer
**When** the step creates the record
**Then** the title is `<script title>'s New Lead`.

### LEAD-AC-315 Access to the conversation of a lead

**Given** a record created from a live chat conversation
**When** a salesperson who is not a member of that conversation reads it
**Then** the read succeeds, and an attempt to write, create or delete the conversation is refused.

### LEAD-AC-316 Linking a record to an inaccessible conversation is refused

**When** a user creates a record pointing at a conversation they may not read
**Then** the creation is refused with `You cannot create leads linked to channels you don't have access to.`
**And when** the same user writes that link on an existing record
**Then** the write is refused with `You cannot update a lead and link it to a channel you don't have access to.`

### LEAD-AC-317 Event rules per attendee

**Given** an active rule triggered at registration creation, with the basis per attendee, the team `Events` and the tag `Trade show`
**When** three registrations are created for a matching event
**Then** three records are created, each linked to its registration, with the team `Events`, the tag `Trade show`, the rule, the event, the event name as referrer, and a description enumerating the participant as `<name> (<email> - <phone>)`.

### LEAD-AC-318 Event rules per order

**Given** the same rule with the basis per order
**When** three registrations of the same order are created
**Then** one record is created, linked to the three registrations, whose description enumerates the three participants.

### LEAD-AC-319 A registration never produces two records for one rule

**Given** a rule triggered at registration confirmation
**When** a registration is confirmed, then un-confirmed, then confirmed again
**Then** exactly one record exists for that registration and that rule, archived records counted.

### LEAD-AC-320 Two rules matching one event produce two records

**Given** two active rules both matching the same event and the same trigger
**When** a registration is created
**Then** two records are created, one per rule.

### LEAD-AC-321 Updating a registration updates its record

**Given** a record created per order from three registrations
**When** two more registrations of the same order are created
**Then** the existing record receives the two new registrations and its description gains a block titled `New registrations`.
**And when** the contact of one registration changes
**Then** the customer of the record is rewritten and its description gains a block titled `Updated registrations` with the suffix `(updated)`.

### LEAD-AC-322 Rules do not run during an import

**When** registrations are created as part of a data import
**Then** no rule runs and no record is created.

### LEAD-AC-323 Regenerating the leads of an event

**Given** an event with fewer registrations than the batching threshold
**When** an event manager regenerates the leads
**Then** the run is synchronous and the records appear immediately.
**And given** an event above the threshold
**Then** a generation request is created, at most one per event, and the scheduled job processes the registrations in batches, remembering the last processed one.
**And when** a user who is not an event manager requests it
**Then** it is refused with `Only Event Managers are allowed to re-generate all leads.`

### LEAD-AC-324 A second generation request for one event is refused

**When** a second generation request is created for an event that already has one
**Then** it is refused with `You can only have one generation request per event at a time.`

### LEAD-AC-325 Survey leads

**Given** a survey with one answer option flagged as lead generating and the team `Inbound` configured on it
**When** a participant completes the survey having chosen that answer
**Then** a record of type `opportunity` is created with the team `Inbound`, the survey as origin survey, a source named after the survey title, the medium `Survey`, the title `<participant name> survey results`, the contact name of the participant, a description listing the questions and the answers, and the participant's contact as customer when it is an active contact, or the typed address as email otherwise
**And** the participation points at the created record.

### LEAD-AC-326 A survey participation with no generating answer creates nothing

**When** a participant completes the survey without choosing any lead-generating answer
**Then** no record is created.

### LEAD-AC-327 Lead mining with companies only

**Given** a request for twenty-five leads targeting companies only, with the country Belgium
**When** the request is submitted and the service returns twenty-five companies
**Then** twenty-five records are created, each with the type, the team, the tags and the salesperson of the request, the external company identifier, the company name as title and as company name, the first returned address as email, the returned telephone, the web address built as `https://<domain>`, and the resolved country and subdivision
**And** each record carries a message introduced by `Opportunity created by Lead Generation`
**And** the request receives a number of the form `LMR001` and its state becomes `done`.

### LEAD-AC-328 Lead mining with contacts

**Given** the same request targeting companies and their contacts, with three contacts per company
**When** the service returns companies each with people
**Then** the first returned person of each company overrides the contact name, the email and the job position of the record.

### LEAD-AC-329 Lead mining errors

**When** the service answers that credits are exhausted
**Then** the state becomes `error`, the error kind becomes `credits` and no record is created.
**When** the service answers with no data
**Then** the state stays `draft`, the error kind becomes `no result` and no record is created.
**When** the transport fails
**Then** the operation is refused with `Your request could not be executed: <reason>`.

### LEAD-AC-330 Resetting a mining request

**Given** a request in the state `done` numbered `LMR001`
**When** it is reset to draft
**Then** its state is `draft` and its number is the literal text `New`.

### LEAD-AC-331 Enrichment eligibility

**Given** a record that is active, has the email `robert@nibbler.example.com`, has never been enriched, did not come from the identification service and has a probability below one hundred
**Then** the manual enrichment button is offered.
**And given** a record whose email quality is `incorrect`, or that was already enriched, or whose probability is one hundred, or that came from the identification service
**Then** the button is not offered.

### LEAD-AC-332 Enrichment fills only the empty fields

**Given** a record whose company name is already `Nibbler` and whose city is empty
**When** the service returns the company name `Nibbler Incorporated` and the city `Namur`
**Then** the company name stays `Nibbler` and the city becomes `Namur`.

### LEAD-AC-333 Enrichment marks the record whatever the outcome

**When** the service returns nothing for a record
**Then** the enrichment flag of that record is set to true and a note explaining that nothing was found is posted.
**And when** the email cannot be normalized
**Then** the flag is set and a note explaining the missing address is posted, without calling the service.
**And when** the email domain is a generic provider domain
**Then** the flag is set and the "not found" note is posted, without calling the service.

### LEAD-AC-334 Enrichment credit exhaustion stops the run

**Given** a batch of fifty records to enrich
**When** the service answers that credits are exhausted
**Then** the run stops, the acting user is notified with `Not enough credits for Lead Enrichment`, and the records of the batch that were not processed keep their flag unset.

### LEAD-AC-335 Automatic enrichment on creation

**Given** the enrichment setting `Enrich all leads automatically`
**When** any Lead is created
**Then** the enrichment job is triggered.
**And given** the setting `Enrich leads on demand only`
**Then** creating a Lead triggers nothing.

### LEAD-AC-336 Website identification

**Given** an active Lead Generation Rule record for the country Belgium matching the path pattern `/pricing*`
**When** a visitor located in Belgium opens `/pricing/plans`
**Then** a Reveal View is created for the pair of that rule and that network address, and a second visit from the same address creates nothing more.
**And when** the scheduled job resolves that address into a company
**Then** a record is created with the type, the team, the salesperson, the tags, the priority and the name suffix of the rule, the company data, the network address, the credits consumed and the rule, and the Reveal View is deleted.
**And when** the service cannot resolve it
**Then** the Reveal View moves to the state `not found`.

### LEAD-AC-337 A Reveal View is not repeated for a recent lead

**Given** a record created six weeks ago from the network address `203.0.113.7`, with a retention window of six months
**When** the scheduled job runs and a Reveal View exists for that address
**Then** the Reveal View is deleted before processing and no new record is created.

### LEAD-AC-338 A Reveal View expires

**Given** a Reveal View older than one month
**When** the cleanup runs
**Then** it is deleted.

### LEAD-AC-339 An invalid path pattern is refused

**When** an administrator types a pattern that does not compile in a Lead Generation Rule record
**Then** the save is refused with `Enter Valid Regex.`

### LEAD-AC-340 The contact count of a Lead Generation Rule record is bounded

**When** an administrator sets the number of tracked contacts to zero or to six
**Then** the save is refused with `Maximum 5 contacts are allowed!`

### LEAD-AC-341 The mail plugin

**Given** the contact `Robert Poilvert` belonging to `Company A`
**When** the plugin asks to create a record for that contact, with the subject `Need pricing` and an email body
**Then** a record is created in `Company A`, titled `Need pricing`, with `Robert Poilvert` as customer and the body as description, and the identifier is returned.
**And when** the contact identifier resolves to nothing
**Then** the answer is the error code `partner_not_found` and nothing is created.

### LEAD-AC-342 The mass mailing counter

**Given** a mass mailing whose source produced eleven records
**When** the lead counter of the mailing is read
**Then** it reads `11`, archived records included.

### LEAD-AC-343 Campaign tracking at creation

**Given** a user who belongs to the group **User: Own Documents Only**
**When** that user creates a Lead while tracking cookies for a campaign are present
**Then** the campaign of the record stays empty, because a salesperson never inherits tracking cookies.
**And given** an anonymous visitor submitting the contact form with the same cookies
**Then** the campaign of the created record is the campaign of the cookie.

### LEAD-AC-344 The Add rules button of an event question answer

**Given** the event `Spacecraft Fair` carrying the question `Question test` with the answer option `Answer test`
**When** a sales administrator uses the **Add rules** operation next to that answer option
**Then** a creation dialogue for an Event Lead Rules record opens as a modal, pre-filled with the name `Answer test`, the acting user as the salesperson written on the created records, and the registration condition `registration_answers.question IN (Question test) AND registration_answer_options.answer_option IN (Answer test)`, and no rule exists yet.
**And when** the user saves the dialogue with the name `event_question_answer_rule`
**Then** exactly one Event Lead Rules record with that name exists.
**And when** a registration for `Spacecraft Fair` is then created with the email `visitor@nibbler.example.com` and the answer `Answer test`
**Then** exactly one Lead whose normalized email is `visitor@nibbler.example.com` is created by that rule.

## 14. Resellers and the partner portal

### LEAD-AC-360 A Lead with no country is skipped

**Given** two Leads, one with the country Belgium and one with none
**When** the partner assignment operation runs on both
**Then** the first is geolocated and assigned, the second is untouched, and the acting user is warned with `There is no country set in addresses for <name of the second record>.`

### LEAD-AC-361 The nearest window wins

**Given** a Lead geolocated at latitude 50.47 and longitude 4.87 in Belgium
**And** three Belgian partners inside the first window, with weights ten, five and one
**When** the assignment runs
**Then** the chosen partner is drawn among those three with probabilities of `62.5 %`, `31.25 %` and `6.25 %`, and no wider window is searched.

### LEAD-AC-362 A wider window is searched when the first is empty

**Given** the same Lead and a single Belgian weighted partner at latitude 51.20 and longitude 3.22, which is outside the first window and inside the second
**When** the assignment runs
**Then** that partner is chosen.

### LEAD-AC-363 A partner with a weight of zero is never chosen

**Given** a single Belgian partner whose level weight is zero, inside the first window
**When** the assignment runs
**Then** no partner is found, the shipped tag `No more partner available` is added to the Lead, and no partner is assigned.

### LEAD-AC-364 A partner that declined is excluded

**Given** a Lead whose declined partner list holds the only weighted partner of the first window
**When** the assignment runs
**Then** that partner is excluded and the search widens to the next window.

### LEAD-AC-365 Assignment writes the forwarding date and the salesperson

**Given** a chosen partner whose salesperson is Lucy
**When** the assignment writes it on a Lead that is active with a probability below one hundred
**Then** the assigned partner is written, the forwarding date becomes today, and the salesperson of the Lead becomes Lucy.

### LEAD-AC-366 Clearing the assigned partner clears the forwarding date

**When** the assigned partner of a Lead is cleared
**Then** the forwarding date becomes empty.

### LEAD-AC-367 Forwarding requires an email address

**Given** a chosen partner with no email address
**When** the forwarding dialogue is confirmed in automatic mode
**Then** it is refused with `Set an email address for the partner(s): <partner name>`
**And in single mode**
**Then** it is refused with `Set an email address for the partner <partner name>`.

### LEAD-AC-368 Forwarding sends one email per recipient

**Given** four Leads, two proposed to the partner `Alpha` and two to the partner `Beta`
**When** the forwarding is confirmed
**Then** exactly two emails are sent, each listing the two records of its recipient with their portal link and stating whether that recipient already has a portal account
**And** each record receives its assigned partner and the salesperson of that partner, with no notification to the followers, and the partner becomes a follower.

### LEAD-AC-369 The reseller accepts

**Given** a portal user whose commercial entity is the assigned partner of a Lead
**When** that user accepts the Lead with the comment `We know this customer`
**Then** a message reading `I am interested by this lead.` followed by `We know this customer` is posted, and the record becomes an opportunity keeping its customer.

### LEAD-AC-370 The reseller declines

**When** the same user declines the Lead, stating they contacted it, with the comment `Out of our region`
**Then** a message reading `I am not interested by this lead. I contacted the lead.` followed by `Out of our region` is posted, every contact of that reseller's commercial family is unsubscribed and added to the declined partner list, and the assigned partner is cleared.

### LEAD-AC-371 The reseller declines as spam

**When** the same user declines the Lead as spam
**Then** the same happens and the shipped tag `Spam` is added.

### LEAD-AC-372 A stranger cannot write on the Lead

**Given** a portal user of another reseller
**When** that user tries to accept, decline, update or change the stage of the Lead
**Then** the operation is refused with `Only users with commercial partner which is a parent of the assigned partner can edit this lead.`

### LEAD-AC-373 The reseller updates the Lead

**Given** a portal user of the assigned reseller
**When** that user submits an expected revenue of `5 000.00`, a probability of `0`, the priority `2`, an expected closing date, an activity type, an activity summary and an activity deadline
**Then** the expected revenue becomes `5 000.00`, the probability becomes empty, the priority becomes `2`, the expected closing date is written
**And** the portal user's own activity is updated when one exists, and a new activity of that type, summary and deadline assigned to that user is created otherwise.

### LEAD-AC-374 The reseller may only change a closed set of contact fields

**When** the portal user submits a change to the company name, the telephone, the email, the street, the second address line, the city, the postal code, the subdivision or the country
**Then** the change is applied.
**And when** the submission includes the expected revenue
**Then** it is refused with `Not allowed to update the following field(s): expected_revenue.`

### LEAD-AC-375 The reseller creates an opportunity

**Given** a portal user whose commercial entity carries a grade
**When** that user submits a contact name, a description and a title
**Then** a record is created with the priority `2`, that commercial entity as assigned partner and the shipped tag `Created by Partner`, it receives the salesperson of that partner, and it is converted into an opportunity.
**And when** any of the three fields is empty
**Then** the answer is `All fields are required!` and nothing is created.
**And when** neither the user's contact nor its commercial entity carries a grade
**Then** access is denied.

### LEAD-AC-376 A portal user is redirected to the portal page

**Given** a portal user who may read a Lead
**When** that user follows a link to the record
**Then** the destination is the portal page of the record, not the internal form.

### LEAD-AC-377 A grade with a default pricelist propagates it

**Given** the grade `Gold` whose default pricelist is `Reseller Gold`
**When** that grade is written on a contact
**Then** the pricelist of the contact becomes `Reseller Gold`.
**And when** a different pricelist is written in the same operation
**Then** it is refused with `You are trying to assign two different pricelists (one directly and one from grade (Gold)).`

### LEAD-AC-378 A membership product grants a grade

**Given** a product whose service tracking is the membership value and whose grade is `Silver`
**When** an order containing it is confirmed for the customer `Nibbler`
**Then** the grade of the commercial entity of `Nibbler` becomes `Silver`.
**And when** the order contains two membership products granting different grades
**Then** the confirmation is refused with `You cannot confirm Sale Order S00042 because there are products assigning different grades.`

### LEAD-AC-379 The public reseller directory

**Given** three published resellers with the grades `Gold`, `Silver` and `Bronze`
**When** the public directory is requested
**Then** the three appear ordered by grade sequence, and a filter by grade or by country restricts the list accordingly
**And when** an unpublished grade is requested by a public reader
**Then** it is not visible.

### LEAD-AC-380 Who may publish a reseller page

**Given** the published company contact `Agrolait`, carrying the grade `Grade Test`, and its public reseller page
**When** the platform administrator opens the directory in editing mode
**Then** the publication control is offered and switching it publishes the page.
**And when** a user who belongs to the salesperson group and to the website editor group does the same
**Then** the publication control is offered and switching it publishes the page.
**And when** a user who belongs to the salesperson group but not to the website editor group does the same
**Then** the publication control is still offered and switching it publishes the page, because the right comes from write access on the contact (`LEAD-152`).

### LEAD-AC-381 Website editing rights alone do not allow publishing

**Given** the same reseller page
**And given** a user who belongs to the website editor group but neither to the salesperson group nor to the contact manager group
**When** that user opens the directory in editing mode
**Then** no publication control is offered for the reseller, and the publication flag cannot be changed.

### LEAD-AC-382 A visitor from a country without resellers still sees the directory

**Given** exactly one listable reseller, a company contact with a published grade, published, located in Belgium
**And given** a visitor whose network address is located in Mexico
**When** that visitor requests the directory without naming a country
**Then** the country inferred from the address, Mexico, holds no listable reseller, the country filter is dropped, the Belgian reseller is listed, and the country selector shows **All Countries** as the active entry (`LEAD-153`).
**And when** a second reseller located in Mexico is published
**Then** the same request lists the Mexican reseller only, because the inferred country now holds one.

### LEAD-AC-383 The directory is a page of the site and answers not-found when empty

**Given** an installation with the reseller capability package
**When** the pages of the site are listed
**Then** the directory path `/partners` is among them, under the name `Partners`, together with one path per grade and one path per country that holds at least one listable reseller.
**And when** no reseller at all is listable and the directory is requested
**Then** the page is rendered with the not-found status.

### LEAD-AC-384 The directory ordering and paging

**Given** forty-one listable resellers, of which `Alpha` and `Beta` carry the grade of sequence 1, `Alpha` having three implementation references and `Beta` five
**When** the first page of the directory is requested
**Then** the first entry is `Beta`, the second is `Alpha`, forty entries are listed, and the forty-first is on the second page.

## 15. Access and visibility

### LEAD-AC-390 A salesperson sees only their own records

**Given** three Leads: one with the salesperson Lucy, one with the salesperson Bob and one with no salesperson
**When** Lucy, who belongs only to the group **User: Own Documents Only**, lists the Leads
**Then** she sees the first and the third and not the second.

### LEAD-AC-391 A salesperson with all documents sees everything

**When** a user of the group **User: All Documents** lists the same three records
**Then** all three are visible.

### LEAD-AC-392 Company scoping

**Given** a Lead of `Company A`, a Lead of `Company B` and a Lead with no company
**When** a user whose session is restricted to `Company A` lists the Leads
**Then** the first and the third are visible and the second is not.

### LEAD-AC-393 No salesperson may delete a Lead

**When** a user of the group **User: All Documents** deletes a Lead
**Then** the deletion is refused.
**And when** a sales administrator does it
**Then** it succeeds.

### LEAD-AC-394 A portal user sees only the records assigned to their family

**Given** a portal user whose commercial entity is `Alpha`
**When** that user lists the opportunities
**Then** only the records whose assigned partner is `Alpha` or a descendant of `Alpha` are visible, and they are read only.

### LEAD-AC-395 The lost reason catalogue is read only for a salesperson

**When** a user of the group **User: Own Documents Only** creates a lost reason
**Then** it is refused.
**And when** a sales administrator does it
**Then** it succeeds.

### LEAD-AC-396 The scoring statistics are read only for everybody

**When** any user, including a sales administrator, writes on a frequency row directly
**Then** it is refused; only the internal maintenance paths, which run with elevated rights, may write.

### LEAD-AC-397 The probability update dialogue is reserved

**When** a user who is not a platform administrator opens the probability update dialogue
**Then** the access rules refuse the record.

### LEAD-AC-398 The duplicate counter crosses visibility

**Given** a Lead of `Company A` and an archived Lead of `Company B` with the same email domain criterion
**When** a user restricted to `Company A` reads the duplicate counter of the first
**Then** it counts the record of `Company B`.

## 16. Reporting and screens

### LEAD-AC-410 The pipeline column set

**Given** a user whose team is `Europe`, which has the stages `New`, `Europe qualified` and `Won`, and a database that also holds `Asia qualified` restricted to `Asia`
**When** the pipeline is opened with the option that shows the reader's team stages
**Then** the columns are the stages present in the data, plus every stage with no team, plus the stages of `Europe`, ordered by sequence, name and identifier; `Asia qualified` does not appear unless a record of the grouping sits in it.

### LEAD-AC-411 A folded stage with no record is collapsed

**Given** the stage `Archived` whose folded flag is set and which holds no record
**When** the pipeline is opened
**Then** the column is shown collapsed.

### LEAD-AC-412 Default ordering

**Given** four Leads with priorities `3`, `1`, `1` and `0`, the two of priority `1` having identifiers 40 and 55
**When** the list is opened with no explicit ordering
**Then** the order is the record of priority `3`, then the record of identifier 55, then the record of identifier 40, then the record of priority `0`.

### LEAD-AC-413 Ordering by the reader's own activity deadline

**Given** ten Leads, four of which carry an activity assigned to the reader with deadlines spread over a week, and six of which carry none
**When** the list is ordered by the reader's own activity deadline ascending, with a window of five records
**Then** the first four are the records carrying the reader's activities ordered by earliest deadline, and the fifth is the first of the remaining records in the default ordering.

### LEAD-AC-414 The forecast measures the prorated revenue

**Given** three opportunities with expected revenues of `10 000.00`, `20 000.00` and `30 000.00` and probabilities of ten, fifty and ninety, all with an expected closing date in the same month
**When** the forecast is read for that month
**Then** the measured total is `1 000.00 + 10 000.00 + 27 000.00 = 38 000.00`.

### LEAD-AC-415 The activity report

**Given** a Lead carrying two messages, one with an activity type and one without
**When** the activity report is read
**Then** exactly one row exists for that record, carrying the lead's salesperson, team, stage, country, company, customer, type, active flag, won status and tags.

### LEAD-AC-416 The lost reason counter

**Given** the reason `Too expensive` used by seven Leads, two of which are archived
**When** the counter is read
**Then** it reads `7`.

### LEAD-AC-417 The opportunity counter of a contact

**Given** the company `Nibbler` with the children `Robert` and `Alice`, three opportunities on `Robert`, one on `Alice` and two on `Nibbler`
**When** the counters are read
**Then** `Robert` reads `3`, `Alice` reads `1` and `Nibbler` reads `6`.

### LEAD-AC-418 The digest indicators

**Given** a company that created thirty-four Leads and closed six opportunities at a probability of one hundred during the period
**When** the digest is computed
**Then** New Leads reads `34` and Opportunities Won reads `6`.
**And when** a user outside the salesperson group computes them
**Then** the access error `Do not have access, skip this data for user's digest email` is raised and the indicator is skipped.

### LEAD-AC-419 The celebration message for a first deal

**Given** a salesperson who has closed no deal this year
**When** that salesperson marks an opportunity won from the form
**Then** the message is `Go, go, go! Congrats for your first deal.`

### LEAD-AC-420 The celebration message for a team record

**Given** a salesperson who has already closed seven deals this year, an opportunity of `18 000.00` for the team `Direct Sales`, a best team deal of `22 000.00` over the last thirty-one days and a best team deal of `9 500.00` over the last seven days
**When** the opportunity is marked won from the form
**Then** the message is `Yeah! Best deal out of the last 7 days for the team.`

### LEAD-AC-421 The celebration message for the fifth deal of the day

**Given** a salesperson who has already closed four deals today and an opportunity whose expected revenue is zero
**When** the opportunity is marked won from the form
**Then** the message is `You're on fire! Fifth deal won today 🔥`

### LEAD-AC-422 The effort message wins over everything

**Given** an opportunity carrying twenty-five messages or more
**When** it is marked won from the form
**Then** the message is `Phew, that took some effort — but you nailed it. Good job!` whatever the other conditions.

### LEAD-AC-423 No salesperson means no celebration

**Given** an opportunity with no salesperson
**When** it is marked won from the form
**Then** no celebration message is returned.

### LEAD-AC-424 The meeting screen picks a relevant period

**Given** an opportunity with exactly one future meeting
**When** the meeting screen is opened from the record
**Then** the mode is the week mode and the initial date is the day of that meeting.
**And given** several future meetings all falling in the same week of the reader's language
**Then** the mode is the week mode and the initial date is the day of the earliest.
**And given** several future meetings spread over more than one week
**Then** the mode is the month mode and the initial date is the day of the earliest.
**And given** no meeting at all
**Then** the mode is the week mode and no initial date is imposed.

### LEAD-AC-425 Creating a meeting logs it on the opportunity

**Given** an opportunity
**When** a meeting of two hours is created on it, outside an activity
**Then** a note is posted on the opportunity reading `Meeting scheduled at <date and time in the reader's time zone>`, then `Subject: <link to the meeting>`, then `Duration: 2 hours`.
**And when** the meeting has no duration
**Then** the last line reads `Duration: unknown`.

### LEAD-AC-426 Deleting a Lead detaches its meetings

**Given** an opportunity with one meeting that points at it through the generic document reference
**When** the opportunity is deleted
**Then** the meeting still exists and its document reference is empty.

### LEAD-AC-427 The duplicate screen

**Given** a Lead with three potential duplicates, one of which is archived
**When** the duplicate screen is opened
**Then** the four records, the original and its three duplicates, are listed, archived records included, and creation is disabled.

### LEAD-AC-428 Action links from an email

**Given** a notification email carrying the won, the lost and the conversion links for a record
**When** an authenticated internal user follows the won link with a valid token
**Then** the record is marked won and the user is redirected to it.
**And when** the token does not match the record
**Then** nothing changes.

### LEAD-AC-429 A meeting created from an activity inherits the opportunity defaults

**Given** the opportunity `Spacecraft fleet` of the team `Europe`, whose customer is `Robert Poilvert`, carrying an activity whose meeting starts on 2023-02-15 at 14:00 in the reader's time zone
**When** the calendar event creation is opened from that activity
**Then** the screen offered is pre-filled with the opportunity `Spacecraft fleet`, the customer `Robert Poilvert`, the attendees `Robert Poilvert` and the acting user, the team `Europe`, the title `Spacecraft fleet`, and the initial date 2023-02-15.
**And when** the activity's meeting points at no opportunity
**Then** none of those values is imposed and the screen is the plain calendar event creation of the messaging domain.

### LEAD-AC-430 Time spent in each stage

**Given** a Lead created on 2023-02-15 at 12:00:00 in the stage `New`
**When** it is moved to `Qualified` at 12:05:00
**Then** its time map reads `New` 300 seconds and `Qualified` 0 seconds.
**And when** it is moved to `Proposition` at 13:45:00
**Then** the map reads `New` 300, `Qualified` 6000, `Proposition` 0.
**And when** it is moved back to `Qualified` at 14:15:00
**Then** the map reads `New` 300, `Qualified` 6000, `Proposition` 1800, the two stays in `Qualified` sharing one entry.
**And when** the map is read again at 14:45:00 with no further move
**Then** it reads `New` 300, `Qualified` 7800, `Proposition` 1800, because the open interval since the last move is added to the current stage.

### LEAD-AC-431 Time spent in each stage is computed per record in a batch

**Given** three Leads created on 2023-02-15 at 12:00:00 in the stage `New`
**And given** that the first is moved to `Qualified` at 12:05:00, the second at 12:10:00, and the third is never moved
**When** the time maps of the three records are read together at 12:20:00
**Then** the first reads `New` 300 and `Qualified` 900, the second reads `New` 600 and `Qualified` 600, the third reads `New` 1200 and has no other entry.
**And** the number of reads performed against the stored stage changes does not grow with the number of records in the batch: the tracked changes of the whole batch are read once.

### LEAD-AC-432 The help text of an empty pipeline names the alias of a team of the reader

**Given** that every Sales Team is archived except four, all in the company of the reader, all with an alias that creates Leads: `UserTeamLeads` which qualifies leads and has the reader as member, `TeamLeads` which qualifies leads and has no member, `UserTeamOpportunities` which does not qualify leads and has the reader as member, and `TeamOpportunities` which does neither
**When** the reader opens a Lead screen that matches no record
**Then** the help block offers the alias of `UserTeamLeads`.
**And when** `UserTeamLeads` is archived and the screen is opened again
**Then** the help block offers the alias of `TeamLeads`.
**And when** `TeamLeads` is archived as well
**Then** the help block offers the alias of `UserTeamOpportunities`, then, once that one is archived too, the alias of `TeamOpportunities`.
**And when** the only remaining team with an alias belongs to another company
**Then** its alias is not offered and the help block shows the title alone.

### LEAD-AC-433 The title of the help text depends on the screen

**When** a screen opened with a default type of lead matches no record
**Then** the title reads `Create a new lead`.
**And when** a screen opened with any other default type matches no record
**Then** the title reads `Create an opportunity to start playing with your pipeline.`
**And when** the screen definition already carries its own help text, as the Leads Analysis screen reached from a mass mailing does
**Then** that text is shown unchanged and no alias sentence is added.

## 17. Messaging, text messages and blacklists

### LEAD-AC-440 The sanitized telephone number follows the telephone field

**Given** a Lead in the United States whose telephone is `+1 202 555 0122`
**Then** its sanitized number is `+12025550122`.
**And when** the telephone is cleared
**Then** the sanitized number is empty.
**And when** the telephone is written as the local number `202 555 0999`
**Then** the sanitized number is `+12025550999`, because the country of the record supplies the missing country code.

### LEAD-AC-441 Sending a text message from a list of opportunities

**Given** three opportunities selected in the opportunities list, one of which is lost
**When** the reader uses the text message header button
**Then** the text message composer opens in batch mode over the selection, with the option that keeps a log already enabled, so that each record receives a note recording what was sent.
**And when** the reader uses the row button of a single opportunity
**Then** the composer opens in single mode on that record only, and the row button is not offered on the lost record.

### LEAD-AC-442 Texting a website visitor that has no customer

**Given** a website visitor with no customer, whose number is `+1 555 555 5556`, linked to one Lead carrying the same telephone number
**When** the text message composer is opened from the visitor screen
**Then** it opens on that Lead, taking the number from the telephone field of the Lead.
**And given** a visitor linked to several Leads carrying that number
**Then** it opens on the most confident of them, the confidence order being the one of [business-rules.md](business-rules.md) `LEAD-068`.

### LEAD-AC-443 Texting a website visitor that has a customer

**Given** a website visitor whose customer is the contact `Test Partner`, whose telephone is `+1 555 555 5557`, and which is also linked to a Lead
**When** the text message composer is opened from the visitor screen
**Then** it opens on the contact `Test Partner`, taking the number from the telephone field of the contact, and not on the Lead.

### LEAD-AC-444 A campaign may pick its winning variant on the leads produced

**Given** a campaign running a split test over two variants
**When** the winner criterion of the email split test is set to `Leads`
**Then** the winning variant is the one whose source produced the most Leads.
**And when** the winner criterion of the text message split test is set to `Leads`
**Then** the same rule decides the winner of the text message variants.

### LEAD-AC-445 A blacklisted email address is excluded from mass communication

**Given** the address `robert@nibbler.example.com` on the email blacklist of the messaging domain
**And given** a Lead carrying that address
**Then** the blacklist flag of the Lead is true.
**And when** a mass mailing is sent to a selection containing that Lead
**Then** the Lead is skipped and no message is sent to that address.
**And when** the address is removed from the blacklist
**Then** the flag becomes false and the Lead is no longer skipped.

### LEAD-AC-446 A blacklisted telephone number is excluded from mass text messaging

**Given** the sanitized number `+12025550122` on the telephone blacklist of the messaging domain
**And given** a Lead whose telephone sanitizes to that number
**Then** the telephone blacklist flag of the Lead is true.
**And when** a text message is sent in batch mode to a selection containing that Lead
**Then** the Lead is skipped and no text message is sent to that number.
**And given** a Lead whose customer is blacklisted by email
**Then** the customer blacklist flag of the Lead is true even when the Lead's own address is not blacklisted.

### LEAD-AC-447 Naming a recipient in a message attaches the customer to every matching record

**Given** three Leads with no customer and the email `robert@nibbler.example.com`: the first in the stage `New`, the second in the stage `Qualified`, the third in the stage `Archived` whose folded flag is set
**And given** a fourth Lead with the same address that already has the customer `Somebody Else`
**When** a message is posted on the first Lead naming the contact `Robert Poilvert`, whose address is `robert@nibbler.example.com`
**Then** that contact becomes the customer of the first and of the second Lead
**And** the third Lead keeps no customer because its stage is folded
**And** the fourth Lead keeps `Somebody Else` (`LEAD-038`).

### LEAD-AC-448 Suggested recipients of a record whose address carries a display name

**Given** a Lead titled `Test Suggestion` whose address is `"New Customer" <new.customer.format@test.example.com>`, whose company name is `Format Name`, whose salesperson is Lucy, and which has no customer
**When** the suggested recipients of the record are requested without creating contacts
**Then** exactly one recipient is suggested, named `New Customer`, with the address `new.customer.format@test.example.com`, linked to no existing contact, and carrying the values that would create the contact: company name `Format Name`, not a company, contact kind `contact`, salesperson Lucy.

### LEAD-AC-449 Several addresses in one field suggest several recipients

**Given** a Lead whose address field holds `new.customer.multi.1@test.example.com, new.customer.2@test.example.com` and whose company name is `Multi Name`
**When** the suggested recipients are requested
**Then** two recipients are suggested: the first named with the whole field content, carrying the address `new.customer.multi.1@test.example.com` and the creation values of the record, the second carrying the address `new.customer.2@test.example.com` with an empty name and no creation values, because only the first address is the candidate customer of the record.

### LEAD-AC-450 A company name that matches an existing company suggests a child contact

**Given** the existing company contact `Nibbler`
**And given** a Lead whose address is `new.customer.with.parent@test.example.com` and whose company name is `Nibbler`
**When** the suggested recipients are requested
**Then** one recipient is suggested with that address and creation values that place the new contact under the company `Nibbler` as its parent, with no company name, not a company, contact kind `contact`, and the salesperson of the record.

### LEAD-AC-451 A record with a customer suggests that customer, and a record with a carbon copy suggests it too

**Given** a Lead whose customer is `Robert Poilvert`, whose address is `robert@nibbler.example.com`
**When** the suggested recipients are requested
**Then** one recipient is suggested, the contact `Robert Poilvert` with that address and no creation values, because the contact already exists.
**And given** a Lead whose customer `Test Partner` has no address and whose carbon copy field holds `accounts@nibbler.example.com`
**Then** two recipients are suggested: the contact `Test Partner` with an empty address and no creation values, and the carbon copy address with an empty name and no creation values.

### LEAD-AC-452 The language of a record reaches the suggested creation values only when it is active

**Given** a Lead whose address is `test.lang@test.example.com`, with no salesperson, whose language is the active language `en_US`
**When** the suggested recipients are requested
**Then** the creation values carry the language `en_US`, are marked as not a company and as the contact kind `contact`.
**And given** the same record whose language is an archived language
**Then** the creation values carry no language at all and the contact would be created with the default customer language.

### LEAD-AC-453 The suggested creation values repeat the whole contact block of the record

**Given** a Lead with the contact name `ContactAndCompany`, the company name `Delivery Boy company`, the address `default_create_with_partner@example.com`, the internal note `<p>Top</p>`, the job position `Delivery Boy`, the telephone `678-728-0949`, the street `3rd Floor, Room 3-C`, the street line two `123 Arlington Avenue`, the postal code `13202`, the city `New York`, the country subdivision `New York`, the country `United States`, the web address `https://www.arlington123.example/3f3c`, with the salesperson Lucy
**When** the suggested recipients are requested
**Then** one recipient is suggested, named `ContactAndCompany`, with that address, and its creation values hold the street, the street line two, the postal code, the city, the country, the country subdivision, the web address, the telephone, the job position, the salesperson Lucy, the internal note as the contact note, the company name `Delivery Boy company`, no parent company, and the flag that says the contact is not a company.
**And when** the record has no contact name and its address carries the display name `"Contact Name" <default_create_with_name_in_email@example.com>`
**Then** the suggested name is `Contact Name` and the suggested address is `default_create_with_name_in_email@example.com`.
**And when** the record has neither a contact name nor a display name in the address
**Then** the suggested name is the address itself.

## 18. Scenarios carried over from the second description

These scenarios restate, as verifiable criteria, the worked examples that the other description of
this domain carried. They use its numbers, so that both sets of numbers are covered.

### LEAD-AC-460 A lead converted with a newly created company and contact

**Given** a Lead with the type `lead`, the title `Spare parts for the Antwerp line`, the company name
`Northwind Parts`, the contact name `Anna Devries`, the job position `Procurement Manager`, the
email `anna@northwind-parts.example`, the telephone `+32 3 555 22 10`, the street `Rue Haute 12`, the
postal code `1000`, the city `Brussels`, the country Belgium, the active language Dutch, the notes
`Called on Monday.`, no customer, the salesperson Ines, the team `Direct Sales`, the stage
`Qualified` and an expected revenue of `18 000`
**And** no contact anywhere in the database carrying the address `anna@northwind-parts.example`
**When** the salesperson opens the conversion dialogue and confirms with the proposed defaults
**Then** the dialogue proposes `Create a new customer`, because no contact matched, and the action
`Convert to opportunity`, because the duplicate search returned only this record
**And** a company contact is created with the name `Northwind Parts`, the company flag set, no
parent, the salesperson Ines, the note `Called on Monday.`, the telephone `+32 3 555 22 10`, the
email `anna@northwind-parts.example`, the job position `Procurement Manager`, the address `Rue Haute
12`, `1000`, `Brussels`, Belgium, an empty stored company name, the contact kind `contact` and the
language Dutch
**And** a second contact is created with the name `Anna Devries`, the company flag clear, the parent
`Northwind Parts`, and the same salesperson, note, telephone, email, job position, address, contact
kind and language, its stored company name staying empty because it has a parent
**And** the Lead is linked to `Anna Devries`, its type becomes `opportunity`, its conversion date is
the instant of the operation, its stage is unchanged because it already had one, and its derived
contact fields re-derive to the same values
**And** exactly two contacts were created and exactly one Lead was updated.

### LEAD-AC-461 Two leads that share a postal address and an email domain are merged

**Given** two active leads, both of type `lead`, detected as duplicates because their email domain
criterion is `@northwind-parts.example`:

| Field | Lead Alpha | Lead Beta |
|---|---|---|
| stage | `Qualified`, sequence 2 | `New`, sequence 1 |
| probability | 22 | 35 |
| identifier | 5101 | 5140 |
| title | `Northwind — spare parts` | `Parts enquiry` |
| salesperson | empty | Karl |
| company name | `Northwind Parts` | empty |
| contact name | empty | `Bruno Adler` |
| email | `anna@northwind-parts.example` | `bruno@northwind-parts.example` |
| telephone | empty | `+32 2 555 01 44` |
| expected revenue | 18 000 | 0 |
| tags | Training | Service |
| priority | `1` | `2` |
| notes | `Called on Monday.` | `Sent the catalogue.` |
| street | `Rue Haute 12` | `Rue Haute 12` |
| street line two | empty | `Box 4` |
| postal code | `1000` | `1000` |
| city | `Brussels` | `Brussels` |
| state | empty | empty |
| country | Belgium | Belgium |

**When** they are merged
**Then** the confidence order is Alpha then Beta, because both are active leads and Alpha's stage
sequence is the higher, so record 5101 survives
**And** the merged values are: type `lead`; title `Northwind — spare parts`; salesperson Karl;
company name `Northwind Parts`; contact name `Bruno Adler`; email
`anna@northwind-parts.example`; telephone `+32 2 555 01 44`; expected revenue 18 000; stage
`Qualified`; tags Training **and** Service; priority `2`; notes `Called on Monday.` then a blank line
then `Sent the catalogue.`; and no lost reason, because the survivor's probability is not zero
**And** the whole address comes from Beta, which has five non-empty address fields against Alpha's
four: `Rue Haute 12`, `Box 4`, `1000`, `Brussels`, an **empty** state and Belgium
**And** the probability of the survivor is not taken from the merge; it is recomputed by the write
unless it was manual
**And** record 5140 is deleted after its messages, activities, attachments and meetings have been
re-pointed at record 5101.

### LEAD-AC-462 Thirty leads and three members with capacities ten, fifteen and five

**Given** one team with three members of capacities ten, fifteen and five, none paused, none with an
assignment condition, none having received a lead in the last twenty-four hours
**And** thirty leads already attached to the team with no salesperson and no assignment date
**When** a sales administrator presses the assignment control on that team, which forces the quota
**Then** the allocation phase finds nothing to allocate, because all thirty already have a team
**And** the daily quotas are `round_half_up_to_integer(10 ÷ 30) = 0`,
`round_half_up_to_integer(15 ÷ 30) = 1` and `round_half_up_to_integer(5 ÷ 30) = 0`
**And** only the member of capacity fifteen is eligible; the thirty leads are ordered by probability
descending, the first is given to that member and converted into an opportunity, the member's quota
falls to zero and the member leaves the rotation
**And** the remaining twenty-nine leads stay unassigned, and the notification reads `1 leads assigned
among 1 salespersons.`
**And when** the capacities are changed to three hundred, four hundred and fifty and one hundred and
fifty, which give daily quotas of ten, fifteen and five
**Then** a single run distributes all thirty: ten to the first member, fifteen to the second and five
to the third.

### LEAD-AC-463 An opportunity whose quotation is confirmed and which is then won

**Given** an opportunity titled `Antwerp line — spare parts` with an expected revenue of `18 000.00`
in the company currency, the stage `Proposition` of sequence 3, an automatic probability of `62.40`,
the salesperson Ines and the team `Direct Sales`
**And** one quotation of `21 400.00` untaxed in the same currency, in the sent state, referencing
that opportunity
**When** the quotation is confirmed
**Then** the quotation count falls from `1` to `0`, the order count rises from `0` to `1` and the sum
of orders becomes `21 400.00`
**And** the expected revenue becomes `21 400.00`, because `18 000.00` is strictly less and the
currency matches, and the change is tracked with `Expected revenue has been updated based on the
linked Sales Orders.`
**And** the prorated revenue becomes `round(21 400.00 × 62.40 ÷ 100, 2) = 13 353.60`
**When** the salesperson then presses the won control
**Then** the only won stage available to `Direct Sales` is `Won` of sequence 70, which is strictly
greater than 3, so it is chosen; the stage becomes `Won`, the probability and the automated
probability become one hundred, the active flag is forced true and the closing date is stamped
**And** the prorated revenue becomes `21 400.00`
**And** the won status becomes `won`, the won counters of `New`, `Qualified`, `Proposition` and `Won`
each rise by one, and so do the cells of the country, the state, the source, the language, the email
quality, the telephone quality and each tag of the record
**And** the days to close is computed from the creation instant to the closing instant
**And** a message with the subtype Opportunity Won is posted and the celebration is evaluated
**And** no journal entry is produced.

### LEAD-AC-464 An opportunity lost with a reason and a closing note

**Given** an opportunity titled `Ghent depot — conveyor belts` with an expected revenue of
`9 500.00`, a probability of `41.00`, the stage `Qualified` of sequence 2, the team `Direct Sales`,
the country Belgium, the source `Search Engine`, an email quality of `correct`, a telephone quality
of `correct` and one tag `Service`
**When** the salesperson presses the lost control, picks the reason `Too expensive` and types the
closing note `Competitor quoted 20 per cent below.`
**Then** the record is archived, its lost reason becomes `Too expensive`, its probability and its
automated probability become zero, its closing date is stamped and its days to close is computed
**And** the prorated revenue becomes `round(9 500.00 × 0 ÷ 100, 2) = 0.00`
**And** the won status becomes `lost`
**And** the lost counters of the stage cells `New` and `Qualified` rise by one and those of
`Proposition` and `Won` do not, because their sequences are above the record's
**And** the lost counters of the country, the source, the email quality and the telephone quality
cells rise by one, as do the language and state cells when those fields are set, and the cell of the
tag `Service` rises too — the threshold of fifty applies when the table is read, not when it is
written
**And** a message with the subtype Opportunity Lost is posted, showing the tracked change of the
lost reason and, below it, the block `Lost Comment:` followed by the note.

### LEAD-AC-465 The confidence order of four active leads

**Given** four active leads of type `lead`: P in a stage of sequence 3 with a probability of 25 and
identifier 41; Q in a stage of sequence 3 with a probability of 15 and identifier 42; R in a stage of
sequence 1 with a probability of 20 and identifier 40; S with no stage, therefore a stage sequence of
zero, a probability of 10 and identifier 43
**When** they are ordered by confidence, most trustworthy first
**Then** the order is P, Q, R, S: P beats Q because at an equal stage sequence its probability is
higher, and S ranks last because it has no stage
**And when** an archived lead T is added
**Then** T ranks last whatever its other values, because the first component of the key is false for
it alone.

### LEAD-AC-466 The celebration message for a deal that never left the first stage

**Given** an opportunity whose salesperson has already closed several deals this year, whose
expected revenue beats no record, and which spent at least sixty seconds in exactly one stage, that
stage being the first stage available to its team
**When** it is marked won from the form
**Then** the message is `No detours, no delays - from <stage name> straight to the win! 🚀`
**And given** the same record but a single stage that is **not** the first available one
**Then** that message is not returned and the country and source rules are still evaluated.

### LEAD-AC-467 Days to assign counts complete days only

**Given** a Lead created on the fifteenth of January at 09:12:44.318 and assigned on the eighteenth
of January at 09:12:43
**When** the days to assign is read
**Then** it is `2.0`: the creation instant is truncated to the second, the difference is two days,
twenty-three hours, fifty-nine minutes and fifty-nine seconds, and only complete days count.

### LEAD-AC-468 The unique-name counter

**Given** a Source named `test` already stored
**When** the names `test`, `test [3]`, `bob`, `test` and `test` are created in that order
**Then** the stored names are `test [2]`, `test [3]`, `bob`, `test [4]` and `test [5]`.

### LEAD-AC-469 Delivered attribution records that may not be deleted

**When** the delivered Source named `Referral` is deleted
**Then** the deletion is refused with `You cannot delete the 'Referral' UTM source record.`
**And when** one of the six delivered Media is deleted
**Then** the deletion is refused with `Oops, you can't delete the Medium '<name>'. Doing so would be
like tearing down a load-bearing wall — not the best idea.`

### LEAD-AC-470 A recurring revenue needs a plan

**Given** a form open on an opportunity, for a user in the recurring revenue group
**When** the user types a recurring revenue of `1 200.00` and leaves the plan empty
**Then** the record cannot be saved until a plan is chosen
**And when** the plan `Yearly` is chosen
**Then** the monthly recurring revenue reads `100.00` and, at a probability of twenty-five, the
prorated monthly recurring revenue reads `25.00` and the prorated recurring revenue reads `300.00`.

### LEAD-AC-471 A blacklisted telephone number is unique and reactivated rather than duplicated

**Given** the number `+12025550122` already on the telephone blacklist and then removed, which
archives the record
**When** the same number is blacklisted again
**Then** the archived record is reactivated and no second record is created
**And when** a second record is created for a number that already exists and is active
**Then** the existing record is returned
**And when** a value that cannot be sanitised is submitted
**Then** it is refused with the parsing error followed by `Please correct the number and try again.`

### LEAD-AC-472 The lead generation credit estimate

**Given** a request for twenty-five leads targeting companies and their contacts, with three
contacts per company
**When** the estimate is read
**Then** the company credits read `25`, the contact credits read `75` and the total reads `100`
**And given** the same request targeting companies alone
**Then** the total reads `25`.

### LEAD-AC-473 The stage search helper

**Given** the stages `New` (no team, sequence 1, not folded), `Europe qualified` (team `Europe`,
sequence 2, not folded), `Asia qualified` (team `Asia`, sequence 2, not folded) and `Archived` (no
team, sequence 90, folded)
**When** a Lead of the team `Europe` looks for its first non-folded stage
**Then** the answer is `New`
**And when** a Lead with no team looks for it
**Then** only `New` and `Archived` are candidates and the answer is `New`.

---

## 19. Reconciliation notes

| Subject | The two statements | Resolution |
|---|---|---|
| Which worked examples are mandatory | One description carried a set of mandated worked examples inside its calculation and workflow documents; the other carried a numbered scenario catalogue. | Both are here. Sections 1 to 17 are the numbered catalogue; section 18 restates the mandated worked examples of the other description as scenarios, keeping its numbers so that neither set of figures is lost. |
| The assignment example | One description used capacities of ten, fifteen and five, which give daily quotas of zero, one and zero; the other used ninety, sixty and thirty, which give three, two and one. | Both are scenarios — `LEAD-AC-197` and `LEAD-AC-462` — because together they show that a capacity is a monthly figure and that the rounding of the daily quota is what decides who is eligible. |
| The merge example | The two descriptions merged different pairs of records. | Both are scenarios — `LEAD-AC-164` and `LEAD-AC-461` — one where an opportunity wins on the second key of the confidence order, one where two leads are separated by their stage sequence. |
| The probability example | Both descriptions computed the same deal and reached `74.30`. | Kept once, as `LEAD-AC-234`, with the frequency table stated in full in [predictive-lead-scoring.md](predictive-lead-scoring.md). |
| Scenario identifiers | Only one description numbered its scenarios. | Its scheme is kept and extended; the scenarios added in section 18 continue the same numbering. |
