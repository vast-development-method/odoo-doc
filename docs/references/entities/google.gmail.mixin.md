# Google Gmail Mixin (`google.gmail.mixin`)

**Transport name:** `google.gmail.mixin`  
**Storage name:** `google_gmail_mixin`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `google_gmail`

Description: Google Gmail Mixin

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `active` | Active | boolean |  | default `True` |
| `google_gmail_refresh_token` | Refresh Token | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `google_gmail_access_token` | Access Token | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `google_gmail_access_token_expiration` | Access Token Expiration Timestamp | integer |  | not copied on duplication; visible only to groups `base.group_system` |
| `google_gmail_uri` | URI | single line text |  | computed by rule `_compute_gmail_uri` (not stored); visible only to groups `base.group_system`; Help: The URL to generate the authorization code from Google |

## Operations (9)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_gmail_uri` | computation | self | `google_gmail` |  |  |
| `open_google_gmail_uri` | operation | self | `google_gmail` |  | Open the URL to accept the Gmail permission.  This is done with an action, so we can force the user the save the form. We need him to save the form so the current mail server record exist in DB, and we can include the record ID in the URL. |
| `_fetch_gmail_refresh_token` | internal rule | self, authorization_code | `google_gmail` |  | Request the refresh token and the initial access token from the authorization code.  :return:     refresh_token, access_token, access_token_expiration |
| `_fetch_gmail_access_token` | internal rule | self, refresh_token | `google_gmail` |  | Refresh the access token thanks to the refresh token.  :return:     access_token, access_token_expiration |
| `_fetch_gmail_token` | internal rule | self, grant_type, **values | `google_gmail` |  | Generic method to request an access token or a refresh token.  Return the JSON response of the GMail API and manage the errors which can occur.  :param grant_type: Depends the action we want to do (refresh_token or authorization_code) :param values: Additional parameters that will be given to the GMail endpoint |
| `_fetch_gmail_access_token_iap` | internal rule | self, refresh_token | `google_gmail` |  | Fetch the access token using IAP.  Make a HTTP request to IAP, that will make a HTTP request to the Gmail API and give us the result.  :return:     access_token, access_token_expiration |
| `_raise_iap_error` | internal rule | self, error | `google_gmail` |  |  |
| `_generate_oauth2_string` | internal rule | self, user, refresh_token | `google_gmail` |  | Generate a OAuth2 string which can be used for authentication.  :param user: Email address of the Gmail account to authenticate :param refresh_token: Refresh token for the given Gmail account  :return: The SASL argument for the OAuth2 mechanism. |
| `_get_gmail_csrf_token` | preparation rule | self | `google_gmail` |  | Generate a CSRF token that will be verified in `google_gmail_callback`.  This will prevent a malicious person to make an admin user disconnect the mail servers. |

## Validation and error messages (8)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `open_google_gmail_uri` | AccessError | Only the administrator can link a Gmail mail server. | `google_gmail` |
| `open_google_gmail_uri` | UserError | Please enter a valid email address. | `google_gmail` |
| `open_google_gmail_uri` | UserError | Please configure your Gmail credentials. | `google_gmail` |
| `open_google_gmail_uri` | UserError | Please configure your Gmail credentials. | `google_gmail` |
| `open_google_gmail_uri` | UserError | Oops, we could not authenticate you. Please try again later. | `google_gmail` |
| `_fetch_gmail_token` | UserError | An error occurred when fetching the access token. | `google_gmail` |
| `_fetch_gmail_access_token_iap` | UserError | Oops, we could not authenticate you. Please try again later. | `google_gmail` |
| `_raise_iap_error` | UserError | get_iap_error_message(self.env, error) | `google_gmail` |

Machine-readable definition: `../../../schemas/data/entities/google.gmail.mixin.json`.
