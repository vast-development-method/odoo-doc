# Hypertext Transfer Protocol Routing (`ir.http`)

**Transport name:** `ir.http`  
**Storage name:** `ir_http`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `base`  
**Extended by packages:** `web`, `base_setup`, `bus`, `web_tour`, `html_editor`, `mail`, `http_routing`, `auth_signup`, `portal`, `account`, `payment`, `auth_password_policy_portal`, `auth_password_policy_signup`, `auth_timeout`, `barcodes`, `barcodes_gs1_nomenclature`, `base_import_module`, `spreadsheet`, `calendar`, `cloud_storage`, `utm`, `mail_plugin`, `delivery`, `google_recaptcha`, `hr_attendance`, `survey`, `portal_rating`, `website`, `website_mail`, `hr_timesheet`, `partner_autocomplete`, `point_of_sale`, `website_sale`, `l10n_tr`, `mass_mailing`, `pos_self_order`, `pos_online_payment_self_order`, `website_cf_turnstile`, `website_crm_iap_reveal`, `website_livechat`

Description: HTTP Routing

## Operations (72)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_slugify_one` | internal rule | cls, value, max_length | `base` |  | Transform a string to a slug that can be used in a url path. This method will first try to do the job with python-slugify if present. Otherwise, every character that is not a word character will be replaced by a dash '-', collapsing duplicates and removing boundary dashes. Example: ^h☺e$#!l(%l}o 你好& becomes h-e-l-l-o-你好 |
| `_slugify` | internal rule | cls, value, max_length, path | `base` |  |  |
| `_slug` | internal rule | cls, value | `base`, `http_routing`, `website` |  |  |
| `_unslug` | internal rule | cls, value | `base`, `http_routing` |  | Extract slug and id from a string. Always return a 2-tuple (str\|None, int\|None) |
| `_get_converters` | preparation rule | cls | `base`, `http_routing`, `website` |  | Get the converters list for custom url pattern werkzeug need to match Rule. This override adds the website ones. |
| `_match` | internal rule | cls, path_info | `base`, `http_routing`, `website` |  | Grant multilang support to URL matching by using http 3xx redirections and URL rewrite. This method also grants various attributes such as `lang` and `is_frontend` on the current `request` object.  1/ Use the URL as-is when it matches a non-multilang compatible    endpoint.  2/ Use the URL as-is when the lang is not present in the URL and    that the default lang has been requested.  3/ Use the URL as-is saving the requested lang when the user is    a bot and that the lang is missing from the URL.  4) Use the url as-is when the lang is missing from the URL, that    another lang than the  |
| `_get_public_users` | preparation rule | cls | `base`, `website` |  |  |
| `_auth_method_bearer` | internal rule | cls | `base` |  |  |
| `_auth_method_user` | internal rule | cls | `base` |  |  |
| `_auth_method_none` | internal rule | cls | `base` |  |  |
| `_auth_method_public` | internal rule | cls | `base`, `website` |  | If no user logged, set the public user of current website, or default public user as request uid. |
| `_authenticate` | internal rule | cls, endpoint | `auth_timeout`, `base` |  | Extend the standard `_authenticate` to enforce identity re-confirmation.  This method checks whether the current session requires identity confirmation due to timeout or inactivity. If a logout is required, a `SessionExpiredException` is raised, which the client handles by redirecting to the login page. If a check identity is required, a `CheckIdentityException` is raised, which the client handles by showing the re-authentication dialog.  :param endpoint: The HTTP route endpoint being accessed. :type endpoint: werkzeug.routing.Rule  :raises CheckIdentityException: If the session requires ident |
| `_authenticate_explicit` | internal rule | cls, auth | `base` |  |  |
| `_geoip_resolve` | internal rule | cls | `base` |  |  |
| `_sanitize_cookies` | internal rule | cls, cookies | `base`, `web` |  |  |
| `_pre_dispatch` | internal rule | cls, rule, args | `auth_signup`, `base`, `html_editor`, `http_routing`, `web`, `website_sale`, `website` |  |  |
| `_dispatch` | internal rule | cls, endpoint | `base` |  |  |
| `_post_dispatch` | internal rule | cls, response | `base`, `utm`, `website` |  |  |
| `_post_logout` | internal rule | cls | `base`, `web` |  |  |
| `_handle_error` | internal rule | cls, exception | `auth_timeout`, `base`, `http_routing` |  | Handle exceptions raised during request processing.  If the exception is a `CheckIdentityException` and the route is HTTP-based, the user is redirected to the identity confirmation page. This ensures that re-authentication can be completed before redirecting to the original request.  All other exceptions, e.g. `JSONRPC` calls, are handled by displaying the authentication form in a dialog rather than a page.  :param Exception exception: The exception raised during request dispatch. :return: An HTTP response for identity confirmation, or the default error response. :rtype: werkzeug.wrappers.Resp |
| `_serve_fallback` | internal rule | cls | `base`, `website` |  |  |
| `_redirect` | internal rule | cls, location, code | `base` |  |  |
| `_generate_routing_rules` | internal rule | self, modules, converters | `base`, `website` |  |  |
| `routing_map` | operation | self, key | `base`, `website` |  |  |
| `_gc_sessions` | background operation | self | `base` | autovacuum |  |
| `_get_translations_for_webclient` | preparation rule | self, modules, lang | `base_import_module`, `base` | model |  |
| `_get_web_translations_hash` | preparation rule | self, modules, lang | `base` | model |  |
| `_is_allowed_cookie` | internal rule | cls, cookie_type | `base`, `website` |  |  |
| `_verify_request_recaptcha_token` | internal rule | self, action | `base`, `google_recaptcha`, `website_cf_turnstile` | model | Verify the recaptcha token for the current request. If no recaptcha private key is set the recaptcha verification is considered inactive and this method will return True. |
| `is_a_bot` | operation | cls | `web` |  |  |
| `_handle_debug` | internal rule | cls | `web` |  |  |
| `webclient_rendering_context` | operation | self | `web` |  |  |
| `color_scheme` | operation | self | `web` |  |  |
| `lazy_session_info` | operation | self | `account`, `hr_attendance`, `web` | model |  |
| `session_info` | operation | self | `auth_timeout`, `barcodes_gs1_nomenclature`, `barcodes`, `base_setup`, `bus`, `cloud_storage`, `google_recaptcha`, `hr_timesheet`, `l10n_tr`, `mail`, `partner_autocomplete`, `spreadsheet`, `web_tour`, `web` |  | Override to add the current user data (partner or guest) if applicable. |
| `get_frontend_session_info` | operation | self | `auth_timeout`, `bus`, `google_recaptcha`, `http_routing`, `portal`, `web`, `website_cf_turnstile`, `website_sale`, `website` | model | Extend the frontend session info with inactivity timeout and user login.  dds the user's inactivity timeout (if applicable) to the session info returned to the frontend web client.  :return: The updated session information dictionary. :rtype: dict |
| `get_currencies` | operation | self | `web` |  |  |
| `_get_editor_context` | preparation rule | cls | `html_editor`, `website` |  | Check for ?editable and stuff in the query-string |
| `_get_translation_frontend_modules_name` | preparation rule | cls | `auth_password_policy_portal`, `auth_password_policy_signup`, `delivery`, `html_editor`, `http_routing`, `mass_mailing`, `payment`, `point_of_sale`, `portal_rating`, `portal`, `pos_online_payment_self_order`, `pos_self_order`, `survey`, `website_livechat`, `website_mail`, `website` |  | Return a list of module name where web-translations and dynamic resources may be used in frontend views |
| `_unslug_url` | internal rule | cls, value | `http_routing` |  | From /blog/my-super-blog-1" to "blog/1" |
| `_url_localized` | internal rule | cls, url, lang_code, canonical_domain, prefetch_langs, force_default_lang | `http_routing` |  | Returns the given URL adapted for the given lang, meaning that:  1. It will have the lang suffixed to it 2. The model converter parts will be translated  If it is not possible to rebuild a path, use the current one instead. :func:`url_quote_plus` is applied on the returned path.  It will also force the canonical domain is requested.  >>> _get_url_localized(lang_fr, '/shop/my-phone-14') '/fr/shop/mon-telephone-14' >>> _get_url_localized(lang_fr, '/shop/my-phone-14', True) '<base_url>/fr/shop/mon-telephone-14' |
| `_url_lang` | internal rule | cls, path_or_uri, lang_code | `http_routing` |  | Given a relative URL, make it absolute and add the required lang or remove useless lang. Nothing will be done for absolute or invalid URL. If there is only one language installed, the lang will not be handled unless forced with `lang` parameter.  :param lang_code: Must be the lang `code`. It could also be something                   else, such as `'[lang]'` (used for url_return). |
| `_url_for` | internal rule | cls, url_from, lang_code | `http_routing`, `website` |  | Return the url with the rewriting applied. Nothing will be done for absolute URL, invalid URL, or short URL from 1 char.  :param url_from: The URL to convert. :param lang_code: Must be the lang `code`. It could also be something                   else, such as `'[lang]'` (used for url_return). |
| `_is_multilang_url` | internal rule | cls, local_url, lang_url_codes | `http_routing` |  | Check if the given URL content is supposed to be translated. To be considered as translatable, the URL should either: 1. Match a POST (non-GET actually) controller that is `website=True` and either `multilang` specified to True or if not specified, with `type='http'`. 2. If not matching 1., everything not under /static/ or /web/ will be translatable |
| `_get_default_lang` | preparation rule | cls | `http_routing`, `website` |  |  |
| `get_translation_frontend_modules` | operation | self | `http_routing` | model |  |
| `_get_translation_frontend_modules_domain` | preparation rule | cls | `http_routing` |  | Return a domain to list the domain adding web-translations and dynamic resources that may be used frontend views |
| `get_nearest_lang` | operation | self, lang_code | `http_routing`, `pos_self_order`, `survey`, `website` | model | Try to find a similar lang. Eg: fr_BE and fr_FR :param lang_code: the lang `code` (en_US) |
| `_frontend_pre_dispatch` | internal rule | cls | `http_routing`, `website_sale`, `website` |  |  |
| `_get_exception_code_values` | preparation rule | cls, exception | `http_routing`, `website` |  | Return a tuple with the error code following by the values matching the exception |
| `_get_values_500_error` | preparation rule | cls, env, values, exception | `http_routing`, `website` |  |  |
| `_get_error_html` | preparation rule | cls, env, code, values | `http_routing`, `website` |  |  |
| `url_rewrite` | operation | self, path, query_args | `http_routing` | model |  |
| `_must_check_identity` | internal rule | cls | `auth_timeout` |  | Determine whether the current user session requires identity confirmation.  This method checks two timeout conditions: - `lock_timeout`: maximum allowed session duration before re-authentication is required, regardless of user activity. - `lock_timeout_inactivity`: period of inactivity after which re-authentication is required.  It compares the current time to session timestamps and evaluates whether the thresholds have been exceeded: - `lock_timeout` compares with the session timestamp `create_time` - `lock_timeout_inactivity` compares with the session timestamp `identity-check-next`  :return |
| `_check_identity` | validation | cls, credential | `auth_timeout` |  | Verify the user's identity using the given credentials.  Handles both single and multi-factor authentication flows depending on the current session state and configured timeout rules.  :param dict credential: A dictionary containing authentication data. Must include     a "type" key (e.g., "password", "totp", "webauthn"). If empty, the method     returns the list of available authentication methods.  :return: A dictionary indicating the outcome of the identity check:      - {"auth_methods": [...]} if no credential is provided,     - {"mfa": True, "auth_methods": [...]} if a second factor is re |
| `_set_session_inactivity` | internal rule | self, session, inactivity_period, force | `auth_timeout` |  | Set or clear the session's inactivity timeout flag.  This method is used to track user inactivity and determine when a session should trigger re-authentication. It is called when presence data is received through the websocket, either:  - because the web client, in Javascript, sent an event that the user is inactive - because the websocket connection was closed (e.g., the user closed the browser,   the last tab to the system was closed, internet disconnection, ...)  :param Session session: The user's HTTP session object. :param float inactivity_period: Duration of user inactivity in milliseconds. :p |
| `_session_info_common_auth_timeout` | internal rule | self, session_info | `auth_timeout` |  | Add inactivity timeout metadata to the session info dictionary.  This method is used to include the user's applicable inactivity timeout (in seconds) in the session information returned to the frontend. The timeout is only added for authenticated (non-public) users.  :param dict session_info: The original session information dictionary. :return: The updated session information with inactivity timeout (if applicable). :rtype: dict |
| `_auth_method_calendar` | internal rule | cls | `calendar` |  |  |
| `get_utm_domain_cookies` | operation | cls | `utm` |  |  |
| `_set_utm` | internal rule | cls, response | `utm` |  |  |
| `_auth_method_outlook` | internal rule | cls | `mail_plugin` |  |  |
| `_add_public_key_to_session_info` | internal rule | self, session_info | `google_recaptcha` | model | Add the ReCaptcha public key to the given session_info object |
| `_verify_recaptcha_token` | internal rule | self, ip_addr, token, action | `google_recaptcha` | model | Verify a recaptchaV3 token and returns the result as a string. RecaptchaV3 verify DOC: https://developers.google.com/recaptcha/docs/verify  :return: The result of the call to the google API:          is_human: The token is valid and the user trustworthy.          is_bot: The user is not trustworthy and most likely a bot.          no_secret: No reCaptcha secret set in settings.          wrong_action: the action performed to obtain the token does not match the one we are verifying.          wrong_token: The token provided is invalid or empty.          wrong_secret: The private key provided in se |
| `_is_survey_frontend` | internal rule | self, path | `survey` | model |  |
| `_slug_matching` | internal rule | cls, adapter, endpoint, **kw | `website` |  |  |
| `_rewrite_len` | internal rule | self, website_id | `website` |  |  |
| `_get_rewrites` | preparation rule | self, website_id | `website` |  |  |
| `_register_website_track` | internal rule | cls, response | `website` |  |  |
| `_serve_page` | internal rule | cls | `website_crm_iap_reveal`, `website` |  |  |
| `_serve_redirect` | internal rule | cls | `website` |  |  |
| `get_timesheet_uoms` | operation | self | `hr_timesheet` | model |  |
| `_verify_turnstile_token` | internal rule | self, ip_addr, token, action | `website_cf_turnstile` | model | Verify a turnstile token and returns the result as a string. Turnstile verify DOC: https://developers.cloudflare.com/turnstile/get-started/server-side-validation/  :return: The result of the call to the cloudflare API:          is_human: The token is valid and the user trustworthy.          is_bot: The user is not trustworthy and most likely a bot.          no_secret: No private key in settings.          wrong_action: the action performed to obtain the token does not match the one we are verifying.          wrong_token: The token provided is invalid or empty.          wrong_secret: The private |

## Validation and error messages (11)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_auth_method_bearer` | AccessDenied | e | `base` |
| `_verify_request_recaptcha_token` | ValidationError | The reCaptcha private key is invalid. | `google_recaptcha` |
| `_verify_request_recaptcha_token` | ValidationError | The reCaptcha token is invalid. | `google_recaptcha` |
| `_verify_request_recaptcha_token` | UserError | Your request has timed out, please retry. | `google_recaptcha` |
| `_verify_request_recaptcha_token` | UserError | The request is invalid or malformed. | `google_recaptcha` |
| `_verify_request_recaptcha_token` | UserError | Suspicious activity detected by google reCAPTCHA. | `google_recaptcha` |
| `_verify_request_recaptcha_token` | ValidationError | The Cloudflare turnstile private key is invalid. | `website_cf_turnstile` |
| `_verify_request_recaptcha_token` | ValidationError | The CloudFlare human validation failed. | `website_cf_turnstile` |
| `_verify_request_recaptcha_token` | UserError | Your request has timed out, please retry. | `website_cf_turnstile` |
| `_verify_request_recaptcha_token` | UserError | The request is invalid or malformed. | `website_cf_turnstile` |
| `_verify_request_recaptcha_token` | UserError | Suspicious activity detected by Turnstile CAPTCHA. | `website_cf_turnstile` |

Machine-readable definition: `../../../schemas/data/entities/ir.http.json`.
