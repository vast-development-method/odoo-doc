# Acceptance criteria

Given/When/Then scenarios a replacement must pass. Each criterion is independently verifiable and uses concrete values. Unless a scenario says otherwise, the setting is: one company "Acme" whose accounting currency is the euro (`EUR`); a provider record named "Dummy Provider" whose state is `test`, which is published, which supports one payment method named "Payment method" (code `unknown`) and which allows tokenization; a customer contact "Norbert Buyer" in Belgium; and a transaction amount of 1111.11 euro. The euro and the United States dollar are both active currencies.

---

# 1. Provider configuration

**PAY-AC-001. Enabling a provider activates its default payment methods.**
Given a disabled provider whose default payment method codes are `card`, `visa`, `mastercard`, `amex` and `discover`, and whose supported methods include all five, all inactive,
When an administrator sets its state to `enabled`,
Then the five methods become active, and the brands among them become active as well.

**PAY-AC-002. Enabling a provider that captures manually only activates compatible methods.**
Given a disabled provider whose `capture_manually` is true and whose default methods include one method with `support_manual_capture` equal to `none` and one with `partial`,
When an administrator sets its state to `test`,
Then only the method whose `support_manual_capture` is `partial` becomes active, and the other one stays inactive.

**PAY-AC-003. Disabling a provider deactivates the methods it alone supported.**
Given an enabled provider and an active payment method supported only by that provider,
When an administrator sets the provider's state to `disabled`,
Then the payment method becomes inactive, and its brands become inactive with it.

**PAY-AC-004. Disabling a provider does not deactivate a method that another provider still supports.**
Given two enabled providers that both support the same active payment method,
When one of them is disabled,
Then the payment method stays active.

**PAY-AC-005. Changing the state of a provider archives its tokens.**
Given a provider in state `test` with three active tokens,
When an administrator sets its state to `enabled`,
Then the three tokens become archived.
And Given the same provider in state `enabled`, When it is set to `disabled`, Then its active tokens become archived.
And Given a provider in state `disabled`, When it is set to `enabled`, Then no token is archived, because the provider was not previously in service.

**PAY-AC-006. Enabling a provider switches the post-processing job on.**
Given no provider of the database has a state other than `disabled`, therefore the post-processing job is inactive,
When an administrator sets one provider to `test`,
Then the post-processing job becomes active.

**PAY-AC-007. Disabling the last provider switches the post-processing job off.**
Given exactly one provider has a state other than `disabled` and the post-processing job is active,
When that provider is set to `disabled`,
Then the post-processing job becomes inactive.

**PAY-AC-008. Publishing follows the state in the form.**
Given a provider form open on a disabled, unpublished provider,
When the user selects `Enabled`,
Then `is_published` becomes true before the record is saved.
When the user then selects `Test Mode`,
Then `is_published` becomes false.

**PAY-AC-009. A disabled provider may not be published.**
Given a disabled, unpublished provider,
When an administrator clicks the publish toggle,
Then the operation is refused with `You cannot publish a disabled provider.` and `is_published` stays false.

**PAY-AC-010. The company of a provider with transactions is frozen.**
Given a provider that has at least one transaction,
When a user changes its company on the form,
Then the change is refused with `You cannot change the company of a payment provider with existing transactions.`

**PAY-AC-011. A provider that is put in service without its credentials is refused.**
Given a provider whose code declares a required credential field that is empty,
When an administrator sets its state to `enabled`,
Then the save is refused with `The following fields must be filled: ` followed by the label of that field.

**PAY-AC-012. A shipped provider may not be deleted.**
Given the shipped provider record named "Stripe",
When an administrator tries to delete it,
Then the deletion is refused with `You cannot delete the payment provider Stripe; disable it or uninstall it instead.`

**PAY-AC-013. A copy of a provider may be deleted.**
Given a provider created by duplicating a shipped one, which means it carries no external identifier,
When an administrator deletes it,
Then the deletion succeeds.

**PAY-AC-014. Manual capture may not be switched on while an incompatible method is active.**
Given an enabled provider with an active payment method whose `support_manual_capture` is `none`,
When an administrator switches `capture_manually` on,
Then the change is refused with `The following payment methods must be disabled in order to enable manual capture: ` followed by that method's name.

**PAY-AC-015. A demo provider may never be enabled.**
Given a provider whose code is `demo`,
When an administrator sets its state to `enabled`,
Then the change is refused with `Demo providers should never be enabled.`

**PAY-AC-016. Creating a company duplicates the installed providers.**
Given company "Acme" owns one provider whose package is installed,
When a new company "Beta" is created,
Then a copy of that provider exists in "Beta", with no credentials, state `disabled`, unpublished and with no journal.

**PAY-AC-017. Resetting the credentials disables and unpublishes the provider.**
Given an enabled, published Mercado Pago provider with an access token, a refresh token, a public key and tokenization allowed,
When an administrator runs Reset credentials,
Then the four credential fields are empty, `allow_tokenization` is false, the state is `disabled` and `is_published` is false, and every token of the provider is archived.

**PAY-AC-018. Uninstalling a connector package keeps the provider records.**
Given three provider records with the code `stripe`, one per company, and no Payment uses the accounting payment method `stripe`,
When the Stripe package is uninstalled,
Then the three records still exist with `code` equal to `none`, `state` equal to `disabled`, `is_published` false and the four template fields empty; their tokens are archived; and the accounting payment method `stripe` is deleted.

**PAY-AC-019. Uninstalling is refused when payments exist.**
Given at least one Payment uses the accounting payment method `stripe`,
When the Stripe package is uninstalled,
Then the uninstallation is refused with `You cannot uninstall this module as payments using this payment method already exist.`

**PAY-AC-020. A journal used by a live provider may not be deleted.**
Given the bank journal "Bank" is the journal of an enabled provider,
When an administrator deletes "Bank",
Then the deletion is refused with `You must first deactivate a payment provider before deleting its journal.` followed by `Linked providers: ` and the provider's name.

---

# 2. Provider availability

**PAY-AC-021. A published provider is available to every user.**
Given a published provider in state `test`,
When the compatible providers are computed for a public user, a portal user and an internal user,
Then the provider is in all three results.

**PAY-AC-022. An unpublished provider stays available to an internal user.**
Given an unpublished provider in state `test`,
When the compatible providers are computed for an internal user,
Then the provider is in the result.

**PAY-AC-023. An unpublished provider is hidden from a user who is not internal.**
Given an unpublished provider in state `test`,
When the compatible providers are computed for a portal user and for a public user,
Then the provider is in neither result.

**PAY-AC-024. A provider is available to a branch company.**
Given a provider whose company is "Acme" and a branch company "Acme North" whose parent is "Acme",
When the compatible providers are computed for "Acme North",
Then the provider is in the result.

**PAY-AC-025. A provider is available in its listed countries.**
Given a provider whose `available_country_ids` contains Belgium and a contact whose country is Belgium,
When the compatible providers are computed,
Then the provider is in the result.

**PAY-AC-026. A provider is not available outside its listed countries.**
Given a provider whose `available_country_ids` contains Belgium and a contact whose country is France,
When the compatible providers are computed,
Then the provider is not in the result and the availability report records the reason `incompatible country`.

**PAY-AC-027. A provider with no listed countries is available everywhere.**
Given a provider whose `available_country_ids` is empty and a contact in France,
When the compatible providers are computed,
Then the provider is in the result.

**PAY-AC-028. A maximum amount of zero never blocks a payment.**
Given a provider whose `maximum_amount` is 0 and a payment of 1111.11 euro,
When the compatible providers are computed,
Then the provider is in the result.

**PAY-AC-029. A payment below the maximum is allowed.**
Given a provider whose `maximum_amount` is 1500.00 euro, the company currency being the euro, and a payment of 1111.11 euro,
When the compatible providers are computed,
Then the provider is in the result.

**PAY-AC-030. A payment above the maximum is blocked.**
Given a provider whose `maximum_amount` is 1000.00 euro and a payment of 1111.11 euro,
When the compatible providers are computed,
Then the provider is not in the result and the availability report records `maximum amount exceeded`.

**PAY-AC-031. The maximum amount is compared in the company currency.**
Given a company whose currency is the euro, a provider whose `maximum_amount` is 500.00, and a rate of 0.90 euro per United States dollar,
When the compatible providers are computed for a payment of 600.00 United States dollars,
Then the converted amount is 540.00 euro, `500.00 ≥ 540.00` is false, and the provider is not in the result.
When the compatible providers are computed for a payment of 500.00 United States dollars instead,
Then the converted amount is 450.00 euro, `500.00 ≥ 450.00` is true, and the provider is in the result.

**PAY-AC-032. A validation operation ignores the maximum amount.**
Given a provider whose `maximum_amount` is 1.00 euro,
When the compatible providers are computed for a validation operation with an amount of 0,
Then the provider is in the result.

**PAY-AC-033. Currency restrictions.**
Given a provider whose `available_currency_ids` contains only the euro,
When the compatible providers are computed for a payment in euro,
Then the provider is in the result.
When they are computed for a payment in United States dollars,
Then the provider is not in the result and the availability report records `incompatible currency`.
Given instead a provider whose `available_currency_ids` is empty,
When the compatible providers are computed for either payment,
Then the provider is in the result both times.

**PAY-AC-034. Tokenization forced.**
Given a provider whose `allow_tokenization` is true,
When the compatible providers are computed with tokenization forced,
Then the provider is in the result.
And Given a provider whose `allow_tokenization` is false, Then it is removed with the reason `tokenization not supported`.

**PAY-AC-035. Tokenization required by the context.**
Given a payment context that requires tokenization,
When the compatible providers are computed,
Then a provider whose `allow_tokenization` is true is in the result and a provider whose `allow_tokenization` is false is removed.

**PAY-AC-036. A disabled provider is never available.**
Given a disabled provider,
When the compatible providers are computed for any user,
Then the provider is not in the result.

---

# 3. Payment methods

**PAY-AC-037. A method is available when an enabled provider supports it.**
Given an active primary method supported by a provider in state `test`,
When the compatible payment methods are computed,
Then the method is in the compatible methods.

**PAY-AC-038. A method is not available when its only provider is disabled.**
Given an active primary method whose only provider is disabled,
When the compatible payment methods are computed,
Then the method is not in the compatible methods and the reason recorded is `no supported provider available`.

**PAY-AC-039. A brand is never selectable.**
Given an active method whose `primary_payment_method_id` is the card method,
When the compatible payment methods are computed,
Then the method is not in the compatible methods, whatever the providers.

**PAY-AC-040. Country and currency restrictions of a method.**
Given a method whose `supported_country_ids` contains Belgium,
When the compatible payment methods are computed for a contact in Belgium,
Then the method is available.
When they are computed for a contact in France,
Then the method is removed with the reason `incompatible country`.
Given instead a method whose `supported_country_ids` is empty,
When the compatible payment methods are computed for any contact,
Then the method is available.
Given a method whose `supported_currency_ids` contains only the euro,
When the compatible payment methods are computed for a payment in euro,
Then the method is available.
When they are computed for a payment in United States dollars,
Then the method is removed with the reason `incompatible currency`.
Given instead a method whose `supported_currency_ids` is empty,
When the compatible payment methods are computed in any currency,
Then the method is available.

**PAY-AC-041. Tokenization and express checkout filters on a method.**
Given a method whose `support_tokenization` is true and another whose flag is false,
When the compatible payment methods are computed with tokenization forced,
Then the first is available and the second is removed with the reason `tokenization not supported`.
Given a method whose `support_express_checkout` is true and another whose flag is false,
When the compatible payment methods are computed for an express checkout,
Then the first is available and the second is removed with the reason `express checkout not supported`.

**PAY-AC-042. The availability report records every reason.**
Given one available provider, one unavailable provider, and five methods, each removed by a different filter (no supported provider available, incompatible country, incompatible currency, tokenization not supported, express checkout not supported),
When the compatible methods are computed with tokenization forced and express checkout on,
Then the report contains one entry per method with the matching reason, and each method entry lists its providers with their own availability.

**PAY-AC-043. A method may not be activated without an enabled provider.**
Given an inactive primary method whose providers are all disabled,
When an administrator sets `active` to true,
Then the change is refused with `This payment method needs a partner in crime; you should enable a payment provider supporting this method first.`

**PAY-AC-044. A brand may be activated for a provider that captures manually.**
Given a provider whose `capture_manually` is true, a primary method whose `support_manual_capture` is `partial`, and a brand of that method whose own `support_manual_capture` is `none`,
When the brand is activated,
Then the change succeeds, because the check reads the primary method's flag.

**PAY-AC-045. Detaching a method from a provider archives the tokens.**
Given an active method attached to a provider, with two active tokens that use that method and that provider,
When an administrator removes the provider from the method's `provider_ids`,
Then the two tokens become archived.

**PAY-AC-046. The placeholder method may not be deleted.**
Given the shipped payment method whose code is `unknown` and whose name is `Payment method`,
When an administrator tries to delete the method whose code is `unknown`,
Then the deletion is refused with `You cannot delete the default payment method.`

---

# 4. Payment tokens

**PAY-AC-047. A user cannot read another user's tokens.**
Given a token whose contact is Norbert Buyer,
When another portal user reads the tokens,
Then that token is not in the result.

**PAY-AC-048. A billing user reads every token.**
Given tokens of several contacts,
When a billing user reads the tokens,
Then all of them are in the result.

**PAY-AC-049. A token may not belong to the public contact.**
Given the public contact of the database, which is the contact every unauthenticated visitor browses as,
When a token is created with the public contact,
Then the creation is refused with `No token can be assigned to the public partner.`

**PAY-AC-050. Unarchiving needs an active provider.**
Given an archived token whose provider is disabled,
When an administrator sets `active` to true,
Then the change is refused with `You can't unarchive tokens linked to inactive payment methods or disabled providers.`

**PAY-AC-051. Unarchiving needs an active method.**
Given an archived token whose payment method is inactive and whose provider is enabled,
When an administrator sets `active` to true,
Then the change is refused with the same message.

**PAY-AC-052. The display name is padded.**
Given a token whose `payment_details` is `1234`,
When its display name is computed,
Then its display name is `•••• 1234`.

**PAY-AC-053. The display name is shortened to a maximum length.**
Given the same token and a maximum length of 6,
When its display name is computed,
Then the display name is `• 1234`.

**PAY-AC-054. The display name is not padded when padding is switched off.**
Given the same token and the padding flag false,
When its display name is computed,
Then the display name is `1234`.

**PAY-AC-055. A token without payment details still has a name.**
Given a token whose `payment_details` is empty and which was created on the 31st of January 2024,
When its display name is computed,
Then its display name is `Payment details saved on 2024/01/31`.

**PAY-AC-056. A customer archives their own token from the portal.**
Given a portal user with one active token,
When the user deletes it from the payment method management page,
Then the token's `active` becomes false, the token still exists, and the transactions that used it keep their link.

**PAY-AC-057. A customer cannot archive somebody else's token.**
Given a token belonging to another contact,
When a portal user calls the archive service with that token's identifier,
Then nothing happens and the token stays active.

---

# 5. Transaction creation and identity

**PAY-AC-058. A transaction created by an enabled provider is a production transaction.**
Given a provider in state `enabled`,
When a transaction is created,
Then its `is_live` is true.

**PAY-AC-059. A transaction created by a provider in test mode is not a production transaction.**
Given a provider in state `test`,
When a transaction is created,
Then its `is_live` is false.

**PAY-AC-060. The reference of the first transaction of a sequence carries no number.**
Given no transaction exists,
When a transaction is created with the prefix `S00042`,
Then its reference is `S00042`.

**PAY-AC-061. The reference is singularized on collision.**
Given the reference `S00042` already exists,
When a transaction is created with the prefix `S00042`,
Then its reference is `S00042-1`.
And Given `S00042`, `S00042-1` and `S00042-ref` exist, Then the next reference is `S00042-2`.

**PAY-AC-062. The reference is computed from the document.**
Given create values referencing the invoice `INV/2026/00017`,
When a transaction is created without a prefix,
Then its reference is `INV/2026/00017`.

**PAY-AC-063. The reference is unique across the database.**
Given an empty transaction table and two different providers in two different companies,
When two transactions are created with the same explicit reference,
Then the second creation is refused with `Reference must be unique!`

**PAY-AC-064. The contact is snapshotted.**
Given a contact whose street is `Huge Street`, whose second street line is `2/543`, whose city is `Sin City`, whose postal code is `1000`, whose country is Belgium and whose email address is `norbert.buyer@example.com`,
When a transaction is created for that contact,
Then `partner_address` is `Huge Street 2/543`, `partner_city` is `Sin City`, `partner_zip` is `1000`, `partner_country_id` is Belgium and `partner_email` is `norbert.buyer@example.com`.
And When the contact's street is changed afterwards, Then the transaction's `partner_address` does not change.

**PAY-AC-065. A transaction may not use an archived token.**
Given an archived token,
When a transaction is created with it,
Then the creation is refused with `Creating a transaction from an archived token is forbidden.`

**PAY-AC-066. A transaction may not be authorized by a provider that does not support it.**
Given a provider whose `support_manual_capture` is empty,
When a transaction of that provider is written with the state `authorized`,
Then the write is refused with `Transaction authorization is not supported by the following payment providers: ` followed by the provider name.

---

# 6. The state machine

**PAY-AC-067. A legal transition is applied.**
Given a transaction in state `draft`,
When it is set to `pending`,
Then its state is `pending`, its `last_state_change` is the current moment and `is_post_processed` is false.

**PAY-AC-068. An illegal transition is ignored.**
Given a transaction in state `done`,
When it is set to `pending`,
Then its state stays `done`, nothing is written, and a warning log entry is produced.

**PAY-AC-069. A transition to the current state is ignored.**
Given a transaction in state `done`,
When it is set to `done` again,
Then nothing is written and an informational log entry is produced.

**PAY-AC-070. An extra allowed source state is honoured.**
Given a transaction in state `done`,
When it is set to `cancel` with `done` declared as an extra allowed source state,
Then its state becomes `cancel`.

**PAY-AC-071. Every state change resets the post-processing flag.**
Given a transaction whose `is_post_processed` is true,
When its state changes,
Then `is_post_processed` becomes false.

**PAY-AC-072. A confirmed transaction can be reached from an error state.**
Given a transaction in state `error`,
When the provider later reports a successful payment,
Then the state becomes `done`.

---

# 7. Processing payment data

**PAY-AC-073. The amount check is skipped for a validation transaction.**
Given a transaction whose operation is `validation` and a connector that would return an empty amount and an empty currency,
When the payment data are validated,
Then the transaction does not become `error`.

**PAY-AC-074. A matching amount passes.**
Given a transaction of 1111.11 euro and payment data reporting 1111.11 and `EUR`,
When the amount and the currency of the payment data are checked against the transaction,
Then the transaction does not become `error`.

**PAY-AC-075. The check uses the payment precision, not the accounting precision.**
Given the euro configured with a rounding step of 0.001 and a transaction of 123.452 euro, and payment data reporting 123.45 and `EUR`,
When the amount and the currency of the payment data are checked against the transaction,
Then the transaction does not become `error`, because the transaction amount is rounded down to 123.45 before the comparison.

**PAY-AC-076. A refund amount is negated before the comparison.**
Given a refund transaction of −30.00 euro and payment data reporting 30.0 and `EUR`,
When the amount and the currency of the payment data are checked against the transaction,
Then the transaction does not become `error`.

**PAY-AC-077. A mismatching amount fails and blocks the updates.**
Given a transaction of 1111.11 euro in state `draft` and payment data reporting 100.00 and `EUR`,
When the data are processed,
Then the transaction becomes `error` with `The amount from the payment data doesn't match the one from the transaction.` and the connector's update step is not run, therefore the provider reference is still empty.

**PAY-AC-078. A mismatching currency fails.**
Given a transaction of 1111.11 euro and payment data reporting 1111.11 and `USD`,
When the amount and the currency of the payment data are checked against the transaction,
Then the transaction becomes `error` with `The currency from the payment data doesn't match the one from the transaction.`

**PAY-AC-079. Missing amount data fail.**
Given payment data reporting an empty amount,
When the amount and the currency of the payment data are checked against the transaction,
Then the transaction becomes `error` with `The amount or currency is missing from the payment data.`

**PAY-AC-080. A connector that opts out skips the check.**
Given a connector that returns no amount data at all,
When the amount and the currency of the payment data are checked against the transaction,
Then the transaction does not become `error` and the updates are applied.

**PAY-AC-081. Updates are applied to a transaction already in error when the amount is valid.**
Given a transaction in state `error` and valid payment data reporting a success,
When the data are processed,
Then the updates are applied and the transaction becomes `done`.

**PAY-AC-082. Payment data with no matching transaction are ignored.**
Given payment data whose reference matches no transaction,
When they are processed,
Then nothing is written and a warning log entry `No transaction found matching reference <reference>.` is produced.

**PAY-AC-083. Payment data with no reference are ignored.**
Given payment data from which the connector can extract no reference,
When the payment data are processed,
Then a warning log entry about the missing reference is produced and nothing is written.

---

# 8. Tokenization

**PAY-AC-084. A confirmed transaction that asked for a token creates one.**
Given a transaction whose `tokenize` is true and whose provider allows tokenization,
When the payment data confirm it,
Then a Payment Token is created with the transaction's provider, payment method and contact, plus the connector's values; the transaction's `token_id` points at it; and `tokenize` becomes false.

**PAY-AC-085. An authorized transaction that asked for a token creates one.**
Given a transaction whose `tokenize` is true and whose provider allows tokenization,
When the payment data authorize it, moving it to the state `authorized`,
Then a Payment Token is created with the transaction's provider, payment method and contact, plus the connector's values; the transaction's `token_id` points at it; and `tokenize` becomes false.

**PAY-AC-086. A transaction that did not ask for a token creates none.**
Given a transaction whose `tokenize` is false,
When the payment data confirm it,
Then no token is created.

**PAY-AC-087. A repeated notification does not create a second token.**
Given a confirmed transaction that already created a token,
When the same notification arrives again,
Then no second token is created, because `tokenize` is false.

**PAY-AC-088. A customer cannot force tokenization.**
Given a provider whose `allow_tokenization` is false,
When the browser calls the transaction service with the tokenization request flag set,
Then the created transaction's `tokenize` is false.

**PAY-AC-089. A method that does not support tokenization prevents it.**
Given a provider that allows tokenization and a payment method whose `support_tokenization` is false,
When the browser asks for tokenization,
Then the created transaction's `tokenize` is false.

**PAY-AC-090. The tokenization checkbox is offered to both logged-in and logged-out customers.**
Given a provider that allows tokenization and does not require it,
When the payment form is rendered for a logged-in user and for a public visitor,
Then the checkbox that saves the payment method is present in both cases.

---

# 9. Capture, void and refund

**PAY-AC-091. Capturing an authorized transaction creates a child transaction.**
Given a transaction of 1111.11 euro in state `authorized` whose provider captures manually and supports only full capture,
When a billing user captures it,
Then exactly one child transaction is created with `source_transaction_id` equal to it, `amount` equal to 1111.11, `operation` equal to the source's operation, and the reference `P-<source reference>`.

**PAY-AC-092. Voiding an authorized transaction creates a child transaction.**
Given the same transaction,
When a billing user voids it,
Then exactly one child transaction is created with the amount 1111.11 and the reference prefix `P-`.

**PAY-AC-093. A refund child carries a negative amount and the refund operation.**
Given a confirmed transaction of 1111.11 euro whose provider supports partial refunds,
When a full refund is requested,
Then a child transaction is created with the reference `R-<source reference>`, the amount −1111.11, the same currency, the operation `refund` and the source transaction set.

**PAY-AC-094. A partial capture child carries the requested amount.**
Given a transaction of 1111.11 euro in state `authorized`,
When a child is created for 11.11,
Then that child's reference is `P-<source reference>`, its amount is 11.11, its currency is the source's currency, its operation equals the source's operation and its source transaction is set.

**PAY-AC-095. Only an authorized transaction may be voided.**
Given a transaction in state `done`,
When a billing user voids it,
Then the operation is refused with `Only authorized transactions can be voided.`

**PAY-AC-096. Only a confirmed transaction may be refunded.**
Given a transaction in state `authorized`,
When a billing user refunds it,
Then the operation is refused with `Only confirmed transactions can be refunded.`

**PAY-AC-097. A user without write access may not capture, void or refund.**
Given a portal user who may read a transaction but not write it,
When that user calls the capture, the void or the refund operation,
Then the call is refused by the access layer.

**PAY-AC-098. A user with write access may capture, void and refund.**
Given a billing user,
When that user calls the three operations on a transaction in the right state,
Then they succeed.

**PAY-AC-099. A capture that covers only part of the amount leaves the source authorized.**
Given a source transaction of 1111.11 euro in state `authorized`,
When a capture child of 100.00 is confirmed,
Then the source transaction is still `authorized`.

**PAY-AC-100. A capture and a void that together cover the amount confirm the source.**
Given the same source transaction with a confirmed capture child of 100.00,
When a void child of 1011.11 is canceled,
Then the source transaction becomes `done`.

**PAY-AC-101. Void children that cover the whole amount cancel the source.**
Given a source transaction of 1111.11 euro in state `authorized`,
When a single void child of 1111.11 is canceled,
Then the source transaction becomes `cancel`.

**PAY-AC-102. Confirming a child triggers the source re-evaluation.**
Given a source transaction with one child,
When the child is confirmed,
Then the source re-evaluation runs exactly once.

**PAY-AC-103. Cancelling a child triggers the source re-evaluation.**
Given a source transaction with one child,
When the child is cancelled,
Then the source re-evaluation runs exactly once.

**PAY-AC-104. Refunds never change the state of their source.**
Given a confirmed transaction of 1111.11 euro,
When a refund child of −30.00 is confirmed,
Then the source transaction stays `done`.

**PAY-AC-105. The refunds count.**
Given a confirmed transaction with two refund children, one confirmed and one in error,
When its `refunds_count` is computed,
Then its `refunds_count` is 2, because the count ignores the state.

**PAY-AC-106. A request to a disabled provider is refused.**
Given a transaction whose provider is disabled,
When a capture, a void, a refund or a token charge is attempted,
Then the operation is refused with `Making a request to the provider is not possible because the provider is disabled.`

**PAY-AC-107. The feedback notification reports success.**
Given a capture that produced no child in state `error`,
When the capture operation builds its feedback notification,
Then the notification is of type success with the message `Your payment operation has been successfully submitted.`

**PAY-AC-108. The feedback notification reports a failure.**
Given a capture whose child ended in state `error` with the reference `P-S00043`,
When the capture operation builds its feedback notification,
Then the notification is of type danger with the message `Your payment operation could not be completed for following transactions: P-S00043`.

---

# 10. The capture wizard

**PAY-AC-109. Two successive partial captures close the source transaction.**
Given a provider whose `capture_manually` is true and whose `support_manual_capture` is `partial`, and a source transaction of 1111.11 euro in state `authorized`,
When a capture wizard is opened on it, `amount_to_capture` is set to 511.11 and the capture is confirmed,
Then one child transaction of 511.11 is created in state `draft`.
When that child is confirmed,
Then the source stays `authorized`.
When a second capture wizard is opened on the same source and confirmed with its default amount,
Then a second child is created whose amount is 600.00, because the available amount is `1111.11 − 511.11 − 0.00`.
When that second child is confirmed,
Then the sum of the children's amounts equals 1111.11 and the source transaction becomes `done`.

**PAY-AC-110. The amount to capture must be positive and within the maximum.**
Given a wizard whose `available_amount` is 1111.11 euro,
When `amount_to_capture` is set to 0,
Then the change is refused with `The amount to capture must be positive and cannot be superior to €1,111.11.`
And When it is set to 1200.00, Then it is refused with the same message.

**PAY-AC-111. A partial capture is refused when the provider only supports full captures.**
Given a provider whose `support_manual_capture` is `full_only` and a wizard whose `available_amount` is 1111.11,
When `amount_to_capture` is set to 500.00,
Then the change is refused with `Some of the transactions you intend to capture can only be captured in full. Handle the transactions individually to capture a partial amount.`

**PAY-AC-112. Partial capture support is computed through the primary method.**
Given a provider whose `capture_manually` is true and whose `support_manual_capture` is `partial`, a primary method whose `support_manual_capture` is `partial`, a brand of that method, and a transaction that uses the brand,
When a capture wizard is opened on that transaction,
Then `support_partial_capture` is true.

**PAY-AC-113. The void-the-rest option disappears when there is no rest.**
Given a wizard whose `available_amount` is 1111.11 and whose `amount_to_capture` is 1111.11,
Then `has_remaining_amount` is false and `void_remaining_amount` is false.
When `amount_to_capture` is lowered to 500.00, Then `has_remaining_amount` becomes true.

**PAY-AC-114. The wizard warns about unanswered requests.**
Given a source transaction with one child still in state `draft`,
When a capture wizard is opened on it,
Then `has_draft_children` is true.

**PAY-AC-115. Capture allocation across two source transactions.**
Given two authorized transactions of 120.00 and 200.00 euro, both childless, and a wizard with `amount_to_capture` equal to 150.00 and the void checkbox unticked,
When the capture is confirmed,
Then one child of 120.00 is created on the first transaction and one child of 30.00 on the second, and no void child is created.

**PAY-AC-116. Capture and void in one step.**
Given one authorized transaction of 120.00 euro, `amount_to_capture` equal to 80.00 and the void checkbox ticked,
When the capture is confirmed,
Then one capture child of 80.00 and one void child of 40.00 are created.

---

# 11. The refund wizard

**PAY-AC-117. The refundable amount after one refund.**
Given a posted Payment of 120.00 euro produced by a confirmed transaction, and one posted refund Payment of 30.00 linked to it,
When the refund wizard is opened,
Then `amount_available_for_refund` is 90.00, `refunded_amount` is 30.00 and the default `amount_to_refund` is 90.00.

**PAY-AC-118. The amount to refund must be positive and within the maximum.**
Given `amount_available_for_refund` equal to 90.00,
When `amount_to_refund` is set to 0,
Then the change is refused with `The amount to be refunded must be positive and cannot be superior to 90.0.`
When `amount_to_refund` is set to 100.00 instead,
Then the change is refused with the same message.
The amount in this message is a plain decimal number, not a formatted monetary amount: 90 renders as `90.0`, with no currency symbol and no thousands separator. The capture wizard's parallel message (PAY-AC-110) does format its amount in the currency, as `€1,111.11`. The difference between the two is intentional and is stated in PAY-RULE-056.

**PAY-AC-119. A pending refund is reported.**
Given a refund transaction of the same source transaction in state `pending`,
When the refund wizard is opened,
Then `has_pending_refund` is true.

**PAY-AC-120. Refund capability is the weaker of the two.**
Given a provider whose `support_refund` is `partial` and a primary method whose `support_refund` is `full_only`,
When the refund wizard computes its own `support_refund`,
Then it is `full_only`.
Given instead a method whose `support_refund` is `none`,
When the refund wizard computes its own `support_refund`,
Then it is `none` and the refundable amount is 0.

---

# 12. Post-processing

**PAY-AC-121. Post-processing runs once per state.**
Given a confirmed transaction whose `is_post_processed` is false,
When the status service is polled,
Then the post-processing consequences run and `is_post_processed` becomes true.
When the service is polled again, Then the consequences do not run again.

**PAY-AC-122. The scheduled job picks up an unprocessed transaction.**
Given a transaction confirmed two days ago whose `is_post_processed` is false,
When the post-processing job runs,
Then the transaction is processed.

**PAY-AC-123. The scheduled job abandons an old transaction.**
Given a transaction whose `last_state_change` is five days old and whose `is_post_processed` is false,
When the job runs,
Then the transaction is not selected.

**PAY-AC-124. A conflict leaves the transaction for the next run.**
Given the database refuses the commit of one transaction's post-processing because of a concurrency conflict,
When the job runs,
Then that transaction stays unprocessed and the other transactions of the run are still processed.

**PAY-AC-125. A failure in one transaction does not stop the job.**
Given the post-processing of one transaction raises a failure,
When the job runs,
Then the failure is logged with the transaction reference, the work is rolled back, and the following transactions are still processed.

---

# 13. Accounting consequences

**PAY-AC-126. A confirmed transaction produces a posted Payment.**
Given a confirmed transaction of 120.00 euro linked to the posted invoice `INV/2026/00017` of 120.00,
When it is post-processed,
Then a Payment is created with `amount` 120.00, `payment_type` `inbound`, `partner_type` `customer`, the commercial contact of the transaction's contact, the provider's journal, the provider's payment method line, `memo` equal to `<reference> - <provider reference>` and `payment_transaction_id` set; the Payment is posted; and the message `The payment related to transaction <link> has been posted: <link>` is logged.

**PAY-AC-127. The Payment settles the invoice.**
Given the previous scenario, in which the Payment of 120.00 euro was created and posted for the invoice `INV/2026/00017`,
When the post-processing reconciles the Payment with the invoices of the transaction,
Then the receivable journal items of the Payment and of the invoice are reconciled and the invoice's residual amount becomes 0.00.

**PAY-AC-128. A draft invoice is posted before the Payment is created.**
Given a confirmed transaction linked to a draft invoice,
When it is post-processed,
Then the invoice is posted first.

**PAY-AC-129. A validation transaction produces no Payment.**
Given a confirmed transaction whose operation is `validation`,
When it is post-processed,
Then no Payment is created.

**PAY-AC-130. A split source transaction produces no Payment.**
Given a source transaction of 120.00 that became `done` because a capture child of 80.00 is `done` and a void child of 40.00 is `cancel`,
When the source transaction is post-processed,
Then no Payment is created for it, and exactly one Payment of 80.00 exists, produced by the capture child.

**PAY-AC-131. A capture child settles the invoice of its source transaction.**
Given a source transaction linked to `INV/2026/00017` and a capture child of 80.00 that is confirmed,
When the child is post-processed,
Then its Payment is reconciled against `INV/2026/00017`.

**PAY-AC-132. A refund produces an outbound Payment.**
Given a confirmed refund transaction of −30.00 euro,
When it is post-processed,
Then a Payment is created with `amount` 30.00 and `payment_type` `outbound`, whose `source_payment_id` is the Payment of the source transaction, and nothing is reconciled automatically.

**PAY-AC-133. A cancelled transaction cancels its Payment.**
Given a confirmed transaction with a posted Payment,
When the transaction moves to `cancel`,
Then its Payment is cancelled.

**PAY-AC-134. Early payment discount.**
Given an invoice of 100.00 euro with a 2 percent early payment discount still valid, and a confirmed transaction of 98.00 euro linked to it,
When the transaction is post-processed,
Then the Payment carries write-off lines totalling 2.00 and the invoice becomes fully paid.

**PAY-AC-135. A Payment produced by a transaction cannot be partially reconciled.**
Given a Payment linked to a Payment Transaction,
When a bank transaction is matched against it for a smaller amount,
Then the partial match is refused.

---

# 14. Sales consequences

**PAY-AC-136. A pending transaction marks the quotation as sent.**
Given a draft quotation `S00042` and a pending transaction linked to it,
When the transaction is post-processed,
Then the quotation moves to the "sent" state and the payment-succeeded email is sent.

**PAY-AC-137. A confirmed transaction that covers the required amount confirms the order.**
Given a quotation `S00042` of 120.00 that requires full prepayment, that does not require a signature, and a confirmed transaction of 120.00 linked only to it,
When the transaction is post-processed,
Then the order is confirmed.

**PAY-AC-138. A transaction that covers less than the required amount does not confirm the order.**
Given the same quotation and a confirmed transaction of 50.00,
When the transaction is post-processed,
Then the order is not confirmed and the payment-succeeded email is sent instead.

**PAY-AC-139. A grouped payment never confirms an order.**
Given a confirmed transaction linked to two quotations,
When it is post-processed,
Then neither order is confirmed.

**PAY-AC-140. An order that requires a signature is not confirmed by a payment.**
Given a quotation that requires a signature and a confirmed transaction that covers it,
When the transaction is post-processed,
Then the order is not confirmed.

**PAY-AC-141. Automatic invoicing of a fully paid order.**
Given the automatic invoicing setting is on and a confirmed transaction of 120.00 for an order of 120.00,
When the transaction is post-processed,
Then a final invoice of 120.00 is created, linked to the transaction, posted, and paid by the Payment.

**PAY-AC-142. Automatic invoicing of a partially paid order.**
Given the automatic invoicing setting is on and a confirmed transaction of 50.00 for a confirmed order of 120.00,
When the transaction is post-processed,
Then a down payment invoice of 50.00 is created and linked to the transaction.

---

# 15. Portal flows

**PAY-AC-143. Direct checkout works for every kind of user.**
Given a provider that builds an inline form,
When a public visitor, a portal user and an internal user each pay 1111.11 euro through the payment form,
Then in each case a transaction is created with `operation` equal to `online_direct` and the customer reaches the confirmation page.

**PAY-AC-144. Redirect checkout works for every kind of user.**
Given a provider that redirects the customer to its own page,
When a public visitor, a portal user and an internal user each pay 1111.11 euro through the payment form,
Then in each case a transaction is created with `operation` equal to `online_redirect`, the processing values carry the rendered redirect form, and the customer reaches the confirmation page.

**PAY-AC-145. A direct payment sends no token charge request.**
Given a direct payment,
When the transaction is created,
Then no payment request is sent by the transaction creation itself.

**PAY-AC-146. A redirect payment sends no token charge request.**
Given a redirect payment,
When the transaction is created,
Then no payment request is sent by the transaction creation itself.

**PAY-AC-147. A token payment sends exactly one charge request.**
Given a payment with a saved token,
When the transaction is created,
Then exactly one payment request is sent.

**PAY-AC-148. Tokenizing from the portal creates a token.**
Given a logged-in customer and a provider that allows tokenization,
When the customer pays and ticks the checkbox that saves the payment method,
Then a token is created once the transaction is confirmed.

**PAY-AC-149. Validating a payment method creates a token without paying.**
Given a logged-in customer on the payment method management page,
When the customer saves a payment method,
Then a transaction with `operation` equal to `validation` is created with the provider's validation amount and currency, and a token is created when it succeeds.

**PAY-AC-150. A pay link with no existing contact redirects to the login page.**
Given a public visitor and a pay link whose contact identifier does not exist,
When the visitor opens the link,
Then the visitor is redirected to the login page with the current full path as the return address.

**PAY-AC-151. A pay request with a wrong access token is refused.**
Given a pay link whose access token does not match the contact, the amount and the currency,
When the visitor opens it,
Then the answer is "not found".

**PAY-AC-152. A transaction request with a wrong access token is refused.**
Given a payment form opened for a contact, an amount and a currency,
When the transaction service is called with a wrong access token,
Then the answer is "forbidden" and no transaction is created.

**PAY-AC-153. A transaction request with an unexpected parameter is refused.**
Given a payment form opened for a contact, an amount and a currency, and a valid access token,
When the transaction service is called with a parameter name that is not in the allowed list,
Then the answer is a bad request carrying `The following parameters are not whitelisted: ` followed by the offending name, and no transaction is created.

**PAY-AC-154. A transaction request with an unknown flow is refused.**
Given a payment form opened for a contact, an amount and a currency, and a valid access token,
When the transaction service is called with a flow that is neither `redirect`, `direct` nor `token`,
Then no transaction is created.

**PAY-AC-155. Paying with somebody else's token is refused.**
Given a token whose owner's commercial contact differs from the paying contact's commercial contact,
When the transaction service is called with that token,
Then the call is refused with `You do not have access to this payment token.`

**PAY-AC-156. Paying with a non-existent token is refused.**
Given a payment form opened for a contact, an amount and a currency, and a valid access token,
When the transaction service is called with a token identifier that does not exist,
Then no transaction is created.

**PAY-AC-157. A pay link with an inactive currency is refused.**
Given a pay link whose currency is archived,
When the visitor opens it,
Then the answer is "not found".

**PAY-AC-158. The tokens of a disabled provider are not offered.**
Given a customer with a token whose provider is disabled,
When the payment form is rendered for a payment,
Then that token is not offered.
And When the payment method management page is rendered, Then that token is offered, in order that the customer can delete it.

**PAY-AC-159. The confirmation page checks its access token.**
Given a transaction and a confirmation address whose access token does not match its contact, amount and currency,
When the address is opened,
Then the answer is "not found".

**PAY-AC-160. The status page without a session shows the fallback text.**
Given a visitor whose session holds no monitored transaction,
When the status page is opened,
Then it shows `Your payment is on its way!`, `You should receive an email confirming your payment within a few minutes.` and `Don't hesitate to contact us if you don't receive it.`

**PAY-AC-161. The status page shows the provider's message per state.**
Given a provider whose done message is `Thanks, we got it.` and a confirmed transaction,
When the status page is shown,
Then it displays that message. And for a pending transaction it displays the provider's pending message; for a canceled one, the cancel message; for an authorized one, the authorization message.

---

# 16. Multi-company

**PAY-AC-162. A customer may not pay a document of another company.**
Given a contact whose company is "Acme" and a payment page opened for company "Beta",
When the page is rendered,
Then no payment form is shown and the page displays `Please switch to company Beta to make this payment.`

**PAY-AC-163. A contact with no company may pay in any company.**
Given a contact with no company and a payment page for company "Beta",
When the payment page is opened for that company,
Then the payment form is shown.

**PAY-AC-164. A billing user reads the tokens of every contact of the accessible companies.**
Given tokens in company "Acme",
When a billing user of "Acme" reads them,
Then all of them are returned.

**PAY-AC-165. A customer archives a token of another company.**
Given a logged-in customer whose active company is "Beta" and a token whose company is "Acme" and whose contact is the customer's contact,
When the customer deletes the token from the portal,
Then the token becomes archived.

---

# 17. Custom and demo providers

**PAY-AC-166. A wire transfer payment ends pending.**
Given an enabled provider whose code is `custom` and whose custom mode is `wire_transfer`,
When a customer chooses it and the browser posts to the custom processing address,
Then the transaction becomes `pending`, no amount check is run, and no "received" message is logged on the documents.

**PAY-AC-167. The communication falls back to the transaction reference.**
Given a wire transfer transaction with no linked invoice and no linked sales order and the reference `tx-20260911140509`,
When the customer-facing communication of the transaction is computed,
Then the communication shown to the customer is `tx-20260911140509`.

**PAY-AC-168. The communication of an invoice is its payment reference.**
Given a wire transfer transaction linked to an invoice whose payment reference is `RF18 5390 0754 7034`,
When the customer-facing communication of the transaction is computed,
Then the communication is `RF18 5390 0754 7034`.

**PAY-AC-169. The communication of a sales order is its reference.**
Given a wire transfer transaction linked to a sales order whose reference is `S00042`,
When the customer-facing communication of the transaction is computed,
Then the communication is `S00042`.

**PAY-AC-170. The "sent" message of a custom provider names the provider.**
Given a wire transfer provider named `Wire Transfer`,
When a transaction is created for it,
Then the message logged is `The customer has selected Wire Transfer to make the payment.`

**PAY-AC-171. The demo connector reaches every state.**
Given a demo provider in state `test`,
When a customer picks the simulated state `pending`, `done`, `cancel` or `error`,
Then the transaction reaches `pending`, `done`, `cancel` or `error` respectively, the last one with the message `You selected the following demo payment status: error`.

**PAY-AC-172. The demo connector authorizes when manual capture is on.**
Given a demo provider whose `capture_manually` is true,
When the simulated state is `done` and the event is not a manual capture,
Then the transaction becomes `authorized`.
And When a capture request is simulated, Then the transaction becomes `done`.

**PAY-AC-173. The demo connector stores the simulated state on the token.**
Given a demo transaction that asks for a token with the simulated state `done`,
When it is processed,
Then a token is created whose simulated state is `done` and whose provider reference is the literal text `fake provider reference`.

**PAY-AC-174. A token of the demo connector propagates its simulated state.**
Given a demo token whose simulated state is `error`,
When a transaction is charged with it,
Then the transaction ends in `error`.

---

# 18. Notification authenticity

The following criteria apply to every connector that signs its notifications: Adyen, Amazon Payment Services, AsiaPay, Buckaroo, ECPay, Flutterwave, Nuvei, Paymob, PayU, Razorpay, Redsys, Stripe, Toss Payments, Worldline and Xendit.

**PAY-AC-175. A notification triggers a signature check.**
Given a webhook notification for an existing transaction,
When it is received,
Then the signature check runs before anything is written.

**PAY-AC-176. A valid signature is accepted.**
Given a notification whose signature equals the one recomputed from the payload and the provider's secret,
When the notification is received,
Then the notification is processed.

**PAY-AC-177. A missing signature is refused.**
Given a notification with no signature,
When the notification is received,
Then the answer is "forbidden", a warning is logged and nothing is written.

**PAY-AC-178. An invalid signature is refused.**
Given a notification whose signature differs from the recomputed one,
When the notification is received,
Then the answer is "forbidden", a warning is logged and nothing is written.

**PAY-AC-179. A notification for an unknown transaction is acknowledged without a signature check.**
Given a Stripe notification whose reference matches no transaction,
When the notification is received,
Then the notification is acknowledged with an empty body and no signature check is run.

**PAY-AC-180. A Stripe notification with a missing webhook secret is refused.**
Given a Stripe provider with no webhook secret,
When a notification arrives,
Then the answer is "forbidden".

**PAY-AC-181. A Stripe notification with a missing or outdated timestamp is refused.**
Given a notification whose signature header carries no timestamp, or a timestamp more than ten minutes old,
When the notification is received,
Then the answer is "forbidden".

**PAY-AC-182. A signature is computed on lower-cased keys.**
Given a Buckaroo notification whose parameter names use mixed case,
When the signature of the notification is verified,
Then the recomputed signature uses the lower-cased keys for sorting and the check succeeds.

**PAY-AC-183. A reference mismatch after fetching the authoritative data is refused.**
Given a Mercado Pago, Iyzico, Stripe, Razorpay or Paymob notification whose authoritative data carry a reference different from the matched transaction's reference,
When the notification is processed and the authoritative data are fetched from the provider,
Then the answer is "forbidden" and nothing is written.

**PAY-AC-184. An abandoned Nuvei payment is verified with an access token.**
Given a Nuvei return address carrying the transaction reference and an error access token,
When the address is opened with a valid token,
Then the transaction is canceled with the state message `The customer left the payment page.`
And When the token is invalid, Then the answer is "forbidden".

---

# 19. Connector reference and amount constraints

**PAY-AC-185. Amazon Payment Services references use only allowed characters.**
Given a transaction created for that provider with the document prefix `INV/2026/00017`,
When the transaction is created and its reference is generated,
Then the reference contains only letters, digits, hyphens and underscores.

**PAY-AC-186. AsiaPay references are at most 35 characters.**
Given a transaction created for that provider with a long document prefix,
When the transaction is created and its reference is generated,
Then the reference is at most 35 characters, and its first part is the document prefix truncated which leaves room for the fourteen-digit moment and the separator.

**PAY-AC-187. ECPay references are at most 20 characters and alphanumeric.**
Given a transaction created for that provider,
When the transaction is created and its reference is generated,
Then the reference is at most 20 characters and contains only letters and digits.

**PAY-AC-188. Redsys references are between 9 and 12 characters and alphanumeric.**
Given a transaction created for that provider,
When the transaction is created and its reference is generated,
Then the reference matches those bounds; a collision appends the letter `S` and a sequence number.

**PAY-AC-189. Toss Payments references are between 6 and 64 characters.**
Given a transaction created for that provider,
When the transaction is created and its reference is generated,
Then the reference matches those bounds and contains only letters, digits, hyphens and underscores.

**PAY-AC-190. Worldline references are at most 30 characters.**
Given a transaction whose generic reference would be 40 characters,
When the transaction is created and its reference is generated,
Then the reference is rebuilt from a two-letter prefix and the fourteen-digit moment and is at most 30 characters.

**PAY-AC-191. An amount is sent in minor units.**
Given a transaction of 1111.11 euro for a connector that uses minor units,
When the outbound payment request is built,
Then the amount sent is the integer 111111.

**PAY-AC-192. An integer-only method forces a whole amount.**
Given a Nuvei transaction of 120.75 United States dollars whose method is the Chilean bank redirect,
When the outbound payment request is built,
Then the amount sent is 120 and the amount check is run with zero decimals.

**PAY-AC-193. Xendit treats every currency as having no decimals.**
Given a Xendit transaction of 120.75 Indonesian rupiah,
When the outbound payment request is built,
Then the amount sent is 120.

**PAY-AC-194. Mercado Pago rounds down the currencies that carry no decimals.**
Given a Mercado Pago transaction of 120.75 Colombian pesos,
When the outbound payment request is built,
Then the amount sent is 120.

**PAY-AC-195. Adyen uses its own precision for the Indonesian rupiah.**
Given an Adyen transaction of 120.00 Indonesian rupiah,
When the outbound payment request is built,
Then the amount sent is the integer 120, not 12000.

**PAY-AC-196. A currency restriction blocks an incompatible provider.**
Given an AsiaPay provider whose single supported currency is the Hong Kong dollar,
When the compatible providers are computed for a payment in euro,
Then the provider is not in the result.
The same holds for Flutterwave, Mercado Pago, Nuvei, Razorpay and Xendit with a currency outside their supported list.

**PAY-AC-197. Only one currency may be selected on a single-currency account.**
Given an AsiaPay provider whose state is `test`,
When two currencies are selected,
Then the change is refused with `Only one currency can be selected by AsiaPay account.`
The same holds for Authorize.Net with its own message and for Paymob with `Only one currency can be selected per Paymob account.`

**PAY-AC-198. An unsupported currency is refused on a restricted account.**
Given an ECPay provider,
When a currency other than the new Taiwan dollar is selected,
Then the change is refused with `ECPay only supports TWD.`
Given a Toss Payments provider, When a currency other than the South Korean won is selected, Then the change is refused with `Currencies other than KRW are not supported.`

---

# 20. Connector state mapping

**PAY-AC-199. A successful notification confirms the transaction.**
Given a draft transaction and a notification reporting the connector's success status,
When the notification is processed,
Then the transaction becomes `done` and its provider reference is written from the notification.
This applies to Amazon Payment Services status `14`, AsiaPay success code `0`, Buckaroo code 190, DPO codes `000` and `002`, ECPay codes `1`, `2` and `10100073`, Flutterwave status `successful`, Iyzico status `SUCCESS`, Mercado Pago status `approved`, Mollie status `paid`, Nuvei status `approved`, Paymob success flag `true`, PayPal status `COMPLETED`, PayU status `success`, Razorpay status `captured`, Redsys codes `0000` to `0099`, Stripe status `succeeded`, Toss Payments status `DONE`, Worldline status `CAPTURED` and Xendit statuses `SUCCEEDED`, `PAID` and `CAPTURED`.

**PAY-AC-200. A failure notification sets the transaction in error.**
Given a draft transaction and a notification reporting the connector's failure status,
When the notification is processed,
Then the transaction becomes `error` with the connector's message.

**PAY-AC-201. An authorization notification authorizes the transaction when manual capture is on.**
Given an Adyen, Authorize.Net, Razorpay or Stripe provider whose `capture_manually` is true,
When a notification reporting an authorization is processed,
Then the transaction becomes `authorized`.
Given the same notification and an Adyen provider whose `capture_manually` is false,
When it is processed,
Then the transaction becomes `done`.

**PAY-AC-202. A capture notification confirms the transaction.**
Given an Adyen provider with manual capture on, an authorized transaction and a capture notification,
When the notification is processed,
Then the transaction becomes `done`.

**PAY-AC-203. A cancellation notification cancels the transaction.**
Given an Adyen cancellation notification whose success flag is true,
When the notification is processed,
Then the transaction becomes `cancel`.

**PAY-AC-204. A failed authorization notification leaves the transaction in draft.**
Given an Adyen authorization notification whose success flag is false,
When the notification is processed,
Then the transaction stays `draft`, because the item is skipped.

**PAY-AC-205. A failed capture notification leaves the source transaction authorized.**
Given an Adyen capture notification whose success flag is false, for a source transaction,
When the notification is processed,
Then the transaction stays `authorized` and the message `The capture of the transaction <reference> failed.` is logged.

**PAY-AC-206. A failed cancellation notification leaves the source transaction authorized.**
Given an Adyen cancellation notification whose success flag is false, for a source transaction in the state `authorized`,
When it is processed,
Then the transaction stays `authorized` and the message `The void of the transaction <reference> failed.` is logged.

**PAY-AC-207. A failed refund notification sets the refund transaction in error.**
Given an Adyen refund notification whose success flag is false, for a refund child transaction,
When the notification is processed,
Then that refund transaction becomes `error`.

**PAY-AC-208. A refund reversal sets a confirmed refund in error.**
Given a Stripe refund transaction in state `done` and a refund-update notification reporting a failure,
When the notification is processed,
Then the transaction becomes `error` with `The refund did not go through. Please log into your Stripe Dashboard to get more information on that matter, and address any accounting discrepancies.`

**PAY-AC-209. A void notification is not treated as a refund.**
Given a Stripe charge-refunded notification whose charge was never captured,
When the notification is processed,
Then nothing is processed.

**PAY-AC-210. A refund started at the provider creates a refund transaction.**
Given an Adyen, Razorpay or Stripe notification about a refund that matches no existing transaction, and a source transaction found by its provider reference,
When the notification is processed,
Then a refund child transaction is created with the amount converted to major units and is then processed.

**PAY-AC-211. A capture or void started at the provider creates a child transaction.**
Given an Adyen capture or cancellation notification that matches no existing child and whose source transaction is found by its original provider reference,
When the notification is processed,
Then a child transaction is created with the notification's amount and provider reference.

**PAY-AC-212. A child whose amount disagrees with the notification is set in error.**
Given an Adyen void notification whose amount differs from the amount of the existing child transaction,
When the notification is processed,
Then that child becomes `error` with `The amount processed by Adyen for the transaction <reference> is different than the one requested. Another transaction is created with the correct amount.` and a new child is created with the notification's amount.

**PAY-AC-213. Razorpay refuses a manual void.**
Given a Razorpay transaction in state `authorized`,
When a billing user voids it,
Then the operation is refused with `Transactions processed by Razorpay can't be manually voided from the system.`

**PAY-AC-214. Razorpay refuses a second token payment within 36 hours.**
Given a Razorpay token payment for the document `S00042` still in state `pending` and created ten hours ago,
When a second token payment for the same document with the same token is charged,
Then it becomes `error` with `Your last payment <reference> will soon be processed. Please wait up to 24 hours before trying again, or use another payment method.`

**PAY-AC-215. Razorpay allows a token payment for another document.**
Given the same pending payment and a token charge whose reference prefix differs,
When that second charge is sent,
Then the charge is sent.

**PAY-AC-216. Razorpay does not overwrite the reference of a confirmed transaction.**
Given a confirmed Razorpay transaction and a later notification carrying a different entity identifier,
When the later notification is processed,
Then the provider reference and the payment method are left unchanged.

**PAY-AC-217. Authorize.Net voids a validation transaction after tokenizing it.**
Given an Authorize.Net validation transaction of 0.01 that the provider authorizes,
When the provider's answer is processed,
Then the token is created first and the authorization is voided afterwards, and the transaction ends `done`.

**PAY-AC-218. Authorize.Net voids an unsettled payment instead of refunding it.**
Given a payment whose provider status is `authorizedPendingCapture`,
When a refund is requested,
Then a void request is sent instead of a refund request.

**PAY-AC-219. Authorize.Net recognises a payment already voided at the provider.**
Given a payment whose provider status is `voided`,
When a refund is requested,
Then the transaction is canceled, with `done` allowed as an extra source state.

**PAY-AC-220. Authorize.Net recognises a payment already refunded at the provider.**
Given a payment whose provider status is `refundSettledSuccessfully`,
When a refund is requested,
Then the refund transaction is confirmed and the post-processing job is woken.

**PAY-AC-221. A voided refund transaction is confirmed.**
Given an Authorize.Net refund transaction whose void succeeded because the payment was not settled,
When the void answer is processed,
Then that refund transaction becomes `done`.

**PAY-AC-222. Voiding a confirmed Authorize.Net transaction cancels it.**
Given a confirmed Authorize.Net transaction whose void succeeds,
When the void answer is processed,
Then it becomes `cancel`.

**PAY-AC-223. A Worldline token payment that needs authentication switches to the redirect flow.**
Given a Worldline token payment that ended in `error` with the state message ending in `AUTHORIZATION_REQUESTED`,
When the processing values are computed again,
Then the transaction is reset to `draft` with the operation `online_redirect` and the redirect flow is forced.

**PAY-AC-224. A Worldline validation is confirmed as soon as a token exists.**
Given a Worldline validation transaction whose status is `PENDING_CAPTURE` and whose data carry a token,
When the notification is processed,
Then the transaction becomes `done` and a token is created.

**PAY-AC-225. A Flutterwave token payment awaiting authentication is redirected.**
Given a Flutterwave token payment in state `pending` whose provider reference is an authentication address,
When the processing values are computed,
Then a redirect form pointing at that address is returned.

**PAY-AC-226. A Xendit return sets a draft transaction to pending.**
Given a Xendit transaction in state `draft` and a return address carrying the success flag and a valid access token over the reference and the amount,
When the customer returns to that address,
Then the transaction becomes `pending`.

**PAY-AC-227. A Toss Payments amount is validated before the confirmation request.**
Given a Toss Payments success return whose amount differs from the transaction amount,
When the return is handled,
Then the transaction becomes `error` and no confirmation request is sent.

**PAY-AC-228. A Toss Payments failure return needs a valid access token.**
Given a failure return whose access token over the reference is invalid,
When the return is handled,
Then nothing is written.
Given a failure return whose access token is valid,
When the return is handled,
Then the transaction becomes `error` with the provider's message and code.

**PAY-AC-229. An ECPay simulation notification is ignored.**
Given a webhook notification whose simulation flag is not `0`,
When the notification is received,
Then nothing is processed and a log entry says that the simulation was skipped.

**PAY-AC-230. An ECPay plain return without a payload is not a failure.**
Given a return opened with no payload,
When the return is handled,
Then nothing is processed and the browser is redirected to the payment status page.

**PAY-AC-231. A Mercado Pago notification that is not a payment event is ignored.**
Given a notification whose action is neither `payment.created` nor `payment.updated`,
When the notification is received,
Then nothing is processed.

**PAY-AC-232. A PayPal event that is not handled is ignored.**
Given a webhook event whose kind is not one of the three handled kinds,
When the notification is received,
Then nothing is processed.

**PAY-AC-233. A PayPal notification that cannot be verified sets the transaction in error.**
Given a webhook event whose verification call answers anything other than a success,
When the notification is processed,
Then the transaction becomes `error` with `Unable to verify the payment data`.

---

# 21. Payment links

**PAY-AC-234. A link for a document with nothing to pay is refused.**
Given a link wizard whose `amount_max` is 0,
When the wizard computes its warning message,
Then the warning message is `There is nothing to be paid.`

**PAY-AC-235. A link with a non-positive amount is refused.**
Given `amount` equal to 0 and `amount_max` equal to 120.00,
When the wizard computes its warning message,
Then the warning message is `Please set a positive amount.`

**PAY-AC-236. A link above the maximum is refused.**
Given `amount` equal to 150.00 and `amount_max` equal to 120.00 euro,
When the wizard computes its warning message,
Then the warning message is `Please set an amount lower than €120.00.`

**PAY-AC-237. A link is refused when portal payment is off.**
Given the portal payment setting is off and an otherwise valid amount,
When the wizard computes its warning message,
Then the warning message is `Online payment option is not enabled in Configuration.`

**PAY-AC-238. The link carries a signed access token.**
Given a valid link wizard for a contact, an amount and a currency,
When the payment link is generated,
Then the produced address carries the amount, the currency identifier, the contact identifier, the company identifier and an access token that verifies against the triple (contact identifier, amount, currency identifier).

---

# 22. Offline payments from a Payment

**PAY-AC-239. Posting a payment with a token creates and charges a transaction.**
Given a draft Payment of 120.00 euro with a token whose provider does not capture manually,
When the Payment is posted,
Then a transaction is created with `operation` equal to `offline`, the token, the Payment's amount, currency and contact, and a reference computed from the Payment's memo; the token is charged; the transaction is post-processed; and the Payment is posted once the transaction is `done`.

**PAY-AC-240. A failed charge cancels the payment.**
Given the same setting and a charge that ends in `error`,
When the Payment is posted and the token is charged,
Then the Payment is cancelled.

**PAY-AC-241. A pending charge leaves the payment unposted.**
Given a charge that ends in `pending`,
When the Payment is posted and the token is charged,
Then the Payment is neither posted nor cancelled.

**PAY-AC-242. A payment may only have one transaction.**
Given a Payment that already has a transaction,
When a second transaction is created from it,
Then the creation is refused with `A payment transaction with reference <reference> already exists.`

**PAY-AC-243. A payment without a token may not create a transaction.**
Given a Payment with no token,
When a transaction is created from that Payment,
Then the creation is refused with `A token is required to create a new payment transaction.`

**PAY-AC-244. Only tokens of providers without manual capture are offered.**
Given two tokens of the same contact, one whose provider captures manually and one whose provider does not,
When the suitable tokens of a Payment are computed,
Then only the second one is offered.

---

# 23. Point of sale online payments

**PAY-AC-245. A confirmed transaction registers a point of sale payment.**
Given a point of sale order of 25.00 euro and a confirmed transaction of 25.00 linked to it,
When the transaction is post-processed,
Then a Payment is created, a payment line of 25.00 is added to the order with the online payment method and the date of the last state change, the Payment carries the point of sale method, order and session, and the point of sale screens are notified.

**PAY-AC-246. A negative amount is refused.**
Given a transaction of −5.00 linked to a point of sale order,
When it is post-processed,
Then the operation fails with `The payment transaction (<identifier>) has a negative amount.`

**PAY-AC-247. A fully paid draft order is processed.**
Given a draft point of sale order that becomes fully paid by the registered payment,
When the online payment is registered on it,
Then the order is processed.

---

# 24. Logging and diagnostics

**PAY-AC-248. Sensitive keys are redacted.**
Given a Stripe request whose payload carries a client secret,
When the request is logged,
Then the value of that key is replaced by `[REDACTED]`, including when it appears inside serialised text.

**PAY-AC-249. A request and its answer are logged with the reference.**
Given a request sent for the transaction `S00042`,
When the request is sent and the provider answers,
Then the log entry names the request method, the address and the reference, and the answer entry names the status code, the status text, the address and the reference; an unsuccessful answer is logged at the error level.

**PAY-AC-250. A connection failure is reported.**
Given the provider is unreachable,
When a request is sent,
Then the operation fails with `Could not establish the connection to the payment provider.`

**PAY-AC-251. A rejected request is reported with the provider's message.**
Given the provider answers with an error status and the message `Invalid merchant account`,
When the request is sent,
Then the operation fails with `The payment provider rejected the request.` followed by that message on a new line.

**PAY-AC-252. The availability report is visible only to administrators.**
Given a payment form with an availability report,
When it is rendered for a portal user,
Then neither the report nor the control that expands it is present.
When it is rendered for an administrator, Then both are present.

**PAY-AC-253. The empty-form notice guides the administrator.**
Given a payment form with no compatible provider and no compatible method, rendered for an administrator, in a database where no provider is configured and a guided setup is possible,
Then the notice `No payment method available` is shown together with `No payment providers are configured.`, a button that starts the guided setup, a link to the availability report and a link to the provider list.
When the same form is rendered for a customer, Then the notice is followed by `If you believe that it is an error, please contact the website administrator.`

---

# 25. Connector credentials and guided setup

**PAY-AC-254. A Mercado Pago provider with credentials may be enabled.**
Given a Payment Provider whose code is `mercado_pago`, whose state is `disabled` and whose access token, refresh token and public key are all filled,
When an administrator sets `state` to `enabled`,
Then the write succeeds, the state becomes `enabled` and no message is shown (PAY-RULE-014).

**PAY-AC-255. A Mercado Pago provider without credentials may not be enabled.**
Given the same provider after the Reset credentials operation, which emptied the account country, the access token, the access token expiry, the refresh token and the public key together with setting the state to `disabled`,
When an administrator sets `state` to `enabled`,
Then the write is refused with `Mercado Pago credentials are missing. Click the "Connect" button to set up your account.` and the state stays `disabled` (PAY-RULE-014).

**PAY-AC-256. The same guard applies to the test state.**
Given a Mercado Pago provider with no access token,
When an administrator sets `state` to `test`,
Then the write is refused with the same message, because the guard is on every state other than `disabled` (PAY-RULE-014).

**PAY-AC-257. Tokenization on Mercado Pago needs a connected account.**
Given a Mercado Pago provider whose `public_key` is empty,
When an administrator sets `allow_tokenization` to true,
Then the write is refused with `Connect your account before enabling tokenization.` (PAY-RULE-015).
Given the same provider once the guided setup has written a public key,
When an administrator sets `allow_tokenization` to true,
Then the write succeeds.

**PAY-AC-258. Resetting Mercado Pago credentials switches tokenization off.**
Given an enabled Mercado Pago provider whose `allow_tokenization` is true,
When an administrator runs the Reset credentials operation,
Then `state` becomes `disabled`, `is_published` becomes false, the four credential fields are emptied and `allow_tokenization` becomes false, so that PAY-RULE-015 cannot be left violated by the reset itself.

**PAY-AC-259. A PayU provider with credentials may be enabled.**
Given a Payment Provider whose code is `payu` and whose key identifier and merchant salt are both filled,
When an administrator sets `state` to `enabled`,
Then the write succeeds (PAY-RULE-016).

**PAY-AC-260. A PayU provider missing either credential may not be enabled.**
Given a PayU provider whose merchant salt is empty while its key identifier is filled,
When an administrator sets `state` to `enabled`,
Then the write is refused with `PayU credentials are missing. Click the "Connect" button to set up your account.` and the state stays `disabled`.
Given instead a PayU provider whose key identifier is empty while its merchant salt is filled,
When an administrator sets `state` to `enabled`,
Then the write is refused with the same message (PAY-RULE-016).

**PAY-AC-261. A Razorpay provider with credentials may be enabled.**
Given a Payment Provider whose code is `razorpay` and whose key identifier and key secret are both filled,
When an administrator sets `state` to `enabled`,
Then the write succeeds (PAY-RULE-017).

**PAY-AC-262. A Razorpay provider connected through an account identifier may be enabled without a key pair.**
Given a Razorpay provider whose key identifier and key secret are both empty but whose connected-account identifier is filled,
When an administrator sets `state` to `enabled`,
Then the write succeeds, because either credential shape satisfies the guard (PAY-RULE-017).

**PAY-AC-263. A Razorpay provider without any credential may not be enabled.**
Given a Razorpay provider whose connected-account identifier, key identifier and key secret are all empty,
When an administrator sets `state` to `enabled`,
Then the write is refused with `Razorpay credentials are missing. Click the "Connect" button to set up your account.` and the state stays `disabled` (PAY-RULE-017).

**PAY-AC-264. A connected Stripe account may not be put in test mode.**
Given a Payment Provider whose code is `stripe` and whose connected-account identifier is filled,
When an administrator sets `state` to `test`,
Then the write is refused with `You cannot set the provider to Test Mode while it is linked with your Stripe account.` (PAY-RULE-018).
Given a Stripe provider with no connected-account identifier,
When an administrator sets `state` to `test`,
Then the write succeeds.

**PAY-AC-265. A Stripe provider whose guided setup is unfinished may not be enabled.**
Given a Stripe provider whose connected account exists but whose onboarding has not reported completion,
When an administrator sets `state` to `enabled`,
Then the write is refused with `You cannot set the provider state to Enabled until your onboarding to Stripe is completed.` and the state is unchanged (PAY-RULE-019).

**PAY-AC-266. An accounting payment method line of a live provider may not be deleted.**
Given a Payment Method Line whose `payment_provider_id` is a provider in the state `enabled`,
When a user deletes that line outside of a package removal,
Then the deletion is refused with `You can't delete a payment method that is linked to a provider in the enabled or test state.` followed by a new line, `Linked providers(s): ` and the display name of that provider (PAY-RULE-021).
Given the same line whose provider is in the state `test`,
When a user deletes it,
Then the deletion is refused with the same message.
Given the same line whose provider is in the state `disabled`,
When a user deletes it,
Then the deletion succeeds.

**PAY-AC-267. Merchant details may not be fetched from a disabled provider.**
Given an Authorize.Net provider whose `state` is `disabled`,
When an administrator runs the Update Merchant Details operation,
Then the operation is refused with `This action cannot be performed while the provider is disabled.` and no request is sent (PAY-RULE-023).
Given the same provider in the state `test`,
When the operation is run and the authentication call fails,
Then the operation fails with `Failed to authenticate.` followed by the provider's message.
Given the same provider in the state `test`,
When the authentication succeeds and the merchant details call fails,
Then the operation fails with `Could not fetch merchant details:` followed by the provider's message.

---

# 26. Connector locale, account country and guided setup behaviour

**PAY-AC-268. A supported browsing language resolves to its country's locale.**
Given a Mercado Pago provider and a payment form rendered with the browsing language `pt_BR`,
When the inline form values are built,
Then the locale sent to the provider is `pt-BR`.
Given the browsing language `es_AR`, When the inline form values are built, Then the `locale` member is `es-AR`.
Given the browsing language `es_MX`, When the inline form values are built, Then the `locale` member is `es-MX`.

**PAY-AC-269. A regional language with no country resolves through the company.**
Given a Mercado Pago provider whose company's country is Mexico, and a payment form rendered with the browsing language `es_419`, which names Latin America and no country,
When the inline form values are built,
Then the country code used for the lookup is the company's, `MX`, and the locale sent is `es-MX`.

**PAY-AC-270. An unsupported language falls back to the default locale.**
Given a Mercado Pago provider and a payment form rendered with the browsing language `fr_FR`,
When the inline form values are built,
Then the country code `FR` is not in the locale table and the locale sent is `en-US`.

**PAY-AC-271. An absent language falls back to the default locale.**
Given a Mercado Pago provider and a request that carries no browsing language at all,
When the inline form values are built,
Then the derived country code is the empty text and the locale sent is `en-US`.

**PAY-AC-272. Changing the Paymob account country changes the available currency.**
Given a Payment Provider whose code is `paymob` and whose `paymob_account_country_id` is Egypt, so that its `available_currency_ids` holds exactly the Egyptian pound,
When an administrator sets `paymob_account_country_id` to Saudi Arabia,
Then `available_currency_ids` holds exactly one currency, the Saudi riyal (`SAR`), and the previous currency is removed rather than added to.
Given the same provider with `paymob_account_country_id` set to the United Arab Emirates, When the change is saved, Then `available_currency_ids` holds exactly the United Arab Emirates dirham; and with Oman, exactly the Omani rial.

**PAY-AC-273. The Stripe guided setup returns an address to open.**
Given a Stripe provider whose `state` is `disabled` in a company whose country is one of the forty-four supported countries,
When an administrator starts the guided setup,
Then a connected account is fetched or created, an account link is requested, and the operation returns the address of that link for the browser to open.

**PAY-AC-274. An outlying territory is treated as its parent country by the guided setup.**
Given a company whose country is Réunion,
When an administrator starts the Stripe guided setup,
Then the country is mapped to France before the support test, the test passes, and the account link request is made instead of being refused.
The same mapping applies to Martinique, Guadeloupe, French Guiana, Mayotte and Saint-Martin, all six of which map to France.

**PAY-AC-275. An unsupported country stops the guided setup.**
Given a company whose country is neither one of the forty-four supported countries nor one of the six mapped territories,
When an administrator starts the Stripe guided setup,
Then the setup is refused with `Stripe Connect is not available in your country, please use another payment provider.` and no account link is requested.

**PAY-AC-276. The account link request carries every required value.**
Given a Stripe provider and a connected account identifier,
When the account link is requested,
Then exactly one request is sent and its payload carries the connected account, the return address, the refresh address and the link type; the return and refresh addresses both carry the provider identifier and the menu identifier, and the refresh address additionally carries the account identifier.

**PAY-AC-277. A webhook is created only when none is registered.**
Given a Stripe provider whose webhook secret is empty,
When an administrator runs the Create webhook operation,
Then exactly one request is sent to the provider, it carries the webhook address, the eight handled events and the fixed service version, and the secret returned by the provider is stored on the provider.

**PAY-AC-278. No webhook is created when one is already registered.**
Given a Stripe provider whose webhook secret is already filled,
When an administrator runs the Create webhook operation,
Then no request at all is sent to the provider and the stored secret is left unchanged.

**PAY-AC-279. The amount check passes for the currencies with a deviating precision.**
Given a Stripe transaction of 15 in the Icelandic króna, whose payment intent payload was built by the connector,
When the amount of the answer is checked against the transaction,
Then the transaction does not become `error`, because the payload and the check both use the connector's own precision of 2 for that currency rather than the currency's own.
The same holds for the Ugandan shilling with a precision of 2 and the Malagasy ariary with a precision of 0.

**PAY-AC-280. The return from a tokenization request is accepted.**
Given a Stripe validation transaction created with the amount 0, the operation `validation` and `tokenize` true, whose setup intent the provider has confirmed,
When the customer's browser reaches the Stripe return endpoint with that transaction's reference,
Then the setup intent is fetched with its instrument expanded, its description is compared with the transaction reference and matches, the payment data are processed, a Payment Token is created and the request answers successfully rather than failing.

---

# 27. Donations and website-scoped payment

**PAY-AC-281. A donation form post is turned into a page address.**
Given a public website page carrying a donation block whose recipient address is `info@yourcompany.example.com`, whose prefilled amounts are 10, 25, 50 and 100, whose minimum amount is 5, whose maximum amount is 100, whose slider step is 5 and whose default amount is 25,
When a visitor who is not signed in picks 50 euro and submits the form to the donation pay endpoint with the post method,
Then the amount 50.00, the currency identifier of the euro, the donation options and the four descriptions are stored in the visitor's session, and the answer is a redirection to the same path with the "see other" status and no rendered page.

**PAY-AC-282. The donation page fills its defaults from the session.**
Given the session of PAY-AC-281,
When the visitor's browser follows the redirection with the get method and no parameter at all,
Then the page is rendered with the amount 50.00, the currency euro, the four descriptions and the donation options taken from the session; the paying contact is the public contact of the request; and the access token is the signature of that contact, 50.00 and the euro.

**PAY-AC-283. The donation page falls back to twenty-five in the company currency.**
Given a website whose active company is "Acme" with the accounting currency euro, and an empty session,
When a visitor opens the donation pay endpoint with the get method and no parameter,
Then the page is rendered with the amount 25.00, the currency euro and a free custom amount as the only donation option.

**PAY-AC-284. The donation form hides the save-my-details box for an anonymous donor.**
Given a donation page rendered for a visitor who is not signed in, and a provider that allows tokenization,
When the payment form is built,
Then the "save my payment details" box is hidden for that provider, and the submit button reads `Donate`.

**PAY-AC-285. A donation below the minimum is refused.**
Given a donation block whose minimum amount is 5,
When the donation transaction endpoint is called for the path part 5 with the amount 4.99,
Then the call is refused with `Donation amount must be at least 5.00.` and no transaction is created.

**PAY-AC-286. Missing donor details are refused in order.**
Given a visitor who is not signed in,
When the donation transaction endpoint is called with an amount of 50.00 and donor details that carry no name, no electronic mail address and no country,
Then the call is refused with `Name is required.`; when only the name is supplied, it is refused with `Email is required.`; when the name and the address are supplied, it is refused with `Country is required.`; and in each case no transaction is created.

**PAY-AC-287. An anonymous donation is recorded on the public contact and never tokenized.**
Given a visitor who is not signed in, a donation of 50.00 euro and donor details naming "Ada Giver", the address `ada@example.com` and the country Belgium,
When the donation transaction endpoint is called with a valid access token,
Then a transaction is created with the amount 50.00, the currency euro, the paying contact equal to the website's public contact, the tokenize flag false, the donation flag true, the contact snapshot name "Ada Giver", the snapshot address `ada@example.com`, the snapshot country Belgium and the snapshot language equal to the language of the request.

**PAY-AC-288. A signed-in donor keeps his own contact and gains a country.**
Given a signed-in customer "Norbert Buyer" whose contact carries no country, and donor details naming Belgium,
When the donation transaction endpoint is called,
Then the transaction is created for that customer's own contact and the snapshot country becomes Belgium, while the snapshot name and address stay those of the contact.

**PAY-AC-289. The access token of a donation follows the amount actually chosen.**
Given a donation page opened with the amount 25.00 and its matching access token,
When the visitor changes the amount to 60.00 and confirms, and the transaction is created for 60.00,
Then the landing address of the transaction carries a token computed over the contact, 60.00 and the currency, and the token computed over 25.00 no longer opens the confirmation page.

**PAY-AC-290. The internal notification is sent as soon as the donation is created.**
Given the donation of PAY-AC-287 and the recipient address `info@yourcompany.example.com`,
When the donation transaction endpoint returns,
Then exactly one message has been sent to that address with the subject `A donation has been made on your website`, whose body carries the donor name "Ada Giver", the address `ada@example.com`, the donation date, the amount 50.00 with the currency symbol, the donor comment when one was given, the provider code and the transaction reference; and the transaction is still in the state `draft`.

**PAY-AC-291. The donor receives a confirmation only on success.**
Given the donation transaction of PAY-AC-287,
When it reaches the state `done` and is post-processed,
Then exactly one message with the subject `Donation confirmation` is sent to `ada@example.com`, rendered in the language recorded on the transaction, opening with `Dear Ada Giver,` and stating the amount 50.00 and the creation date; and when the transaction instead reaches `cancel` or `error`, no such message is sent.

**PAY-AC-292. The Payment of a donation carries the donation details.**
Given the confirmed donation transaction of PAY-AC-291,
When post-processing creates its Payment,
Then the Payment carries the donation flag true, and a log entry is written on it reading `Payment received from donation with following details:` followed by one line for the company, one for the contact, one for the contact name, one for the contact country and one for the contact electronic mail address, each prefixed by the label of that field, and no line at all for a value that is empty.

**PAY-AC-293. A provider bound to one website is not offered on another.**
Given two websites, "Site A" and "Site B", both of company "Acme", and a provider whose website is "Site A",
When a payment form is served for "Site B",
Then that provider is absent from the compatible providers, and the availability report records it as unavailable with the reason `incompatible website`; and when the same form is served for "Site A", or when the provider's website is emptied, the provider is present.

**PAY-AC-294. Duplicating a provider keeps the website only inside the company tree.**
Given a provider of company "Acme" bound to the website "Site A",
When the provider is duplicated without a website in the duplication values, and the copy stays in company "Acme",
Then the copy is bound to "Site A"; and when the copy is created in a company that is not "Acme" nor one of its descendants, the copy is bound to no website at all.

**PAY-AC-295. The supported payment methods block lists brands, not their primary method.**
Given a website of company "Acme" with one published provider that supports the primary method "Card", which is active and carries the brands "Visa" and "Mastercard", and the primary method "PayPal", which is active and has no brand,
When the supported payment methods endpoint is called with no limit,
Then the answer contains exactly three entries, "Visa", "Mastercard" and "PayPal", each with its name and the address of its image, and does not contain "Card".

**PAY-AC-296. The supported payment methods block honours its limit and its caching.**
Given the setting of PAY-AC-295,
When the endpoint is called with the limit 2 by a visitor who is not an internal user,
Then at most two entries are returned and the answer declares that it may be cached publicly for seven days with one further day of stale reuse; and when the same call is made by an internal user, the answer declares that it must not be cached.

**PAY-AC-297. A website request builds its addresses from the site it is serving.**
Given a database serving "Site A" at one address and "Site B" at another, and a provider used by both,
When a transaction is created while serving "Site B",
Then the return address and the webhook address given to the provider are built from the root address of the request to "Site B", not from the database-wide base address; and an address written in a non-Latin script is given to the provider in its plain-letter transcription.
