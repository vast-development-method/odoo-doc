# in-app purchase Partner Autocomplete application programming interface (`iap.autocomplete.api`)

**Transport name:** `iap.autocomplete.api`  
**Storage name:** `iap_autocomplete_api`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `partner_autocomplete`

Description: IAP Partner Autocomplete API

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_contact_iap` | internal rule | self, local_endpoint, action, params, timeout | `partner_autocomplete` | model |  |
| `_request_partner_autocomplete` | internal rule | self, action, params, timeout | `partner_autocomplete` | model | Contact endpoint to get autocomplete data.  :returns: a 2-element tuple (results, error code) :rtype: tuple[dict, Literal[False]] \| tuple[Literal[False], str] |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_contact_iap` | ValidationError | Test mode | `partner_autocomplete` |

Machine-readable definition: `../../../schemas/data/entities/iap.autocomplete.api.json`.
