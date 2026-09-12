# in-app purchase Lead Enrichment application programming interface (`iap.enrich.api`)

**Transport name:** `iap.enrich.api`  
**Storage name:** `iap_enrich_api`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `iap`

Description: IAP Lead Enrichment API

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_contact_iap` | internal rule | self, local_endpoint, params | `iap` | model |  |
| `_request_enrich` | internal rule | self, lead_emails | `iap` | model | Contact endpoint to get enrichment data.  :param lead_emails: dict{lead_id: email} :return: dict{lead_id: company data or False} :raise: several errors, notably   * InsufficientCreditError: {     "credit": 4.0,     "service_name": "reveal",     "base_url": "https://example.com/",     "message": "You don't have enough credits on your account to use this service."     } |

Machine-readable definition: `../../../schemas/data/entities/iap.enrich.api.json`.
