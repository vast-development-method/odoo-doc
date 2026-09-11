# Workflows

This file specifies the end-to-end operational sequences of the Customer Relationship Management
domain. Each workflow names the actor, the preconditions, the numbered steps, the records created
or updated at each step, the postconditions and the failure conditions.

Entities are named in full on first use with their transport and storage names: Lead (`crm.lead`,
table `crm_lead`), Stage (`crm.stage`, table `crm_stage`), Sales Team (`crm.team`, table
`crm_team`), Sales Team Member (`crm.team.member`, table `crm_team_member`), Contact
(`res.partner`, table `res_partner`), Sales Order (`sale.order`, table `sale_order`), Lost Reason
(`crm.lost.reason`, table `crm_lost_reason`).

Cross-references: the arithmetic behind these steps is in
[calculations.md](calculations.md); the validations and messages are in
[business-rules.md](business-rules.md); the states are in
[state-machines.md](state-machines.md).

---

## 1. Actors

| Actor | Group | What they may do in this domain |
|---|---|---|
| Salesperson (own documents) | *User: Own Documents Only* | Create, read and modify leads that are theirs or unassigned; create Contacts; run the conversion, merge and loss wizards; **may not delete** a lead. |
| Salesperson (all documents) | *User: All Documents* | Everything above, on every lead. |
| Sales administrator | *Administrator* | Everything above, plus delete leads, configure stages, lost reasons, recurring plans, teams and memberships, and trigger the assignment run. |
| System administrator | *Settings* | Everything, plus the probability rebuild wizard and the scoring configuration. |
| Portal partner | portal, with a partner level | Read and act on the opportunities assigned to them through the partner portal. |
| Anonymous website visitor | public | Submit the public contact form, converse with the live chat script. |
| The system | — | The scheduled allocation job, the scheduled probability rebuild, the scheduled enrichment job, the incoming electronic mail gateway. |

---

## 2. Capturing a lead

### 2.1 By hand in the pipeline

**Actor**: salesperson. **Precondition**: none.

1. The user opens the pipeline (opportunities) or the leads list and presses the create button.
2. The quick-create form asks for the title, the Contact (or the company name and contact name),
   the electronic mail address, the telephone number and the expected revenue.
3. On save a Lead is created. The defaults apply: the type follows the group *Show Lead Menu*;
   the salesperson is the creating user; the team is derived from the salesperson through the
   default-team selection; the company is derived from the team, the salesperson or the Contact;
   the stage is the first non-folded stage available to the team; the priority is Low; the active
   flag is true.
4. If the record is created directly in a stage flagged as won and has no closed date, the closed
   date is stamped with the current instant.
5. If the record was created with a customer company but no Contact, and the electronic mail
   address or telephone number of the lead differs from that company's, only the company **name**
   is copied onto the lead; otherwise the company itself becomes the Contact.
6. The outcome bookkeeping runs: a record created directly as won or as lost immediately adjusts
   the frequency table.
7. A creation message is posted on the record: "A new lead has been created for the team
   "*the team*"." or, with no team, "A new lead has been created and is not assigned to any team."

**Postcondition**: one Lead exists, visible in the pipeline column of its stage.

### 2.2 By electronic mail to a team alias

**Actor**: anyone who can send electronic mail to the alias; **effective actor**: the system.

**Precondition**: the Sales Team has an alias name and an alias domain, and the alias contact
policy allows the sender.

1. The incoming gateway receives a message addressed to the team's alias and finds no existing
   record to attach it to.
2. A Lead is created from the message with these defaults, which the alias defaults may override:
   - the title is the message subject, or the text "No Subject" when there is none;
   - the electronic mail address is the message sender;
   - the Contact is the resolved author of the message, when one was resolved;
   - the priority is taken from the message priority when the message carries one that matches a
     valid priority value.
   The alias defaults additionally set the type and the team.
3. The salesperson default is explicitly **suppressed** for this path, so that the lead is not
   silently attributed to the technical gateway user. The lead is therefore created unassigned.
4. The lead is then offered to the team-leader fallback of section 2.6, with the creation source
   worded "incoming email".
5. The message becomes the first entry of the record's discussion thread, with its attachments.

**Postcondition**: one Lead exists, attached to the team, carrying the original message.

### 2.3 By the public contact form

**Actor**: an anonymous or identified website visitor.

**Precondition**: the website has a contact form configured to create leads, and the Lead entity
is marked as reachable from website forms.

1. The visitor submits the form. The controller extracts the values for the fields the form
   exposes.
2. **Telephone normalisation.** For every telephone field of the Lead, the submitted number is
   reformatted to international notation using a country determined as follows: the country
   submitted in the form if any; otherwise the country of the Contact linked to the visitor record,
   falling back to the company's country; otherwise the country derived from the visitor's network
   location. Failure to parse leaves the number unchanged.
3. **State inference.** When no state was submitted, the state is inferred from the visitor's
   network location when both a country code and a subdivision code are available and resolve to a
   known state.
4. **Input filtering.** Before insertion:
   - if a Contact was determined, the electronic mail address and the telephone submitted in the
     form are **dropped**, so that the form cannot silently overwrite the Contact's details;
   - the medium defaults to the value submitted, else to the record default, else to the medium
     named "website", creating it if necessary;
   - the team defaults to the value submitted, else to the website's default sales team;
   - the salesperson defaults to the value submitted, else to the website's default salesperson;
   - the type becomes `lead` when the resolved team uses leads and `opportunity` otherwise; with no
     resolved team, the group *Show Lead Menu* decides.
5. **Visitor linkage.** A visitor record is created if absent. If the submitted address normalises
   to the same value as the visitor's Contact's address, the visitor's Contact is attached to the
   lead — but only when the telephone numbers do not conflict: either the form or the Contact has
   no number, or the two formatted numbers are equal.
6. The company defaults to the website's company and the language to the request language when not
   supplied.
7. The Lead is created. The visitor is linked to it; when the visitor had no lead and no Contact,
   the visitor's display name becomes the lead's contact name.

**Postcondition**: one Lead exists, linked to a website visitor, attributed to a campaign, source
and medium taken from the visitor's tracking cookies.

### 2.4 By a live chat conversation

Two paths exist.

#### 2.4.1 The operator command

**Actor**: an operator who is a salesperson.

1. During a conversation the operator types the command `/lead` followed by a title.
2. With no title, the operator is shown a private hint explaining the syntax.
3. With a title, a Lead is created:
   - the originating channel is the conversation;
   - the title is the plain text of the typed title;
   - the Contact is the first non-public external participant of the conversation, unless any
     participant is the public user, in which case no Contact is set at all;
   - the salesperson and the team are explicitly empty;
   - the notes hold the conversation transcript;
   - "referred by" is the operator's name;
   - the source is the delivered source record named "Livechat".
4. The operator is shown a private message "Created a new lead: *a link to the lead*".

#### 2.4.2 The chat script step

**Actor**: the script, acting for a visitor.

**Precondition**: a script step of type "Create Lead" or "Create Lead and Forward" is reached.

1. The step gathers the customer values collected earlier in the script: the electronic mail
   address, the telephone number and the transcript. The visitor's Contact is updated with the
   address and the number when the visitor is not the public user; no Contact is created.
2. The lead values are prepared:
   - the title is the first free-text answer the visitor gave, truncated to one hundred
     characters; failing that, "*the script title*'s New Lead";
   - the notes are the collected description followed by the conversation transcript;
   - the originating channel is the conversation;
   - the source is the script's source;
   - the team is the step's configured team, **cleared** when the acting user's company and the
     team's company are both set and differ;
   - the salesperson is explicitly empty;
   - the type follows the team's usage flags when a team remains.
3. For a public visitor the electronic mail address and telephone are written on the lead; for an
   identified user the Contact is written instead.
4. The Lead is created and offered to the team-leader fallback of section 2.6 with the creation
   source worded "livechat discussion".
5. **For the "Create Lead and Forward" variant only**, the conversation is additionally handed to
   a human operator chosen among the salespeople who could receive the lead:
   1. Determine the candidate teams: the lead's team if it has one; otherwise every team that is
      not opted out of assignment, that uses leads or opportunities, that has a non-zero capacity,
      and whose assignment domain the lead matches.
   2. If the acting user's Contact has a company, keep only the teams with no company or with that
      company.
   3. Collect the candidate users: the members of those teams that are not paused, whose effective
      quota is strictly positive, and whose assignment domain the lead matches.
   4. Ask the live chat channel which of those users are actually available.
   5. Forward the conversation to one of them. If the conversation's operator actually changed,
      write that user as the lead's salesperson and write as the lead's team the candidate team
      that has that user as a member, then notify the operator with "Created a new lead: *a link
      to the lead*".

### 2.5 By the electronic mail client plug-in

**Actor**: a salesperson working inside their electronic mail client.

1. The plug-in authenticates against the dedicated route family and asks for the leads already
   linked to the Contact of the correspondent.
2. The user may create a Lead from the correspondent, which produces a record carrying the
   correspondent's name, address and company data.
3. The user may log the body of the current message onto an existing lead, which posts it as a
   message on that lead's discussion thread.

### 2.6 The team-leader fallback

**Applies to**: leads created by the incoming electronic mail gateway and by live chat.

1. If rule-based assignment is **enabled** (the configuration parameter `crm.lead.auto.assignment`
   is set), do nothing: the assignment run will handle the lead.
2. Otherwise, and only when the lead has a team, group the unassigned leads by team and, for each
   team that has a leader, write that leader as the salesperson.
3. Log on each such lead: "This new lead created by *the creation source* was automatically
   assigned to team leader *the leader's name*".

### 2.7 By the lead generation service

**Actor**: a salesperson composing a Lead Generation Request.

1. The user fills a request: how many leads, whether to target companies alone or companies and
   their contacts, the type to produce, the team, the salesperson, the tags, and the criteria
   (countries, states, company size range, industries, and for contacts either a preferred role
   with other roles, or a seniority).
2. The form clamps the numbers: leads between one and two hundred; contacts between one and five;
   the minimum size at least one and at most the maximum; the maximum at least the minimum.
3. On submit, if the request number is still the placeholder it is replaced by the next value of
   the numbering series.
4. The payload is assembled and sent to the external service together with the account token, the
   database identifier, the database version and language, the company's country code, and the
   list of service company identifiers already present in the database so that the same company is
   not returned twice.
5. Outcomes:
   - **credit error**: the error type becomes "Insufficient Credits" and the status becomes
     `error`; nothing is created;
   - **no data**: the error type becomes "No Result"; the status is unchanged; nothing is created;
   - **data**: one Lead per returned company is created, and the status becomes `done`.
6. Each created Lead receives: the requested type, team, tags and salesperson; the service company
   identifier; the company name as both title and company name (falling back to the domain when the
   name is absent); the first returned electronic mail address; the telephone number; the website
   built from the domain; the street, city, postal code, country and state resolved from the codes.
   When contacts were requested, the first returned contact overwrites the contact name, the
   electronic mail address and the job position.
7. A note is posted on each Lead rendering the company data returned by the service.
8. The view switches to the produced leads, filtered on the requested type.

**Failure condition**: any transport error aborts with "Your request could not be executed: *the
error*".

---

## 3. Enriching a lead

**Actor**: the scheduled job, or a salesperson pressing the enrichment button.

**Precondition for the button**: the lead is active, has an electronic mail address whose quality
is not `incorrect`, has not already been enriched, has no service company identifier, and its
probability is not one hundred. The setting must be "on demand".

**Precondition for the job**: the setting is "automatic", which is expressed by the enrichment
scheduled job being active. Creating any Lead triggers the job.

1. The job selects the leads that are active, not yet enriched, with an electronic mail address,
   without a service company identifier, whose probability is below one hundred or empty, and
   created within the last twenty-four hours (the delay is a parameter of the job).
2. The selection is processed in batches of fifty, each batch locked for update; a batch that
   cannot be locked reschedules the job five minutes later.
3. For each lead in a batch:
   - skip it when its probability is one hundred or it is already enriched;
   - skip it when it has no electronic mail address;
   - if the address does not normalise, mark it enriched and post the note "no electronic mail
     address" variant; continue;
   - if the address's domain belongs to a known free public provider, mark it enriched and post
     the "not found" note; continue;
   - otherwise record the pair (lead, domain) for the request.
4. Send the batch of domains to the enrichment service.
   - **insufficient credit**: notify the requester (only when the run is interactive) with the
     title "Not enough credits for Lead Enrichment" and stop the whole run — there is no point
     continuing;
   - **other error**: notify the requester with "An error occurred during lead enrichment" and skip
     the batch;
   - **success**: notify the requester with "The leads/opportunities have successfully been
     enriched".
5. Apply the response. For each lead that received data:
   - mark it enriched;
   - fill, **only where the lead is currently empty**, the company name, the service company
     identifier, the street, the city and the postal code from the service's name, identifier,
     location, city and postal code;
   - fill the telephone from the first returned number when the lead has none;
   - fill the country from the returned country code when the lead has none, and then fill the
     state from the returned state code when the lead has none and a country is known;
   - post a note rendering the company data with the heading "Lead enriched based on email
     address".
   For each lead that received nothing, mark it enriched and post the "not found" note.
6. Each batch is committed separately so that a later failure does not lose the credits already
   spent.

**Postcondition**: every processed lead carries the enrichment flag, whether or not data was
found, so that it is never asked for twice.

---

## 4. Qualifying and working a lead

**Actor**: salesperson.

1. **Complete the data.** The salesperson fills the company name, the contact name, the job
   position, the electronic mail address, the telephone number, the address, the language, the
   website and the tags. Each change to the electronic mail address or the telephone number may
   also write the linked Contact; the form shows a warning ribbon when it will.
2. **Set the expectation.** The expected revenue, and where the recurring revenue capability is
   granted, the recurring revenue and its plan. The prorated figures follow automatically.
3. **Set the priority.** One of Low, Medium, High, Very High. Priority is the primary sort key of
   the pipeline.
4. **Advance the stage.** Dragging the card or clicking the status bar writes the stage, stamps
   the last stage update, accumulates the duration in the previous stage and recomputes the
   probability.
5. **Plan the next step.** Activities are scheduled on the record; the pipeline can be sorted by
   the current user's earliest activity deadline.
6. **Schedule a meeting.** The meeting button opens the calendar pre-filled with the opportunity,
   the Contact, the team and the title, and positioned on the most relevant period (see
   `interfaces.md`). Creating the meeting posts a note on the opportunity.
7. **Watch for duplicates.** The duplicate counter appears as soon as at least one other record
   shares the electronic mail domain, the sanitised telephone number or the commercial entity.

---

## 5. Converting one lead into an opportunity

**Actor**: salesperson. **Precondition**: the record is of type `lead` and is active.

1. The user presses "Convert to Opportunity". The wizard opens on the lead.
   - **Failure condition**: if the lead's probability is exactly 100, the wizard refuses to open
     with "Closed/Dead leads cannot be converted into opportunities."
2. The wizard computes its defaults:
   - it looks for a matching Contact (the lead's own Contact, or a Contact found by the lead's
     electronic mail address without creating one). If found, the related-customer action is "Link
     to an existing customer" and that Contact is proposed; otherwise the action is "Create a new
     customer";
   - it runs the merge-oriented duplicate search with lost records **included**, using the chosen
     Contact and the Contact's address (falling back to the lead's own address);
   - if two or more duplicates were found the conversion action defaults to "Merge with existing
     opportunities", otherwise to "Convert to opportunity";
   - the salesperson defaults to the lead's salesperson; the team follows from the salesperson.
3. The user confirms.

### 5.1 Branch A — convert

1. The set of records to convert is the active records of the calling context (normally the single
   lead).
2. For each record that is active, handle the customer:
   - when the related-customer action is "Link to an existing customer", the chosen Contact (or the
     record's own) is forced onto the record and **no** Contact is created;
   - when it is "Create a new customer", the chosen Contact is forced onto the record if one was
     given, and otherwise a Contact is created by the customer-creation routine, using the wizard's
     chosen company as the parent when one was chosen. The new Contact's salesperson is the
     wizard's salesperson.
3. Each record is converted: the type becomes `opportunity`, the conversion date is stamped, the
   Contact is written when it changed, and a stage is chosen when the record had none.
4. The salesperson list is applied: when "Force assignment" is off, only the records that have no
   salesperson receive one. The single salesperson is written on every record; the team is written
   too.
5. The view redirects to the resulting opportunity in form mode, with the type as a filter and as a
   default.

### 5.2 Branch B — merge

1. The set to merge is the detected duplicates plus the lead itself.
2. The merge algorithm runs with automatic deletion **disabled**.
3. The surviving record is unarchived (a lost duplicate can therefore be revived by the merge).
4. If the survivor is still of type `lead`, it goes through the same convert branch as above, with
   the wizard's salesperson and team.
5. If the survivor is already an opportunity, its salesperson and team are overwritten when it has
   no salesperson or when "Force assignment" is on.
6. The wizard repoints itself at the survivor so that deleting the others does not cascade into it.
7. Every record of the merge set other than the survivor is deleted with elevated rights.
8. The view redirects to the survivor.

### 5.3 Worked example (mandated) — a lead converted with a newly created party

**Given.** A Lead with:

| Field | Value |
|---|---|
| type | `lead` |
| title | "Spare parts for the Antwerp line" |
| company name | "Northwind Parts" |
| contact name | "Anna Devries" |
| job position | "Procurement Manager" |
| electronic mail address | `anna@northwind-parts.example` |
| telephone | `+32 3 555 22 10` |
| street | "Rue Haute 12" |
| postal code | "1000" |
| city | "Brussels" |
| country | Belgium |
| language | Dutch (active) |
| notes | "Called on Monday." |
| Contact | *(none)* |
| salesperson | Ines |
| team | Direct Sales |
| stage | Qualified |
| expected revenue | 18 000 |

No Contact anywhere in the database carries the address `anna@northwind-parts.example`.

**When** the salesperson opens the conversion wizard and confirms with the defaults.

**Then**:

1. The wizard proposes "Create a new customer", because no matching Contact was found.
2. The duplicate search returns only this record, so the conversion action is "Convert to
   opportunity".
3. The customer-creation routine runs. The individual's name is "Anna Devries". The lead has a
   company name, so **a Contact flagged as a company is created first**:

   | Contact field | Value |
   |---|---|
   | name | "Northwind Parts" |
   | is a company | yes |
   | parent | *(none)* |
   | salesperson | Ines |
   | notes | "Called on Monday." |
   | telephone | `+32 3 555 22 10` |
   | electronic mail address | `anna@northwind-parts.example` |
   | job position | "Procurement Manager" |
   | street, postal code, city, country | "Rue Haute 12", "1000", "Brussels", Belgium |
   | free-text company name | *(empty — the record is itself a company)* |
   | type | contact |
   | language | Dutch |

4. **A second Contact is then created** for the individual, with the company as its parent:

   | Contact field | Value |
   |---|---|
   | name | "Anna Devries" |
   | is a company | no |
   | parent | "Northwind Parts" |
   | salesperson | Ines |
   | notes | "Called on Monday." |
   | telephone | `+32 3 555 22 10` |
   | electronic mail address | `anna@northwind-parts.example` |
   | job position | "Procurement Manager" |
   | street, postal code, city, country | "Rue Haute 12", "1000", "Brussels", Belgium |
   | free-text company name | *(empty — the record has a parent)* |
   | type | contact |
   | language | Dutch |

5. The **individual** Contact is written on the lead.
6. The lead's type becomes `opportunity` and its conversion date is stamped.
7. The lead's stage is unchanged, because it already had one.
8. The salesperson Ines and the team Direct Sales are written (already their values, so no change
   in practice).
9. Because the Contact is now set, the computed contact fields re-derive: the contact name stays
   "Anna Devries" (the Contact is not a company), the company name becomes the parent's name
   "Northwind Parts", the job position and the website follow the Contact, the language follows the
   Contact, and the whole address block is taken from the Contact — which holds the same values.
10. The user lands on the opportunity form.

**Records created: two Contacts. Records updated: one Lead.**

---

## 6. Converting many leads at once

**Actor**: salesperson. **Precondition**: several leads selected in a list.

1. The user chooses the mass conversion action. The wizard defaults the selected leads into its
   "active leads" field, defaults the related-customer action to "Use existing partner or create",
   defaults deduplication to on and force assignment to off, and offers a list of salespeople.
2. The wizard computes the duplicate set: for each selected lead, the merge-oriented duplicate
   search with lost records **excluded**; the lead is listed as duplicated when more than one
   record comes back.
3. The user confirms.
4. **Deduplication pass** (only when deduplication is on):
   1. Walk the selected leads in order. Skip a lead already consumed by a previous merge.
   2. Run the duplicate search for it. When more than one record comes back, merge them (with
      automatic deletion on), record every input as consumed and record the survivor.
   3. Rebuild the list of records to convert: first the originally selected leads that were not
      consumed, keeping the original order; then the merge survivors that are not already in that
      list.
5. **Conversion pass**: the same convert branch as section 5.1, with two differences:
   - the customer handling is per lead: the related-customer action "Use existing partner or
     create" is translated, for each lead, into "force the Contact found for that lead, and create
     one when none was found";
   - the salespeople are applied round-robin over the converted records (see `calculations.md`),
     and, because force assignment is off by default, only the records that have no salesperson
     receive one.
6. The view redirects to the first converted record.

---

## 7. Merging duplicates

**Actor**: salesperson. **Precondition**: two or more records selected.

1. The user chooses the merge action from the list. The wizard defaults its record set to the
   selected records **excluding** those whose outcome is won, with archived records included.
2. The user may set a salesperson, which proposes a team.
3. On confirmation the merge algorithm runs with the wizard's salesperson and team as overrides
   and with automatic deletion **on**.
4. The view redirects to the surviving record.

**Failure conditions**: fewer than two records — "Select at least two Leads/Opportunities from the
list to merge them."; more than five records — "To prevent data loss, Leads and Opportunities can
only be merged by groups of 5."

### 7.1 Worked example (mandated) — two leads sharing an address

The complete field-by-field outcome of merging two leads that share the postal address and the
electronic mail domain is given in [calculations.md](calculations.md), section 7.6. In summary:
the record whose stage sequence is higher becomes the survivor; the address block is taken **whole**
from the record that has the most non-empty address fields, including that record's empty state
field; every other single-valued field takes the first non-empty value in confidence order; the
tags are unioned; the priority is the maximum; the notes are concatenated; and the second record is
deleted after its messages, activities, attachments and meetings have been repointed.

---

## 8. Rule-based allocation and assignment

### 8.1 Enabling it

**Actor**: system administrator.

1. In the settings, switch on "Rule-Based Assignment". This writes the configuration parameter
   `crm.lead.auto.assignment`.
2. Choose "Manually" or "Repeatedly". "Repeatedly" activates the lead-assignment scheduled job and
   sets its interval unit, interval number and next execution instant from the settings.
   "Manually" deactivates the job but keeps the feature, so that the action button appears on the
   team form.
   - **Failure conditions** when editing the interval: a number of zero or below gives "Repeat
     frequency should be positive."; a number of one hundred or above gives "Invalid repeat
     frequency. Consider changing frequency type instead of using large numbers."
3. Give each Sales Team Member a capacity, and optionally an assignment domain and a preferred
   assignment domain. Optionally give the team an assignment domain and mark teams that should be
   skipped.

### 8.2 Running it

**Actor**: the scheduled job, or a sales administrator pressing "Assign Leads" on a team.

**Precondition**: the actor is a sales administrator or the system. Otherwise the run is refused
with "Lead/Opportunities automatic assignment is limited to managers or administrators".

| Aspect | Scheduled job | Manual action |
|---|---|---|
| teams considered | every team that uses leads or opportunities and is not opted out | the teams the button was pressed on |
| daily quota | not forced (the last twenty-four hours are deducted) | **forced** (the full daily quota is available) |
| creation window | the last seven days | unlimited |

1. **Allocation.** Leads with no team and no salesperson are allocated to teams by the weighted
   random draw of `calculations.md`. Duplicates encountered during allocation are merged, and the
   losing records are deleted in bundles.
2. **Assignment.** The team's leads that have no salesperson and no assignment date are assigned to
   members by the preference-first round robin of `calculations.md`. **Each assignment also
   converts the lead into an opportunity**, using the lead's existing Contact; no Contact is
   created by this path.
3. **Reporting.** The manual action posts a note on each team and shows a success notification
   built from these parts, in order:
   - when duplicates were merged: "*n* duplicates leads have been merged."
   - when nothing at all was allocated or assigned:
     - one team with no capacity: "No allocated leads to *the team* team because it has no
       capacity. Add capacity to its salespersons."
     - one team with capacity: "No allocated leads to *the team* team and its salespersons because
       no unassigned lead matches its domain."
     - several teams: "No allocated leads to any team or salesperson. Check your Sales Teams and
       Salespersons configuration as well as unassigned leads."
   - when nothing was allocated but something was assigned:
     - one team: "No new lead allocated to *the team* team because no unassigned lead matches its
       domain."
     - several teams: "No new lead allocated to the teams because no lead match their domains."
   - when something was allocated:
     - one team: "*n* leads allocated to *the team* team."
     - several teams: "*n* leads allocated among *m* teams."
   - when nothing was assigned but something was allocated: "No lead assigned to salespersons
     because no unassigned lead matches their domains."
   - when something was assigned: "*n* leads assigned among *m* salespersons."
   The note on the team is prefixed by "Lead Assignment requested by *the user's name*".

### 8.3 Worked example (mandated) — thirty leads, three members with capacities ten, fifteen and five

The arithmetic is given in [calculations.md](calculations.md), section 8.5. Operationally:

**Given** one team, three members with capacities 10, 15 and 5, none paused, none with an
assignment domain, no leads assigned in the last twenty-four hours, and thirty leads already
belonging to the team with no salesperson and no assignment date.

**When** a sales administrator presses "Assign Leads" on that team.

**Then**:

1. Allocation finds no lead to allocate, because all thirty already have a team.
2. The daily quotas are computed as 0, 1 and 0 respectively.
3. Only the second member is eligible. The thirty leads are sorted by probability descending. The
   first one is assigned to the second member and converted into an opportunity. The member's
   quota drops to zero and the member leaves the rotation.
4. No eligible member remains, so the remaining twenty-nine leads stay unassigned.
5. The notification reads "1 leads assigned among 1 salespersons."

To distribute all thirty in one run in the ratio ten to fifteen to five, the capacities must be set
to 300, 450 and 150, which give daily quotas of 10, 15 and 5. The rotation then produces ten
assignments to the first member, fifteen to the second and five to the third.

---

## 9. Selling from an opportunity

**Actor**: salesperson. **Precondition**: the sales capability is installed and the record is an
opportunity.

1. The user presses "New Quotation".
2. **If the opportunity has no Contact**, the quotation contact wizard opens first and asks
   whether to create a new customer from the lead information, to link an existing one, or not to
   link a customer at all.
   - **Failure condition**: opening the wizard from anything other than a lead gives "You can only
     apply this action from a lead."
   - "Create a new customer" runs the customer-creation routine and writes the result on the
     opportunity; "Link to an existing customer" forces the chosen Contact; "Do not link" leaves
     the opportunity without one.
3. A new quotation form opens, pre-filled from the opportunity: the opportunity reference, the
   Contact, the campaign, the medium, the source, the origin (the opportunity's title), the
   company, the tags, and — when set — the team and the salesperson.
4. The salesperson completes and confirms the quotation. Confirmation belongs to the Sales domain;
   its only effect here is the expected-revenue feedback of `calculations.md`, section 2.6.
5. The opportunity's counters update: the number of quotations counts the orders in state draft or
   sent; the number of sales orders counts those beyond that; the sum of orders totals the
   confirmed ones converted into the company currency.

---

## 10. Winning a deal

**Actor**: salesperson. **Precondition**: the record is an opportunity, is active, and its outcome
is not already won.

1. The user presses "Won" in the form header, or moves the card into a stage flagged as won.
2. The win routine runs:
   1. Unarchive the record if it was archived.
   2. Find every stage flagged as won that is available to the record (no team restriction or the
      record's team among its teams), ordered by sequence.
   3. Choose the **first won stage whose sequence is strictly greater** than the record's current
      stage sequence. If there is none, choose the **last won stage whose sequence is less than or
      equal to** the current one. If there is still none, use the whole set (which in a
      well-formed configuration is a single stage).
      This handles a pipeline where won stages are interleaved with ordinary ones: a record in
      stage *y* moves to the won stage that follows *y*, not to an earlier one.
   4. Group the records by their target won stage and write, per group, the stage and probability
      one hundred.
3. The write rules fire: the record is forced active, the probability and the automated probability
   become one hundred, the closed date is stamped (unless the record was already in a won stage),
   the last stage update is stamped, the outcome becomes `won` and the frequency table receives a
   won increment for every cell of the record with the "all stages" expansion.
4. A tracked message with the *Opportunity Won* subtype is posted.
5. When the action was pressed from the form, the celebration message of `calculations.md`,
   section 11, is evaluated and, if one applies, returned as an animation with the team leader's
   portrait.

### 10.1 Worked example (mandated) — an opportunity won and its quotation confirmed

**Given.** An opportunity "Antwerp line — spare parts" with expected revenue 18 000.00, stage
Proposition (sequence 3), probability 62.40 computed automatically, salesperson Ines, team Direct
Sales, company currency the company's own. A quotation of 21 400.00 untaxed exists in state
"sent", in the company currency, referencing this opportunity.

**When** the salesperson confirms the quotation and then presses "Won".

**Then**, in order:

1. **Confirmation** moves the order out of the draft and sent states. The opportunity's counters
   change: the number of quotations goes from 1 to 0, the number of sales orders goes from 0 to 1,
   and the sum of orders becomes 21 400.00.
2. The expected-revenue feedback fires: 18 000.00 is strictly less than 21 400.00 and the currency
   matches, so the expected revenue becomes **21 400.00** and the tracked change carries the log
   message "Expected revenue has been updated based on the linked Sales Orders."
3. The prorated revenue immediately follows: round to two decimals of 21 400.00 × 62.40 ÷ 100 =
   **13 353.60**.
4. **Pressing "Won"** finds the won stages available to Direct Sales. With the delivered
   configuration there is exactly one, "Won", of sequence 70, which is strictly greater than 3, so
   it is chosen.
5. The write sets the stage to Won and the probability to 100; the write rules force the active
   flag true and the automated probability to 100 and stamp the closed date.
6. The prorated revenue becomes round to two decimals of 21 400.00 × 100 ÷ 100 = **21 400.00**.
7. The outcome becomes `won`. The frequency table of team Direct Sales receives, for every cell of
   the opportunity, a won increment: the won counters of New, Qualified, Proposition **and** Won
   each rise by one, and so do the cells for the country, the state, the source, the language, the
   electronic mail quality, the telephone quality and each tag.
8. Days to close is computed from the creation instant to the closed instant.
9. A message with the *Opportunity Won* subtype is posted, and the celebration evaluation runs.

**Records updated: one Lead, one Sales Order (by the Sales domain), and the frequency cells of the
team.** No journal entry is produced by this domain; see [accounting-effects.md](accounting-effects.md).

---

## 11. Losing a deal

**Actor**: salesperson. **Precondition**: the outcome is `pending` and the record is active.

1. The user presses "Lost" in the form header, or picks "Mark Lost" from a list selection.
2. The loss wizard opens with the selected records. The user may choose a Lost Reason and may type
   a closing note.
3. On confirmation:
   1. If the closing note is not empty, it is registered as the log message that will accompany the
      tracked change, rendered as a block whose first line is "Lost Comment:" followed by the note.
   2. The records are archived.
   3. The records are written with the chosen lost reason, probability zero and automated
      probability zero.
4. The write rules fire: the closed date is stamped because the active flag is being set to false,
   the outcome becomes `lost`, and the frequency table receives a lost increment for every cell of
   each record with the "stages up to and including the current one" expansion.
5. A tracked message with the *Opportunity Lost* subtype is posted, carrying the closing note when
   one was given.

### 11.1 Worked example (mandated) — one lost with a reason

**Given.** An opportunity "Ghent depot — conveyor belts" with expected revenue 9 500.00,
probability 41.00, stage Qualified (sequence 2), team Direct Sales, country Belgium, source Search
Engine, electronic mail quality `correct`, telephone quality `correct`, one tag "Service".

**When** the salesperson presses "Lost", picks the reason "Too expensive" and types the closing
note "Competitor quoted 20 per cent below."

**Then**:

1. The closing note is registered as the log message.
2. The record is archived: the active flag becomes false.
3. The lost reason becomes "Too expensive"; the probability and the automated probability become
   zero.
4. The closed date is stamped with the current instant; days to close is computed.
5. The prorated revenue becomes round to two decimals of 9 500.00 × 0 ÷ 100 = **0.00**.
6. The outcome becomes `lost`.
7. The frequency table of team Direct Sales receives a lost increment of one on: the stage cells of
   New and Qualified (sequences 1 and 2, both at or below the record's stage sequence) but **not**
   Proposition nor Won; the country cell for Belgium; the source cell for Search Engine; the
   electronic mail quality cell for `correct`; the telephone quality cell for `correct`; the
   language cell if a language is set; the state cell if a state is set; and, if the tag "Service"
   already has at least fifty closed deals behind it, that tag's cell too — the threshold applies
   only when the table is *read*, so the increment itself always happens.
8. A message with the *Opportunity Lost* subtype is posted, showing the tracked change of the lost
   reason and, below it, the block "Lost Comment: Competitor quoted 20 per cent below."

**Records updated: one Lead and the frequency cells of the team.**

---

## 12. Restoring a lost deal

**Actor**: salesperson. **Precondition**: the outcome is `lost`.

1. The user presses "Restore".
2. The record is unarchived. Because it was actually inactive, its lost reason is cleared and its
   probability is recomputed.
3. The manual probability is then set equal to the automated probability, re-attaching the record
   to the automatic computation.
4. The outcome becomes `pending` and the frequency table receives a lost **decrement** for every
   cell of the record.
5. A tracked message with the *Opportunity Restored* subtype is posted.

Plain unarchiving (without "Restore") performs steps 2, 4 and 5 but not step 3: the probability
stays at zero while the automated probability is recomputed, so the record is reported as having a
manual probability.

---

## 13. Realigning a probability

**Actor**: salesperson. **Precondition**: the record's probability differs from its automated
probability.

1. The form shows the probability as manual.
2. The user presses the realignment control.
3. The probability is recomputed and the manual probability is set equal to it.

The explanation tooltip performs the same realignment as a side effect of being opened, and
additionally returns the ranked list of the three most positive and the three most negative
observations.

---

## 14. Rebuilding the probability model

**Actor**: system administrator.

1. Open the probability rebuild wizard from the settings. It shows the current scoring start date
   and the current scoring variables.
2. Change either.
3. Confirm.
   - **Guard**: the action does nothing at all unless the acting user is an administrator.
4. The configuration parameters `crm.pls_fields` and `crm.pls_start_date` are written. Writing
   `crm.pls_fields` reloads the Lead entity definition, because the list of scoring variables is
   part of the dependency list of the probability computation.
5. The rebuild runs immediately: the frequency table is emptied, rebuilt from every won and lost
   lead created on or after the start date, and every pending lead is rescored in batches.
   - **Failure condition**: an actor who may not delete leads gets "You don't have the access
     needed to run this cron."

The same rebuild runs on the schedule of the probability job when that job is active.

---

## 15. Forwarding an opportunity to a reselling partner

**Actor**: sales administrator. **Precondition**: the partner network capability is installed.

1. The user selects opportunities and presses "Assign Partner".
2. Opportunities with no country are skipped and reported in a notification: "There is no country
   set in addresses for *the titles*."
3. For each remaining opportunity, coordinates are obtained if missing, a partner is chosen by the
   widening geographic search with weighted choice, and the partner is written on the record along
   with that partner's salesperson. Opportunities for which no partner was found receive the tag
   reserved for "no partner available".
4. Optionally the user opens the forwarding wizard, which proposes one pairing per opportunity,
   renders a message per pairing and sends it to the partners.
5. The partner opens the opportunity in the partner portal and either declares interest or
   declines; see [state-machines.md](state-machines.md), section 7.

### 15.1 What the partner may do in the portal

| Operation | Effect |
|---|---|
| Declare interest | Posts "I am interested by this lead." plus the optional comment; converts the record to an opportunity. |
| Decline | Posts the corresponding message; unsubscribes the partner's whole commercial hierarchy; adds them to the declined list; clears the assigned partner; optionally adds the "spam" tag. |
| Update the deal | Writes the expected revenue, the probability, the priority and the expected closing date; creates or updates **the acting user's own** activity with the given type, summary and deadline. |
| Update the contact details | Writes only the company name, the telephone, the electronic mail address and the six address fields. Any other field gives "Not allowed to update the following field(s): *the names*." |
| Update the stage | Writes the stage. |
| Create an opportunity | Requires the acting user's Contact or its commercial entity to have a partner level, otherwise access is denied. Requires the contact name, the description and the title, otherwise "All fields are required!". Creates a record with priority High, assigned to the user's commercial entity, tagged as the partner's own opportunity, then realigns the salesperson from the assigned partner and converts it. |

Every one of these operations first checks that the acting portal user's commercial entity is an
ancestor of the record's assigned partner, otherwise "Only users with commercial partner which is a
parent of the assigned partner can edit this lead."

---

## 16. Reorganising teams

### 16.1 Adding a member

**Actor**: sales administrator.

1. Open the team and add a user to the salespeople list, or create a Sales Team Member directly.
2. In single-membership mode every other active membership of that user is archived first.
3. The membership is created with the default capacity of thirty.
4. The user is added to the team's favourites.
5. The team's capacity rises by the member's capacity.
6. The user's main team is recomputed as the team of their **first-created** membership.

**Failure conditions**: a duplicate active membership in single-membership mode gives "You are
trying to create duplicate membership(s). We found that *the pairs* already exist(s)."; a user
without the team's company gives "User '*the user*' is not allowed in the company '*the company*'
of the Sales Team '*the team*'."

### 16.2 Removing a member

1. Remove the user from the salespeople list, or archive the membership.
2. The membership becomes inactive; its history and its counters are retained.
3. The team's capacity drops.

### 16.3 Changing the team's usage flags

1. Toggle "Leads" or "Pipeline".
2. The alias creation values are recomputed and written: the aliased entity is the Lead; the alias
   defaults are rewritten with the resulting type and the team; clearing both flags clears the
   alias name so that the address stops accepting messages.

### 16.4 Deleting a team

1. Before the record is removed, the team's frequency cells are folded into the "no team" bucket by
   the routine of [entities.md](entities.md), section 9.9. Seed-only cells are discarded; matching
   cells are added after rounding each counter half-up to a whole number; missing cells are
   created.
2. The team is deleted; its remaining frequency cells are removed by the cascading reference; leads
   pointing at it have their team cleared.

**Failure condition**: the delivered teams "Website" and "Point of Sale" cannot be deleted —
"Cannot delete default team "*the name*"".

---

## 17. Reporting

### 17.1 Pipeline analysis

**Actor**: salesperson or administrator.

The pipeline itself is the report: the same Lead records presented as a list, a pivot, a graph, a
calendar, a kanban board and a forecast board. The measures available are the expected revenue, the
prorated revenue, the recurring revenue, the expected monthly recurring revenue, the prorated
variants, the probability (averaged), the days to assign, the days to close and the record count.
The dimensions are the stage, the team, the salesperson, the country, the state, the campaign, the
source, the medium, the tags, the lost reason, the priority, the type, the creation date, the
conversion date, the closed date and the expected closing date.

### 17.2 Activity analysis

A read-only view over completed activities logged on leads. One row per completed activity, with
the activity type, its author, its completion date and the lead's dimensions.

### 17.3 Periodic digest

Two figures are contributed to the periodic digest: the number of leads created in the period and
the number of opportunities won in the period, both scoped to the recipient's company. A recipient
who is not a salesperson simply does not receive them.

### 17.4 Recognition goals

When the recognition capability is installed, five goal definitions are contributed, each computed
per salesperson and batched:

| Goal | Computation | Direction |
|---|---|---|
| Total Invoiced | sum of the untaxed subtotal of customer invoices that are not cancelled, dated by invoice date | higher is better |
| New Leads | count of leads and opportunities, dated by creation date | higher is better |
| Time to Qualify a Lead | sum of the days-to-close of records of type `lead`, dated by closed date | **lower** is better |
| Days to Close a Deal | sum of the days-to-assign, dated by assignment date | **lower** is better |
| New Opportunities | count of records of type `opportunity`, dated by assignment date | higher is better |
