# Glossary

Terms used in this domain, with their precise meaning here. Where a term also exists in another domain, the definition states the boundary. Every entry is written in full words.

## A

**Activation level.** A classification of how far a reseller partnership has progressed, independent of the grade. Three levels are shipped: `Fully Operational`, `Ramp-up`, `First Contact`. Used for reporting and for filtering the partner directory.

**Activity.** A planned action on a record, with a type, a summary, a deadline and a responsible user. Activities belong to the messaging domain; this domain exposes the next activity of a Lead and orders lists by the reader's own earliest deadline.

**Activity analysis.** The read-only report built on the messages of Leads that carry an activity type; one row is one completed activity.

**Alias.** The electronic mail address owned by a Sales Team. A message received at that address creates a Lead for the team, and every notification about a Lead of that team carries the alias as its reply address. A team always owns one.

**Allocation.** The first phase of rule-based assignment: attaching an unclaimed Lead to a Sales Team by a weighted random draw proportional to team capacity, merging its duplicates on the way. The second phase is the distribution to members.

**Assigned partner.** The external reseller a Lead has been forwarded to. Writing it stamps the forwarding date; clearing it clears that date.

**Assignment condition.** A search condition, set on a Sales Team or on a Sales Team Member, that a Lead must satisfy before being allocated to that team or given to that member. An empty condition accepts everything.

**Assignment date.** The instant at which a Lead first received a salesperson, or the instant of the most recent change of salesperson. Cleared when the salesperson is removed. It is the basis of the member workload counters and it excludes a record from redistribution.

**Assignment delay.** A number of hours, held in a system parameter, that a Lead must have existed before the allocation may consider it, so that automated processes have time to enrich it first.

**Automated probability.** The value the scoring model computes for a Lead. It is always refreshed by the model; it is a separate field from the probability the salesperson sees.

**Automatic probability.** The state in which the probability of a Lead equals its automated probability to two decimal places, which means the model is free to overwrite the probability. The opposite state is a manual probability.

## B

**Base quota.** The whole number of Leads a member may receive in one day, computed as the monthly capacity divided by thirty and rounded to the nearest whole number with halves moving away from zero.

**Blacklist.** A list of email addresses, and a separate list of telephone numbers, that must not be contacted. Owned by the messaging domain. A Lead exposes whether its address or its number, or the address of its customer, is on a list.

## C

**Campaign, source and medium.** The three campaign-tracking references carried by a Lead: which campaign produced it, from which source, through which delivery method. Filled from the visitor's tracking cookies at creation, but never for a record created by a salesperson.

**Capacity.** The average number of Leads a member can absorb over thirty days. The capacity of a team is the sum of the capacities of its active memberships.

**Celebration message.** A congratulation shown when an opportunity is marked won from its form. One message is chosen from an ordered list of conditions evaluated against the historical statistics of the salesperson and of the team; the animation shows the team leader's portrait when the leader has one.

**Census stage.** The first Stage in sequence order that has no team restriction. Its frequency row holds the total number of won and lost records of a statistical universe, because every won record increments every stage and every lost record increments at least the first one. When no stage is unrestricted there is no census and no probability can be computed.

**Clamping.** Forcing the computed probability of a pending record into the closed interval from 0.01 to 99.99, so that exactly zero and exactly one hundred stay reserved for lost and won.

**Closing date.** The instant at which a Lead reached a probability of one hundred or was archived. Cleared when the record becomes open again. Not to be confused with the expected closing date.

**Commercial entity.** The company at the top of a contact's parent chain, or the contact itself when it is a company with no parent. On a Lead, a derived helper used to pick or create the customer company, and one of the three duplicate criteria.

**Commit bundle.** The number of assignment steps grouped into one committed unit of work, held in a system parameter. It bounds how much work a failure late in a long run can discard.

**Confidence order.** The ordering used by the merge to decide which record survives: not lost first, then opportunity before lead, then the higher stage sequence, then the higher probability, then the higher identifier.

**Contact name.** The name of the natural person behind a Lead, as opposed to the company name.

**Conversion.** The transformation of a Lead into an opportunity. It sets the type, stamps the conversion date, links or creates the customer, and may assign a salesperson and a team.

**Cross-team statistics.** The scoring frequency rows whose team is empty. They are used for a Lead whose team has no statistics of its own, and they receive the statistics of a team that is deleted.

**Lead Generation Rule.** The entity holding one rule that turns identified website traffic into Leads. One record carries the countries, the country subdivisions, the website, the path pattern, the industry, role and seniority filters, the company size bounds and the values written on the created records. This is the only name used for the entity in this domain.

**Reveal View.** The entity holding one visit of one network address to a page matched by a Lead Generation Rule record, waiting to be resolved into a company by the identification service. This is the only name used for the entity in this domain.

## D

**Days to assign.** The absolute number of whole days between the creation of a Lead and its assignment date.

**Days to close.** The absolute number of whole days between the creation of a Lead and its closing date.

**Declined partner.** A reseller that has refused a given Lead. Recorded per Lead so that the geographic search never proposes that reseller for that Lead again.

**Deduplication.** In the assignment algorithm, the step that merges a candidate Lead with its duplicates before attaching it to a team, so that a team never receives several copies of the same commercial interest.

**Display detector.** The duplicate detector that feeds the potential duplicate counter: the union of the matches on the electronic mail domain criterion, on the commercial entity and on the sanitized telephone number, with a discriminating test on two of the three. It answers "which records might a user want to look at alongside this one".

**Duplicate, potential.** A Lead that shares with another Lead its email domain criterion, its sanitized telephone number or its commercial entity. Detected and counted, never prevented.

## E

**Electronic mail.** Written in full throughout this folder. Where a field table or a message reproduces a stored identifier that abbreviates it, the identifier is reproduced exactly and the full name is given beside it.

**Email domain criterion.** The duplicate key derived from the normalized email address of a Lead: the address itself when its domain is a generic provider domain or when there is no domain at all, and otherwise the `@` sign followed by the domain.

**Email quality.** A derived flag, `correct` or `incorrect` or empty, telling whether at least one address contained in the email field passes validation.

**Enrichment.** The operation that sends the domain of the email address of a Lead to an external service and fills the empty company fields of the record with the answer.

**Event lead rule.** A rule that turns event registrations into Leads, per attendee or per order, at creation, at confirmation or at attendance, filtered by event, event category, company and an extra registration condition.

**Expected closing date.** The date on which the salesperson expects the deal to be won. The basis of the forecast.

**Expected revenue.** The untaxed amount the salesperson expects to invoice if the deal is won. An estimate; it never reaches the general ledger.

## F

**Folded stage.** A stage whose pipeline column is collapsed when it holds no record, and which is excluded from the automatic choice of a first stage.

**Forecast.** The presentation of opportunities grouped by expected closing period, measuring the prorated revenue rather than the expected revenue.

**Forwarding.** The act of sending a Lead to an external reseller: writing the assigned partner, writing the salesperson of that partner, subscribing the partner and sending the forwarding email.

**Frequency row.** One cell of the scoring statistics: one variable, one value, one team, a won count and a lost count.

## G

**Grade.** The level of a reseller partnership, for example `Gold`, `Silver` or `Bronze`. It carries a sequence used for ordering, an optional default pricelist and a level weight used by the geographic draw.

## I

**Identification service.** The external service that resolves a network address, or an email domain, into a company description. Used by the Lead Generation Rule, by the Lead Mining Requests and by the enrichment.

**Industry, role and seniority.** The three filter catalogues offered by the company data service. Each carries the external code the service expects, and each has a unique name.

## L

**Lead.** A record of commercial interest. The same entity holds the unqualified lead and the qualified opportunity, distinguished by the type field. In prose, "Lead" with a capital letter names the entity; "lead" in lower case names a record whose type is `lead`.

**Lead generation rule.** A rule that turns identified website traffic into Leads, filtered by website, path pattern, country, state, industry and company size, and carrying the values written on the records it creates.

**Lead mining.** The purchase of new Leads from an external service, filtered by country, subdivision, industry and company size, optionally with contacts.

**Level weight.** A non-negative whole number attached to a grade and copied onto the reseller contact. It is the relative probability of receiving a forwarded Lead. Zero means never.

**Lost.** The state of a Lead that is archived **and** whose probability is zero. Neither condition alone is enough.

**Lost reason.** A catalogue entry naming why a deal was lost. Referenced restrictively: a reason in use cannot be deleted.

## M

**Manual probability.** A probability typed by a salesperson, which the model no longer overwrites because it differs from the automated probability.

**Medium.** See campaign, source and medium.

**Membership.** The link between one user and one Sales Team, carrying the assignment configuration of that user inside that team. Archiving a membership is how a user is moved out of a team without losing the history.

**Membership product.** A product whose service tracking is the membership value. Confirming an order that contains it writes the level the product grants onto the commercial entity of the customer.

**Merge.** The operation that combines several Leads into one survivor, with a deterministic precedence per field, and moves the history, the activities, the attachments, the meetings and the recently active followers onto the survivor.

**Merge detector.** The duplicate detector used by the conversion dialogues and by the allocation: the records matching the normalized electronic mail address or the chosen customer, restricted by the outcome. It answers "which records describe the same deal".

**Monthly recurring revenue.** The recurring revenue divided by the number of months of the recurring plan.

## N

**Naive Bayes classifier.** The statistical model used to compute the automated probability: the observed frequency of each value among won records and among lost records, multiplied under an assumption of independence between the variables, then normalized.

## O

**Opportunity.** A Lead whose type is `opportunity`, that is, a qualified commercial interest that sits in the pipeline.

## P

**Partner directory.** The public pages listing the published resellers, filterable by grade and by country.

**Pending.** The state of a Lead that is neither won nor lost.

**Pipeline.** The ordered set of stages an opportunity moves through, and by extension the kanban screen that shows them as columns.

**Pipeline analysis.** The report built directly on the Lead entity, presenting the same records as a list, a pivot, a graph, a calendar, a kanban board and a map.

**Preference condition.** A second search condition on a membership, describing the Leads the member should receive first. Leads matching both the assignment condition and the preference condition are distributed in a dedicated pass before everything else.

**Priority.** A four-level classification of a Lead: `0` Low, `1` Medium, `2` High, `3` Very High. Part of the default ordering and of the merge precedence.

**Probability.** The chance, expressed as a percentage between zero and one hundred, that the deal will be won. Either automatic, and then equal to the automated probability, or manual.

**Properties.** User-defined extra fields carried by a Lead, whose definition lives on its Sales Team. Changing the team of a Lead therefore changes which properties exist on it.

**Prorated monthly recurring revenue.** The monthly recurring revenue weighted by the probability. It differs from the prorated recurring revenue by exactly the number of months of the plan.

**Prorated revenue.** The expected revenue weighted by the probability, rounded to two decimal places. The measure of the forecast.

## Q

**Quota.** The number of Leads a member may still receive in the current run: the base quota for a manual run, and the base quota minus the leads received in the last twenty-four hours for a scheduled run.

## R

**Recurring plan.** A named number of months over which a recurring revenue amount spreads. Four are shipped: one, twelve, thirty-six and sixty months.

**Recurring revenue.** An amount expected to repeat over the number of months of the recurring plan. Distinct from the expected revenue, which is a one-off amount.

**Requirements.** Free text on a stage, describing the internal criteria for a record to enter it. Shown as a tooltip over the stage name.

**Reseller.** An external partner that receives forwarded Leads, works them on the partner portal, and carries a grade, an activation level, a level weight and geographic coordinates. Also called an affiliate, a member or a partner depending on the company's chosen label.

**Restore.** The operation that brings a lost Lead back into the normal life cycle: unarchive, clear the lost reason, recompute the automated probability and realign the probability on it.

**Reveal view.** One visit of one network address to a page matched by a lead generation rule, waiting to be resolved into a company by the identification service. One row exists per pair of rule and address.

**Rotting.** The state of a pending opportunity that has stayed in the same stage longer than the rotting threshold of that stage.

**Rotting threshold.** A number of days per stage. Zero disables rotting for that stage.

**Round robin.** The distribution rule that serves one member, sends that member to the back of the queue, and drops the member from the queue once its quota reaches zero. A second, simpler round robin deals a fixed list of salespeople over a list of records in strides.

## S

**Sales Team.** A group of salespeople with a leader, an optional company, an email alias, pipeline options, a capacity derived from its members, an assignment condition and a definition of the dynamic properties available on its Leads.

**Sales Team Member.** See membership.

**Sanitized telephone number.** The telephone number reduced to its international, digits-only form. One of the three duplicate criteria.

**Scoring start date.** The date before which closed records are ignored by the scoring model and open records are not recomputed.

**Scoring variable.** A field of the Lead whose values are counted in the frequency table. The stage is always one; the others are configurable.

**Show Lead Menu.** The access group that makes the unqualified stage of the life cycle visible: it shows the leads menu, makes `lead` the default type of a new record and switches the alias defaults of the teams that use leads.

**Show Recurring Revenues Menu.** The access group that makes the recurring revenue amount, the recurring plan and the four derived recurring figures visible, and that adds the recurring plans menu.

**Smoothing.** The offset of `0.1` added to every count when a frequency row is created, and the floor of `0.1` applied when a decrement would reach zero. It prevents a never-observed combination from driving the probability to an extreme.

**Source.** See campaign, source and medium.

**Stage.** An ordered step of the pipeline, shared by every team or restricted to a set of teams, optionally flagged as the won stage, optionally folded, with a rotting threshold, internal requirements and a colour.

**Stage expansion.** The rule that decides which stage rows a closed record contributes to in the frequency table: every stage for a won record, and every stage whose sequence is at or below the record's own for a lost record.

**Suggested recipient.** A contact or an address that the message composer proposes when a message is composed on a Lead, together with the values that would create the contact when the address matches none. Proposing costs nothing: nothing is written until the user confirms the composition.

**Survivor.** The record that remains after a merge, chosen by the confidence order.

## T

**Tag.** A free classification label shared by Leads and by the sales documents. Tag names are unique.

**Team leader.** The user in charge of a Sales Team. Receives the Leads created without a salesperson when rule-based assignment is not activated, and appears with the suffix `(Team Leader)` in the member list of the team.

**Team-leader fallback.** The rule that gives a Lead created without a salesperson by the incoming message gateway or by a live chat step to the leader of its team, and logs a note saying so. It applies only while rule-based assignment is switched off.

**Telephone blacklist.** The list of telephone numbers that must not be contacted, held in international notation. A number is archived rather than deleted when it is taken off the list.

**Text message.** A short message sent to a telephone number rather than to an email address. Owned by the messaging domain; a Lead is a valid target for it, the number being the sanitized form of its telephone field.

**Time in stage.** The number of seconds a record spent in each stage, derived from the tracked stage changes. Used to detect a record that went straight from the first stage to the win.

## U

**Unique-name counter.** The rule that makes a campaign identifier, a source name or a medium name unique by appending a bracketed counter instead of rejecting the input.

## W

**Weighted random draw.** The mechanism that picks one element from a population with a probability proportional to a weight. Used twice in this domain: to pick a team during the allocation, with the capacity as weight, and to pick a reseller during the geographic assignment, with the level weight as weight.

**Won.** The state of a Lead whose probability is one hundred **and** whose stage is flagged as won. Neither condition alone is enough.

**Won stage.** A stage whose won flag is set. Writing such a stage on a record forces the record active with a probability of one hundred.

**Won status.** The derived three-valued state of a Lead: `won`, `lost` or `pending`. Never written directly; it drives the scoring statistics.

---

## Reconciliation notes

| Subject | The two statements | Resolution |
|---|---|---|
| Vocabulary covered | One description defined the vocabulary of the pipeline, the scoring model and the reseller programme; the other added the terms of the allocation, the celebration, the attribution and the acquisition channels. | Every term of both is defined here, in one alphabetical list. |
| The name of the website identification entities | One description called them by a long name beginning with the domain name; the other by their transport name. | They are named Lead Generation Rule and Reveal View throughout this folder, each carrying its transport name on first use in every file. |
| The two duplicate detectors | Only one description separated them. | Both are defined here, as the display detector and the merge detector, because they answer different questions and are used by different operations. |
| Spelling of "electronic mail" | One description wrote the term in full; the other used the short form. | Each file is internally consistent; the entry **Electronic mail** records that the two mean the same thing and that a reproduced identifier keeps its own spelling. |
