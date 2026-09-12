# Configuration of the Customer Portal

Every setting, system parameter, constant, shipped record, access group, permission and master-data prerequisite of the Customer Portal, with its data type, its default and its effect.

---

## 1. Capability packages

| Package | Delivers | Requires | Installed automatically |
|---|---|---|---|
| Customer Portal | The whole domain: the access mixin, the three wizards, the portal layout and pages, the account and address pages, the security page, the discussion thread on portal pages, the share and invitation flows. | The web client foundation, the rich-text editor, the front-end routing, the messaging domain and the sign-up capability. | no |
| Portal Rating | The rating bridge: the star widget, the pop-up rating composer, the publisher reply fields and endpoint, and the rating information inside the portal message projection. | Customer Portal and the rating capability. | yes, whenever both requirements are present |
| Password Policy on the Portal | Adds the configured minimum password length to every portal page and renders the strength meter next to the new-password input. | Customer Portal and the password policy capability. | yes |
| Second Factor on the Portal | Adds the second-factor section to the security page, embeds the enrollment layout and grants the portal group access to the enrollment wizard; points the "invite to enable a second factor" address at the portal for non-employees. | Customer Portal and the second-factor capability. | yes |
| Passkeys on the Portal | Adds the passkey section to the security page with creation, renaming and deletion. | Customer Portal and the passkey capability. | yes |

Every document domain that wants a portal page adds its own package (Sales, Accounts Receivable, Accounts Payable, Purchasing, Projects and Tasks, Timesheets, Events, Loyalty, Manufacturing subcontracting, Customer Relationship Management partnerships). Those packages contribute the home cards, the list pages, the record pages, the record rules and the portal web address rules; they are specified in their own folders.

---

## 2. Settings

| Setting | Identifier | Where | Data type | Default | Stored in | Effect |
|---|---|---|---|---|---|---|
| Customer application programming interface keys, labelled `Customer API Keys` | `portal_allow_api_keys` on Configuration Settings | General settings, in the block that is visible only in developer mode, next to the administrator's own application key button | boolean | false | the `portal.allow_api_keys` system parameter | When true, an external person may create and revoke their own application keys from the security page, and the section is rendered there (also requiring developer mode). When false, the section is not rendered and key creation by an external person is refused. |

The label and the caption are reproduced as the system emits them: the field label is `Customer API Keys` and the checkbox caption is `Customers can generate API keys`, in which the three letters abbreviate "application programming interface". This specification writes the term in full everywhere outside these two reproduced strings.

**Reading and writing.** The field is not stored on the settings record. Reading it returns the value of the system parameter `portal.allow_api_keys`, read with elevated rights and interpreted as a boolean. Writing it writes the boolean into that parameter, again with elevated rights. Independently of the derivation, the settings-page loader puts the same boolean into the page payload under the same name on every load, so that the checkbox reflects the stored parameter even before the derivation has run.

---

## 3. System parameters

Each parameter names one row of the platform's system parameter store; the keys are reproduced exactly, because an installation carries them in its data.

| Parameter | Data type | Default | Owner | Effect in this domain |
|---|---|---|---|---|
| `portal.allow_api_keys` | boolean text | unset (equivalent to false) | this domain | See section 2. Read with elevated rights on every render of the security page and on every key-creation attempt. Any non-empty stored text counts as true. |
| `auth_signup.invitation_scope` | selection: `b2b`, `b2c` | `b2b` | [Identity and Access](../identity-and-access/README.md) | Decides the share split (`PORT-RULE-010`) and whether a sign-up token can be produced for a Contact that has no user. |
| `base.template_portal_user_id` | integer text | the shipped portal user template | [Identity and Access](../identity-and-access/README.md) | The user duplicated to create a new external user from the invitation wizard. |
| `auth_signup.signup.validity.hours` | integer | 144 | [Identity and Access](../identity-and-access/README.md) | How long an invitation link stays valid. |
| `auth_signup.reset_password.validity.hours` | integer | 4 | [Identity and Access](../identity-and-access/README.md) | How long a password-reset link stays valid. |
| `database.secret` | text | generated at installation | the platform foundation, [configuration parameters](../../runtime/configuration-parameters.md) | The key used to sign a recipient identity and to sign notification links. Rotating it invalidates every outstanding signed link. |
| `auth_password_policy.minlength` | integer | 0, that is, no minimum | [Identity and Access](../identity-and-access/README.md) | Added to every portal page's rendering values when the password-policy bridge is installed; used as the client-side minimum of the new-password input. |
| `web.base.url` | text | the platform's own address | the platform foundation, [configuration parameters](../../runtime/configuration-parameters.md) | Prefixed to every share link so that the copied link is absolute. |

---

## 4. Constants

| Constant | Value | Where it applies |
|---|---|---|
| Default page size of a portal list page | 80 | Every shipped list page except the timesheet page. |
| Page size of the timesheet list page | 100 | The timesheet page only. |
| Default page size of the generic navigator helper | 30 | The helper's own default, used by callers that do not pass a size. |
| Navigator scope | 5 | Accepted by the helper and not used by its algorithm. |
| Messages per page in a portal discussion thread | 10 | The thread anchor, overridable per page. |
| Session history length | 100 | The identifiers stored by a list page for the record navigator. |
| Deletion queue batch size | 50 | One scheduled run of the account deletion queue. |
| Author picture dimensions in a portal thread | 50 by 50 pixels | The projected author picture address. |
| Attachment thumbnail dimensions | as produced by the attachment domain | The projected thumbnail token. |
| Accepted image media types for the shared-project image upload | the joint photographic experts group format, the portable network graphics format, the bitmap format and the tagged image file format | The image upload endpoint of the projects domain; a file of any other media type is refused with `Only jpeg, png, bmp and tiff images are allowed as attachments.` |

---

## 5. Session keys written by this domain and by the document domains

| Key | Written by | Content |
|---|---|---|
| `portal_counters` | The counter endpoint | A map from counter name to a boolean saying whether the counter was non-zero. Used to reveal cards immediately on the next page load. |
| `my_quotations_history` | The quotation list page | Up to one hundred sales order identifiers. |
| `my_orders_history` | The sales order list page | Up to one hundred sales order identifiers. |
| `my_invoices_history` | The invoice list page | Up to one hundred journal entry identifiers. |
| `my_rfqs_history` | The request-for-quotation list page | Up to one hundred purchase order identifiers. |
| `my_purchases_history` | The purchase order list page | Up to one hundred purchase order identifiers. |
| `my_projects_history` | The project list page | Up to one hundred project identifiers. |
| `my_tasks_history` | The task list page and the task record page reached from a back-office button | Up to one hundred task identifiers. |
| `my_project_tasks_history` | The task list inside a project page | Up to one hundred task identifiers. |
| `view_quote_<order identifier>` | The sales order record page | The date on which the "viewed by the customer" note was last logged for that order, so that at most one such note is logged per order per day per session. |

---

## 6. Shipped actions

| Record | Kind | Name | Target | Effect |
|---|---|---|---|---|
| `partner_wizard_action_create_and_open` | server action, bound to the Contact entity | `Grant portal access` | not applicable | Creates one Portal Access Wizard row and returns the dialog for it. This is the entry point offered in the Contact list and form action menu. |
| `partner_wizard_action` | window action | `Grant portal access` | new dialog, form mode | Opens the Portal Access Wizard directly. Not bound to any entity, so it does not appear in an action menu. |
| `portal_share_action` | window action, bound to the Portal Share Wizard entity | `Share Document` | new dialog, form mode, medium size | Opens the Portal Share Wizard. Reached from a document through the share operation of the access mixin, which merges the active model and record into the action's context. |

---

## 7. Shipped screen definitions

### 7.1 Back-office screens

| Definition | Entity | Content |
|---|---|---|
| `wizard_view` | Portal Access Wizard | The invitation dialog, titled `Portal Access Management`: the explanatory paragraph, the `welcome_message` input, the editable line list with the columns Contact (forced to be saved), Email (read-only for an employee line), three state icon buttons (a green check for `ok`, a red cross for `ko`, a struck-through person for `exist`), Latest Authentication, then the buttons `Grant Access`, `Revoke Access`, `Re-Invite` and the disabled `Internal User` marker; and a footer with the `Close` button that saves and closes. The list forbids creating and deleting lines. |
| `portal_share_wizard` | Portal Share Wizard | The share dialog, titled `Share Document`: the warning banner shown when `access_warning` is not empty, the hidden model and identifier, the `share_link` with a copy-to-clipboard control labelled `Copy Link`, the recipient selector with electronic mail address tags and the placeholder `Add contacts to share the document...`, the note input with the placeholder `Add a note`, and the footer with `Send` (hidden when the warning is not empty) and `Cancel`. |
| `res_config_settings_view_form` | Configuration Settings (`res.config.settings`) | Adds the developer-only block with the `Customer API Keys` switch next to the administrator's application key button. |
| `rating_rating_view_form` | Rating | Adds, after the rating moment, the label `Comment`, the `publisher_comment` input and, when a comment exists, the line `by <publisher> on <publication moment>`. |

### 7.2 Front-end layout fragments

| Fragment | Purpose |
|---|---|
| `frontend_layout` | Extends the shared front-end layout: sets the document text direction from the active language, adds the right-to-left marker and the portal marker to the page wrapper, adds a `Skip to Content` link for non-employees, replaces the header with the portal navigation bar (company logo linking to the root, the sign-in placeholder and the user menu), adds the full-width alert slot, and adds the social preview metadata (title, description, site name, image, image dimensions and the summary card marker) when a page declares a preview object. |
| `footer_language_selector` | Inserts the language selector in the footer, opening upwards. Shipped as a separate fragment so that it can be switched off. |
| `language_selector` | The language menu itself: rendered only when more than one front-end language exists; shows the full label or the compact code; each entry rewrites the current address for that language. |
| `user_dropdown` | The signed-in user's menu: avatar or icon and name, then `Apps` (employees only, pointing at the back office), a divider and `Logout`. |
| `my_account_link` | Inserts `My Account`, pointing at `/my/home`, before the logout divider of the user menu. |
| `placeholder_user_sign_in` | An empty insertion point for the sign-in entry. |
| `user_sign_in` | Fills that insertion point with a `Sign in` entry for anonymous visitors, pointing at the sign-in page, with a red dot marker when a profiling session is active. |
| `user_sign_in_redirect` | A variant of the sign-in entry that carries the current path as the destination. |
| `portal_breadcrumbs` | The breadcrumb: a home icon linking to `/my/home`, then `Addresses` and `Details` for the account pages. Each document domain inserts its own entries with an explicit priority, which is what fixes their order (sales 20, purchasing 25, invoicing 30, timesheets 35, projects and tasks their own). |
| `portal_layout` | The page frame: when the page is not the account page and does not use the search-bar breadcrumb, a breadcrumb row with the record navigator on the right; then the content container. For the account pages, the heading `My account`, the content column, the account sidebar on a wide screen and the offcanvas sidebar on a narrow one. |
| `portal_back_in_edit_mode` | The banner shown to a website editor previewing the portal: `This is a preview of the customer portal.` and a link `Back to edit mode`. |
| `portal_my_home` | The home page: the four optional category rows and the always-present common row with the address book and the security cards, plus the loading spinner. |
| `portal_docs_entry` | One home card: the optional pictogram, the counter placeholder, the title, the descriptive line and the link. Hidden unless its counter is known non-zero or it is a configuration card. |
| `portal_docs_entry_layout` | The optional variant that shows the pictogram of every card. Marked as an optional inherit, so a website editor may switch it off. |
| `portal_table` | The standard list table frame with the navigator underneath. |
| `portal_record_sidebar` | The record sidebar frame: a title block, an entry block and the credit line. |
| `portal_searchbar` | The search bar described in [workflows.md](workflows.md): breadcrumb or title, sort menu, filter menu, group menu, scoped search form and the narrow-screen toggle labelled `Toggle filters`. |
| `portal_contact` | The `Your contact` block of the sidebar: the representative's name, electronic mail address, telephone and city. |
| `portal_my_contact` | A compact contact block with an avatar, a name and a `Send message` link anchored at the discussion thread. |
| `side_content` | The account sidebar: avatar, name, company name, the Contact rendered with its address, the `Edit information` link and the representative block. |
| `portal_my_security` | The security page, with the sections described in [interfaces.md](interfaces.md). |
| `record_pager` | The previous and next controls of a record page. |
| `pager` | The page navigator of a list page. |
| `message_thread` | The discussion thread anchor, carrying the model, the record identifier, the token, the recipient identifier, the signed identity, the page size, whether the composer is allowed and whether the two-column layout is used. |
| `signature_form` | The signature panel, carrying the address to call, the default name, the drawing mode, the button label, the drawing area ratio, the signature kind and the pen colour. |
| `portal_sidebar` | A page frame for a record page that uses a scroll-spy sidebar. |
| `my_addresses` | The address book page with its two sections and the `Same as delivery address` switch. |
| `address_list` | The list of address cards of one section. |
| `address_card` | One address card with its badge, its edit control and its remove control. |
| `address_form_fields` | The address input set shared by the account form and the address form. |
| `address_footer` | The `Discard` and `Save Address` controls. |
| `portal_my_details` | The account details page. |
| `address_management` | The address creation and update page, titled `Billing address` or `Delivery address`. |
| `message_document_unfollowed` | Renders the unfollow confirmation inside the portal layout instead of the plain public layout. |
| `portal_share_template` | The invitation body posted by the share dialog. |
| `rating_widget_stars_static` | The static star row, in the full and the compressed variant. |
| `rating_stars_static_popup_composer` | The star row plus the review button and the modal that hosts the rating composer. |
| `message_thread` (rating variant) | Adds the "show ratings" marker to the discussion thread anchor. |

### 7.3 Front-end dialog fragments

| Fragment | Purpose |
|---|---|
| Identity check | The dialog `Security Control` with the heading `Please enter your password to confirm you own this account`, a password input and a `Forgot password?` link, confirmed with `Confirm Password`. |
| Key description | The dialog asking for a key description (`Name your key`, `Enter a description of and purpose for the key.`, the input placeholder `What's this key for?`, the warning that the description is the only way to identify the key, the heading `Give a duration for the key's validity`, the duration selector and the note `The key will be deleted once this period has elapsed.`). |
| Key display | The dialog `Write down your key` showing the generated key once, with the explanation and the warning that the key cannot be retrieved later and grants full access to the account. |
| Composer | The portal message composer: the "you must be signed in to post" notice for anonymous readers when no identity is proven, the avatar, the text area with the placeholder `Write a message...`, the attachment button titled `Add attachment`, the submit button and the pending-attachment list with its delete controls. |

---

## 8. Shipped message template

| Template | Entity it renders against | Subject | Sender | Recipient | Body |
|---|---|---|---|---|---|
| `Settings: New Portal User Invite` | User | `Your account at {{ company name }}` | the company's formatted address when set, otherwise the acting user's formatted address | the user's formatted address | Described in section 1.1 of [workflows.md](workflows.md). Its description reads `Sent to new portal user after you invited them`. It renders the welcome message of the invitation session and a tracking medium of `portalinvite`. |

The share dialog does not use a message template: it posts a rendered fragment as an internal note with the light notification layout.

---

## 9. Access groups

| Group | Canonical identifier | What it may do in this domain |
|---|---|---|
| Public | `base.group_public` (`Role / Public`) | Open a shared document page with a valid token; read a report through a token; post, edit, react and rate only when a signed recipient identity or a token resolves a Contact; fetch a country's address layout. Nothing else: every account page requires a session. |
| Portal | `base.group_portal` (`Role / Portal`) | Everything the public visitor may do, plus the home page, the list pages, the record pages allowed by the record rules of each document domain, the address book, the account details form, the security page with its password, second factor, passkey, session and (when allowed) application key management, and the account deletion request. Read, write, create and delete on the second-factor enrollment wizard, bounded by the platform rule that a person may only touch their own. |
| Internal user | `base.group_user` (`Role / User`) | Everything a portal user may do on their own account, plus toggling a message between public and internal. An employee may not change their own name or electronic mail address through the address form. |
| Contact manager | `base.group_partner_manager` (`Creation`, under the privilege `Contact`) | Read, write and create the three wizards of this domain; therefore grant, revoke and re-invite portal access, and share a document. |
| Website editor | `website.group_website_restricted_editor` | Publish a reply under a rating without needing write permission on the rated record; enable or disable the optional layout variants. |
| Settings administrator | `base.group_system` | Set the `Customer API Keys` switch. |

---

## 10. Model access rules shipped by this domain

| Entity | Group | Read | Write | Create | Delete |
|---|---|---|---|---|---|
| Portal Share Wizard | Contact manager | yes | yes | yes | no |
| Portal Access Wizard | Contact manager | yes | yes | yes | no |
| Portal Access Wizard User | Contact manager | yes | yes | yes | no |

Shipped by the second-factor bridge:

| Entity | Group | Read | Write | Create | Delete |
|---|---|---|---|---|---|
| Second Factor Enrollment Wizard | Portal | yes | yes | yes | yes |

No other model access rule is shipped by this domain. The permissions that decide what an external person sees on each document are shipped by that document's own domain and are summarized in `PORT-RULE-171`.

---

## 11. Record rules

This domain ships no record rule. It depends on the ones shipped by the document domains, listed in `PORT-RULE-171` of [business-rules.md](business-rules.md).

---

## 12. Scheduled job

| Job | Runs as | Interval | Priority | Work |
|---|---|---|---|---|
| `Base: Portal Users Deletion` | the system robot user | once a day | 8 | Processes the User Deletion Request queue in batches of fifty, as specified in section 30 of [workflows.md](workflows.md). Owned by [Identity and Access](../identity-and-access/README.md); listed here because the portal is the only place that fills the queue. |

---

## 13. Master data that must exist

| Prerequisite | Why |
|---|---|
| The portal access group | Every external user is a member of it; every document domain attaches its portal record rules to it. |
| The public access group | An anonymous visitor is a member of it; granting portal access removes it. |
| The portal user template user | Duplicated to create every new external user, so that the new account starts with the right groups, the right notification preferences and the right home action. |
| The anonymous public user | The identity used for a request without a session; its Contact is archived, which is why a newly created address is never attached to it. |
| At least one company with a name and, ideally, a logo, an electronic mail address, a telephone number and a website | Used in the invitation message, in the report layout and in the portal navigation bar. |
| The country list with, per country, an address layout, a "subdivision required" flag and a "postal code required" flag | Drives the address form, its field order and its mandatory set. |
| The country subdivision list | Fills the subdivision selector. |
| At least one published front-end language | The layout reads the language direction from it; more than one makes the selector appear. |
| The private-note message subtype and the public comment message subtype | The first is used by the share invitation and is excluded from portal threads; the second is what makes a posted message visible on a portal page. |
| The shipped placeholder image | Served when an author picture cannot be resolved. |
| The database secret | Signs every recipient identity and every notification link. |

---

## 14. What a document domain must provide to have a portal presence

This is the contract a replacement must expose so that a new document kind can be added without touching this domain.

1. Adopt the Portal Access Mixin on the document entity, and override the portal web address rule to return the page path of that document.
2. Optionally override the access warning rule to block sharing in certain states.
3. Declare, on the document entity, which field holds the secret used for external posting (the default is the access token contributed by the mixin), and optionally a logical-parent signature rule.
4. Declare model access and record rules for the portal group.
5. Contribute a home card fragment into one of the four category rows, declaring a counter name.
6. Contribute the counter computation for that name, returning zero when the reader has no read permission on the model.
7. Contribute a list page endpoint that requires a session, builds its condition, uses the shared navigator and stores its history key in the session.
8. Contribute a record page endpoint that accepts anonymous requests, resolves the document through the shared access check, redirects to `/my` on failure, handles the report parameters, and passes its values through the shared record-page value builder with its history key.
9. Contribute the breadcrumb entries with an explicit priority.
10. Contribute the sidebar title and entries, using the shared sidebar frame.
11. Contribute the discussion thread anchor with the token and, when applicable, the recipient parameters.
12. Contribute, when the document can be signed, an endpoint that receives a name and a signature image and returns a redirection address.
13. Contribute, when the document can be paid, the payment context values and a transaction endpoint that resolves the document through the shared access check.
