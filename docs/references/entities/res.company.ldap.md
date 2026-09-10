# Company directory access protocol configuration (`res.company.ldap`)

**Transport name:** `res.company.ldap`  
**Storage name:** `res_company_ldap`  
**Kind:** persistent entity (one table)  
**Defined by package:** `auth_ldap`

Description: Company LDAP configuration

## Identity and behavior

- Default ordering: `sequence`
- Display name field: `ldap_server`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (11)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `sequence` | Sequence | integer |  | default `10` |
| `company` | Company | many to one | `res.company` | required; on delete of the target: cascade |
| `ldap_server` | directory access protocol Server address | single line text |  | required; default `127.0.0.1` |
| `ldap_server_port` | directory access protocol Server port | integer |  | required; default `389` |
| `ldap_binddn` | directory access protocol binddn | single line text |  | Help: The user account on the LDAP server that is used to query the directory. Leave empty to connect anonymously. |
| `ldap_password` | directory access protocol password | single line text |  | Help: The password of the user account on the LDAP server that is used to query the directory. |
| `ldap_filter` | directory access protocol filter | single line text |  | required; Help: Filter used to look up user accounts in the LDAP database. It is an    arbitrary LDAP filter in string representation. Any `%s` placeholder    will be replaced by the login (identifier) provided by the user, the filter    should contain at least one such placeholder.      The filter must result in exactly one (1) result, otherwise the login will    be considered invalid.      Example (actual attributes depend on LDAP server and setup):          (&(objectCategory=person)(objectClass=user)(sAMAccountName=%s))      or          (\|(mail=%s)(uid=%s)) |
| `ldap_base` | directory access protocol base | single line text |  | required; Help: DN of the user search scope: all descendants of this base will be searched for users. |
| `user` | Template User | many to one | `res.users` | Help: User to copy when creating new users |
| `create_user` | Create User | boolean |  | default `True`; Help: Automatically create local user accounts for new users authenticating via LDAP |
| `ldap_tls` | Use TLS | boolean |  | Help: Request secure TLS/SSL encryption when connecting to the LDAP server. This option requires a server with STARTTLS enabled, otherwise all authentication attempts will fail. |

## Operations (9)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_ldap_dicts` | preparation rule | self | `auth_ldap` |  | Retrieve res_company_ldap resources from the database in dictionary format. :return: ldap configurations :rtype: list of dictionaries |
| `_connect` | internal rule | self, conf | `auth_ldap` |  | Connect to an LDAP server specified by an ldap configuration dictionary.  :param dict conf: LDAP configuration :return: an LDAP object |
| `_get_entry` | preparation rule | self, conf, login | `auth_ldap` |  |  |
| `_authenticate` | internal rule | self, conf, login, password | `auth_ldap` |  | Authenticate a user against the specified LDAP server.  In order to prevent an unintended 'unauthenticated authentication', which is an anonymous bind with a valid dn and a blank password, check for empty passwords explicitely (:rfc:`4513#section-6.3.1`) :param dict conf: LDAP configuration :param login: username :param password: Password for the LDAP user :return: LDAP entry of authenticated user or False :rtype: dictionary of attributes |
| `_query` | internal rule | self, conf, filter, retrieve_attributes | `auth_ldap` |  | Query an LDAP server with the filter argument and scope subtree.  Allow for all authentication methods of the simple authentication method:  - authenticated bind (non-empty binddn + valid password) - anonymous bind (empty binddn + empty password) - unauthenticated authentication (non-empty binddn + empty password)  .. seealso::    :rfc:`4513#section-5.1` - LDAP: Simple Authentication Method.  :param dict conf: LDAP configuration :param filter: valid LDAP filter :param list retrieve_attributes: LDAP attributes to be retrieved.         If not specified, return all attributes. :return: ldap entri |
| `_map_ldap_attributes` | internal rule | self, conf, login, ldap_entry | `auth_ldap` |  | Compose values for a new resource of model res_users, based upon the retrieved ldap entry and the LDAP settings. :param dict conf: LDAP configuration :param login: the new user's login :param tuple ldap_entry: single LDAP result (dn, attrs) :return: parameters for a new resource of model res_users :rtype: dict |
| `_get_or_create_user` | preparation rule | self, conf, login, ldap_entry | `auth_ldap` |  | Retrieve an active resource of model res_users with the specified login. Create the user if it is not initially found.  :param dict conf: LDAP configuration :param login: the user's login :param tuple ldap_entry: single LDAP result (dn, attrs) :return: res_users id :rtype: int |
| `_change_password` | internal rule | self, conf, login, old_passwd, new_passwd | `auth_ldap` |  |  |
| `test_ldap_connection` | operation | self | `auth_ldap` |  | Test the LDAP connection using the current configuration. Returns a dictionary with notification parameters indicating success or failure. |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_get_or_create_user` | AccessDenied | No local user found for LDAP login and not configured to create one | `auth_ldap` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `auth_ldap` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `auth_ldap.view_ldap_installer_form` | form |  | `company`, `ldap_server`, `ldap_server_port`, `ldap_tls`, `ldap_binddn`, `ldap_password`, `ldap_base`, `ldap_filter`, `sequence`, `create_user`, `user` | `Test Connection` |  | `auth_ldap` |
| `auth_ldap.res_company_ldap_view_tree` | list |  | `company`, `ldap_server`, `ldap_server_port` |  |  | `auth_ldap` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `auth_ldap.action_ldap_installer` | Setup your LDAP Server | list,form |  |  |  | `auth_ldap` |

Machine-readable definition: `../../../schemas/data/entities/res.company.ldap.json`; views: `../../../schemas/interfaces/views/res.company.ldap.json`.
