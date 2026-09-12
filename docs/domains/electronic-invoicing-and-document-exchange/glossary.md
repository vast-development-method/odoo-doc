# Glossary

The terms of the Electronic Invoicing and Document Exchange domain, in full words. A term that belongs to another domain is defined here only in the sense this domain uses it, with a pointer to the domain that owns it.

---

**Access point.** A service that is entitled to hand documents to the regulated document exchange network and to receive documents from it on behalf of participants. This platform never operates one; it talks to a proxy service that does.

**Additional document reference.** The element group of the universal business language family that carries a document attached to the invoice, either embedded as encoded bytes or referenced by an address. It is how the printed document and the user supplied attachments travel inside the structured file.

**Aggregated delivery state.** The single delivery state shown on an accounting document, derived from the delivery states of its delivery records whose format needs a remote call. Its derivation, including the two cases where it is empty although delivery records exist, is in section 3 of [state-machines.md](state-machines.md).

**Allowance.** A negative adjustment to an amount, expressed as its own element group with a charge indicator whose value is false. A line discount, an early payment discount and a global discount are all reported as allowances.

**Applicability.** The answer a format gives when asked whether it applies to one accounting document. An empty answer means the format produces nothing for that document. A non empty answer names the operation to run when the document is posted, the operation to run when a withdrawal is asked for, the optional batching operations of both flows, and the optional content preview operation.

**Archival variant of the portable document format.** The long term preservation profile of the printed document format. A hybrid invoice for France or Germany is converted to it, and an extension metadata block is added, so that the file remains readable and verifiable for the legal retention period.

**Base line.** The intermediate representation of one line of an accounting document that the tax engine of the [taxes](../taxes/calculations.md) domain consumes and enriches with tax details. Every export mapping and every import mapping of this domain is expressed in terms of base lines rather than in terms of stored lines.

**Batching key.** The tuple that decides which delivery records travel in one job. It always holds the format, the delivery state and the company; a format that declares a batching operation extends it with the values that operation returns, and a format that declares none extends it with the identifier of the accounting document, which produces one job per document. Specified in section 1.8 of [entities.md](entities.md).

**Behaviour definition.** One of the eighteen entities of this domain that hold no field and store no record, and that exist only as a named set of node builders, import steps and checks arranged in an inheritance graph. Enumerated with their graph in section 19 of [entities.md](entities.md).

**Blocking level.** The severity carried by a delivery record: informational, warning or error. Only the error level blocks the operation the delivery record belongs to.

**Business response.** A message sent back over the exchange network about a document that was received, stating that it has been received, that it is accepted, or that it is rejected with reasons and suggested actions.

**Cancellation request.** The act of asking the remote service of a format to withdraw a payload it has already accepted. It moves the delivery record from sent to to cancel and posts a note; it does not itself cancel the accounting document, which is cancelled only when the remote service grants the withdrawal.

**Cash rounding line.** A line added to an accounting document so that its total ends on a round amount in the smallest coin in circulation. It is never reported as a document line; its amount becomes the payable rounding amount of the exported file.

**Charge.** A positive adjustment to an amount, expressed as its own element group with a charge indicator whose value is true. A recycling contribution tax, an excise tax, a negative discount and a general upsell are all reported as charges.

**Charge indicator.** The element whose value, true or false, says whether an allowance and charge group is a charge or an allowance.

**Classified tax category.** The element group inside an item that states the tax category code, the rate and the tax scheme that apply to that line. The European semantic standard demands exactly one of them per line.

**Client identifier.** The identifier the proxy service assigns to a credential when the connection is created. It is sent as a header on every call.

**Commercial contact.** The topmost contact of a contact hierarchy, the one that carries the legal identity. Most identification elements of an exported file are read from the commercial contact rather than from the contact of the document. Owned by the [contacts and organizations](../contacts-and-organizations/README.md) domain.

**Credential.** The record that identifies this company on a proxy service: the client identifier, the participant identification, the private key, the rotating token and the operating mode. Called the Electronic Interchange Proxy User in [entities.md](entities.md).

**Cross industry invoice.** The structured invoice syntax whose file is carried inside the printed document, producing a hybrid document that a human and a machine can both read. Its mapping is in [cross-industry-invoice-mapping.md](cross-industry-invoice-mapping.md).

**Customization identifier.** The element that names the exact profile a structured file follows. It is what a receiving system matches in order to choose its decoder, and what a participant publishes in order to say which documents it accepts.

**Delivery record.** One row per accounting document per registered format, carrying the produced file, the state, the last error and the blocking level. Called the Electronic Document in [entities.md](entities.md).

**Delivery state.** The state of a delivery record: to send, sent, to cancel or cancelled.

**Demonstration mode.** An operating mode in which no call leaves the deployment; every proxy call is answered locally and a shipped demonstration vendor bill arrives in the inbox.

**Document exchange network.** The regulated four corner network on which a sender hands a document to its own access point, that access point delivers it to the access point of the receiver, and the receiver collects it there. Referred to by its proper name, Peppol, where a product name is intended.

**Early payment discount.** A discount granted when the invoice is paid before a stated day. A mixed early payment splits into two lines on the accounting document and into one allowance and one charge in the exported file, so that only the tax effect of the discount is reported.

**Electronic address scheme.** The code that says what kind of identifier a participant endpoint holds, for example a tax identification number, a company registry number or an electronic mail address. The catalogue is in section 3 of [peppol-network.md](peppol-network.md).

**Electronic document state.** See aggregated delivery state.

**Emptying tax.** A fixed amount tax, or a tax computed by code, that does not increase the base of the taxes that follow it. It is removed from its line and reported as an extra document line carrying the tax name, because the standard has no element for it.

**Endpoint.** The value half of a participant identification. See participant identification.

**European economic area.** The set of countries listed in section 9 of [universal-business-language-mapping.md](universal-business-language-mapping.md), used to decide the intra-community tax category and the intra-area delivery date rule.

**European semantic invoice standard.** The semantic model that defines what an electronic invoice must contain in Europe, independently of the syntax used to express it. This domain implements it as a layer that sits between the generic builder and every European profile.

**Excise tax.** A tax computed by code that increases the base of the taxes that follow it. It is reported as a line level charge rather than as a tax.

**Exemption reason.** The sentence, and optionally the code, that says why a supply carries no tax. The prediction is in section 10.2 of [calculations.md](calculations.md) and the code list is in section 23 of [universal-business-language-mapping.md](universal-business-language-mapping.md).

**File document sign.** The factor, plus one or minus one, applied to every amount and quantity read from an inbound file, so that a credit note expressed as an invoice with negative amounts is imported correctly.

**Format.** A registered structured document profile that may be switched on per journal. Called the Electronic Document Format in [entities.md](entities.md).

**Global discount line.** A line of an accounting document that carries a discount or an upsell applying to the whole document. It is never reported as a document line; it becomes a document level allowance or charge.

**Global location number.** The code that identifies one physical location in the international article number location scheme, whose scheme identifier is `0088`. It is held on a delivery address, is free text and is not validated by the system. On export it becomes the identifier of the delivery location in the universal business language family and the identifier of the ship-to trade party in the cross industry invoice family; no import step ever writes it. Specified in section 15.6 of [entities.md](entities.md).

**Goods and services tax area.** The set of countries listed in section 8.7 of [universal-business-language-mapping.md](universal-business-language-mapping.md) whose parties use the goods and services tax scheme code instead of the value added tax scheme code.

**Hybrid document.** A printed document that carries a structured file inside it, so that one file serves both a human reader and a machine.

**Intra-community supply.** A supply between two parties established in two different countries of the European economic area, where the liability for the tax is shifted to the buyer. It has its own tax category code and its own delivery date requirement.

**Item.** The element group of a document line that describes what was sold: the name, the description, the seller and buyer and standard identifiers, the commodity classifications, the classified tax categories and the free attribute pairs.

**Job.** A set of delivery records that share one format, one delivery state, one company and one batching key, processed by one call to the operation of that format. Records at the error blocking level are never put into a job.

**Layered builder.** The family of node builders assembled from five layers, described in section 1 of [universal-business-language-mapping.md](universal-business-language-mapping.md), that serves every profile derived from the European semantic standard.

**Line net amount.** The amount of a document line excluding tax, after the line allowances have been subtracted and the line charges added. It is the amount the document total must be the sum of.

**Message identifier.** The identifier the proxy service assigns to a message when it accepts it, and by which the delivery state of that message is later polled and acknowledged.

**Migration key.** A key issued by a previous access point that lets an existing participant move its publication to this access point. It is cleared as soon as it has been sent.

**Monetary total.** The element group holding the eight document totals: the line extension amount, the tax exclusive amount, the tax inclusive amount, the allowance total, the charge total, the prepaid amount, the payable rounding amount and the payable amount.

**Network state.** The state of one accounting document on the document exchange network: ready to send, queued, pending reception, done, error, or one of the three answer states received, approved and rejected. Specified in section 5 of [state-machines.md](state-machines.md).

**Operating mode.** Whether a credential talks to the live network, to the acceptance network, or to nothing at all in the demonstration mode.

**Out of step.** The condition of a credential whose rotating token no longer matches the one the proxy service expects, which happens after a database is restored from a backup or copied without neutralisation. While a credential is out of step every call is refused locally and only two operations remain available, both signed with the participant private key: reconnect this database, and disconnect this database.

**Participant.** A company that is identified on the exchange network by a participant identification, and, when it is published, that announces the document types it accepts.

**Participant identification.** The pair made of an electronic address scheme code and an endpoint value, written as the scheme, a colon and the value, and compared without regard to letter case.

**Participant lookup.** The enquiry that answers whether a participant identification exists on the network and which document types it publishes.

**Participant state.** The registration state of a company: not registered, sender, pending registration to receive, receiver, or rejected.

**Payable rounding amount.** The element that absorbs, in one figure, both a cash rounding applied by the accounting document and any residual discrepancy between the sum of the presented amounts and the true document total.

**Plain builder.** The older builder that serves the plain 2.0 syntax, the plain 2.1 syntax, the Belgian profile and the point of sale receipt profile. Described in section 17 of [universal-business-language-mapping.md](universal-business-language-mapping.md).

**Prepaid amount.** The part of the document total that has already been paid, computed as the total minus the residual amount. Under the international billing layer it also carries the withholding tax amounts.

**Presentation rounding.** The rule that turns an amount into the text written into a file, given a minimum and an optional maximum number of decimal places. Specified in section 1 of [calculations.md](calculations.md).

**Profile.** A concrete structured document specification, identified by its customization identifier, for example the international billing profile or a country variant of it.

**Proxy service.** The remote counterpart operated for this platform that holds the participant registration, accepts outgoing payloads, returns message identifiers and delivery states, stores inbound payloads until they are acknowledged, and calls back into the deployment.

**Reception journal.** The single purchase journal of a company in which documents received from the exchange network are filed.

**Recycling contribution tax.** A fixed amount tax that increases the base of the taxes that follow it, levied to finance the collection and recycling of a product. It is reported as a line level charge rather than as a tax.

**Rotating token.** The shared secret used to sign an ordinary proxy call. It expires after twenty four hours, which is what prevents two databases from using one credential at the same time.

**Self billed document.** A document that the customer produces on behalf of the seller. It carries its own type codes and its own customization identifier, and the roles of supplier and customer are swapped relative to the accounting document that produced it.

**Sending method.** How an accounting document is delivered: by electronic mail, by post, over the exchange network, or by download. Owned by the [accounts receivable](../accounts-receivable/README.md) domain; this domain adds the network method.

**Sent flag.** The derived flag that says whether an accounting document has left the platform on the network. It is true exactly when the network state is pending reception, done, received, approved or rejected. It is the guard of the refusal to cancel a queued send, of the hiding of the reset to draft button and of the resequencing refusal.

**Service metadata register.** The register in which a participant publishes the document types it accepts and the address of its access point.

**Soft reset.** A reset of the participant configuration of a company that clears only the registration state and the migration key, keeping the electronic address scheme, the endpoint and the contact details, so that the user can register again with the same identification. Used by the out of step disconnect and nowhere else; every other reset is a full reset, which also clears and recomputes the configuration.

**Structured file.** The machine readable file produced from an accounting document, or read into one. Called the markup file where the emphasis is on its syntax.

**Tax category code.** The single letter or two letter code that classifies a tax for reporting purposes: standard rate, zero rated, exempt, reverse charge, intra-community supply, export, outside the scope, and the three regional codes. The catalogue is in section 9 of [configuration.md](configuration.md).

**Tax subtotal.** One entry of the tax breakdown of a document, holding the taxable amount, the tax amount and the tax category that produced them. The standard requires exactly one per distinct pair of category and rate.

**Unit code.** The code of the unit of measure written as an attribute of a quantity element. The mapping is in section 11 of [calculations.md](calculations.md).

**Universal business language.** The document syntax family in which most profiles of this domain are expressed. Its mapping is in [universal-business-language-mapping.md](universal-business-language-mapping.md).

**Verification state.** The result of the last participant lookup for one contact seen from one company: unchecked, not on the network, cannot receive the format, or on the network. Only the last of the four allows a document to be sent to that contact over the network. Specified in section 10 of [state-machines.md](state-machines.md).

**Withholding tax.** A tax whose amount is negative, deducted by the buyer and paid to the authority on behalf of the seller. The international billing layer forbids reporting it as a tax total, so it is reported as a prepaid amount instead, together with an explanatory sentence in the note.

