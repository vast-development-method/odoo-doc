# Google Service (`google.service`)

**Transport name:** `google.service`  
**Storage name:** `google_service`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `google_account`

Description: Google Service

## Operations (5)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_client_id` | preparation rule | self, service | `google_account` |  |  |
| `_get_authorize_uri` | preparation rule | self, service, scope, redirect_uri, state, approval_prompt, access_type | `google_account` | model | This method return the url needed to allow this instance of the system to access to the scope of gmail specified as parameters |
| `_get_google_tokens` | preparation rule | self, authorize_code, service, redirect_uri | `google_account` | model | Call Google API to exchange authorization code against token, with POST request, to not be redirected. |
| `_refresh_google_token` | internal rule | self, service, rtoken | `google_account` |  |  |
| `_do_request` | internal rule | self, uri, params, headers, method, preuri, timeout | `google_account` | model | Execute the request to Google API. Return a tuple ('HTTP_CODE', 'HTTP_RESPONSE') :param uri : the url to contact :param params : dict or already encoded parameters for the request to make :param headers : headers of request :param method : the method to use to make the request :param preuri : pre url to prepend to param uri. |

Machine-readable definition: `../../../schemas/data/entities/google.service.json`.
