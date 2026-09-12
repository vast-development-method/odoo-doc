# Microsoft Service (`microsoft.service`)

**Transport name:** `microsoft.service`  
**Storage name:** `microsoft_service`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `microsoft_account`

Description: Microsoft Service

## Operations (9)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_microsoft_client_id` | preparation rule | self, service | `microsoft_account` |  |  |
| `_get_calendar_scope` | preparation rule | self | `microsoft_account` |  |  |
| `_get_auth_endpoint` | preparation rule | self | `microsoft_account` | model |  |
| `_get_token_endpoint` | preparation rule | self | `microsoft_account` | model |  |
| `_refresh_microsoft_token` | internal rule | self, service, rtoken | `microsoft_account` | model | Call Microsoft API to refresh the token, with the given authorization code :param service : the name of the microsoft service to actualize :param rtoken : the code to exchange against the new refresh token :returns the new access token and its time to live |
| `_refresh_microsoft_token_with_refresh` | internal rule | self, service, rtoken | `microsoft_account` | model | Call Microsoft API to refresh the token, with the given authorization code :param service : the name of the microsoft service to actualize :param rtoken : the code to exchange against the new refresh token :returns the new access token, its time to live and the new refresh token |
| `_get_authorize_uri` | preparation rule | self, from_url, service, scope, redirect_uri | `microsoft_account` | model | This method return the url needed to allow this instance of the system to access to the scope of gmail specified as parameters |
| `_get_microsoft_tokens` | preparation rule | self, authorize_code, service, redirect_uri | `microsoft_account` | model | Call Microsoft API to exchange authorization code against token, with POST request, to not be redirected. |
| `_do_request` | internal rule | self, uri, params, headers, method, preuri, timeout | `microsoft_account` | model | Execute the request to Microsoft API. Return a tuple ('HTTP_CODE', 'HTTP_RESPONSE') :param uri : the url to contact :param params : dict or already encoded parameters for the request to make :param headers : headers of request :param method : the method to use to make the request :param preuri : pre url to prepend to param uri. |

Machine-readable definition: `../../../schemas/data/entities/microsoft.service.json`.
