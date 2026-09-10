# Microsoft Outlook Mixin (`microsoft.outlook.mixin`)

**Transport name:** `microsoft.outlook.mixin`  
**Storage name:** `microsoft_outlook_mixin`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `microsoft_outlook`

Description: Microsoft Outlook Mixin

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `active` | Active | boolean |  | default `True` |
| `microsoft_outlook_refresh_token` | Outlook Refresh Token | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `microsoft_outlook_access_token` | Outlook Access Token | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `microsoft_outlook_access_token_expiration` | Outlook Access Token Expiration Timestamp | integer |  | not copied on duplication; visible only to groups `base.group_system` |
| `microsoft_outlook_uri` | Authentication URI | single line text |  | computed by rule `_compute_outlook_uri` (not stored); visible only to groups `base.group_system`; Help: The URL to generate the authorization code from Outlook |

## Operations (10)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_outlook_uri` | computation | self | `microsoft_outlook` |  |  |
| `open_microsoft_outlook_uri` | operation | self | `microsoft_outlook` |  | Open the URL to accept the Outlook permission.  This is done with an action, so we can force the user the save the form. We need him to save the form so the current mail server record exist in DB and we can include the record ID in the URL. |
| `_fetch_outlook_refresh_token` | internal rule | self, authorization_code | `microsoft_outlook` |  | Request the refresh token and the initial access token from the authorization code.  :return:     refresh_token, access_token, id_token, access_token_expiration |
| `_fetch_outlook_access_token` | internal rule | self, refresh_token | `microsoft_outlook` |  | Refresh the access token thanks to the refresh token.  :return:     access_token, access_token_expiration |
| `_fetch_outlook_token` | internal rule | self, grant_type, **values | `microsoft_outlook` |  | Generic method to request an access token or a refresh token.  Return the JSON response of the Outlook API and manage the errors which can occur.  :param grant_type: Depends the action we want to do (refresh_token or authorization_code) :param values: Additional parameters that will be given to the Outlook endpoint |
| `_fetch_outlook_access_token_iap` | internal rule | self, refresh_token | `microsoft_outlook` |  | Fetch the access token using IAP.  Make a HTTP request to IAP, that will make a HTTP request to the Outlook API and give us the result.  :return:     access_token, access_token_expiration |
| `_raise_iap_error` | internal rule | self, error | `microsoft_outlook` |  |  |
| `_generate_outlook_oauth2_string` | internal rule | self, login | `microsoft_outlook` |  | Generate a OAuth2 string which can be used for authentication.  :param login: Email address of the Outlook account to authenticate :return: The SASL argument for the OAuth2 mechanism. |
| `_get_outlook_csrf_token` | preparation rule | self | `microsoft_outlook` |  | Generate a CSRF token that will be verified in `microsoft_outlook_callback`.  This will prevent a malicious person to make an admin user disconnect the mail servers. |
| `_get_microsoft_endpoint` | preparation rule | self | `microsoft_outlook` | model |  |

## Validation and error messages (9)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `open_microsoft_outlook_uri` | AccessError | Only the administrator can link an Outlook mail server. | `microsoft_outlook` |
| `open_microsoft_outlook_uri` | UserError | Please enter a valid email address. | `microsoft_outlook` |
| `open_microsoft_outlook_uri` | UserError | Please configure your Outlook credentials. | `microsoft_outlook` |
| `open_microsoft_outlook_uri` | UserError | Please configure your Outlook credentials. | `microsoft_outlook` |
| `open_microsoft_outlook_uri` | UserError | Oops, we could not authenticate you. Please try again later. | `microsoft_outlook` |
| `_fetch_outlook_token` | UserError | An error occurred when fetching the access token. %s | `microsoft_outlook` |
| `_fetch_outlook_access_token_iap` | UserError | Oops, we could not authenticate you. Please try again later. | `microsoft_outlook` |
| `_raise_iap_error` | UserError | get_iap_error_message(self.env, error) | `microsoft_outlook` |
| `_generate_outlook_oauth2_string` | UserError | Please connect with your Outlook account before using it. | `microsoft_outlook` |

Machine-readable definition: `../../../schemas/data/entities/microsoft.outlook.mixin.json`.
