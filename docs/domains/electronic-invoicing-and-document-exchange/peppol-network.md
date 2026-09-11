# The Peppol document exchange network

The reference for everything that concerns the regulated four corner document exchange network: the participant roles and their state machine, the electronic address scheme catalogue per country, the participant lookup, the document types a participant publishes, the contract with the proxy service that operates the access point on behalf of this platform, the request signing and credential refresh, the outgoing message states, the inbound message handling, the business level responses with their code lists, the demonstration mode, the recovery of a connection whose credentials went out of step, and the complete catalogue of the errors the proxy returns. The procedures that use all of this, step by step, are in [workflows.md](workflows.md); the records are specified in [entities.md](entities.md); the numbered rules are in [business-rules.md](business-rules.md).

The network is a regulated document exchange network on which a sender hands a document to its own access point, that access point delivers it to the access point of the receiver, and the receiver collects it there. A participant is identified by a pair made of an electronic address scheme code and an endpoint value. A participant publishes, in a service metadata register, the list of document types it is able to receive. The platform never operates an access point itself: it talks to a proxy service that operates one, and this file specifies that conversation.

---

# 1. Roles and the registration state machine

| State | Label | Meaning |
|---|---|---|
| `not_registered` | Not registered | The company has no connection. It can neither send nor receive over the network. |
| `sender` | Can send but not receive | The company is connected and may hand documents to the network, but it is not published in the service metadata register, so nobody can address a document to it. |
| `smp_registration` | Can send, pending registration to receive | The publication of the company in the service metadata register has been requested and is not yet effective. The company may already send. |
| `receiver` | Can send and receive | The company is published and may both send and receive. |
| `rejected` | Rejected | The identity verification of the company failed. |

| From state | Trigger | Guard | To state | Side effects |
|---|---|---|---|---|
| `not_registered` | create the connection | the identification, the contact electronic mail address and the mobile number are set, and the identity verification, when the proxy requires one, succeeded | the state the proxy returns: `sender`, `smp_registration`, `receiver` or `rejected` | the credential record is created with the client identifier, the rotating token and a freshly generated private key; a welcome message is sent when the returned state is `sender` |
| `sender` | ask to receive as well | the state is exactly `sender`, and the identification is not already published by another provider | `smp_registration` | the published document types are sent to the proxy, the stored migration key is cleared, the external provider name is cleared, and the participant state poll is scheduled to run in one hour |
| `smp_registration` | the participant state poll returns `receiver` | none | `receiver` | none |
| `smp_registration` or `receiver` | ask to stop receiving | none | `sender` | the pending delivery states and the pending inbox are drained first, then the proxy is asked to remove the publication |
| any state with a credential | deregister | none | `not_registered` | the pending delivery states and the pending inbox are drained first, the proxy is asked to cancel the registration, the whole participant configuration is reset and the credential record is deleted |
| any state | the participant state poll returns `not_registered` | none | `not_registered` | the participant configuration is reset and the credential record is archived |
| any state | the participant state poll fails with the code saying the client is gone | none | `not_registered` | the participant configuration is reset and the credential record is archived |
| any state | a call fails with the code saying no such user exists, while the credential is already inactive and the company has no other credential of the same kind | none | `not_registered` | the stored migration key is cleared and the refusal `We could not find a user with this information on our server. Please check your information.` is raised |

**Resetting the participant configuration** sets the state to `not_registered` and clears the migration key. Unless the reset is a soft reset, it also clears the external provider name, the electronic address scheme, the endpoint, the contact electronic mail address and the mobile number, and then recomputes the contact electronic mail address, the mobile number, the scheme and the endpoint from the ordinary derivations, so that a branch company that borrowed the configuration of its parent gets its defaults back. A soft reset keeps the configuration so that the user can register again with the same identification.

**Which states may send.** The states `sender`, `smp_registration` and `receiver`. Every operation that hands a document to the network, and the delivery state polling, is restricted to companies in one of those three states.

---

# 2. Participant identification

The participant identification of a company is the electronic address scheme code, a colon, and the endpoint value, for example `0208:0477472701`. It is compared without regard to letter case everywhere. When the company has no scheme or no endpoint, an attempt to register raises `Please fill in the electronic address scheme code and the Participant Identifier code.`

A company may not register when another connection of a different kind already exists for it; the attempt is refused with `A connection to '<the other kind>' already exists.`

A branch company whose scheme and endpoint equal those of an ancestor company that can send, or that has no endpoint of its own while an ancestor has one, is considered to use the connection of that ancestor. Documents it issues are exported with the ancestor as the supplier, and it may be disconnected from that ancestor, which deregisters it.

---

# 3. The electronic address scheme catalogue

## 3.1 Schemes offered per country

For each country, the schemes are listed in the order in which they are preferred, together with the field of the contact that supplies the endpoint value. An entry with no field means the scheme is offered but no value can be derived automatically.

| Country | Scheme codes in preference order, with the source of the value |
|---|---|
| Andorra | `9922` tax identification number |
| United Arab Emirates | `0235` tax identification number |
| Albania | `9923` tax identification number |
| Austria | `9915` tax identification number |
| Australia | `0151` tax identification number |
| Bosnia and Herzegovina | `9924` tax identification number |
| Belgium | `0208` company registry, `9925` tax identification number |
| Bulgaria | `9926` tax identification number |
| Switzerland | `9927` tax identification number, `0183` no source |
| Cyprus | `9928` tax identification number |
| Czechia | `9929` tax identification number |
| Germany | `9930` tax identification number, `0246` the national economic identification number |
| Denmark | `0184` tax identification number, `0198` tax identification number |
| Estonia | `9931` tax identification number |
| Spain | `9920` tax identification number |
| Finland | `0216` no source |
| France | `0225` the endpoint itself, reserved for country specific logic; `0009` company registry; `9957` tax identification number; `0002` no source |
| Singapore | `0195` the national unique entity number |
| United Kingdom | `9932` tax identification number |
| Greece | `9933` tax identification number |
| Croatia | `9934` tax identification number, `0088` company registry |
| Hungary | `9910` the European tax identification number of the country |
| Ireland | `9935` tax identification number |
| Iceland | `0196` tax identification number |
| Italy | `0211` tax identification number, `0210` the national fiscal code |
| Japan | `0221` tax identification number |
| Liechtenstein | `9936` tax identification number |
| Lithuania | `9937` tax identification number |
| Luxembourg | `9938` tax identification number |
| Latvia | `0218` company registry, `9939` tax identification number |
| Monaco | `9940` tax identification number |
| Montenegro | `9941` tax identification number |
| North Macedonia | `9942` tax identification number |
| Malta | `9943` tax identification number |
| Malaysia | `0230` no source |
| Nigeria | `0244` tax identification number |
| Netherlands | `0106` no source, `0190` no source |
| Norway | `0192` the national organisation number |
| New Zealand | `0088` company registry |
| Poland | `9945` tax identification number |
| Portugal | `9946` tax identification number |
| Romania | `9947` tax identification number |
| Serbia | `9948` tax identification number |
| Sweden | `0007` company registry, `9955` tax identification number |
| Slovenia | `9949` tax identification number |
| Slovakia | `9950` tax identification number, `0245` company registry |
| San Marino | `9951` tax identification number |
| Turkey | `9952` tax identification number |
| Vatican City | `9953` tax identification number |
| Saint Barthélemy | `0009` establishment identification number, `9957` tax identification number, `0002` no source |
| French Guiana | `0009` establishment identification number, `9957` tax identification number, `0002` no source |
| Guadeloupe | `0225` the endpoint itself, `0009` establishment identification number, `9957` tax identification number, `0002` no source |
| Saint Martin | `0009` establishment identification number, `9957` tax identification number, `0002` no source |
| Martinique | `0225` the endpoint itself, `0009` establishment identification number, `9957` tax identification number, `0002` no source |
| New Caledonia | `0009` establishment identification number, `9957` tax identification number, `0002` no source |
| French Polynesia | `0009` establishment identification number, `9957` tax identification number, `0002` no source |
| Saint Pierre and Miquelon | `0009` establishment identification number, `9957` tax identification number, `0002` no source |
| Réunion | `0225` the endpoint itself, `0009` establishment identification number, `9957` tax identification number, `0002` no source |
| French Southern and Antarctic Lands | `0009` establishment identification number, `9957` tax identification number, `0002` no source |
| Wallis and Futuna | `0009` establishment identification number, `9957` tax identification number, `0002` no source |
| Mayotte | `0225` the endpoint itself, `0009` establishment identification number, `9957` tax identification number, `0002` no source |
| Åland Islands | `0216` no source |

The Netherlands is deliberately mapped to the chamber of commerce scheme and the public body scheme only, never to the tax identification scheme, because the national profile requires the legal entity of a Netherlands party to be identified by one of those two.

Four scheme codes are withdrawn from the list offered in the interface, and are only kept visible on a contact that already carries one of them: `0037`, `0213`, `9955` and `0193`.

## 3.2 Endpoint sanitising and validity

Both are specified in section 15.2 of [entities.md](entities.md): which characters are stripped per scheme, and the five validity rules with their exact messages.

The company applies two further rule sets of its own.

**Rules that refuse a value.** When one of these fails, writing the endpoint raises `The Peppol endpoint identification number is not correct.`

| Scheme | Requirement |
|---|---|
| `0007` | a valid Swedish organisation number |
| `0088` | a valid international article number |
| `0184` | a valid Danish central business register number |
| `0192` | a valid Norwegian organisation number |
| `0208` | a valid Belgian tax identification number |

**Rules that only warn.** When one of these fails, the registration wizard shows the warning `The endpoint number might not be correct. Please check the value.`, but the value is accepted.

| Scheme | Requirement |
|---|---|
| `0151` | a valid Australian business number |
| `0201` | exactly six letters or digits |
| `0210` | a valid Italian fiscal code |
| `0211` | a valid Italian tax identification number |
| `9906` | a valid Italian tax identification number |
| `9907` | a valid Italian fiscal code |

**Rules that silently correct the value.** Applied whenever a company record is created or written, before validation.

| Scheme | Correction |
|---|---|
| `0007` | keep the first run of exactly ten digits found in the value |
| `0184` | keep the first run of exactly eight digits found in the value |
| `0192` | keep the first run of exactly nine digits found in the value |
| `0208` | keep the first run of exactly ten digits found in the value |

## 3.3 Countries where the network is used

**Countries where the network is the default sending method:** Austria, Belgium, Switzerland, Cyprus, Czechia, Germany, Denmark, Estonia, Spain, Finland, France, Ireland, Iceland, Lithuania, Luxembourg, Latvia, Malta, the Netherlands, Norway, Sweden, Slovenia.

**Countries where the network is available at all:** the countries above, plus Andorra, Albania, the Åland Islands, Bosnia and Herzegovina, Bulgaria, Saint Barthélemy, the United Kingdom, French Guiana, Guadeloupe, Greece, Croatia, Hungary, Italy, Liechtenstein, Monaco, Montenegro, Saint Martin, North Macedonia, Martinique, New Caledonia, French Polynesia, Poland, Saint Pierre and Miquelon, Portugal, Réunion, Romania, Serbia, Slovakia, San Marino, the French Southern and Antarctic Lands, Turkey, Vatican City, Wallis and Futuna, Mayotte.

**Countries where an electronic mail message about the network is added to the invoice mail:** Belgium, Luxembourg, the Netherlands, Sweden, Norway. The footnote is added only when the company can send, the company country and the customer country are both in that list, and it carries the customer country, whether the document was already sent over the network, whether the customer looks like a consumer, that is whether its tax identification number is one character or less, and whether the customer is a published participant.

---

# 4. Participant lookup

The lookup answers two questions about a trading partner: is it published on the network at all, and does it publish the document type this company intends to send.

1. When the operating mode is the demonstration mode, the lookup is not performed at all and the verification state is derived locally, see section 11.
2. The participant identification is lower cased and passed to the lookup endpoint of the proxy as a query parameter. The call is a plain request without signing, because the lookup is public.
3. A transport failure, a response that is not a structured document, or a response carrying an error other than the one meaning "not found" is logged and produces no answer.
4. A successful answer holds the published identification of the participant and a list of published services, each carrying a document type identifier and an address at which the service description can be fetched.

**Deciding the verification state:**

```
if the scheme or the endpoint is missing, or the chosen format is not a format the network accepts:
    state = not_verified
else:
    answer = lookup(scheme + ":" + endpoint)
    if answer is empty:                       state = not_valid
    else if the participant does not exist:   state = not_valid
    else if the document type is published:   state = valid
    else:                                     state = not_valid_format
```

**Deciding whether the participant exists.** The published identification of the answer must equal the identification asked for, compared without regard to letter case, and the address of the first published service must not belong to the national pre-registration register of Belgium. Every Belgian company is pre-registered there, so a match on that register does not mean the company is a real participant.

**Deciding whether the document type is published.** The customization identifier that the chosen profile would write, for the billing process or for the self billing process as required, must appear inside the document type identifier of at least one published service.

**Logging a change.** When the verification state of a contact changes, a message is posted on its discussion thread showing the previous label, an arrow, the new label, the name of the field and the name of the company for which the check ran. The change is logged this way rather than by the ordinary field tracking because the value is held per company.

**When the lookup runs.** On creation of a contact; whenever the scheme, the endpoint or the chosen format is edited, immediately, so that the interface can warn before a document is sent; when the sending wizard is opened; and after an inbox batch, for every partner of the imported documents whose verification state was unchecked or empty.

## 4.1 Deciding whether the company itself is already published

Before a company asks to become a receiver, the same lookup is run on its own identification. When the participant exists, the request is refused with `A participant with these details has already been registered on the network. If you have previously registered to a Peppol service, please deregister.` When the service description of the first published service can be fetched and names a provider that is not this platform, the sentence `The Peppol service that is used is <the provider name>.` is appended and the provider name is stored on the company.

---

# 5. Published document types

A receiver publishes a set of document type identifiers. The set is assembled from a per capability package table, so that installing a package extends what the participant announces.

| Contributed by | Document type identifier | Human name |
|---|---|---|
| the base set | `urn:oasis:names:specification:ubl:schema:xsd:Invoice-2::Invoice##urn:cen.eu:en16931:2017#compliant#urn:fdc:peppol.eu:2017:poacc:billing:3.0::2.1` | the document type identifier of the Peppol billing invoice, version 3 |
| the base set | `urn:oasis:names:specification:ubl:schema:xsd:CreditNote-2::CreditNote##urn:cen.eu:en16931:2017#compliant#urn:fdc:peppol.eu:2017:poacc:billing:3.0::2.1` | the document type identifier of the Peppol billing credit note, version 3 |
| the base set | `urn:oasis:names:specification:ubl:schema:xsd:Invoice-2::Invoice##urn:cen.eu:en16931:2017#compliant#urn:fdc:peppol.eu:2017:poacc:selfbilling:3.0::2.1` | the document type identifier of the Peppol self billing invoice, version 3 |
| the base set | `urn:oasis:names:specification:ubl:schema:xsd:CreditNote-2::CreditNote##urn:cen.eu:en16931:2017#compliant#urn:fdc:peppol.eu:2017:poacc:selfbilling:3.0::2.1` | the document type identifier of the Peppol self billing credit note, version 3 |
| the base set | `urn:oasis:names:specification:ubl:schema:xsd:Invoice-2::Invoice##urn:cen.eu:en16931:2017#compliant#urn:fdc:nen.nl:nlcius:v1.0::2.1` | the document type identifier of the Netherlands standard invoice, version 2 |
| the base set | `urn:oasis:names:specification:ubl:schema:xsd:CreditNote-2::CreditNote##urn:cen.eu:en16931:2017#compliant#urn:fdc:nen.nl:nlcius:v1.0::2.1` | the document type identifier of the Netherlands standard credit note, version 2 |
| the business response package | `urn:oasis:names:specification:ubl:schema:xsd:ApplicationResponse-2::ApplicationResponse##urn:fdc:peppol.eu:poacc:trns:invoice_response:3::2.1` | the document type identifier of the Peppol invoice response transaction, version 3 |

**Keeping the published set in step.** A scheduled action, whose declared interval is so long that it effectively only runs when triggered, compares the published set of every receiver with the target set and corrects it. The target set is the full set when the company has a reception journal and the full set minus the response transaction otherwise, because a participant that cannot file received documents must not announce that it answers them. The action removes the published identifiers that are not in the target set, then adds the target identifiers that are not published, and reschedules itself in four hours when any receiver failed. It is also triggered whenever the reception journal of a company is written.

**Removing the document types of a package.** When a capability package that contributed document types is removed, every receiver is asked to remove exactly the identifiers that package contributed. A failure per receiver is logged and does not stop the others.

---

# 6. The proxy call contract

## 6.1 Operating modes and base addresses

| Mode | Base address of the proxy | Meaning |
|---|---|---|
| `prod` | the production address of the proxy service | the live network |
| `test` | the acceptance address of the proxy service | the acceptance network |
| `demo` | none; every call is answered locally | the demonstration mode of section 11 |

The mode of a company is decided in this order: the demonstration mode when the chosen scheme is the demonstration scheme `odemo`; otherwise the mode stored on the existing credential; otherwise the value of the configuration parameter that names the default mode; otherwise the live mode.

## 6.2 Shape of a call

Every authenticated call is a remote procedure call carried over a secure transport, whose body is a structured document holding the protocol version `2.0`, the method name `call`, the parameters of the call, and a freshly generated request identifier. The response is a structured document holding either a result or an error. The call times out after thirty seconds.

Every endpoint address is built as the base address of the mode, then a fixed application path segment, then the kind of the connection, which is `peppol`, then the version number of the operation and the operation name. The two operations that create a credential without the identity verification flow, and the one that renews the rotating token, sit under a different fixed path segment reserved for the platform services.

## 6.3 Request signing

Three headers are added to every authenticated call:

| Header | Value |
|---|---|
| the client identifier header | the client identifier of the credential |
| the timestamp header | the current time expressed in whole seconds since the epoch |
| the signature header | the signature of the message defined below |
| the signature kind header | `hmac` or `asymmetric` |

The message that is signed is the concatenation, separated by vertical bars, of: the timestamp, the path of the address, the client identifier, the query parameters of the address rendered as a structured document with the keys sorted, and the body of the request rendered as a structured document with the keys sorted.

| Signing kind | How the signature is produced | When it is used |
|---|---|---|
| shared secret | a keyed hash of the message using the secure hash algorithm with a digest of two hundred and fifty six bits, keyed with the rotating token decoded from its base sixty four form, rendered in hexadecimal | every ordinary call |
| participant key | a signature of the message with the private key of the credential, rendered in base sixty four | only the three calls that repair a connection whose credentials went out of step |

Signing the sorted rendering of the parameters and of the body, rather than the raw bytes, makes the signature independent of the serialisation order and preserves the integrity of the message end to end.

## 6.4 The rotating token

The rotating token expires after twenty four hours. When a call fails with the error code saying the token has expired, the credential is locked for update, a renewal call is made, the returned token replaces the stored one, the change is committed immediately so that it cannot be lost, and the original call is retried once with the shared secret signing. When the lock cannot be taken, the renewal is abandoned silently, because another process is already renewing it.

The expiry exists so that two databases cannot use the same credential at the same time: the second one to renew invalidates the token of the first.

## 6.5 Errors returned by a call

| Error code | Handling |
|---|---|
| `refresh_token_expired` | renew the token and retry once, as in section 6.4 |
| `no_such_user` | deactivate the credential; the same code is also raised when the identification was claimed by somebody else before any data was exchanged |
| `invalid_signature` | raise `Failed to connect to the access point server. This might be due to another connection to the access point server. It can occur if you have duplicated your database. \n\nIf you are not sure how to fix this, please contact our support.` and, in the network specific wrapper, mark the connection as out of step |
| `connection_superseded` | another database has taken over the connection; disconnect this database |
| any other code | raise the message that came with it |

A transport failure raises `The web address that this service requested returned an error. The web address it tried to contact was <the address>`. A response whose error says the address does not exist raises `The web address that this service tried to contact does not exist. The web address was “<the address>”`. Any other transport level error raises `The web address that this service requested returned an error. The web address it tried to contact was <the address>. <the message>`.

A call made while the credential is marked as out of step raises, before any request is sent, `Failed to connect to Peppol Access Point. This might happen if you restored a database from a backup or copied it without neutralization. To fix this, please go to Settings > Accounting > Peppol Settings and click on 'Reconnect this database'.`

A call made while the operating mode is the demonstration mode and the caller did not intercept it raises the internal refusal `Can't access the proxy in demo mode`.

## 6.6 The endpoints

| Endpoint | Parameters | Answer | Purpose |
|---|---|---|---|
| `create_user`, version 2, under the platform services segment | the database identifier, the company key, the participant identification, the public key in text encoding, the kind of the connection | the client identifier and the rotating token | create a credential without the identity verification flow |
| `renew_token`, version 1, under the platform services segment | none | the rotating token | renew the expired token |
| `can_connect`, version 2 | the database identifier, the participant identification, the return address, the connection request token, the contact electronic mail address, the callback address | whether the identification is invalid, whether the database is unsuitable, whether an identity verification is required and which methods are available with their authorisation addresses | ask whether a connection may be created, before creating it |
| `connect`, version 2 | the participant identification, the database identifier, the company key, the public key, the verification token, and the company details of section 7.1 | the client identifier, the rotating token and the resulting participant state | create the connection |
| `participant_status`, version 2 | none | the participant state | poll the state of the participant |
| `register_sender_as_receiver`, version 1 | the migration key, the list of published document type identifiers | none | ask to be published as a receiver |
| `unregister_to_sender`, version 1 | none | none | ask to stop being published |
| `cancel_peppol_registration`, version 1 | none | none | cancel the registration entirely |
| `update_user`, version 1 | a structure holding the fields to update; only the contact electronic mail address is ever sent | none | change the contact electronic mail address held by the proxy |
| `send_document`, version 1 | the list of payloads of section 8.1 | one message entry per payload, in the same order, each holding a message identifier | hand documents to the network |
| `get_all_documents`, version 1 | a filter holding the direction, whether errors are wanted and the receiver identification | the list of messages awaiting collection, each holding at least its identifier, its sender and its receiver | list the inbox |
| `get_document`, version 1 | the list of message identifiers | a map from message identifier to its content: the encrypted document, the encrypted symmetric key, the file name, the state, the document type, the identifier of the message it answers, and an error when there is one | fetch messages and delivery states |
| `ack`, version 1 | the list of message identifiers | none | tell the proxy that those messages have been handled and may stop being returned |
| `send_response`, version 1 | the list of referenced message identifiers, the response status and the clarification list | one message entry per response, each holding a message identifier | send business level responses |
| `lookup`, version 1 | the participant identification, as a query parameter, unsigned | the published identification and the published services | look a participant up |
| `get_services`, version 2 | none | the published document type identifiers of this participant | read the published set |
| `add_services`, version 2 | the list of document type identifiers | none | publish further document types |
| `remove_services`, version 2 | the list of document type identifiers | none | withdraw document types |
| `set_webhook`, version 2 | the callback address and a signed callback token | none | register the callbacks of section 10 |
| `mark_connection_out_of_sync`, version 1 | the synchronisation counter | none | declare that this database believes its credentials are out of step |
| `resync_connection`, version 1 | the incremented synchronisation counter | the rotating token | take the connection back for this database |

---

# 7. Creating a connection

## 7.1 The company details sent to the proxy

| Detail | Source |
|---|---|
| company name | the display name of the company |
| tax identification number | the tax identification number of the company |
| street | the street of the company |
| city | the city of the company |
| postal code | the postal code of the company |
| country code | the two letter code of the country of the company |
| mobile number | the participant mobile number of the company |
| contact electronic mail address | the participant contact electronic mail address of the company |
| migration key | the stored migration key of the company, which lets an existing participant move to this access point |
| callback address | the callback address of the company, which is the base address of the deployment followed by the callback path |
| callback token | a signed token binding the company key and the callback address, valid for thirty days |
| published document type identifiers | the target set of section 5 |

## 7.2 The connection request token

Before the identity verification is started, a token is produced that binds the participant identification, the company key, the contact of the current user and the moment it was produced. It is signed and is valid for two weeks. It travels to the identity verification service and comes back on the return address, which is how the platform knows which company and which user the returning browser belongs to. A token that cannot be verified, or whose company or contact no longer exists, is rejected.

## 7.3 The answers of the connection enquiry

| Answer field | Meaning | Refusal raised |
|---|---|---|
| the answer is empty | the proxy could not be reached | `Could not connect to Proxy Server.` |
| the identification is invalid with the code saying it is not on the network | the identification cannot be used | `Your identifier you entered is invalid for Peppol.` |
| the identification is invalid with the code saying the format is wrong | the shape of the identification is wrong | `Your identifier does not have a valid format.` followed, when the answer supplies an example, by ` Expected format: <the example>.` |
| the identification is invalid for any other reason | | `Your identifier is invalid.` |
| the database is unsuitable | the deployment may not connect | `The database you are trying to connect to is not suitable for Peppol.` |
| an identity verification is required and none was chosen | | `You need to authenticate to continue.` |
| the chosen identity verification method is not among the available ones | | `Selected authentication method is not available.` |

When an identity verification is required, the user is sent to the authorisation address of the chosen method. The method is the generic one when it is available, and the national electronic identity method otherwise.

## 7.4 Returning from the identity verification

The return address is called by the verification service with the kind of verification, the connection request token, a verification token and a state.

| State returned | Effect |
|---|---|
| `canceled` | a notification of kind `canceled` is pushed to the contact of the requesting user and the browser is sent back to the settings screen |
| `pending` | a notification of kind `pending` is pushed and the browser is sent to the main screen, because the decision is taken asynchronously |
| anything else with no verification token | a notification of kind `failure` is pushed and the browser is sent back to the settings screen |
| anything else with a verification token | the connection is created; on failure a notification of kind `failure` carrying the refusal message is pushed |

A separate callback address is called by the verification service when a pending decision turns positive. It verifies the same token, does nothing when the company already has a credential and answers that it is already connected, otherwise creates the connection and pushes a notification of kind `success`. A failure answers that it failed and leaves the user able to finish from the link sent by electronic mail.

## 7.5 Notifications shown after a successful registration

| Resulting state | Message |
|---|---|
| `sender` | `You can now send electronic invoices via Peppol.` |
| `smp_registration` | `Your Peppol registration will be activated soon. You can already send invoices.` |
| `receiver` | `You can now send and receive electronic invoices via Peppol` |
| `rejected` | title `Registration rejected.`, message `Your registration has been rejected. Please contact the support for further assistance.` |

## 7.6 Refusals raised before the connection enquiry

| Condition | Message |
|---|---|
| the company has no country | `Please select a country for your company.` |
| the contact electronic mail address or the mobile number is missing | `Contact email and phone number are required.` |
| the participant endpoint is missing | `Peppol Address should be provided.` |
| a branch is registering with the identification of its parent while it should register with its own | `Peppol Identifier should be different from main company.` |
| the company is not in the `not_registered` state | `Cannot register a user with a <the label of the current state> application` |
| the mobile number is not in international form | `Please enter the mobile number in the correct international format.\nFor example: +32123456789, where +32 is the country code.` |

Registering archives every credential of the same kind that already exists for the company with the same identification, so that a second registration never leaves two active credentials behind.

---

# 8. Sending

## 8.1 One payload

| Field | Value |
|---|---|
| file name | the file name produced by the profile |
| receiver | the electronic address scheme of the commercial partner, a colon, and its participant endpoint |
| document | the structured file encoded in base sixty four |

A payload larger than sixty four million bytes is refused locally with `Invoice %s exceeds the size limit of 64 MB to be sent via Peppol.`, with the unit expanded to "megabytes".

## 8.2 The outgoing message states

| State | Label | Meaning |
|---|---|---|
| `ready` | Ready to send | The document is posted, the company can send, the partner is a verified participant, and nothing has been sent yet. |
| `to_send` | Queued | The document has been queued by the sending wizard for asynchronous sending. |
| `processing` | Pending Reception | The document has been handed to the proxy and a message identifier has come back. |
| `done` | Done | The network has confirmed the delivery. |
| `error` | Error | The document could not be handed over, or the network reported a failure. |
| `AB` | Received | The recipient answered that it has received the document. |
| `AP` | Approved | The recipient answered that it accepts the document; the approval code identifier. |
| `RE` | Rejected | The recipient answered that it rejects the document. |

The three last states exist only when the business response package is installed, and are derived from the responses received about the document: a rejection wins over everything, then an approval or a payment notice gives the approved state, then anything else gives the received state. Only responses whose own delivery state is `done` are considered.

**Derivation of the state.** When the company can send, the partner is a verified participant, the document is posted, it is a sale document or a receipt, and it has no state yet, the state becomes `ready`. When the document is a draft sale document that has not been sent, the state is cleared. In every other case the state is left as it is.

**Derivation of the sent flag.** A document counts as sent when its state is none of: empty, `ready`, `to_send`, `error` and `skipped`. A document that counts as sent may not be reset to draft, and may not be resequenced.

**Cancelling a queued send.** Clearing the state and the queued sending data is refused with `Cannot cancel an entry that has already been sent to PEPPOL` when any of the selected documents counts as sent.

---

# 9. Receiving

The inbox polling procedure, with its duplicate detection and its batching, is in section 11 of [workflows.md](workflows.md). Three points belong here.

**Decryption.** Each message carries the document encrypted with a symmetric key, and that symmetric key encrypted with the public key of the credential. The symmetric key is decrypted with the private key of the credential, then the document is decrypted with the symmetric key using an authenticated symmetric scheme.

**File type.** The base rule declares every inbound document to be a markup file with the extension `xml` and the media type `application/xml`. A country specific package may declare another one.

**Reading the document type code of an inbound file.** The type code is read from the invoice type code element, falling back to the credit note type code element. When the file could not be parsed at all, every embedded binary object is stripped from the raw bytes and the file is parsed again, because a very large embedded printed document can defeat the parser. The codes `389`, `527` and `261` mean a self billed document.

---

# 10. Callbacks

Three public callback addresses accept a signed token as a query parameter and answer with an empty success. Each one verifies the token, resolves the credential it names, and triggers the corresponding scheduled action so that the work happens in the ordinary batch path rather than inside the request.

| Path | Triggered scheduled action |
|---|---|
| `/peppol/webhook/new-message` | the inbox polling |
| `/peppol/webhook/message-state-update` | the delivery state polling |
| `/peppol/webhook/user-state-update` | the participant state polling |

**The callback token** is a signed payload binding the company key and the callback address, valid for thirty days. Verification requires that the address of the incoming request starts with the address inside the token. When the token names a company that has a credential, that credential is used; otherwise, when the payload names a credential directly, that credential is used. A token that fails verification resolves nothing and the callback does nothing, while still answering with an empty success, so that a caller cannot probe which tokens are valid.

**Keeping the callbacks alive.** A scheduled action running every two weeks re-registers the callback address and a fresh token for every company in the `sender` or `receiver` state, because the token expires after thirty days.

---

# 11. The demonstration mode

In the demonstration mode no call ever leaves the deployment. Six operations are intercepted.

| Operation | Behaviour in the demonstration mode |
|---|---|
| a call to the proxy | answered from the table below |
| deciding the verification state of a partner | `not_verified` when the scheme or the endpoint is missing; `not_valid` when no format is chosen; `not_valid_format` when the chosen format is not one the network accepts; `valid` otherwise |
| deciding whether a participant already exists | always no |
| the connection enquiry | answers that no identity verification is required |
| creating the connection | creates a credential whose client identifier is derived from the company key, whose token is the text `demo`, whose mode is the demonstration mode and whose private key is a shipped demonstration key, reusing an existing key with the same content rather than creating a duplicate; the participant state becomes `receiver` |
| deregistering | resets the participant configuration and deletes the credential, with no call |

| Intercepted endpoint | Answer |
|---|---|
| `register_sender`, `register_receiver`, `register_sender_as_receiver`, `update_user`, `cancel_peppol_registration`, `set_webhook`, `ack`, `add_services`, `remove_services` | an empty answer |
| `unregister_to_sender` | sets the participant state to `sender` |
| `participant_status` | the state `receiver` |
| `get_services` | the full supported document type set |
| `get_all_documents` | one demonstration vendor bill, unless a document carrying its message identifier already exists, in which case no message |
| `get_document` | the demonstration vendor bill when the identifier is the demonstration one, otherwise a message in state `done` of the invoice document type answering itself |
| `send_document` | one message entry per payload, each with a freshly generated identifier, and the inbox polling scheduled action is triggered so that the demonstration vendor bill arrives |

**The demonstration vendor bill** is an inbound message whose direction is incoming, whose receiver is the participant identification of the credential, whose identifier is derived from the company key, whose sender identification is `0208:2718281828`, whose state is `done`, whose document type is the invoice type, and whose content is a shipped encrypted file with its shipped encrypted symmetric key.

The demonstration electronic address scheme `odemo` is offered on a contact only while the company operates in the demonstration mode; otherwise it is removed from the list of schemes the interface offers.

---

# 12. Recovering a connection whose credentials went out of step

A database that was restored from a backup, or copied without being neutralised, holds a credential that the proxy has already reassigned. The symptom is a call failing with the invalid signature code.

1. **Marking.** The credential is marked as out of step and its rotating token is cleared, in one write that is flushed immediately so that a token renewed in another transaction cannot resurrect it. A call is then made with the participant key signing to declare the situation to the proxy, carrying the current synchronisation counter.
2. **Superseded.** When that call answers that the connection has been superseded, this database is disconnected and the refusal `This connection has been superseded by another database. Register again.` is raised.
3. **Reconnecting this database.** The synchronisation counter is increased by one and a call is made with the participant key signing to take the connection back. On success the returned token is stored and the out of step mark is cleared, and the participant state poll is scheduled to run asynchronously, because confirming the token on the proxy side and failing before the local commit would leave an unrecoverable state.
4. **Disconnecting this database.** The participant configuration is soft reset, which keeps the identification so that the user can register again, and the credential record is deleted. The connection held by the proxy is left untouched, so the other database keeps working.

Both actions are offered as buttons in the participant settings block, and both require the credential to be marked as out of step.

---

# 13. Business responses

## 13.1 Response codes

| Code | Label | Sent by this platform |
|---|---|---|
| `AB` | Acknowledgement | yes, automatically after a received document has been imported |
| `IP` | In Process | no; stored when received |
| `UQ` | Under query | no; stored when received |
| `CA` | Conditionally accepted | no; stored when received |
| `RE` | Rejection | yes, from the rejection wizard |
| `AP` | Approval | yes, automatically when a received vendor bill is posted; the approval code identifier |
| `PD` | Paid | no; stored when received |

## 13.2 Delivery state of a response

| State | Label | Meaning |
|---|---|---|
| `processing` | Pending Reception | the response has been handed to the proxy |
| `done` | Done | the network has confirmed the delivery |
| `error` | Error | the network reported a failure |
| `not_serviced` | Not Serviced | the recipient cannot receive this document type |

## 13.3 When a document may be answered

A document may be answered when it carries a message identifier, it is a vendor bill or a vendor credit note, it has no response that is `not_serviced` and no response that is neither in error nor of a code other than approval or rejection, and its partner publishes the response transaction. In other words, a document may be acknowledged once, and then approved or rejected once, and never again after that.

## 13.4 The rejection reason code list

Every entry belongs to the reason list, whose list identifier is `OPStatusReason`, except the first, which is shipped without a list identifier and therefore is not offered in the rejection wizard.

| Code | Name | Description |
|---|---|---|
| `NON` | No Issue | Indicates that receiver of the documents sends the message just to update the status and there are no problems with document processing. |
| `REF` | References incorrect | Indicates that the received document did not contain references as required by the receiver for correctly routing the document for approval or processing. |
| `LEG` | Legal information incorrect | Information in the received document is not according to legal requirements. |
| `REC` | Receiver unknown | The party to which the document is addressed is not known. |
| `QUA` | Item quality insufficient | Unacceptable or incorrect quality. |
| `DEL` | Delivery issues | Delivery proposed or provided is not acceptable. |
| `PRI` | Prices incorrect | Prices not according to previous expectation. |
| `QTY` | Quantity incorrect | Quantity not according to previous expectation; the quantity reason code identifier. |
| `ITM` | Items incorrect | Items not according to previous expectation. |
| `PAY` | Payment terms incorrect | Payment terms not according to previous expectation. |
| `UNR` | Not recognized | Commercial transaction not recognized. |
| `FIN` | Finance incorrect | Finance terms not according to previous expectation. |
| `PPD` | Partially Paid | Payment is partially but not fully paid. |
| `OTH` | Other | Reason for status is not defined by code. |

The default reason offered by the rejection wizard is `UNR`.

## 13.5 The suggested action code list

Every entry belongs to the action list, whose list identifier is `OPStatusAction`.

| Code | Name | Description |
|---|---|---|
| `NOA` | No action required | No action required. |
| `PIN` | Provide information | Missing information requested without re-issuing invoice. |
| `NIN` | Issue new invoice | Request to re-issue a corrected invoice. |
| `CNF` | Credit fully | Request to fully cancel the referenced invoice with a credit note. |
| `CNP` | Credit partially | Request to issue partial credit note for corrections only. |
| `CNA` | Credit the amount | Request to repay the amount paid on the invoice. |
| `OTH` | Other | Requested action is not defined by code. |

## 13.6 Reading an inbound response

The response code is read from the response code element of the document response. Every status element of the response is then classified:

| Case | Effect |
|---|---|
| both a status reason code and a status reason text are present, and the list attribute of the code is one of the two lists | the text is added to that list |
| only a status reason code is present, and its list attribute is one of the two lists | the shipped entry of that list with that code supplies its name; when no entry matches, the raw code is added |
| only a status reason text is present | the shipped entry whose name or whose description equals that text supplies its list; when none matches, the text is added to a third list called miscellaneous |

The formatted message is then built: when the reason list is not empty, the bold title `Reasons:`, a line break and the reasons rendered as a natural language list; when the action list is not empty, a line break, the bold title `Suggested actions:` and the actions; when the miscellaneous list is not empty, a line break, the bold title `Miscellaneous:` and those entries.

## 13.7 Messages logged about a response

| Situation | Message logged on the accounting document |
|---|---|
| an inbound rejection whose delivery state is `done` | `The Peppol receiver of this document has rejected it with the following information:` followed by a line break and the formatted message of section 13.6 |
| an inbound acknowledgement whose delivery state is `done` | `The Peppol receiver of this document replied that he has received it.` |
| an inbound approval whose delivery state is `done` | `The Peppol receiver of this document replied that he has accepted it.` |
| an outbound response that was accepted by the proxy | `A Peppol response was sent to the Peppol Access Point declaring you <received, accepted or rejected> this document.` |
| an outbound response that the proxy did not accept | `A Peppol response declaring you <received, accepted or rejected> this document could not be sent to the Peppol Access Point.` |
| an outbound response whose call raised | `An error occurred while responding to this invoice's expeditor.` followed by a line break and `Status: <the code> - <the error text>` |
| the delivery state of an outbound response turns to error | `Peppol business response error: <the error text>` |

A rejection with no reason from the reason list is refused with `At least one reason must be given when rejecting a Peppol invoice.`

---

# 14. Advanced document fields

A capability package adds seven further text fields to an accounting document, each intended to feed one further element of the outgoing file: a contract document reference, a project reference, an originator document reference, a despatch document reference, one additional document reference, an accounting cost and a delivery location global location number. They are stored on the document and are available to a profile that maps them. The shipped profiles of this specification do not map them; a deployment that needs them maps them through the user defined extension mechanism of section 20.3 of [universal-business-language-mapping.md](universal-business-language-mapping.md).

---

# 15. The error catalogue of the proxy

An error answered by the proxy carries a code, an optional set of arguments, an optional subject and an optional message. Two code spaces exist: the standard codes and the codes of the message exchange standard. The rule for turning an error into a message is:

```
if the error carries a message exchange code that is not 4:
    text = the message of that code, from section 15.2
else if the error carries a standard code that is not 105 and not 106:
    text = the message of that code, from section 15.1
else:
    text = the message that came with the error, or "Not able to retrieve error message"
message = "Peppol Error [code=" + the standard code + "]: " + the subject + newline + text
```

The two standard codes `105` and `106` are excluded because their own detail text is far more useful than the generic sentence. A code that is in neither table produces `Unknown Peppol Error: <the whole error structure>`. When a message of the table expects arguments and the error supplies a different number of them, the placeholders are filled with the text `<unknown>`.

## 15.1 Standard codes

The sentences below are the ones shown to the user, with every short form written out in full words.

| Code | Message |
|---|---|
| 101 | `Something went wrong with your request` |
| 102 | `Proxy error, please contact support (missing "%s" - please make sure that it was loaded from the configuration panel)` |
| 103 | `The document could not be validated.` |
| 104 | `Could not find a schema definition with which to validate the document with identifier "%s".` |
| 105 | `The markup document is not valid according to the schema definition.` |
| 106 | `The markup document is not valid according to the schematron.` |
| 107 | `The markup document could not be canonicalized.` |
| 201 | `There was an issue with the Peppol Participant.` |
| 202 | `This user's is not a Peppol User. The proxy_type associated must be "peppol". This is required to create a peppol participant.` |
| 203 | `The Service Metadata Publisher associated to this participant is not reachable. web address: %s` |
| 204 | `The Peppol Participant service group found on his Service Metadata Provider is invalid.` |
| 205 | `No valid AS4 participant endpoint were found.` |
| 206 | `The Peppol Participant domain name system lookup resulted in a web address that does not belong to this server.` |
| 207 | `The Peppol Participant cannot receive this document type.` |
| 208 | `The Peppol Participant certificate is invalid.` |
| 301 | `No universal business language document was provided to the Peppol Outbound Service.` |
| 302 | `The universal business language document provided to the Peppol Outbound Service is malformed.` |
| 303 | `The document could not be sent to the Peppol Participant.` |
| 304 | `There was an issue with the Hermes Migration Token Collection Interface (MTCI).` |
| 501 | `There was an error with the incoming message.` |
| 502 | `This user is not registered on our Access Point.` |
| 503 | `Unexpected end of MIME multipart message.` |
| 504 | `Unable to verify the digest value.` |
| 505 | `The encrypted data reference the Security header intended for the "ebms" SOAP actor could not be decrypted by the Security Module.` |
| 506 | `The message does not comply with the AS4 policy.` |
| 507 | `The incoming message could not be decompressed.` |
| 701 | `There was an error with the Peppol Request` |
| 702 | `Your request is still being processed.` |
| 703 | `Your identification has not been approved for this action yet` |
| 704 | `An internal error occurred` |
| 705 | `You don't have enough credit` |
| 706 | `The document could not be found` |
| 707 | `You have reached the limit of documents you can send today. Retry later. Please contact the support if you think you need to increase that limit.` |
| 708 | `You are not authorized to change the contact email.` |

Two codes have a behaviour of their own beyond their message: code `702` means the proxy is still working and the message must be polled again without acknowledging it; code `207` on a business response means the recipient cannot receive a response, and turns the delivery state of that response into `not_serviced`.

## 15.2 Codes of the message exchange standard

The sentences below are the ones shown to the user, with every short form written out in full words where a short form would otherwise stand alone.

| Code | Message |
|---|---|
| 1 | `Although the message document is well formed and schema valid, some element/attribute contains a value that could not be recognized and therefore could not be used by the MSH.` |
| 2 | `Although the message document is well formed and schema valid, some element/attribute value cannot be processed as expected because the related feature is not supported by the MSH.` |
| 3 | `Although the message document is well formed and schema valid, some element/attribute value is inconsistent either with the content of other element/attribute, or with the processing mode of the MSH, or with the normative requirements of the ebMS specification.` |
| 4 | `Other` |
| 5 | `The MSH is experiencing temporary or permanent failure in trying to open a transport connection with a remote MSH.` |
| 6 | `There is no message available for pulling from this MPC at this moment.` |
| 7 | `The use of MIME is not consistent with the required usage in this specification.` |
| 8 | `Although the message document is well formed and schema valid, the presence or absence of some element/ attribute is not consistent with the capability of the MSH, with respect to supported features.` |
| 9 | `The exchange standard header is either not well formed as a markup document, or does not conform to the exchange standard packaging rules.` |
| 10 | `The ebMS header or another header (e.g. reliability, security) expected by the MSH is not compatible with the expected content, based on the associated P-Mode.` |
| 11 | `The MSH is unable to resolve an external payload reference (i.e. a Part that is not contained within the ebMS Message, as identified by a PartInfo/href URI).` |
| 20 | `An Intermediary MSH was unable to route an ebMS message and stopped processing the message.` |
| 21 | `An entry in the routing function is matched that assigns the message to an MPC for pulling, but the intermediary MSH is unable to store the message with this MPC` |
| 22 | `An intermediary MSH has assigned the message to an MPC for pulling and has successfully stored it. However the intermediary set a limit on the time it was prepared to wait for the message to be pulled, and that limit has been reached.` |
| 23 | `An MSH has determined that the message is expired and will not attempt to forward or deliver it.` |
| 30 | `The structure of a received bundle is not in accordance with the bundling rules.` |
| 31 | `A message unit in a bundle was not processed because a related message unit in the bundle caused an error.` |
| 40 | `A fragment is received that relates to a group that was previously rejected.` |
| 41 | `A fragment is received but more than one fragment message in a group of fragments specifies a value for this element.` |
| 42 | `A fragment is received but more than one fragment message in a group of fragments specifies a value for this element.` |
| 43 | `A fragment is received but more than one fragment message in a group of fragments specifies a value for this element.` |
| 44 | `A fragment is received but more than one fragment message in a group of fragments specifies a value for this element.` |
| 45 | `A fragment is received but more than one fragment message in a group of fragments specifies a value for a compression element.` |
| 46 | `A fragment is received but a previously received fragment message had the same values for GroupId and FragmentNum` |
| 47 | `The href attribute does not reference a valid MIME data part, MIME parts other than the fragment header and a data part are in the message. are added or the SOAP Body is not empty.` |
| 48 | `An incoming message fragment has a a value greater than the known FragmentCount.` |
| 49 | `A value is set for FragmentCount, but a previously received fragment had a greature value.` |
| 50 | `The size of the data part in a fragment message is greater than %s` |
| 51 | `More time than %s has passed since the first fragment was received but not all other fragments are received.` |
| 52 | `Message properties were present in the fragment SOAP header that were not specified in %s` |
| 53 | `The eb3:Message header copied to the fragment header does not match the eb3:Message header in the reassembled source message.` |
| 54 | `Not enough disk space available to store all (expected) fragments of the group.` |
| 55 | `An error occurred while decompressing the reassembled message.` |
| 60 | `A responding MSH indicates that it applies the alternate MEP binding to the response message.` |
| 101 | `The signature in the Security header intended for the "ebms" SOAP actor, could not be validated by the Security module.` |
| 102 | `The encrypted data reference the Security header intended for the "ebms" SOAP actor could not be decrypted by the Security Module.` |
| 103 | `The processor determined that the message's security methods, parameters, scope or other security policy-level requirements or agreements were not satisfied.` |
| 201 | `Some reliability function as implemented by the Reliability module, is not operational, or the reliability state associated with this message sequence is not valid.` |
| 202 | `Although the message was sent under Guaranteed delivery requirement, the Reliability module could not get assurance that the message was properly delivered, in spite of resending efforts.` |
| 301 | `A Receipt has not been received  for a message that was previously sent by the MSH generating this error.` |
| 302 | `A Receipt has been received  for a message that was previously sent by the MSH generating this error, but the content does not match the message content (e.g. some part has not been acknowledged, or the digest associated does not match the signature digest, for NRR).` |
| 303 | `An error occurred during the decompression.` |

The short forms that appear in those messages expand as follows: the message handler of the exchange standard, the message partition channel, the multipurpose internet mail extensions packaging, the markup language, the processing mode agreed between two handlers, the simple object access protocol envelope, the non repudiation of receipt evidence, the message exchange pattern binding, the web address, and the exchange standard itself. They are reproduced verbatim because a replacement must show the recipient the same sentence the network produced.
