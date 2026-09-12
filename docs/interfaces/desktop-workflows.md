# Desktop workflows

How a user works with the system on a computer: how the applications are reached, what each menu entry opens, what a
user can do on every kind of view, what the buttons of each major document do and what guards them, and what the
dashboards show. Screens are described as workflows on views, with the fields shown, the actions offered and the
conditions that enable them; no client technology is named, because a replacement may render these views in any way that
preserves the behavior.

## Part 1: the navigation model

### Applications, menus and actions

The navigation is a tree. The root entries of the tree are the **applications**; every other entry is a menu inside one
application. An entry has a name, a parent, an order number, an optional action, an optional icon (roots only) and an
optional list of access groups.

| Rule | Statement |
|---|---|
| NAV-RULE-001 | Entries are ordered by their order number, then by name. Entries without an order number sort after the numbered ones. |
| NAV-RULE-002 | An entry restricted to access groups is shown only to users who belong to at least one of them. |
| NAV-RULE-003 | An entry whose action the user may not run, and which has no visible child, is hidden. An application whose every child is hidden is itself hidden. |
| NAV-RULE-004 | An entry without an action is a heading: opening it opens its first visible child. |
| NAV-RULE-005 | The menu tree is loaded once per session and per language, and it is not cached between sessions, because group membership may change. |

An **action** is what an entry opens. Five kinds exist.

| Kind | What it opens | Main properties |
|---|---|---|
| Window action | A set of views over one entity | entity, ordered list of view kinds, filter condition, context (default filters, groupings and values for new records), target (current screen, dialog, full screen or new browser tab), row limit, empty-state help text, access groups |
| Client action | A screen that is not a set of views over one entity (messaging, the point of sale station, the site editor, a dashboard, the settings screen) | an identifier and its own parameters |
| Report action | A printed document, described in [`report-and-export-documents.md`](report-and-export-documents.md) | report definition |
| Address action | An address to open, either inside the client or in a new tab | the address, computed or fixed |
| Server action | Server-side work that returns one of the other action kinds, or nothing | the operation and its parameters |

### The action stack and the navigation trail

Opening an action from a menu clears the stack and puts that action at its root. Opening a record from a list pushes the
form on the stack. Opening a related record from a form pushes again. The navigation trail shows one step per stack
entry, with the display name of the record or the name of the action; pressing a step pops the stack back to it. The
address of the screen encodes the stack, therefore reloading the page or sharing the address reopens the same screen at
the same record, subject to the access rights of whoever opens it.

A dialog target opens the action in a window on top of the current screen and does not touch the stack. A full-screen
target hides the navigation chrome. A new-tab target opens the address outside the client.

### Contextual actions

An action may be bound to an entity as a contextual action: it is then offered in the action menu of a list or form of
that entity, applied to the selected records. Report definitions bound this way appear in the print menu; server actions
bound this way appear in the action menu.

### The frame of every screen

| Element | Behavior |
|---|---|
| Application switcher | Lists the visible applications with their icons; searching filters both applications and their menu entries by name. |
| Menu bar | The entries of the current application, expanded on demand. |
| Navigation trail | The action stack, described above. |
| Company selector | Shown when the user may access several companies. It lets the user switch the active company and toggle further companies on and off; the set of active companies filters every list and defaults every new record. |
| Search bar | Present on every multi-record view; described in part 3. |
| Systray | The activity counter and its list, the messaging counter and its list, the presence and check-in control when the attendance capability is installed, the running timer when time tracking is installed, the debug indicator when diagnostics are on, and the user menu. The counters are kept current by pushed notifications, specified in part 3. |
| User menu | The profile of the user, the documentation link, the support link, the keyboard shortcut list (part 3), the log-out action and, for an administrator, the developer tools. |
| Command palette | Opened from the keyboard; it searches menu entries, conversations and the commands of the current screen. Specified in part 3. |

## Part 2: the menu map of every application

The tables below list every menu entry that opens an action, with the action it runs, the entity that action shows, the
views it offers in order (the first is the one opened), the default filters, groupings and new-record values the action
applies, and the access groups the entry is restricted to. An empty entity means the action is not a set of views over
one entity (a client screen, an address, or work that returns another action).

The installation ships eight hundred and ninety-three menu entries, of which six hundred and forty-five open an action;
the remaining two hundred and forty-eight only group other entries and appear as the intermediate segments of the menu
paths. The tables below list six hundred and thirty of them, spread over the thirty-five applications a user works in.
Fifteen entries that open an action are deliberately not listed: the ten entries of the application that drives the
automated test suite, the single entry of a translation test fixture application, one page-configuration entry of the
same fixture inside the site application, and the three application entries whose own action is repeated by a child
entry that is listed (the discussion application, the dashboard application and the course application each carry an
action on the application entry itself, and that same action already appears as a child entry).

An application entry that carries an action of its own and has no child entry is listed as a single row bearing the name
of the application, as the personal task application is.

### Discuss

Menu order 5; contributed by the Discuss capability package; visible to the access groups: Role / User. The application contains 7 menu entries that open an action.

| Menu path | Action | Entity | Views (first is the default) | Default filters, groupings and values | Restricted to |
|---|---|---|---|---|---|
| Discuss | Messaging |  |  |  |  |
| Channels | Channels | Discussion Channel | card, form |  |  |
| Configuration > Notifications | Notification settings |  |  |  |  |
| Configuration > Voice & Video | Voice and video settings |  |  |  |  |
| Configuration > Canned Responses | Canned Responses | Canned Response | list, form, card |  |  |
| Configuration > Roles | Roles | Messaging Role | list, form |  |  |
| Technical > Call History | Call History | Keep the call history | list, form |  |  |

### Calendar

Menu order 10; contributed by the Calendar capability package; visible to the access groups: Role / User. The application contains 4 menu entries that open an action.

| Menu path | Action | Entity | Views (first is the default) | Default filters, groupings and values | Restricted to |
|---|---|---|---|---|---|
| Calendar | Meetings | Calendar Event | calendar, list, form |  | Role / User |
| Configuration | Meetings | Calendar Event | calendar, list, form |  | Role / Administrator, Technical Features |
| Configuration > Settings | Settings | Configuration Settings | form |  | Role / Administrator |
| Configuration > Reminders | Calendar Alarm | Calendar Reminder | list, form |  | Technical Features |

### Contacts

Menu order 20; contributed by the Contacts capability package; visible to the access groups: Role / User, Creation. The application contains 12 menu entries that open an action.

| Menu path | Action | Entity | Views (first is the default) | Default filters, groupings and values | Restricted to |
|---|---|---|---|---|---|
| Contacts | Contacts | Contact | list, card, form, activity | new records: is company = True |  |
| Configuration > Contact Tags | Contact Tags | Contact Tag |  |  |  |
| Configuration > Website Tags | Website Tags | Partner Tags - These tags can be used on website |  |  |  |
| Configuration > Industries | Industries | Industry | list, form |  |  |
| Configuration > Localization > Countries | Countries | Country |  |  |  |
| Configuration > Localization > Cities | Cities | City | list |  |  |
| Configuration > Localization > Fed. States | Fed. States | Country Subdivision |  |  |  |
| Configuration > Localization > Country Group | Country Group | Country Group |  |  |  |
| Configuration > Localization > GIB Tax Offices | GIB Tax Offices | Turkish Tax Office |  |  |  |
| Configuration > Bank Accounts > Banks | Banks | Bank | list, form |  |  |
| Configuration > Bank Accounts > Bank Accounts | Bank Accounts | Bank Account | list, form |  |  |
| Configuration > Identification Type | Identification Type | Localization Latam Identification Type | list |  |  |

### Customer Relationship Management

Menu order 25; contributed by the Customer Relationship Management capability package; visible to the access groups: User: Own Documents Only, Administrator. The application contains 26 menu entries that open an action.

| Menu path | Action | Entity | Views (first is the default) | Default filters, groupings and values | Restricted to |
|---|---|---|---|---|---|
| Sales > My Pipeline | My pipeline (opportunities of the current user) |  |  |  |  |
| Sales > My Activities | My Activities | Lead | list, card, chart, pivot, calendar, form, activity | filter: assigned to me; new records: type = opportunity | User: Own Documents Only |
| Sales > My Quotations | Quotations | Sales Order | list, card, form, calendar, pivot, chart, activity | filter: my quotation |  |
| Sales > Teams | Teams | Sales Team | card, form |  |  |
| Sales > Customers | New contact form |  | list, card, form, activity |  |  |
| Leads | Leads | Lead | list, card, chart, pivot, calendar, form, activity | filter: type, to process; new records: type = lead | Show Lead Menu |
| Reporting > Forecast | Opportunity forecast |  |  |  |  |
| Reporting > Pipeline | Pipeline Analysis | Lead | chart, pivot, list, form | filter: opportunity, current |  |
| Reporting > Leads | Leads Analysis | Lead | chart, pivot, list | filter: active, inactive, create date |  |
| Reporting > Activities | Activities | Activity Analysis Report | chart, pivot, list | filter: completion date |  |
| Reporting > Partnerships | Partnership Analysis | Customer Relationship Management Partnership Ana | chart |  |  |
| Reporting > Lead Generation Views | Lead Generation Views | Customer Relationship Management Reveal View | list, form |  | Technical Features |
| Configuration | My pipeline (opportunities of the current user) |  |  |  | Administrator |
| Configuration > Settings | Settings | Configuration Settings | form |  | Role / Administrator |
| Configuration > Sales Teams | Sales Teams | Sales Team | list, form |  |  |
| Configuration > Teams Members | Team Members | Sales Team Member | card, list, form |  | Technical Features |
| Configuration > Activities > Activity Types | Activity Types | Activity Type | list, card, form |  |  |
| Configuration > Activities > Activity Plans | Lead Activity Plans | Activity Plan | list, card, form | new records apply to: Lead | Administrator |
| Configuration > Recurring Plans | Recurring Plans | Recurring Revenue Plan | list |  | Show Recurring Revenues Menu |
| Configuration > Pipeline > Stages | Stages | Pipeline Stage |  |  | Technical Features |
| Configuration > Pipeline > Tags | Tags | Sales Tag |  |  |  |
| Configuration > Pipeline > Lost Reasons | Lost Reasons | Lost Reason | list, form |  |  |
| Configuration > Customer Relationship Management Partners > Levels | Levels | Partner Grade |  |  |  |
| Configuration > Customer Relationship Management Partners > Partner Activations | Partner Activations | Partner Activation | list, form |  |  |
| Configuration > Lead Generation > Lead Mining Requests | Lead Mining Requests | Lead Mining Request | list, form |  |  |
| Configuration > Lead Generation > Visits to Leads Rules | Visits to Leads Rules | Customer Relationship Management Lead Generation | list, form |  |  |

### Sales

Menu order 30; contributed by the Sales capability package. The application contains 32 menu entries that open an action.

| Menu path | Action | Entity | Views (first is the default) | Default filters, groupings and values | Restricted to |
|---|---|---|---|---|---|
| Orders > Quotations | Quotations | Sales Order | list, card, form, calendar, pivot, chart, activity | filter: my quotation | User: Own Documents Only |
| Orders > Orders | Sales Orders | Sales Order | list, card, form, calendar, pivot, chart, activity | filter: sales | User: Own Documents Only |
| Orders > Sales Teams | Sales Teams | Sales Team | card, form |  | Administrator |
| Orders > Customers | Customers | Contact | list, card, form | filter: customer; new records: is company = True, customer rank = 1 | User: Own Documents Only |
| To Invoice > Orders to Invoice | Orders to Invoice | Sales Order | list, form, calendar, chart, pivot, card, activity |  |  |
| To Invoice > Orders to Upsell | Orders to Upsell | Sales Order | list, form, calendar, chart, pivot, card, activity |  |  |
| Products > Products | Products | Product Template |  |  |  |
| Products > Product Variants | Product Variants | Product Variant | card, list, form, activity |  | Manage Product Variants |
| Products > Pricelists | Pricelists | Pricelist | list, card, form |  | Basic Pricelists |
| Products > Discount & Loyalty | Discount & Loyalty | Loyalty Program | list, form |  | Administrator |
| Products > Gift cards & Electronic Wallet | Gift cards & Electronic Wallet | Loyalty Program | list, form | new records: program type = gift_card | Administrator |
| Reporting > Sales | Sales Analysis | Sales Analysis Report | chart, pivot, list, form | filter: Sales, order date |  |
| Reporting > Salespersons | Sales Analysis By Salespersons | Sales Analysis Report | chart, pivot | filter: User, order date; group by: user |  |
| Reporting > Products | Sales Analysis By Products | Sales Analysis Report | chart, pivot | filter: Sales, Product, order date; group by: product |  |
| Reporting > Customers | Sales Analysis By Customers | Sales Analysis Report | chart, pivot | filter: Customer, order date; group by: partner |  |
| Configuration > Settings | Settings | Configuration Settings | form |  | Role / Administrator |
| Configuration > Sales Teams | Sales Teams | Sales Team | list, form |  |  |
| Configuration > Sales Orders > Quotation Templates | Quotation Templates | Quotation Template | list, form |  | Quotation Templates |
| Configuration > Sales Orders > Headers/Footers | Headers/Footers | Quotation Document | card, list, form |  |  |
| Configuration > Sales Orders > Delivery Methods | Delivery Methods | Shipping Method | list, form | group by: provider |  |
| Configuration > Sales Orders > Tags | Tags | Sales Tag |  |  |  |
| Configuration > Products > Attributes | Attributes | Product Attribute | list, form |  | Manage Product Variants |
| Configuration > Products > Categories | Categories | Product Category |  |  |  |
| Configuration > Products > Combo Choices | Combo Choices | Product Combo | list, form |  |  |
| Configuration > Products > Product Tags | Product Tags | Product Tag | list, form |  |  |
| Configuration > Products > Units & Packagings | Units & Packagings | Unit of Measure |  |  | Manage Multiple Units of Measure |
| Configuration > Activities > Activity Types | Activity Types | Activity Type | list, card, form | new records apply to: Sales Order | Technical Features |
| Configuration > Activities > Activity Plans | Sale Order Plans | Activity Plan | list, card, form | new records apply to: Sales Order | Administrator |
| Configuration > Online Payments > Payment Providers | Payment Providers | Payment Provider | card, list, form |  |  |
| Configuration > Online Payments > Payment Methods | Payment Methods | Payment Method | list, card, form | filter: available pms |  |
| Configuration > Online Payments > Payment Tokens | Payment Tokens | Payment Token | list, form |  | Technical Features |
| Configuration > Online Payments > Payment Transactions | Payment Transactions | Payment Transaction | list, card, form, chart, pivot |  | Technical Features |

### Dashboards

Menu order 37; contributed by the Spreadsheet dashboard capability package. The application contains 3 menu entries that open an action.

| Menu path | Action | Entity | Views (first is the default) | Default filters, groupings and values | Restricted to |
|---|---|---|---|---|---|
| Dashboards | Accounting dashboard |  |  |  |  |
| My Dashboard | My Dashboard | Dashboard Board | form |  |  |
| Configuration > Dashboards | Dashboards | Spreadsheet Dashboard Group | list, form |  |  |

### Point of Sale

Menu order 50; contributed by the Point of Sale capability package; visible to the access groups: Administrator, User. The application contains 30 menu entries that open an action.

| Menu path | Action | Entity | Views (first is the default) | Default filters, groupings and values | Restricted to |
|---|---|---|---|---|---|
| Dashboard | Point of Sale | Point of Sale Configuration | card, list, form |  |  |
| Orders > Orders | Orders | Point of Sale Order | list, form, card, pivot |  | Administrator, User |
| Orders > Sessions | Sessions | Point of Sale Session | list, card, form |  | User |
| Orders > Payments | Payments | Point of Sale Payment | list, form | group by: payment method | Administrator, User |
| Orders > Consolidated Invoice | Consolidated Invoices | MyInvois Document | list, form |  |  |
| Orders > Preparation Printers | Preparation Printers | Point of Sale Printer | list, card, form |  |  |
| Orders > Customers | Customers | Contact | list, card, form | filter: customer; new records: is company = True, customer rank = 1 |  |
| Products > Products | Products | Product Template | card, list, form, activity | filter: available at the point of sale; new records: available at the point of salet of sale = True |  |
| Products > Product Variants | Product Variants | Product Variant | card, list, form, activity | filter: available at the point of sale; new records: available at the point of salet of sale = True | Manage Product Variants |
| Products > Combo Choices | Combo Choices | Product Combo | list, form |  |  |
| Products > Pricelists | Pricelists | Pricelist | list, card, form |  | Basic Pricelists |
| Products > Discount & Loyalty | Discount & Loyalty | Loyalty Program | list, form |  | Administrator |
| Products > Gift cards & Electronic Wallet | Gift cards & Electronic Wallet | Loyalty Program | list, form | new records: program type = gift_card | Administrator |
| Reporting > Orders | Orders Analysis | Point of Sale Orders Report | chart, pivot | filter: not cancelled |  |
| Reporting > Sales Details | Sales Details (dialog) | Point of Sale Sales Details Wizard | form |  |  |
| Reporting > Session Report | Session Report (dialog) | Point of Sale Daily Sales Report Wizard | form |  |  |
| Reporting > Malta EXO > Compliance Letter | Compliance Letter (dialog) | Compliance Letter for EXO Number Wizard | form |  |  |
| Reporting > French Statements > Sales Closings | Sales Closings | Sale Closing | list, form |  |  |
| Reporting > French Statements > Check Move Integrity Reporting | Point of sale chain integrity check |  |  |  |  |
| Configuration > Settings | Settings | Configuration Settings | form |  | Role / Administrator |
| Configuration > Payment Methods | Payment Methods | Point of Sale Payment Method | list, card, form | group by: account | Administrator, User |
| Configuration > Presets | Presets | Point of Sale Preset | list, form |  | Preset Menu |
| Configuration > Coins/Bills | Coins/Bills | Point of Sale Cash Denomination | list, form |  | Administrator |
| Configuration > Floor Plans | Floor Plans | Restaurant Floor | list, card, form |  | User |
| Configuration > Point of Sales | Point of Sale List | Point of Sale Configuration | list, form |  |  |
| Configuration > Note Models | Note Models | Point of Sale Note | list |  |  |
| Configuration > Products > Point of Sale Product Categories | Point of Sale Product Categories | Point of Sale Category | list, card, form |  |  |
| Configuration > Products > Attributes | Attributes | Product Attribute | list, form |  | Manage Product Variants |
| Configuration > Products > Product Tags | Product Tags | Product Tag | list, form |  |  |
| Configuration > Taxes | Taxes | Tax | list, card, form | filter: sale, purchase | Technical Features |

### Invoicing

Menu order 55; contributed by the Invoicing capability package; visible to the access groups: Show Accounting Features - Readonly, Invoicing. The application contains 67 menu entries that open an action.

| Menu path | Action | Entity | Views (first is the default) | Default filters, groupings and values | Restricted to |
|---|---|---|---|---|---|
| Dashboard | Dashboard | Journal | card, form | filter: dashboard | Basic |
| Customers > Invoices | Invoices | Journal Entry | list, card, form, activity | filter: customer invoice, customer receipt; new records: move type = customer invoice |  |
| Customers > Credit Notes | Credit Notes | Journal Entry | list, card, form, activity | filter: customer credit note; new records: move type = customer credit note |  |
| Customers > Sale Invoices and Credit Notes (CL) | Sale Invoices and Credit Notes | Journal Entry | list, form | new records: move type = customer invoice |  |
| Customers > Payments | Customer Payments | Payment | list, card, form, chart, activity | filter: inbound filter; new records: payment type = inbound, partner type = customer, move journal types = ('bank |  |
| Customers > Third Party Checks | Third Party Checks | Account payment check | list, form, calendar, chart, pivot | filter: checks on hand |  |
| Customers > Products | Products | Product Template |  | filter: to sell |  |
| Customers > Customers | Customers | Contact | list, card, form | filter: customer; new records: is company = True, customer rank = 1 |  |
| Vendors > Bills | Bills | Journal Entry | list, card, form, activity | filter: vendor bill, vendor receipt; new records: move type = vendor bill |  |
| Vendors > Refunds | Refunds | Journal Entry | list, card, form, activity | filter: vendor credit note; new records: move type = vendor credit note |  |
| Vendors > Vendor Bills and Refunds (CL) | Vendor Bills and Refunds | Journal Entry | list, form | new records: move type = vendor bill |  |
| Vendors > Payments | Vendor Payments | Payment | list, card, form, chart, activity | filter: outbound filter; new records: payment type = outbound, partner type = supplier, move journal types = ('bank |  |
| Vendors > Employee Expenses | Employee Expenses | Expense | list, card, form, pivot, chart | filter: all approved, all to pay | All Approver |
| Vendors > Own Checks | Own Checks | Account payment check | list, form, calendar, chart, pivot | filter: checks on hand |  |
| Vendors > Products | Products | Product Template |  | filter: to purchase |  |
| Vendors > Vendors | Vendors | Contact | list, card, form | filter: supplier; new records: is company = True, supplier rank = 1 |  |
| Accounting > Transactions > Journal Entries | Journal Entries | Journal Entry | list, card, form, activity | filter: posted; new records: move type = entry | Show Accounting Features - Readonly |
| Accounting > Transactions > Analytic Items | Analytic Items | Analytic Line | list, card, form, chart, pivot |  | Analytic Accounting |
| Accounting > Transactions > Permanent Account Number Entity | Permanent Account Number Entity | Indian Permanent Account Number Entity | list, form |  |  |
| Accounting > Closing > Secure Entries | Secure Journal Entries (dialog) | Entry Securing Wizard | form |  | Technical Features, Show Inalterability Features |
| Review > Control > Journal Items | Journal Items | Journal Item | list, pivot, chart, card | filter: posted | Show Accounting Features - Readonly |
| Review > Logs > Audit Trail | Audit Trail | Message | list |  |  |
| Reporting > Management > Product Margins… | Product Margins (dialog) | Product Margin Wizard | form |  |  |
| Reporting > Management > Invoice Analysis | Invoices Analysis | Invoices Statistics | chart, pivot | filter: current, customer; group by: invoice date by month |  |
| Reporting > Management > Analytic Report | Analytic Reporting | Analytic Line |  | filter: group by analytic account, fiscal date, profit and loss accounts | Show Accounting Features - Readonly |
| Reporting > Argentinean Statements > Gross Income Tax - Sales by jurisdiction | Gross Income Tax - Sales by jurisdiction | Invoices Statistics | pivot | filter: current, customer, with document, company, group by provincial jurisdiction, group by account, accounting date this year |  |
| Reporting > Argentinean Statements > Gross Income Tax - Purchases by jurisdiction | Gross Income Tax - Purchases by jurisdiction | Invoices Statistics | pivot | filter: current, supplier, with document, company, group by provincial jurisdiction, group by account, accounting date this year |  |
| Reporting > France > E-reporting | E-Reporting | French Approved Dematerialization Platform Flow | list, form |  |  |
| Reporting > France > Sales Closings | Sales Closings | Sale Closing | list, form |  |  |
| Reporting > Hungary > Tax audit export - Adóhatósági Ellenőrzési Adatszolgáltatás | Tax audit export - Adóhatósági Ellenőrzési Adatszolgáltatás (dialog) | Tax audit export - Adóhatósági Ellenőrzési Adats | form |  |  |
| Reporting > Accounting Tests | Accounting Tests | Accounting Assert Test | list, form |  | Technical Features |
| Configuration > Settings | Settings | Configuration Settings | form |  | Role / Administrator |
| Configuration > Accounting > Chart of Accounts | Chart of Accounts | Account | list, card, form |  | Show Accounting Features - Readonly |
| Configuration > Accounting > Taxes | Taxes | Tax | list, card, form | filter: sale, purchase |  |
| Configuration > Accounting > Tax Groups | Tax Groups | Tax Group | list, form |  | Technical Features |
| Configuration > Accounting > Journals | Journals | Journal | list, card, form |  | Administrator |
| Configuration > Accounting > Multi-Ledger | Multi-ledger | Journal Group |  |  | Show Accounting Features - Readonly |
| Configuration > Accounting > Fiscal Positions | Fiscal Positions | Fiscal Position | list, card, form |  |  |
| Configuration > Accounting > Currencies | Currencies | Currency | list, card, form |  |  |
| Configuration > Accounting > Cash Roundings | Cash Roundings | Cash Rounding | list, form |  | Allow the cash rounding management |
| Configuration > Accounting > GIB Codes | GIB Codes | Turkish Tax Codes (GIB Codes) |  |  | Technical Features |
| Configuration > Accounting > DDT | Transport Document | Transport Document | list, form |  | Technical Features |
| Configuration > Accounting > Document Types | Document Types | Latam Document Type |  |  |  |
| Configuration > Invoicing > Payment Terms | Payment Terms | Payment Terms | list, card, form |  |  |
| Configuration > Invoicing > Incoterms | Incoterms | International Commercial Term | list, form |  | Technical Features |
| Configuration > Invoicing > Product Categories | Categories | Product Category |  |  |  |
| Configuration > Invoicing > Electronic Data Interchange Proxy Users | Electronic Data Interchange Proxy User | Electronic Interchange Proxy User | list, form |  | Technical Features |
| Configuration > Invoicing > Tax Office | Tax Office | Tax office in Czech Republic | list, form | group by: region | Administrator |
| Configuration > Analytic Accounting > Analytic Accounts | Analytic Accounts | Analytic Account | list, card, form | filter: active | Analytic Accounting |
| Configuration > Analytic Accounting > Analytic Distribution Models | Analytic Distribution Models | Analytic Distribution Model | list, form |  | Analytic Accounting |
| Configuration > Analytic Accounting > Analytic Plans | Analytic Plans | Analytic Plan | list, form |  | Analytic Accounting |
| Configuration > ARCA > Document Types | Document Types | Latam Document Type |  | filter: localization |  |
| Configuration > ARCA > Responsibility Types | ARCA Responsibility Types | ARCA Responsibility Type |  |  |  |
| Configuration > ARCA > Earnings Scale | ARCA tax | Argentinean Earnings Scale | list, form |  |  |
| Configuration > Ecuadorian SRI > Payment Methods SRI | Payment Methods SRI | SRI Payment Method | list, form |  | Administrator |
| Configuration > Online Payments > Payment Providers | Payment Providers | Payment Provider | card, list, form |  |  |
| Configuration > Online Payments > Payment Methods | Payment Methods | Payment Method | list, card, form | filter: available pms |  |
| Configuration > Online Payments > Payment Tokens | Payment Tokens | Payment Token | list, form |  | Technical Features |
| Configuration > Online Payments > Payment Transactions | Payment Transactions | Payment Transaction | list, card, form, chart, pivot |  | Technical Features |
| Configuration > Egyptian Tax Authority > Thumb Drive | Thumb Drive | Thumb drive used to sign invoices in Egypt | list |  |  |
| Configuration > SInvoice > Symbols | Symbols | SInvoice symbol | list, form |  |  |
| Configuration > SInvoice > Templates | Templates | SInvoice template | list, form |  |  |
| Configuration > Spain Facturae Electronic Data Interchange > Certificates | Certificates for Facturae Electronic Data Interchange invoices on Spain | Digital Certificate | list, form | filter: scope facturae |  |
| Configuration > Spain Immediate Information Supply > Certificates | Certificates for Immediate Information Supply Electronic Data Interchange invoices on Spain | Digital Certificate | list, form | filter: scope immediate information supply | Administrator |
| Configuration > Spain TicketBAI > Licenses | Companies | Company | list, card, form |  | Administrator |
| Configuration > Spain TicketBAI > Certificates | Certificates for Electronic Data Interchange TicketBAI invoices on Spain | Digital Certificate | list, form | filter: scope tbai | Administrator |
| Configuration > Veri*Factu (Spain) > Certificates | Certificates for Veri*Factu | Digital Certificate | list, form | filter: scope verifactu | Administrator |

### Project

Menu order 70; contributed by the Project capability package; visible to the access groups: Administrator, User. The application contains 15 menu entries that open an action.

| Menu path | Action | Entity | Views (first is the default) | Default filters, groupings and values | Restricted to |
|---|---|---|---|---|---|
| Projects | All projects |  |  |  |  |
| Projects | All projects grouped by stage |  |  |  | Use Stages on Project |
| Tasks > My Tasks | My tasks |  |  | filter: open tasks; new records: assignees include the current user |  |
| Tasks > All Tasks | All tasks |  |  | filter: open tasks; new records: users = [(4, user_identifier)] |  |
| Reporting > Tasks Analysis | Task analysis |  |  |  |  |
| Reporting > Customer Ratings | Customer Ratings | Rating | card, list, pivot, chart, form | filter: rated on |  |
| Configuration > Settings | Settings | Configuration Settings | form |  | Role / Administrator |
| Configuration > Projects | Project configuration grouped by stage |  |  |  | Use Stages on Project |
| Configuration > Projects | Project configuration |  |  |  |  |
| Configuration > Project Stages | Project Stages | Project Stage | list, card, form |  | Use Stages on Project |
| Configuration > Activity Plans | Activity Plans | Activity Plan | list, card, form | new records apply to: Task |  |
| Configuration > Activity Types | Activity Types | Activity Type | list, card, form | new records apply to: Task |  |
| Configuration > Project Roles | Project Roles | Project Role | list, card, form |  |  |
| Configuration > Tags | Tags | Project Tag |  |  |  |
| Configuration > Task Stages | Task Stages | Task Stage | list, card, form | new records: user = False | Technical Features |

### Timesheets

Menu order 75; contributed by the Task Logs capability package; visible to the access groups: User: own timesheets only. The application contains 9 menu entries that open an action.

| Menu path | Action | Entity | Views (first is the default) | Default filters, groupings and values | Restricted to |
|---|---|---|---|---|---|
| Timesheets > All Timesheets | All Timesheets | Analytic Line | list, form, card, pivot, chart | filter: week | User: all timesheets |
| Timesheets > My Timesheets | My Timesheets | Analytic Line | list, form, card, pivot, chart |  | User: all timesheets |
| My Timesheets | My Timesheets | Analytic Line | list, form, card, pivot, chart |  | User: own timesheets only |
| Reporting > Timesheets > By Billing Type | Timesheets by Billing Type | Timesheet Analysis Report | pivot, chart |  |  |
| Reporting > Timesheets > By Employee | Timesheets by Employee | Timesheet Analysis Report | pivot, chart |  | User: all timesheets |
| Reporting > Timesheets > By Project | Timesheets by Project | Timesheet Analysis Report | pivot, chart |  |  |
| Reporting > Timesheets > By Task | Timesheets by Task | Timesheet Analysis Report | pivot, chart |  |  |
| Reporting > Timesheets / Attendance Analysis | Timesheets / Attendance Analysis | Timesheet Attendance Report | chart, pivot |  |  |
| Configuration | Settings | Configuration Settings | form |  | Role / Administrator |

### Website

Menu order 95; contributed by the Website capability package; visible to the access groups: Role / User. The application contains 57 menu entries that open an action.

| Menu path | Action | Entity | Views (first is the default) | Default filters, groupings and values | Restricted to |
|---|---|---|---|---|---|
| Site > Homepage | Site preview (opens the public site inside the client) |  |  |  |  |
| Site > Menu Editor | Site preview (opens the public site inside the client) |  |  |  |  |
| Site > Content > Model Pages | Website Model Pages | Website Model Page | list, card, form |  | Technical Features |
| Site > Content > Pages | Website Pages | Website Page | list, card |  |  |
| Site > Content > Blog Posts | Blog Post Pages | Blog Post | list, card, form |  |  |
| Site > Content > Products | Product Pages | Product Template | list, card |  |  |
| Site > Content > Events | Event Pages | Event | list, card |  |  |
| Site > Content > Courses | Course Pages | Course | list, card, form |  |  |
| Site > Content > Jobs | Job Pages | Job Position | list, card, form |  | Interviewer |
| Site > Content > Forum Posts | Forum Post Pages | Forum Post | list, card, chart | filter: posts |  |
| Site > Content > Technical Pages | Technical Pages | Website Technical Page | list |  |  |
| Site > This page > Properties | Site preview (opens the public site inside the client) |  |  |  |  |
| Site > This page > Optimize SEO | Site preview (opens the public site inside the client) |  |  |  |  |
| Site > This page > Link Tracker | Site preview (opens the public site inside the client) |  |  |  |  |
| Site > This page > Page source editor | Site preview (opens the public site inside the client) |  |  |  |  |
| Site > This page > Edit Menu | Site preview (opens the public site inside the client) |  |  |  |  |
| Electronic Commerce > Orders > Orders | Orders | Sales Order | list, form, card, activity | filter: order confirmed, from website |  |
| Electronic Commerce > Orders > Unpaid Orders | Unpaid Orders | Sales Order | list, card, form, activity |  |  |
| Electronic Commerce > Orders > Abandoned Carts | Abandoned Carts | Sales Order | list, card, form, activity | filter: recovery email |  |
| Electronic Commerce > Orders > Customers | New customer form |  | list, card, form, activity |  |  |
| Electronic Commerce > Products > Products | Products | Product Template | card, list, form, activity | filter: published |  |
| Electronic Commerce > Products > Pricelists | Pricelists | Pricelist | list, card, form |  | Basic Pricelists |
| Electronic Commerce > Products > Electronic Commerce Categories | Electronic Commerce Categories | Website Product Category | list, form |  |  |
| Electronic Commerce > Products > Attributes | Attributes | Product Attribute | list, form |  | Manage Product Variants |
| Electronic Commerce > Products > Combo Choices | Combo Choices | Product Combo | list, form |  |  |
| Electronic Commerce > Products > Product Ribbons | Product Ribbons | Product Ribbon | list, form |  |  |
| Electronic Commerce > Products > Product Tags | Product Tags | Product Tag | list, form |  |  |
| Electronic Commerce > Products > Attribute Categories | Attribute Categories | Product Attribute Category | list |  | Technical Features |
| Electronic Commerce > Loyalty > Discount & Loyalty | Discount & Loyalty | Loyalty Program | list, form |  |  |
| Electronic Commerce > Loyalty > Gift cards & Electronic Wallet | Gift cards & Electronic Wallet | Loyalty Program | list, form | new records: program type = gift_card |  |
| Reporting > Analytics | Site analytics |  |  |  |  |
| Reporting > Electronic Commerce | Site dashboard |  |  |  | Role / Administrator, Editor and Designer |
| Reporting > Online Sales | Online Sales Analysis | Sales Analysis Report | pivot, chart | filter: confirmed | Administrator |
| Reporting > Visitors | Visitors | Website Visitor | card, list, form, chart | filter: last 7 days |  |
| Reporting > Page Views | Page Views | Website Visit Track | list | filter: type web address |  |
| Configuration > Settings | Settings | Configuration Settings | form |  | Role / Administrator |
| Configuration > Websites | Websites | Website | list, form |  | Technical Features |
| Configuration > Apps | Apps | Module | card, list, form | filter: category | Role / Administrator |
| Configuration > Redirects | Rewrite | Web Address Rewrite |  |  | Technical Features |
| Configuration > Menus | Website Menu | Website Menu | list, form | group by: website | Technical Features |
| Configuration > Electronic Commerce > Payment Providers | Payment Providers | Payment Provider | card, list, form |  |  |
| Configuration > Electronic Commerce > Payment Methods | Payment Methods | Payment Method | list, card, form | filter: available pms |  |
| Configuration > Electronic Commerce > Payment Tokens | Payment Tokens | Payment Token | list, form |  | Technical Features |
| Configuration > Electronic Commerce > Payment Transactions | Payment Transactions | Payment Transaction | list, card, form, chart, pivot |  | Technical Features |
| Configuration > Electronic Commerce > Delivery Methods | Delivery Methods | Shipping Method | list, form | group by: provider |  |
| Configuration > Electronic Commerce > Zip Prefix | Zip Prefix | Delivery Postal Code Prefix | list, form |  | Technical Features |
| Configuration > Electronic Commerce > Product Feeds | Product Feeds | Product Feed | list, form |  | Product Feed |
| Configuration > Blog > Blogs | Blogs | Blog | list, form |  |  |
| Configuration > Blog > Tags | Blog Tags | Blog Tag | list, form |  |  |
| Configuration > Blog > Tag Categories | Tag Category | Blog Tag Category | list, form |  |  |
| Configuration > Forum > Forums | Forums | Forum | list, form |  |  |
| Configuration > Forum > Ranks | Ranks | Karma Rank | list, form |  |  |
| Configuration > Forum > Tags | Forum Tags | Forum Tag | list, form |  |  |
| Configuration > Forum > Badges | Badges | Badge | card, list, form |  |  |
| Configuration > Forum > Close Reasons | Post Close Reason | Forum Post Closing Reason | list |  |  |
| Configuration > Mailing Lists > Mailing Lists | Mail Groups | Mail Group | card, list, form |  |  |
| Configuration > Mailing Lists > Moderation Rules | Moderation | Mailing List black/white list | list, form |  | Mail Group Administrator |

### Electronic Learning

Menu order 100; contributed by the Electronic Learning capability package; visible to the access groups: Officer. The application contains 14 menu entries that open an action.

| Menu path | Action | Entity | Views (first is the default) | Default filters, groupings and values | Restricted to |
|---|---|---|---|---|---|
| Courses > Courses | All Courses | Course | card, list, form |  |  |
| Courses > Contents | Contents | Course Content | card, list, form | filter: own publications |  |
| Courses > Certifications | Certifications | Survey | card, list, pivot, chart, form | new records: certification = True, scoring type = scoring_with_answers |  |
| Forum > Forums | Forums | Forum | list, form |  |  |
| Forum > Posts | Forum Posts | Forum Post | list, chart, pivot, form | filter: questions |  |
| Reporting > Courses | Courses | Course | list, chart, pivot, form |  |  |
| Reporting > Contents | Contents | Course Content | chart, list, form, pivot |  |  |
| Reporting > Revenues | Electronic Learning Revenues | Sales Analysis Report | chart, pivot | group by: date, product |  |
| Reporting > Attendees | Attendees | Course Enrollment | chart, pivot, list, card | filter: groupby member status |  |
| Reporting > Reviews | Reviews | Rating | card, list, chart, pivot, form |  |  |
| Reporting > Quizzes | Quizzes | Quiz Question | list, chart, pivot, form |  |  |
| Configuration > Settings | Settings | Configuration Settings | form |  | Role / Administrator |
| Configuration > Course Groups | Course Groups | Course Tag Group | list, form |  |  |
| Configuration > Content Tags | Content Tags | Content Tag | list, form |  |  |

### Email Marketing

Menu order 115; contributed by the Email Marketing capability package; visible to the access groups: User. The application contains 13 menu entries that open an action.

| Menu path | Action | Entity | Views (first is the default) | Default filters, groupings and values | Restricted to |
|---|---|---|---|---|---|
| Mailings | Mailings | Mass Mailing | list, card, form, calendar | filter: assigned to me; new records: user = user_identifier, mailing type = mail |  |
| Mailing Lists > Mailing Lists | Mailing Lists | Mailing List | card, list, form |  |  |
| Mailing Lists > Mailing List Contacts | Mailing List Contacts | Mailing Contact | list, card, form, chart, pivot | filter: not email bl |  |
| Campaigns | Campaigns | Campaign | card, list, form |  | Manage Mass Mailing Campaigns |
| Reporting > Mass Mailing Analysis | Mass Mailing Analysis | Mailing Trace Report | chart, pivot, list |  |  |
| Reporting > Opt-Out Report | Opt-Out Report | Mailing Subscription | chart, pivot, list, form | group by: opt out reason |  |
| Configuration > Settings | Settings | Configuration Settings | form |  | Role / Administrator |
| Configuration > Campaign Stages | Campaign Tracking Stages | Campaign Stage | list, form |  | Manage Mass Mailing Campaigns |
| Configuration > Campaign Tags | Campaign Tags | Campaign Tag |  |  | Manage Mass Mailing Campaigns |
| Configuration > Link Tracker | Link Tracker | Link Tracker | list, form, chart |  |  |
| Configuration > Blacklisted Email Addresses | Blacklisted Email Addresses | Email Blacklist |  |  |  |
| Configuration > Optout Reasons | Optout Reasons | Mailing Opt-Out Reason | list, form |  |  |
| Configuration > Favorite Filters | Favorite Filters | Mailing Filter | list, form | filter: saved by me |  |

### Text Message Marketing

Menu order 120; contributed by the Text Message Marketing capability package; visible to the access groups: User. The application contains 7 menu entries that open an action.

| Menu path | Action | Entity | Views (first is the default) | Default filters, groupings and values | Restricted to |
|---|---|---|---|---|---|
| Text Message Marketing | Text Message Marketing | Mass Mailing | list, card, form, calendar, chart | filter: assigned to me; new records: user = user_identifier, mailing type = sms | User |
| Mailing Lists > Mailing Lists | Mailing Lists | Mailing List | card, list, form |  | User |
| Mailing Lists > Mailing List Contacts | Mailing List Contacts | Mailing Contact | list, form | filter: not phone bl | User |
| Campaigns | Campaigns | Campaign | card, list, form |  | Manage Mass Mailing Campaigns |
| Reporting | Text Message Marketing Analysis | Mailing Trace Report | chart, pivot, list |  | User |
| Configuration > Blacklisted Phone Numbers | Blacklisted Phone Numbers | Phone Blacklist |  |  | User |
| Configuration > Link Tracker | Link Tracker | Link Tracker | list, form, chart |  | User |

### Events

Menu order 125; contributed by the Events Organization capability package; visible to the access groups: Registration Desk. The application contains 23 menu entries that open an action.

| Menu path | Action | Entity | Views (first is the default) | Default filters, groupings and values | Restricted to |
|---|---|---|---|---|---|
| Events | Events | Event | card, calendar, list, form, pivot, chart, activity |  | Registration Desk |
| Registration Desk | Attendance scanning surface |  |  |  | Registration Desk |
| Tracks | Event Tracks | Event Track | card, list, form, calendar, chart, activity |  | Technical Features |
| Reporting > Attendees | Attendees | Event Registration | chart, pivot, card, list, form | filter: last month creation, taken, status, group event; group by: create date day | User |
| Reporting > Revenues | Revenues | Event Sales Report | chart, pivot | filter: priced tickets, event date start | User |
| Configuration > Settings | Settings | Configuration Settings | form |  | Role / Administrator |
| Configuration > Lead Generation | Lead Generation Rule | Event Lead Rules | list, form |  | Administrator |
| Configuration > Event Templates | Event Templates | Event Template |  |  |  |
| Configuration > Event Stages | Event Stages | Event Stage | list, form |  |  |
| Configuration > Event Tags Categories | Event Tags Categories | Event Tag Category | list, form |  |  |
| Configuration > Event Questions | Event Question | Event Question | list, form |  |  |
| Configuration > Mail Schedulers | Events Mail Schedulers | Event Communication |  |  | Technical Features |
| Configuration > Booth Categories | Booth Category | Event Booth Category | list, form |  |  |
| Configuration > Booths | Booths | Event Booth | card, list, form, chart, pivot | group by: state | Technical Features |
| Configuration > Track Stages | Track Stages | Event Track Stage | list, card, form |  | Technical Features |
| Configuration > Track Locations | Event Locations | Event Track Location |  |  |  |
| Configuration > Track Tag Categories | Track Tag Categories | Event Track Tag Category | list, form |  | Technical Features |
| Configuration > Track Tags | Track Tags | Event Track Tag |  |  | Technical Features |
| Configuration > Track Visitors | Track Visitors | Event Track Visitor | list, form |  | Technical Features |
| Configuration > Sponsor Levels | Sponsor Levels | Event Sponsor Type |  |  | Technical Features |
| Configuration > Quizzes | Event Quizzes | Quiz | list, form |  | Technical Features |
| Configuration > Quiz Questions | Event Quiz Questions | Content Quiz Question | list, form |  | Technical Features |
| Configuration > Website Menus | Menus | Website Event Menu | list, form |  | Technical Features |

### Surveys

Menu order 130; contributed by the Surveys capability package; visible to the access groups: User. The application contains 5 menu entries that open an action.

| Menu path | Action | Entity | Views (first is the default) | Default filters, groupings and values | Restricted to |
|---|---|---|---|---|---|
| Participants | Participants | Survey Participation | list, card, form | group by: survey |  |
| Surveys | Surveys | Survey | card, list, form, activity |  |  |
| Questions & Answers > Questions | Questions | Survey Question | list, form | group by: page |  |
| Questions & Answers > Suggested Values | Suggested Values | Survey Answer Option | list, form | group by: question |  |
| Questions & Answers > Detailed Answers | Detailed Answers | Survey Participation Answer | list, form | group by: survey, user input |  |

### Purchase

Menu order 135; contributed by the Purchase capability package; visible to the access groups: Administrator, User. The application contains 12 menu entries that open an action.

| Menu path | Action | Entity | Views (first is the default) | Default filters, groupings and values | Restricted to |
|---|---|---|---|---|---|
| Orders > Requests for Quotation | Requests for Quotation | Purchase Order | list, card, form, pivot, chart, calendar, activity |  |  |
| Orders > Purchase Orders | Purchase Orders | Purchase Order | list, card, form, pivot, chart, calendar, activity |  |  |
| Orders > Purchase Agreements | Purchase Agreements | Purchase Agreement | list, card, form |  |  |
| Orders > Vendors | Vendors | Contact | list, card, form | filter: supplier; new records: is company = True, supplier rank = 1 |  |
| Products > Products | Products | Product Template |  |  |  |
| Products > Product Variants | Product Variants | Product Variant | list, card, form, activity |  | Manage Product Variants |
| Reporting > Purchase | Purchase Analysis | Purchase Analysis Report | chart, pivot | filter: orders, date approve | Administrator |
| Configuration > Settings | Settings | Configuration Settings | form |  | Role / Administrator |
| Configuration > Vendor Pricelists | Vendor Pricelists | Vendor Price | list, form, card | filter: active products |  |
| Configuration > Products > Attributes | Attributes | Product Attribute | list, form |  | Manage Product Variants |
| Configuration > Products > Categories | Categories | Product Category |  |  |  |
| Configuration > Products > Units & Packagings | Units & Packagings | Unit of Measure |  |  | Manage Multiple Units of Measure |

### Inventory

Menu order 140; contributed by the Inventory capability package; visible to the access groups: Administrator, User. The application contains 38 menu entries that open an action.

| Menu path | Action | Entity | Views (first is the default) | Default filters, groupings and values | Restricted to |
|---|---|---|---|---|---|
| Overview | Inventory Overview | Operation Type | card, form |  |  |
| Operations > Transfers > Receipts | Receipts of every incoming operation type |  |  |  | Administrator, User |
| Operations > Transfers > Deliveries | Deliveries of every outgoing operation type |  |  |  | Administrator, User |
| Operations > Transfers > Internal | Internal transfers of every internal operation type |  |  |  | Manage Multiple Stock Locations |
| Operations > Transfers > Manufacturings | Manufacturings | Manufacturing Order | list, card, form, calendar, activity | new records: company = the first active company | Administrator, User |
| Operations > Transfers > Dropships | Dropships | Transfer | list, card, form, calendar | filter: dropships; new records: company = the first active company | Administrator, User |
| Operations > Jobs > Batch Transfers | Batch Transfers | Batch Transfer | list, card, form | filter: draft, in progress |  |
| Operations > Jobs > Wave Transfers | Wave Transfers | Batch Transfer | list, card, form | filter: draft, in progress |  |
| Operations > Adjustments > Physical Inventory | Physical inventory count |  |  |  |  |
| Operations > Adjustments > Scrap | Scrap Orders | Scrap Order | list, form, card, pivot, chart |  |  |
| Operations > Adjustments > Landed Costs | Landed Costs | Landed Cost | list, form, card |  |  |
| Operations > Procurement > Replenishment | Replenishment |  |  |  | Administrator |
| Operations > Procurement > References | References | Reference between stock documents | list, form |  | Technical Features |
| Operations > Procurement Compute | Scheduled actions |  |  |  | Technical Features |
| Products > Products | Products | Product Template | card, list, form | new records: is storable = True |  |
| Products > Product Variants | Product Variants | Product Variant | list, form, card |  | Manage Product Variants |
| Products > Lots / Serial Numbers | Lots / Serial Numbers | Lot or Serial Number |  | group by: location | Manage Lots / Serial Numbers |
| Products > Packages | Packages | Package | list, card, form | filter: location, internal | Manage Packages |
| Reporting > Stock | Stock | Product Variant | list, form | new records: is storable = True |  |
| Reporting > Locations | Stock quantities |  |  |  | Manage Multiple Stock Locations, Manage Different Stock Owners, Technical Features |
| Reporting > Moves History | Moves History | Stock Move Line | list, card, pivot, form | filter: done |  |
| Reporting > Moves Analysis | Moves Analysis | Stock Move |  | filter: done |  |
| Configuration > Settings | Settings | Configuration Settings | form |  | Role / Administrator |
| Configuration > Warehouse Management > Warehouses | Warehouses | Warehouse |  |  |  |
| Configuration > Warehouse Management > Operations Types | Operations Types | Operation Type | list, form |  |  |
| Configuration > Warehouse Management > Locations | Locations | Location | list, form | filter: internal location | Manage Multiple Stock Locations |
| Configuration > Warehouse Management > Routes | Routes | Route | list, form |  | Manage Push and Pull inventory flows |
| Configuration > Warehouse Management > Rules | Rules | Stock Rule | list, form |  | Manage Push and Pull inventory flows |
| Configuration > Warehouse Management > Storage Categories | Storage Categories | Storage Category | list, form |  | Manage Multiple Stock Locations |
| Configuration > Warehouse Management > Putaway Rules | Putaway Rules | Putaway Rule | list |  | Manage Multiple Stock Locations |
| Configuration > Products > Categories | Categories | Product Category |  |  |  |
| Configuration > Products > Attributes | Attributes | Product Attribute | list, form |  | Manage Product Variants |
| Configuration > Products > Units & Packagings | Units & Packagings | Unit of Measure |  |  | Manage Multiple Units of Measure |
| Configuration > Products > Barcode Nomenclatures | Barcode Nomenclatures | Barcode Nomenclature | list, card, form |  | Technical Features |
| Configuration > Delivery > Delivery Methods | Delivery Methods | Shipping Method | list, form | group by: provider |  |
| Configuration > Delivery > Package Types | Package Types | Package Type |  |  | Manage Packages |
| Configuration > Delivery > Zip Prefix | Zip Prefix | Delivery Postal Code Prefix | list, form |  | Technical Features |
| Configuration > GİB e-Dispatch > GİB Plate Numbers | GİB Plate Numbers | GİB Plate numbers | list, form |  |  |

### Manufacturing

Menu order 145; contributed by the Manufacturing capability package; visible to the access groups: User, Administrator. The application contains 14 menu entries that open an action.

| Menu path | Action | Entity | Views (first is the default) | Default filters, groupings and values | Restricted to |
|---|---|---|---|---|---|
| Operations > Manufacturing Orders | Manufacturing Orders | Manufacturing Order | list, card, form, calendar, pivot, chart, activity | filter: to do; new records: company = the first active company |  |
| Operations > Work Orders | Work Orders | Work Order | list, card, form, calendar, pivot, chart | filter: ready, progress, blocked | Manage Work Order Operations |
| Operations > Unbuild Orders | Unbuild Orders | Unbuild Order | list, card, form, activity |  |  |
| Operations > Scrap | Scrap Orders | Scrap Order | list, form, card, pivot, chart |  |  |
| Planning > Procurement Compute Manufacturing | Scheduled actions |  |  |  | Technical Features |
| Products > Products | Products | Product Template | card, list, form | new records: is storable = True |  |
| Products > Product Variants | Product Variants | Product Variant | card, list, form |  | Manage Product Variants |
| Products > Bills of Materials | Bills of Materials | Bill of Materials | list, card, form | new records: company = the first active company |  |
| Products > Lots/Serial Numbers | Lots / Serial Numbers | Lot or Serial Number |  | group by: location | Manage Lots / Serial Numbers |
| Reporting > Work Orders | Work Orders Analysis | Work Order | chart, pivot, list, form | filter: workcenter, ready, blocked, progress | Manage Work Order Operations |
| Reporting > Overall Equipment Effectiveness | Overall Equipment Effectiveness | Work Center Productivity Record | chart, pivot, list, form | filter: workcenter group, loss group | Manage Work Order Operations |
| Configuration > Settings | Settings | Configuration Settings | form |  | Role / Administrator |
| Configuration > Work Centers | Work Centers | Work Center | list, card, form |  | Manage Work Order Operations |
| Configuration > Operations | Operations | Operation | list, card, form |  | Manage Work Order Operations |

### Maintenance

Menu order 160; contributed by the Maintenance capability package. The application contains 10 menu entries that open an action.

| Menu path | Action | Entity | Views (first is the default) | Default filters, groupings and values | Restricted to |
|---|---|---|---|---|---|
| Dashboard | Maintenance Teams | Maintenance Team | card, form |  | Equipment Manager, Role / User |
| Maintenance > Maintenance Requests | Maintenance Requests | Maintenance Request | card, list, form, pivot, chart, calendar, activity | filter: active; new records: user = user_identifier | Equipment Manager, Role / User |
| Maintenance > Maintenance Calendar | Maintenance Requests | Maintenance Request | calendar, card, list, form, pivot, chart, activity | filter: active, to do | Equipment Manager, Role / User |
| Equipment | Equipment | Equipment | card, list, form |  | Equipment Manager, Role / User |
| Reporting > Maintenance Requests Analysis | Maintenance Requests Analysis | Maintenance Request | chart, pivot, card, list, form, calendar, activity | filter: active |  |
| Configuration > Settings | Settings | Configuration Settings | form |  | Role / Administrator |
| Configuration > Maintenance Teams | Teams | Maintenance Team | list, card, form |  | Equipment Manager |
| Configuration > Equipment Categories | Equipment Categories | Equipment Category | list, card, form |  |  |
| Configuration > Maintenance Stages | Stages | Maintenance Stage | list, card, form |  | Technical Features |
| Configuration > Activity Types | Activity Types | Activity Type | list, card, form | new records apply to: Maintenance Request | Technical Features |

### Repairs

Menu order 165; contributed by the Repairs capability package; visible to the access groups: User. The application contains 5 menu entries that open an action.

| Menu path | Action | Entity | Views (first is the default) | Default filters, groupings and values | Restricted to |
|---|---|---|---|---|---|
| Orders | Repair Orders | Repair Order | list, card, chart, pivot, form, activity |  | User |
| Reporting > Repairs | Repair Orders Analysis | Repair Order | list, card, chart, pivot, form | filter: product, createDate |  |
| Configuration > Products | Products | Product Template | card, list, form | new records: is storable = True |  |
| Configuration > Product Variants | Product Variants | Product Variant | list, form, card |  | Manage Product Variants |
| Configuration > Repair Orders Tags | Tags | Repair Tag |  |  | Technical Features |

### Employees

Menu order 185; contributed by the Employees capability package; visible to the access groups: Administrator, Officer: Manage all employees, Role / User. The application contains 23 menu entries that open an action.

| Menu path | Action | Entity | Views (first is the default) | Default filters, groupings and values | Restricted to |
|---|---|---|---|---|---|
| Employees | Employees | Employee | card, list, form, activity, chart, pivot |  | Officer: Manage all employees |
| Directory | Employees | Public Employee Profile | card, list, form |  |  |
| Departments | Departments | Department | card, list, form |  | Role / User |
| Learning > Certifications | Certifications | Employee Skill | list, form | group by: type |  |
| Learning > Training Attendances | Training Attendances | Resume Line | list, card, form, calendar |  |  |
| Learning > Courses > Electronic Learning | Electronic Learning Courses | Course | list, card, form |  |  |
| Learning > Courses > Onsite | Onsite Courses | Event | card, calendar, list, form, pivot, chart, activity |  |  |
| Reporting > Skills > Skills Inventory | Skills Inventory | Employee Skills Report | list, pivot | filter: skill type, skill |  |
| Reporting > Skills > Certifications | Certification | Employee Certification Report | list, pivot | filter: employee |  |
| Configuration > Settings | Settings | Configuration Settings | form |  | Role / Administrator |
| Configuration > Employee > Onboarding / Offboarding | Employee Plans | Activity Plan | list, card, form | new records apply to: Employee |  |
| Configuration > Employee > Work Locations | Work Locations | Work Location | list, form |  |  |
| Configuration > Employee > Working Schedules | Working Schedules | Working Schedule | list, form |  |  |
| Configuration > Employee > Departure Reasons | Departure Reasons | Departure Reason | list |  |  |
| Configuration > Employee > Skill Types | Skill Types | Skill Type | list, form |  | Officer: Manage all employees |
| Configuration > Employee > Tags | Employee Tags | Employee Tag | list, form |  | Technical Features |
| Configuration > Resume > Sections | Resume Sections | Resume Line Type | list, form |  | Technical Features |
| Configuration > Recruitment > Job Positions | Job Positions | Job Position | list, form |  |  |
| Configuration > Recruitment > Contract Templates | Contract Templates | Employee Version | list, form |  | Administrator |
| Configuration > Recruitment > Employment Types | Employment Types | Contract Type | list |  | Officer: Manage all employees |
| Configuration > Challenges > Badges | Badges | Badge | card, list, form |  |  |
| Configuration > Challenges > Challenges | Challenges | Challenge | card, list, form | filter: inprogress; new records: inprogress = True | Officer: Manage all employees |
| Configuration > Challenges > Goals History | Goals History | Goal | list, card | group by: user, definition | Officer: Manage all employees |

### Attendances

Menu order 205; contributed by the Attendances capability package; visible to the access groups: Officer: Manage attendances. The application contains 9 menu entries that open an action.

| Menu path | Action | Entity | Views (first is the default) | Default filters, groupings and values | Restricted to |
|---|---|---|---|---|---|
| Overview > Dashboard | Attendances | Attendance | list, form |  |  |
| Overview > Employees | Employees | Employee | card, list, form, activity, chart, pivot |  | Officer: Manage attendances |
| Management | Management | Attendance | list, form |  | Officer: Manage attendances |
| Kiosk Mode | Check-in station address |  |  |  | Officer: Manage all attendances |
| Reporting > Attendances | Attendances | Attendance | pivot, chart |  |  |
| Reporting > Time Off Ledger | Time Off Ledger | Attendance and Leave Analysis Report | list, pivot, form | filter: less than 0, last two months; group by: date, employees |  |
| Configuration > Settings | Settings | Configuration Settings | form |  | Administrator |
| Configuration > Onboarding | Check-in station trial |  |  |  | Officer: Manage all employees |
| Configuration > Overtime Rulesets | Rulesets | Attendance Overtime Ruleset | list, form |  | Administrator |

### Recruitment

Menu order 210; contributed by the Recruitment capability package; visible to the access groups: Officer: Manage all applicants, Interviewer. The application contains 19 menu entries that open an action.

| Menu path | Action | Entity | Views (first is the default) | Default filters, groupings and values | Restricted to |
|---|---|---|---|---|---|
| Applications > By Job Positions | Job Positions | Job Position | card, list, form |  | Officer: Manage all applicants |
| Applications > By Job Positions | Job Positions | Job Position | card, form |  | Interviewer |
| Applications > By Talent Pools | Talent Pool | Talent Pool | card, list, form |  | Officer: Manage all applicants |
| Applications > All Applications | Applications | Applicant | card, list, form, pivot, chart, calendar, activity | filter: applicants |  |
| Reporting > Recruitment Analysis | Recruitment Analysis | Applicant | chart, pivot | filter: creation month, job |  |
| Configuration > Settings | Settings | Configuration Settings | form |  | Role / Administrator |
| Configuration > Job Positions > Stages | Stages | Recruitment Stage | list, card, form |  | Technical Features |
| Configuration > Job Positions > Employment Types | Employment Types | Contract Type | list |  | Officer: Manage all employees |
| Configuration > UTMs > Mediums | Mediums | Campaign Medium | list, form |  | Technical Features |
| Configuration > UTMs > Sources | Sources | Campaign Source | list, form |  | Technical Features |
| Configuration > Applications > Degrees | Degrees | Degree |  |  |  |
| Configuration > Applications > Refuse Reasons | Refuse Reasons | Refuse Reason | list, form |  |  |
| Configuration > Applications > Tags | Tags | Applicant Tag |  |  |  |
| Configuration > Employees > Departments | Departments | Department | list, form |  |  |
| Configuration > Employees > Skill Types | Skill Types | Skill Type | list, form |  |  |
| Configuration > Activities > Activity Types | Activity Types | Activity Type | list, card, form | new records apply to: Applicant |  |
| Configuration > Activities > Activity Plans | Recruitment Plans | Activity Plan | list, card, form | new records apply to: Applicant | Administrator |
| Configuration > Interviews | Interviews | Survey | card, list, activity, form | new records: survey type = recruitment | Administrator |
| Configuration > Job Boards > Emails | Emails | Job Platform | list, form |  |  |

### Fleet

Menu order 220; contributed by the Fleet capability package; visible to the access groups: Officer: Manage all vehicles. The application contains 14 menu entries that open an action.

| Menu path | Action | Entity | Views (first is the default) | Default filters, groupings and values | Restricted to |
|---|---|---|---|---|---|
| Fleet > Fleet | Vehicles | Vehicle | card, list, form, pivot, activity |  | Officer: Manage all vehicles |
| Fleet > Contracts | Contracts | Vehicle Contract | list, card, form, chart, pivot, activity | filter: open | Officer: Manage all vehicles |
| Fleet > Services | Services | Vehicle Service Log | list, card, form, chart, pivot, activity | filter: groupby service type | Officer: Manage all vehicles |
| Fleet > Odometers | Odometers | Vehicle Odometer Reading | list, form, chart |  | Officer: Manage all vehicles |
| Reporting > Costs | Costs Analysis | Vehicle Cost Report | chart, pivot | filter: date start | Administrator |
| Reporting > Odometers | Odometer Analysis | Fleet Odometer Analysis Report | chart | filter: groupby date, groupby category | Administrator |
| Configuration > Settings | Settings | Configuration Settings | form |  | Role / Administrator |
| Configuration > Models > Manufacturers | Manufacturers | Vehicle Brand | card, list, form | filter: with models |  |
| Configuration > Models > Models | Models | Vehicle Model | list, form |  |  |
| Configuration > Models > Categories | Categories | Vehicle Category | list |  |  |
| Configuration > Services > Types | Types | Vehicle Service Type | list, form |  | Technical Features |
| Configuration > Vehicle > Status | Status | Vehicle State | list, form |  | Technical Features |
| Configuration > Vehicle > Tags | Tags | Vehicle Tag |  |  | Technical Features |
| Configuration > Activity Types | Activity Types | Activity Type | list, card, form | new records apply to: Vehicle Contract | Technical Features |

### Time Off

Menu order 225; contributed by the Time Off capability package; visible to the access groups: Role / User. The application contains 16 menu entries that open an action.

| Menu path | Action | Entity | Views (first is the default) | Default filters, groupings and values | Restricted to |
|---|---|---|---|---|---|
| My Time > Dashboard | Dashboard | Time Off Request | calendar, list, form, activity | filter: year |  |
| My Time > My Time Off | My Time Off | Time Off Request | list, form, card, activity |  |  |
| My Time > My Allocations | My Allocations | Time Off Allocation | list, card, form, activity | filter: year |  |
| Overview | All Time Off | Time Off Calendar Report | calendar | filter: my team, current year, validate, approve |  |
| Management > Time Off | All Time Off | Time Off Request | card, list, form, calendar, activity | filter: waiting for me, waiting for me manager, current year |  |
| Management > Allocations | Allocations | Time Off Allocation | card, list, form, activity | filter: my team, approve |  |
| Reporting > by Employee | Time Off by Employee | Time Off Request | list, chart, pivot, calendar, form | filter: date from, group employee, group type, to approve, validated |  |
| Reporting > by Type | Time Off by Type | Time Off Summary Report | chart, list, pivot | filter: group type, approve, validated |  |
| Reporting > Balance | Time off by employee and type |  |  |  | Administrator |
| Configuration > Time Off Types | Time Off Types | Time Off Type | list, card, form |  | Administrator |
| Configuration > Accrual Plans | Accrual Plans | Accrual Plan | list, form |  | Administrator |
| Configuration > Public Holidays | Public Holidays | Resource Time Off | list, form | filter: date | Administrator |
| Configuration > Mandatory Days | Mandatory Days | Mandatory Working Day | list, form | filter: date | Administrator |
| Configuration > Optional Holidays | Optional Holidays | Optional Holidays | list | filter: date | Administrator |
| Configuration > Activity Types | Activity Types | Activity Type | list, card, form | new records apply to: Time Off Request | Technical Features |
| Configuration > Settings | Settings | Configuration Settings | form |  | Role / Administrator |

### Expenses

Menu order 230; contributed by the Expenses capability package. The application contains 6 menu entries that open an action.

| Menu path | Action | Entity | Views (first is the default) | Default filters, groupings and values | Restricted to |
|---|---|---|---|---|---|
| My Expenses > Expenses to Process | Expenses to Process | Expense | list, card, form, chart, pivot, activity |  | Role / User |
| My Expenses > My Expenses | My Expenses | Expense | list, card, form, chart, pivot, activity | filter: my open expenses |  |
| Reporting > Expenses Analysis | Expenses Analysis | Expense | chart, pivot, list, form |  |  |
| Configuration > Settings | Settings | Configuration Settings | form |  | Role / Administrator |
| Configuration > Activity Types | Activity Types | Activity Type | list, card, form | new records apply to: Expense | Technical Features |
| Configuration > Expense Categories | Expense categories |  |  | new records: can be expensed = 1, type = service, expense policy = cost | Administrator |

### Lunch

Menu order 235; contributed by the Lunch capability package; visible to the access groups: User : Order your meal. The application contains 13 menu entries that open an action.

| Menu path | Action | Entity | Views (first is the default) | Default filters, groupings and values | Restricted to |
|---|---|---|---|---|---|
| My Lunch > New Order | Order Your Lunch | Lunch Product | card, list |  |  |
| My Lunch > My Order History | My Orders | Lunch Order | list, card, pivot |  |  |
| My Lunch > My Account History | My Account | Lunch Cash Move Report | list |  |  |
| Manager > Cash Moves | Cash Moves | Lunch Cash Move | list, card, form |  |  |
| Manager > Control Accounts | Control Accounts | Lunch Cash Move Report | list, card, form |  |  |
| Manager > Control Vendors | Control Vendors | Lunch Order | list, card, pivot |  |  |
| Manager > Today's Orders | Today's Orders | Lunch Order | list, card |  |  |
| Configuration > Settings | Settings | Configuration Settings | form |  |  |
| Configuration > Vendors | Vendors | Lunch Vendor | card, list, form |  |  |
| Configuration > Locations | Lunch Locations | Lunch Location | list, form, card |  |  |
| Configuration > Products | Products | Lunch Product | list, card, form |  |  |
| Configuration > Product Categories | Product Categories | Lunch Product Category | list, form, card |  |  |
| Configuration > Alerts | Lunch Alerts | Lunch Alert | list, form, card |  |  |

### Live Chat

Menu order 240; contributed by the Live Chat capability package; visible to the access groups: User. The application contains 16 menu entries that open an action.

| Menu path | Action | Entity | Views (first is the default) | Default filters, groupings and values | Restricted to |
|---|---|---|---|---|---|
| Channels | Live Chat Channels | Live Chat Channel | card, form |  | User |
| Sessions > All Conversations | Sessions | Discussion Channel | card, list, pivot, chart, form |  |  |
| Sessions > Looking for Help | Looking for Help | Discussion Channel | list, card, form |  |  |
| Visitors | Visitors | Website Visitor | card, list, form, chart | filter: last 7 days | User |
| Reporting > Agents | Agents | Keep the channel member history | pivot, chart |  |  |
| Reporting > Sessions | Sessions | Live Chat Channel Report | chart, pivot |  |  |
| Configuration > Canned Responses | Canned Responses | Canned Response | list, form, card |  | User |
| Configuration > Chatbots | Chatbot | Chatbot Script | list, form |  | Administrator |
| Configuration > Expertise | Expertise | Live Chat Expertise | list, form |  |  |
| Configuration > Tags | Tags | Live Chat Conversation Tags | list, form |  | User |
| Technical > Escalated Sessions | Sessions | Discussion Channel |  | filter: ongoing, escalated | Administrator |
| Technical > Ongoing Call Sessions | Sessions | Discussion Channel |  | filter: ongoing, in a call | Administrator |
| Technical > Ongoing Sessions | Sessions | Discussion Channel |  | filter: ongoing | Administrator |
| Technical > Sessions Handled by Agent | Sessions | Discussion Channel |  | filter: ongoing, handled by agent | Administrator |
| Technical > Sessions Handled by Bot | Sessions | Discussion Channel |  | filter: ongoing, handled by bot | Administrator |
| Technical > Member History | Member History | Keep the channel member history | list, form |  |  |

### Data Cleaning

Menu order 250; contributed by the Data Recycle capability package. The application contains 2 menu entries that open an action.

| Menu path | Action | Entity | Views (first is the default) | Default filters, groupings and values | Restricted to |
|---|---|---|---|---|---|
| Recycle Records | Field Recycle Records | Data Recycling Record | list, form |  |  |
| Configuration > Rules > Recycle Records | Recyle Records Rules | Data Recycling Rule | list, form |  |  |

### Link Tracker

Menu order 270; contributed by the Campaign Tracking Trackers capability package; visible to the access groups: Technical Features. The application contains 4 menu entries that open an action.

| Menu path | Action | Entity | Views (first is the default) | Default filters, groupings and values | Restricted to |
|---|---|---|---|---|---|
| Link Tracker | Link Tracker | Link Tracker | list, form, chart |  | Technical Features |
| UTMs > Campaigns | Campaigns | Campaign | list, card, form |  | Technical Features |
| UTMs > Mediums | Mediums | Campaign Medium | list, form |  | Technical Features |
| UTMs > Sources | Sources | Campaign Source | list, form |  | Technical Features |

### Marketing Card

Menu order 270; contributed by the Marketing Card capability package; visible to the access groups: Marketing Card User. The application contains 1 menu entry that opens an action.

| Menu path | Action | Entity | Views (first is the default) | Default filters, groupings and values | Restricted to |
|---|---|---|---|---|---|
| Campaigns | Card Campaign | Marketing Card Campaign | list, card, form |  | Marketing Card User |

### Apps

Menu order 500; contributed by the Base capability package; visible to the access groups: Role / Administrator. The application contains 6 menu entries that open an action.

| Menu path | Action | Entity | Views (first is the default) | Default filters, groupings and values | Restricted to |
|---|---|---|---|---|---|
| Apps > Main Apps | Capability packages |  |  |  |  |
| Apps > Theme Store | Appearance themes |  |  |  |  |
| Apps > Third-Party Apps | Third-party checks |  |  |  |  |
| Update Apps List | Module Update (dialog) | Module List Update Wizard | form |  | Technical Features |
| Apply Scheduled Upgrades | Apply Schedule Upgrade (dialog) | Module Upgrade Wizard | form |  | Technical Features |
| Import Module | Import Module (dialog) | Import Module Wizard | form |  | Technical Features |

### Settings

Menu order 550; contributed by the Base capability package; visible to the access groups: Access Rights. The application contains 97 menu entries that open an action.

| Menu path | Action | Entity | Views (first is the default) | Default filters, groupings and values | Restricted to |
|---|---|---|---|---|---|
| General Settings | Settings | Configuration Settings | form |  | Role / Administrator |
| Users & Companies > Users | Users | User | list, card, form | filter: no share |  |
| Users & Companies > Groups | Groups | Access Group |  | filter: no share | Technical Features |
| Users & Companies > Privileges | Privileges | Access Privilege |  |  | Technical Features |
| Users & Companies > Companies | Companies | Company | list, card, form |  |  |
| Users & Companies > Open Authorization Providers | Providers | Open Authorization Provider | list, form |  | Technical Features |
| Translations > Languages | Languages | Language |  |  |  |
| Translations > Import / Export > Export Translation | Export Translation (dialog) | Translation Export Wizard | form |  |  |
| Translations > Import / Export > Import Translation | Import Translation (dialog) | Translation Import Wizard | form |  |  |
| Translations > Application Terms > Transifex Code Translations | Interface translations |  |  |  |  |
| Gamification Tools > Challenges | Challenges | Challenge | card, list | filter: inprogress; new records: inprogress = True |  |
| Gamification Tools > Goals | Goals | Goal | list, form, card | group by: user, definition |  |
| Gamification Tools > Goal Definitions | Goal Definitions | Goal Definition | list, form |  |  |
| Gamification Tools > Badges | Badges | Badge | card, list, form |  |  |
| Gamification Tools > Ranks | Ranks | Karma Rank | list, form |  |  |
| Gamification Tools > Karma Tracking | Karma Tracking | Karma Tracking | list, form |  |  |
| Technical > Discuss > Messages | Messages | Message | list, form |  |  |
| Technical > Discuss > Scheduled Messages | Scheduled Messages | Scheduled Messages | list, form |  |  |
| Technical > Discuss > Subtypes | Subtypes | Message Subtype | list, form |  |  |
| Technical > Discuss > Tracking Values | Tracking Values | Field Change Tracking Value | list, form |  |  |
| Technical > Discuss > Notifications | Notifications | Notification | list, form |  | Technical Features |
| Technical > Discuss > Followers | Followers | Follower | list, form |  | Technical Features |
| Technical > Discuss > Email Blacklist | Blacklisted Email Addresses | Email Blacklist |  |  |  |
| Technical > Discuss > Ratings | Ratings | Rating | card, list, chart, pivot, form |  |  |
| Technical > Discuss > Mail Groups | Mail Groups | Mail Group | card, list, form |  |  |
| Technical > Discuss > User Settings | User Settings | User Preferences | list, form |  |  |
| Technical > Discuss > Guests | Guests | Guest | list, form |  |  |
| Technical > Discuss > Moderation Rules | Moderation | Mailing List black/white list | list, form |  |  |
| Technical > Discuss > RTC sessions | RTC sessions | Real Time Communication Session | list, form | group by: channel |  |
| Technical > Discuss > ICE Servers | ICE Servers | Interactive Connectivity Server | list, form, card |  |  |
| Technical > Discuss > Message Reactions | Message Reactions | Message Reaction | list, form |  |  |
| Technical > Discuss > Link Previews | Link Previews | Link Preview | list, form |  |  |
| Technical > Discuss > Favourite animated images | Favourite animated images | Favourite Animated Image | list, form |  |  |
| Technical > Email > Emails | Emails | Outgoing Email | list, form |  |  |
| Technical > Email > Outgoing Mail Servers | Outgoing Mail Servers | Outgoing Mail Server | list, form |  | Technical Features |
| Technical > Email > Incoming Mail Servers | Incoming Mail Servers | Incoming Mail Server | list, form |  | Technical Features |
| Technical > Email > Email Templates | Email Templates | Email Template | form, list | filter: base templates |  |
| Technical > Email > Aliases | Aliases | Email Alias |  | filter: active | Technical Features |
| Technical > Email > Alias Domains | Alias Domains | Email Alias Domain | list, form |  | Technical Features |
| Technical > Email > Channels | Join a group | Discussion Channel | card, list, form |  | Technical Features |
| Technical > Email > Channels/Members | Channels/Members | Discussion Channel Member | list, form |  | Technical Features |
| Technical > Email > Mail Gateway Allowed | Mail Gateway Allowed | Mail Gateway Allowed Sender | list |  | Technical Features |
| Technical > Email > Snailmail Letters | Snailmail Letters | Postal Letter | form, list |  |  |
| Technical > Email > Digest Emails | Digest Emails | Digest Email |  | filter: activated | Access Rights |
| Technical > Email > Digest Tips | Digest Tips | Digest Tip |  |  | Access Rights |
| Technical > Activities > Activity Overview | Activity Overview | Activity | list, form |  |  |
| Technical > Activities > Activity Types | Activity Types | Activity Type | list, card, form |  |  |
| Technical > Activities > Activity Plans | Activity Plans | Activity Plan | list, card, form |  |  |
| Technical > Phone / Text Message > Text Message | Text Message | Outgoing Text Message | list, form |  |  |
| Technical > Phone / Text Message > Text Message Templates | Templates | Text Message Template | list, form |  |  |
| Technical > Phone / Text Message > Phone Blacklist | Blacklisted Phone Numbers | Phone Blacklist |  |  |  |
| Technical > Marketing Card > Card Template | Card Template | Marketing Card Template | list, form |  | Marketing Card Manager, Technical Features |
| Technical > Mass Mailing > Mailing Traces | Mailing Traces | Mailing Trace | list, form, chart, pivot |  |  |
| Technical > Actions > Actions | Actions | Action |  |  |  |
| Technical > Actions > Client Actions | Client Actions | Client Action |  |  |  |
| Technical > Actions > Configuration Wizards | Configuration Wizards | Configuration Step |  |  |  |
| Technical > Actions > Embedded Actions | Embedded Actions | Embedded Action |  |  |  |
| Technical > Actions > Reports | Reports | Report Action |  |  |  |
| Technical > Actions > Server Actions | Server Actions | Server Action | list, form | filter: toplevel actions |  |
| Technical > Actions > User-defined Defaults | User-defined Defaults | User Default Value | list, form |  |  |
| Technical > Actions > Window Actions | Window Actions | Window Action |  |  |  |
| Technical > In Application Purchase > In Application Purchase Accounts | In Application Purchase Account | In-Application Purchase Account | list, form |  |  |
| Technical > In Application Purchase > In Application Purchase Partners | In Application Purchase Partner | Contact Enrichment | list, form |  |  |
| Technical > Automation > Automation Rules | Automation Rules | Automation Rule | card, list, form | filter: inactive |  |
| Technical > Automation > base.ir_cron_act | Scheduled actions |  |  |  |  |
| Technical > Automation > Scheduled Actions Triggers | Scheduled Actions Triggers | Scheduled Action Trigger | list, form |  |  |
| Technical > Database Structure > Assets | Assets | Web Asset |  | filter: active |  |
| Technical > Database Structure > Decimal Accuracy | Decimal Accuracy | Decimal Precision |  |  |  |
| Technical > Database Structure > Fields | Fields | Field Definition |  |  |  |
| Technical > Database Structure > Fields Selection | Fields Selection | Selection Value Definition |  |  |  |
| Technical > Database Structure > Logging | Logging | Log Entry | list, form |  |  |
| Technical > Database Structure > ManyToMany Relations | ManyToMany Relations | Relation Table Registry |  |  | Technical Features |
| Technical > Database Structure > Model Constraints | Model Constraints | Model Constraint Registry |  |  | Technical Features |
| Technical > Database Structure > Models | Models | Model Definition |  |  |  |
| Technical > Database Structure > Profiling | Ir profile | Performance Profile | list, form | filter: group session |  |
| Technical > Database Structure > base.action_attachment | Attachments |  | card, list, form |  |  |
| Technical > User Interface > Menu Items | Menu Items | Menu Item |  |  |  |
| Technical > User Interface > Onboardings | Onboardings | Onboarding Panel | list, form |  |  |
| Technical > User Interface > Onboardings Steps | Onboarding Steps | Onboarding Step | list, form |  |  |
| Technical > User Interface > Views | Views | View Definition |  | filter: active |  |
| Technical > User Interface > Customized Views | Customized Views | User Customized View |  |  |  |
| Technical > User Interface > Tours | Tours | Guided Tour |  |  |  |
| Technical > User Interface > User-defined Filters | User-defined Filters | Saved Filter |  |  |  |
| Technical > Reporting > Paper Format | Paper Format General Configuration | Paper Format | list, form |  | Technical Features |
| Technical > Reporting > Reports | Reports | Report Action | list, form |  | Technical Features |
| Technical > Sequences & Identifiers > External Identifiers | External Identifiers | External Identifier |  |  | Technical Features |
| Technical > Sequences & Identifiers > Sequences | Sequences | Sequence |  |  |  |
| Technical > Parameters > System Parameters | System Parameters | System Parameter |  |  |  |
| Technical > Security > Record Rules | Record Rules | Record Rule |  |  |  |
| Technical > Security > Access Rights | Access Rights | Model Access Rule |  |  |  |
| Technical > Security > User Devices | User Devices | User Device | list, card, form |  |  |
| Technical > Privacy > Privacy Logs | Privacy Logs | Privacy Log | list, form |  |  |
| Technical > Calendar > Calendar Alarm | Calendar Alarm | Calendar Reminder | list, form |  | Technical Features |
| Technical > Calendar > Meeting Types | Meeting Types | Calendar Event Tag |  |  | Technical Features |
| Technical > Resource > Working Schedules | Working Schedules | Working Schedule | list, form |  |  |
| Technical > Resource > Resource Time Off | Resource Time Off | Resource Time Off | list, form, calendar |  |  |
| Technical > Resource > Resources | Resources | Resource | list, form |  |  |

### To-do

Contributed by the To-Do capability package. The application contains 1 menu entry that opens an action.

| Menu path | Action | Entity | Views (first is the default) | Default filters, groupings and values | Restricted to |
|---|---|---|---|---|---|
| To-do | To-dos | Task | card, form, list, calendar, activity | filter: open tasks; new records: project = False |  |
## Part 3: common workflows on views

Every multi-record view shares the search bar, the pager, the view switcher and the action menu. Every view kind is
described below with what it shows, what a user may do on it, and the rules that govern it.

### Searching, filtering and grouping

The search bar of a view accepts three kinds of input, which combine into **facets** displayed under the bar.

| Input | Behavior |
|---|---|
| Typed text | Offers one proposal per searchable field of the view ("Search Customer for: Acme"). Choosing one adds a facet that matches that field against the text. Several proposals may be accepted at once, and they combine with a logical or inside one facet. A proposal on a relational field may be expanded to pick the exact record instead of matching by name. |
| Filter | A named condition defined on the view. Filters of the same group combine with a logical or inside one facet; facets combine with a logical and. Date filters expand into period entries (the current month, the previous month, each quarter, the year) and a comparison entry that repeats the view for the same period of the previous year or period. |
| Grouping | A named grouping defined on the view, or any groupable field chosen from the list. Groupings stack: the first is the outer level, the next is nested inside it, up to the depth the view allows. A date grouping offers year, quarter, month, week and day. |

A **search panel** may accompany the view: a left-hand column of categories (one per chosen relational or selection
field) whose values filter the view when selected. Category values show the record count when the view asks for it.

**Favourites.** The current combination of facets, groupings and ordering may be saved as a favourite with a name. A
favourite may be marked as the default of that action for that user, in which case it is applied every time the action
is opened, and it may be shared with all users, in which case every user sees it in the favourite list. Favourites are
per action and per user, and removing one does not affect the underlying action.

**The pager.** Shows the window of records displayed out of the total ("1-80 / 342"). It moves forward and backward and
accepts a typed range. The page size comes from the action limit, defaulting to eighty records.

**Ordering.** A list is ordered by the ordering of the entity unless the user presses a column header, which sorts by
that column ascending, then descending, then returns to the default. The chosen ordering is part of what a favourite
stores.

### The list view

| Element | Behavior |
|---|---|
| Columns | The fields the view declares. Columns marked optional are hidden or shown from the column menu at the right end of the header row, and the choice is remembered per user and per view. |
| Aggregates | A numeric column that declares an aggregate shows the sum (or average) of the visible rows under the column, and per group when the list is grouped. |
| Grouping | A grouped list shows one collapsible row per group with the group label, the record count and the aggregates. Opening a group loads its records. Groups are paged themselves when they hold more records than the page size. |
| Editing in place | When the view allows it, pressing a cell opens the row for editing; the row is saved when focus leaves it or when another row is opened, and discarded with the discard action. A new row is added at the top or the bottom, as the view declares. |
| Multiple record edit | When several rows are selected and the view allows it, editing one cell offers to apply the new value to every selected record, with a confirmation that states how many records will change. |
| Selection | Checkboxes select rows; the header checkbox selects the page. When the whole page is selected and more records match, a banner offers to select all matching records, after which actions apply to the whole selection. |
| Manual ordering | When the view declares a sequence field, rows carry a drag handle and dropping a row rewrites the sequence of the affected rows. |
| Row actions | A trailing column may carry per-row buttons, each with its own visibility condition. |
| Opening a record | Pressing a row opens the form of that record, pushing it on the stack, and the form then offers previous and next navigation over the list. |

### The form view

| Element | Behavior |
|---|---|
| Status bar | At the top right, the states of the record in their declared order, with the current one highlighted. States the view hides are shown only when the record is in them. When the field allows it, pressing a state moves the record to it directly. |
| Header buttons | At the top left, the operations offered on the record. Each has a label, a visibility condition, an optional confirmation text, and an optional access group. A highlighted button is the suggested next step. |
| Title zone | The main identifying fields, rendered larger. |
| Statistic buttons | Boxed counters at the top right of the sheet, each showing a count or an amount and opening the related records when pressed. Each has its own visibility condition. |
| Sheet | The fields, grouped into columns, notebook pages and labelled groups. A field may be required, read-only or invisible depending on the values of other fields; those conditions are evaluated continuously as the user types. |
| Line editors | One-to-many fields are edited as embedded lists with their own add, remove and reorder actions, and optionally as embedded cards or an embedded form. |
| Unsaved changes | The record is dirty as soon as a field changes; leaving the record, switching view or pressing a button asks to save or discard first, except for buttons the view marks as saving by themselves. |
| Recomputation | Changing a field triggers the on-change rules of the entity, which may fill or clear other fields and may raise a warning dialog; the warning is shown and the change is kept or reverted as the rule states. |
| Discussion thread | Under the sheet: the followers with their subscription settings, the message list with internal notes and outgoing messages, the activity list, the attachment list, and the composition actions (send a message, log a note, schedule an activity, attach a file). |
| Record navigation | Previous and next over the list the form was opened from, with the position shown in the pager. |

### The card view

Records are shown as cards, arranged in columns when the view is grouped. Cards may be dragged between columns, which
writes the grouping field, and dragged inside a column, which rewrites the sequence. A column may show a progress bar
summarising its cards by a chosen field, a sum of a numeric field in its header, and a fold state. Quick creation adds a
card with a minimal form at the top of a column; the full form is one press away. A column may be added, renamed,
folded, archived or deleted when the grouping field allows it.

### The calendar view

Records are placed by their date or by their start and end. Day, week, month and year scales are offered. Dragging a
record moves it in time and writes the date fields; resizing changes the duration. Creating in an empty slot opens a
quick form pre-filled with that slot. A side panel lists the record owners and lets the user show or hide each of them.
When the entity supports attendance answers, the colour of a record reflects the answer of the current user.

### The pivot view

Rows and columns are groupings, cells are measures. Any grouping may be expanded into a sub-grouping by pressing its
header. Measures are chosen from the numeric fields of the entity plus the record count. Totals are shown per row, per
column and overall. The table may be flipped, expanded entirely, downloaded as a spreadsheet workbook, or inserted into
a spreadsheet. Comparison with a previous period adds a second value and the variation in each cell.

### The chart view

Bar, line and pie renderings of one measure over one or two groupings, with stacked, ascending and descending options.
The same measure and grouping choices as the pivot apply.

### The activity view

A grid of records by activity type: rows are records, columns are activity types, cells hold the activities with their
due state (overdue, today, planned). A cell offers to schedule an activity, and the header offers to schedule for the
whole column.

### The schedule view

A time grid of records with rows grouped by a chosen field, used for planning. Records are dragged to move and resized
to change duration; overlaps are highlighted when the entity declares a capacity.

### The hierarchy view

A tree of records following a parent field, used for organization charts and category trees. Nodes expand and collapse,
and a node may be dragged onto another to change its parent.

### The map view

Records placed on a map by the coordinates of their address, with a list beside the map and an itinerary link.

### Actions offered on records

| Action | Where | Behavior and guards |
|---|---|---|
| Print | Print menu of list and form | Lists the report definitions bound to the entity that the user may run; renders for the selected records. |
| Send by electronic mail | Action menu, and dedicated buttons on documents | Opens the composer with the template of the document, the recipients derived from the document, the attachments the template declares, and the language of the recipient. |
| Export | Action menu of a list | Opens the export dialogue described in [`report-and-export-documents.md`](report-and-export-documents.md). |
| Import | List of an entity the user may create | Opens the import flow described in the same document. |
| Archive and unarchive | Action menu | Available on entities that carry the active flag. Archiving hides the records from every default list; the archived filter brings them back. Archiving asks for confirmation when the records are referenced elsewhere. |
| Duplicate | Action menu of a form | Creates an unsaved copy: fields declared as not copyable are reset, one-to-many lines are copied unless declared otherwise, the name is suffixed with "(copy)" where the entity declares it, and the copy opens in the form for review. |
| Delete | Action menu | Refused when a record is referenced by a restricting relation, with the message that names the referencing records; refused on posted or otherwise locked documents by the rules of the domain. |
| Add to dashboard | Search menu of a multi-record view | Adds the current view, with its facets, groupings and ordering, as a block of the personal dashboard under a name the user types. |
| Insert in spreadsheet | Pivot, chart and list views | Inserts the current table, chart or list into a spreadsheet document, linked to the live data. |
| Share by link | Documents that support portal sharing | Generates the portal address of the document with its access token and offers to send it by electronic mail. |
| Open the record log | Developer tools | Shows the technical identity, the creation and last change stamps and the external identifier of the record. |

### Activities

An activity is a planned piece of work attached to a record: a type (call, meeting, to do, electronic mail, upload
document, and the types the packages add), a summary, a note, a due date and an assignee. The activity list of a record
offers to schedule, edit, mark as done (optionally with a feedback note that is posted in the thread), and to schedule a
follow-up. Overdue activities colour the record in lists and cards, and the systray counter groups the activities of the
user by application with the overdue count first.

### Attachments and documents

Every record with a discussion thread accepts attachments, uploaded by drag and drop or from the composer. Attachments
are listed with their name, size and preview; images and printable documents preview in place. An attachment may be
deleted by its author or by a user with write access on the record. The main attachment of a document (the printed form
of an invoice, the received vendor bill) is the one previewed beside the form.

### The command palette

The command palette is a single search box opened from the keyboard with `control` and `k`, from anywhere in the client
and even while a field is being edited. It replaces hunting through menus.

The first character typed may select a **namespace**, which decides what is searched:

| First character | Namespace | Prompt shown | Message when nothing matches | What is searched |
|---|---|---|---|---|
| none | Commands | "Search for a command..." | the default empty message | The commands the current screen offers, including every control of the screen that declares a shortcut letter. |
| `/` | Menus | "Search for a menu..." | "No menu found" | The applications and every menu entry the user may see. |
| `@` | Conversations | "Search a conversation" | "No conversation found" | The user's conversations, channels and the people they may write to; a search that matches nobody offers to open a new conversation. |

Rules of the palette:

1. Results are grouped by category and the categories appear in a fixed order. In the menu namespace, applications come
   first and menu entries second.
2. Matching is tolerant: the typed characters must appear in the label in order, not necessarily consecutively. A menu
   entry is matched on its whole path with the segments reversed, therefore typing the name of the leaf finds it even
   when the path is long.
3. Duplicate results with the same label in the same category are shown once.
4. Every control of the current screen that declares a shortcut letter becomes a command automatically: its label is the
   control's own text, or its placeholder text when it has no text, its category is the one its container declares, and
   the palette displays its shortcut beside it. A container may declare that its controls are excluded.
5. Choosing a result performs exactly the same operation as operating the control it stands for, and the palette closes.
6. The conversation namespace waits 200 milliseconds after the last keystroke before searching, because that search
   reaches the server; the other namespaces search what the client already holds.

Commands the standard screens contribute, beyond the controls: switch the active company, show a named view of the
current screen, move the record to the next or the previous state, set the priority, assign the record to oneself,
schedule an activity, and, when diagnostics are on, the developer tools.

### Keyboard shortcuts

The shortcut modifier is the `alt` key. Holding it alone reveals a small badge on every control that has a shortcut
letter, so the shortcuts of the current screen are discoverable without documentation.

| Shortcut | Control |
|---|---|
| `alt` + `h` | Open the application switcher. |
| `alt` + `b` | Go back one step: the last but one entry of the navigation trail. |
| `alt` + `c` | New record. |
| `alt` + `s` | Save the record. |
| `alt` + `j` | Discard the changes, or close the dialog. |
| `alt` + `n` | Save and start another record; on a pager, the next page. |
| `alt` + `p` | On a pager, the previous page. |
| `alt` + `x` | Remove the current line, or the record from the dialog that owns it. |
| `alt` + `k` | Remove the record shown in a relational dialog. |
| `alt` + `v` | The confirming action of a dialog (export, select) and the scale selector of the calendar. |
| `alt` + `z` | Close a dialog without acting. |
| `alt` + `q` | Confirm a confirmation dialog. |
| `alt` + `u` | Open the action menu of the current screen. |
| `alt` + `shift` + `u` | Open the action menu over the selected records, and open the company selector. |
| `alt` + `shift` + `q` | Open the search options: filters, groupings and favourites. |
| `alt` + `shift` + `v` | Cycle through the views the current action offers. |
| `alt` + `x` / `alt` + `shift` + `x` | Move the record to the next or the previous state of its status bar. |
| `alt` + `r` | Raise the priority of the record. |
| `control` + `k` | Open the command palette. |
| `control` + `enter` | Confirm the dialog, and confirm the company selection. |
| `escape` | Cancel a quick creation, close an overlay, leave an inline edit. |
| arrow keys | Move the focus between cards of the card view. |
| `space`, `shift` + `space` | Select or unselect the focused card, extending the selection with `shift`. |

Three rules govern dispatch:

1. A shortcut is ignored while the focus is in a text field, unless the shortcut declares that it bypasses that
   protection; the command palette shortcut does.
2. Only the controls of the top-most active layer answer. A dialog masks the shortcuts of the screen behind it.
3. A shortcut bound to a control that is hidden or disabled does nothing.

### Notifications

A notification is a transient message shown in a corner of the screen, stacked with the others.

| Property | Values | Meaning |
|---|---|---|
| Severity | success, warning, danger, info | Selects the colour and the icon. |
| Title | text, optional | The heading. |
| Message | text | The body. |
| Buttons | a list, optional | Each with a label, an optional icon and an action; one may be marked as the suggested one. |
| Sticky | true or false, default false | A sticky notification stays until the user dismisses it; a non-sticky one closes itself after 4000 milliseconds. |

Notifications arrive from three sources:

1. **The operation the user just ran.** A business operation may answer with a notification instead of, or in addition
   to, an action; the client shows it and then runs the follow-up action the answer carries. The structure of that
   answer is in [`service-layer.md`](service-layer.md).
2. **The client itself,** when it catches a failure it can express in one sentence, when a save succeeded, or when a
   long download has been prepared.
3. **The server, outside any request the user made.** The server pushes the same structure on the user's own channel and
   the client displays it when it arrives. This is how the result of a background job, an action taken by another user
   on a record the user follows, and an incoming message reach the screen. The transport is specified in
   [`remote-transport-contracts.md`](remote-transport-contracts.md).

The same channel updates the counters of the systray: the number of pending activities grouped by application with the
overdue ones first, and the number of unread conversations.

## Part 4: the form workflow of each major document

Each subsection gives the status bar of the document, the header buttons in the order they appear with the condition
that hides each one, the confirmation text where one is asked, the access group required where one is required, and the
statistic buttons. The business meaning of each operation is specified in the workflow file of the owning domain; the
tables here state what the screen offers and when.

### Sales Order

Status bar: `draft` (Quotation), `sent` (Quotation Sent), `sale` (Sales Order). The cancelled state is shown only when
the record is in it.

| Button | Operation | Hidden when | Notes |
|---|---|---|---|
| Send | `action_quotation_send` | the state is not `draft` | Opens the composer with the quotation template and attaches the printed quotation. |
| Confirm | `action_confirm` | the state is not `draft` | The suggested action on a new quotation. |
| Confirm | `action_confirm` | the state is not `sent` | Same operation offered after the quotation was sent. |
| Send | `action_quotation_send` | the state is neither `sent` nor `sale` | Resends the document, and sends the order confirmation once confirmed. |
| Send Proforma Invoice | `action_quotation_send` | the state is `draft`, or the order already has an invoice | Requires the proforma group. |
| Send Proforma Invoice | `action_quotation_send` | the state is not `draft`, or the order already has an invoice | Requires the proforma group. |
| Print | report of the quotation | the state is `sale` | Prints the quotation form. |
| Create Invoice | invoicing assistant | the invoicing state is not `to invoice` | Opens the down payment and invoicing assistant. |
| Create Invoice | invoicing assistant | the invoicing state is not `no`, or the state is not `sale` | Offered on a confirmed order with nothing left to invoice, for a down payment. |
| Capture Transaction | `payment_action_capture` | the order has no authorized transaction | Captures an authorized online payment. |
| Void Transaction | `payment_action_void` | the order has no authorized transaction | Confirmation: "Are you sure you want to void the authorized transaction? This action can't be undone." |
| Preview | `action_preview_sale_order` | never | Opens the customer portal view of the order. |
| Unlock | `action_unlock` | the order is not locked | Requires the sales manager group. |
| Cancel | `action_cancel` | the state is `cancel`, or the order is locked | Asks for confirmation when invoices exist. |

Statistic buttons: Invoices, hidden when the order has none.

### Journal Entry, customer invoice, vendor bill and credit note

Status bar: `draft`, `posted`; `cancel` is shown only when the record is in it. A secured entry shows the lock marker
beside the state.

| Button | Operation | Hidden when | Group |
|---|---|---|---|
| Post | `action_post` | the posting button is hidden by the rules of the document, or the document is not a plain entry | Invoicing |
| Confirm | `action_post` | the posting button is hidden, or the document is a plain entry, or the document is in a state that forbids posting | Invoicing |
| Send | `action_invoice_sent` | the send button is not displayed, or the highlight rule of the document says otherwise | |
| Print | `action_print_pdf` | the state is not `posted`, or a send is in progress, or the document rules hide it | |
| Pay | `action_register_payment` | the state is not `posted`, or the payment state is not one that accepts a payment | Invoicing |
| Preview | `preview_invoice` | the document is neither a customer invoice nor a customer credit note | |
| Reverse Entry | reversal assistant | the document is not a plain entry, or the state is not `posted` | Invoicing |
| Credit Note | `action_reverse` | the document is neither a customer invoice nor a vendor bill | Invoicing |
| Cancel Entry | `button_cancel` | the record is unsaved, the state is not `draft`, or the document is not a plain entry | Invoicing |
| Cancel | `button_cancel` | the record is unsaved, the state is not `draft`, or the document is a plain entry | Invoicing |
| Request Cancel | `button_request_cancel` | the state is not `posted`, or the reset button is available | Invoicing |
| Reset to Draft | `button_draft` | the reset button is not available (a locked period, a secured chain or a sent electronic document forbid it) | Invoicing |
| Lock | `button_hash` | the journal does not restrict changes, or the entry is already secured | Invoicing |
| Reviewed | `button_set_checked` | the state is not `posted`, or the entry is already marked as reviewed | Accounting user |

Statistic buttons: Payment (the payment the entry belongs to), Payments, Reconciled Items, Cash Basis Entries,
Adjusting Entries and the origin entries of an adjusting entry; each hidden when its count is zero.

### Payment

Status bar: `draft`, `in_process`, `paid`. The rejected and cancelled states are shown when the record is in them.

| Button | Operation | Hidden when | Group |
|---|---|---|---|
| Confirm | `action_post` | the state is not `draft` | |
| Validate | `action_validate` | the state is not `in_process`, or a journal entry already exists | |
| Reject | `action_reject` | the state is not `in_process`, or the payment was not sent | |
| Mark as Sent | `mark_as_sent` | the state is not `in_process`, the payment is already marked as sent, or the payment method does not support it | |
| Unmark as Sent | `unmark_as_sent` | the state is not `in_process`, the payment is not marked as sent, or the payment method does not support it | |
| Request Cancel | `button_request_cancel` | the state is not `in_process`, there is no journal entry, or no cancellation is needed | Invoicing |
| Reset to Draft | `action_draft` | the state is `draft` | Invoicing |
| Cancel | `action_cancel` | the record is unsaved, or the state does not allow cancelling | |

### Purchase Order

Status bar: `draft` (Request for Quotation), `sent` (Request for Quotation Sent), `purchase` (Purchase Order).

| Button | Operation | Hidden when | Group |
|---|---|---|---|
| Send Request for Quotation | `action_rfq_send` | the state is not `draft` | |
| Confirm Order | `button_confirm` | the state is not `draft` | |
| Send Request for Quotation | `action_rfq_send` | the state is not `sent` | |
| Confirm Order | `button_confirm` | the state is not `sent` | |
| Approve Order | `button_approve` | the state is not `to approve` | Purchase manager |
| Send Purchase Order | `action_rfq_send` | the state is not `purchase` | |
| Acknowledge | `action_acknowledge` | the order is already acknowledged, or the state is not `purchase` | |
| Print | quotation report | the state is `purchase` | Any internal user |
| Print | purchase order report | the state is not `purchase` | Any internal user |
| Cancel | `button_cancel` | the state is not one of `draft`, `sent`, `to approve`, `purchase` | |
| Set to Draft | `button_draft` | the state is not `cancel` | |
| Lock | `button_lock` | the order is already locked, the state is not `purchase`, or locking is not configured | |
| Unlock | `button_unlock` | the order is not locked | Purchase manager |

### Transfer

Status bar for an incoming transfer: `draft`, `assigned` (Ready), `done`. For every other operation kind:
`draft`, `confirmed` (Waiting), `assigned` (Ready), `done`. The waiting-another-operation and cancelled states are
shown when the record is in them.

| Button | Operation | Hidden when | Group |
|---|---|---|---|
| Mark as To Do (label reproduced as "Mark as Todo") | `action_confirm` | the state is not `draft` | Any internal user |
| Check Availability | `action_assign` | the availability button is not applicable (nothing is reservable) | Any internal user |
| Validate | `button_validate` | the state is `draft`, `confirmed`, `done` or `cancel` | Inventory user |
| Validate | `button_validate` | the state is `waiting`, `assigned`, `done` or `cancel` | Inventory user |
| Print | `do_print_picking` | the state is not `assigned` | Inventory user |
| Print | delivery slip report | the state is not `done` | Any internal user |
| Return | return assistant | the state is not `done` | Any internal user |
| Cancel | `action_cancel` | the state is not one of `draft`, `confirmed`, `assigned` | Confirmation: "Are you sure you want to cancel this transfer?" |

Statistic buttons: Returns, Scraps, Packages (before completion and after completion, each with its own source),
Traceability (only on a done transfer of tracked products), Allocation, Operations, Moves, and Next Transfer.

### Manufacturing Order

Status bar: `draft`, `confirmed`, `done`. The in-progress, to-close and cancelled states are shown when the record is in
them.

| Button | Operation | Hidden when | Confirmation |
|---|---|---|---|
| Confirm | `action_confirm` | the state is not `draft` | |
| Plan | `button_plan` | the state is not one of `confirmed`, `progress`, `to_close`, or the order is already planned | |
| Unplan | `button_unplan` | the order is not planned, or the state is `done` or `cancel` | |
| Start | `action_start` | the state is not `confirmed` | |
| Check availability | `action_assign` | the state is `draft`, `done` or `cancel`, or nothing is reservable | |
| Unreserve | `do_unreserve` | nothing is reserved | |
| Produce | `button_mark_done` | the order has components and the produce button is not applicable | |
| Produce All | `button_mark_done` | the order has components and the produce-all button is not applicable | |
| Produce | `button_mark_done` | the order has components | "There are no components to consume. Are you sure you want to continue?" |
| Produce All | `button_mark_done` | the order has components | "There are no components to consume. Are you sure you want to continue?" |
| Unbuild | `button_unbuild` | the state is not `done` | |
| Cancel | `action_cancel` | the record is unsaved, or the state is `done` or `cancel` | "Are you sure you want to cancel this manufacturing order?" |

Statistic buttons: Allocation, child orders, source orders, backorders, unbuild orders, scraps, Transfers,
Traceability, Product Moves, Overview and serial numbers; each hidden when its count is zero or the state forbids it.

### Expense

Status bar: `draft`, `approved`, `posted`, `paid`. The submitted and refused states are shown when the record is in
them.

| Button | Operation | Hidden when | Group |
|---|---|---|---|
| Submit | `action_submit` | the state is not `draft` (two variants exist: one for an expense that carries at least one attachment and one for an expense that carries none, in order that the missing receipt is pointed out) | |
| Approve | `action_approve` | the user may not approve, or the state is not `submitted` | |
| Post Journal Entries | `action_post` | the state is not `approved` | Invoicing |
| Refuse | `action_refuse` | the state is neither `submitted` nor `approved` | Expense approver |
| Reset | `action_reset` | the expense may not be reset, or it is already a draft (a second variant applies to users without accounting rights, and also hides the button once the expense is posted or paid) | |
| Split Expense | `action_split_wizard` | the state is not `draft`, or the product carries a fixed cost (a second variant applies to approvers and accountants and hides the button once the expense is in payment, paid, refused or posted) | |

### Time Off Request

Status bar: `confirm` (To Approve), `validate1` (Second Approval), `validate` (Approved). The refused and cancelled
states are shown when the record is in them.

| Button | Operation | Hidden when |
|---|---|---|
| Approve | `action_approve` | the current user may not give the first approval, or the record is unsaved |
| Validate | `action_approve` | the current user may not give the second approval, or the first approval is still pending, or the record is unsaved |
| Back to Approval | `action_back_to_approval` | the request cannot be returned to approval, or the record is unsaved |
| Refuse | `action_refuse` | the current user may not refuse, or the record is unsaved |
| Cancel | `action_cancel` | the request may not be cancelled, or the record is unsaved |

### Repair Order

Status bar: `draft`, `confirmed`, `under_repair`, `done`. The cancelled state is shown when the record is in it.

| Button | Operation | Hidden when | Confirmation |
|---|---|---|---|
| Confirm Repair | `action_validate` | the state is not `draft` | |
| Start Repair | `action_repair_start` | the state is not `confirmed` | |
| End Repair | `action_repair_end` | the state is not `under_repair`, or every part line is complete | "For some of the parts, there is a difference between the initial demand and the actual quantity that was used. Are you sure you want to confirm?" |
| End Repair | `action_repair_end` | the state is not `under_repair`, or a part line is incomplete | |
| Check availability | `action_assign` | nothing is reservable | |
| Unreserve | `action_unreserve` | nothing is reserved | |
| Create Quotation | `action_create_sale_order` | no customer is set, the state is `cancel`, or a quotation already exists | |
| Cancel Repair | `action_repair_cancel` | the state is `done` or `cancel` | |
| Set to Draft | `action_repair_cancel_draft` | the state is not `cancel` | |

### Lead and Opportunity

A lead has no state bar; its progress is the pipeline **stage**, shown as a clickable bar of the stages of its sales
team, plus a separate won or lost marker.

| Button | Operation | Hidden when |
|---|---|---|
| Won | `action_set_won_rainbowman` | the record is already won, it is still a lead rather than an opportunity, or it is archived |
| Convert to Opportunity | conversion assistant | the record is already an opportunity, or it is archived |
| Lost | lost-reason assistant | the record is not pending (it is already won or lost), or it is archived |
| Restore | `action_restore` | the record is not lost |

### Point of Sale Session

Status bar: `opened`, `closing_control` (Closing Control), `closed`.

| Button | Operation | Hidden when |
|---|---|---|
| Continue Selling | `open_frontend_cb` | the session is a rescue session, or its state is neither opening control nor opened |
| Close Session and Post Entries | `action_pos_session_closing_control` | the state is not closing control, unless the session is a rescue session |

### Point of Sale Order

Status bar: `draft` (New), `paid`, `done` (Posted); a cancelled order shows `draft` then `cancel`.

### Batch and wave transfers

Status bar of a batch transfer: `draft`, `confirmed`, `assigned` (Ready), `done`. Status bar of a wave transfer:
`draft`, `in_progress`, `done`.

### Purchase Agreement

Status bar: `draft` (New), `confirmed` (Confirmed), `done` (Closed), hidden entirely when the record is a purchase
template rather than an agreement.

### Survey participation

Status bar: `new`, `in_progress`, `done`.

### Event

An event has no state field on its form: its progress is the event **stage**, shown as a clickable bar of the configured
stages (New, Booked, Announced, Ended, Cancelled by default).

### Event Registration

Status bar: `open` (Registered), `done` (Attended). The draft and cancelled states are shown when the record is in them.

### Stage-based documents in general

Tasks, applicants, leads, events, help requests and every other document whose progress is a configurable pipeline
follow one pattern: the stage bar at the top of the form is clickable, dragging a card between the columns of the card
view writes the same field, and each stage may declare that it folds in the card view, that it marks the record as
closed, and that it triggers an automatic message or an activity. A task additionally carries a personal stage per user,
shown instead of the project stage when the task belongs to no project.

## Part 5: dashboards

Five kinds of dashboard exist.

### The application dashboard of the accounting workspace

Entity: Journal, in card view, filtered to the journals marked for the dashboard. Each card shows one journal with: its
name and type; for a bank journal the balance in the system and the balance of the last imported statement with the
difference; for a sale or purchase journal the number and total of documents to validate, of late documents and of
unpaid documents; a chart of the last months; and a button row (create an invoice, import statements, reconcile,
register a payment) whose entries depend on the journal type and the access rights of the user.

### The application dashboards of the operational applications

| Application | Dashboard | Content |
|---|---|---|
| Inventory | Overview | One card per operation type with the count of transfers to process, late transfers, waiting transfers and back orders, and the buttons to open each of those lists and to create a new transfer of that type. |
| Manufacturing | Overview | The same shape for manufacturing operation types, plus the work centre load when the work centre capability is installed. |
| Point of Sale | Overview | One card per station with its state (closed, opening control, in session, closing control), the cashier of the open session, the session figures, and the buttons to open the station, to close the session and to see the orders. |
| Sales, Purchase, Projects, Recruitment, Events | Team or pipeline cards | One card per team, project, job position or event with its counters (open documents, overdue documents, forecast amounts), a progress indicator and the buttons to open the filtered lists. |
| Site | Site dashboard | Visits, conversions, sales, the most viewed pages and the products most added to carts, over a chosen period. |
| Live conversations | Live chat dashboard | Sessions, messages, rating and response time per operator and per channel, over a chosen period. |

### The personal dashboard

A user may add any multi-record view to a personal dashboard through the search menu: the view, its facets, its
groupings and its ordering are stored as a block with a name. The personal dashboard shows the blocks in a column
layout; each block may be renamed, moved, folded and removed, and it re-runs its stored view every time it is displayed.

### Spreadsheet dashboards

A spreadsheet dashboard is a workbook laid out as a dashboard and grouped under a theme. The dashboard screen lists the
themes on the left and renders the chosen dashboard read-only on the right, refreshing its live data at each opening.
A dashboard may be shared by link: the link carries a token, freezes the data at the moment of sharing, and opens a
read-only page for any visitor holding it, with a download action that returns the workbook.

### Reporting views used as dashboards

Every analysis entity (sales analysis, invoice analysis, inventory valuation, manufacturing performance, project
profitability, time off summary, recruitment funnel, event revenue, marketing statistics) is a read-only entity shown as
pivot and chart views with a rich search view. They are reached from the Reporting menu of their application, they honour
the same facets, groupings and favourites as any other view, and they may be inserted into a spreadsheet or added to the
personal dashboard.

## Part 6: rules and acceptance criteria

| Rule | Statement |
|---|---|
| DSK-RULE-001 | A menu entry whose action the user may not run, and which has no visible child, is not shown. |
| DSK-RULE-002 | Opening an action from a menu clears the navigation stack; opening a record or a related record pushes onto it. |
| DSK-RULE-003 | The default filters, groupings and new-record values of an action are applied every time the action is opened from a menu, and a user default favourite replaces the default filters. |
| DSK-RULE-004 | A form with unsaved changes asks to save or discard before leaving, before switching view and before running a button that does not save on its own. |
| DSK-RULE-005 | A header button is shown only when its visibility condition holds and the user belongs to the access groups it declares; a hidden button cannot be run by any other means from the screen. |
| DSK-RULE-006 | A button that declares a confirmation text runs only after the user confirms that exact text. |
| DSK-RULE-007 | Archiving hides records from every default list; the archived filter and the archived state of a relation field are the only ways to see them. |
| DSK-RULE-008 | Duplicating a record produces an unsaved copy in which fields declared as not copyable are reset. |
| DSK-RULE-009 | The active company selection filters every list and defaults every new record; a record of a company the user has not activated is read-only when visible at all. |
| DSK-RULE-010 | A favourite marked as default applies to its action for its owner only; a shared favourite is offered to every user but is not applied automatically. |
| DSK-RULE-011 | A shortcut is dispatched only to the top-most active layer, and only when the focus is not in a text field, unless the shortcut declares that it bypasses that protection. |
| DSK-RULE-012 | A non-sticky notification closes itself after 4000 milliseconds; a sticky one stays until the user dismisses it. |
| DSK-RULE-013 | A command palette result performs exactly the operation of the control it stands for, and never one the user could not reach on the screen. |
| DSK-RULE-014 | A duplicated menu entry that opens the same action as an entry of another application is a separate entry: opening it starts a new navigation stack rooted at that entry. |

### Acceptance criteria

**AC-DESKTOP-001 — A hidden application is unreachable.** Given a user who belongs to no accounting group, when the user
opens the application switcher, then the invoicing application is not listed, and opening its address directly answers
with the access refusal.

**AC-DESKTOP-002 — The status bar and the header of a quotation.** Given a sales order in the quotation state, when the
user opens its form, then the status bar shows Quotation, Quotation Sent and Sales Order with Quotation highlighted, the
header offers Send, Confirm, Print, Preview and Cancel, and it does not offer Create Invoice.

**AC-DESKTOP-003 — A refused operation leaves the record where it was.** Given a transfer in the ready state whose
products are tracked by lot, when the user presses Validate without filling the lot numbers, then the operation is
refused with the message of the inventory domain and the transfer stays in the ready state.

**AC-DESKTOP-004 — A view added to the personal dashboard.** Given a list of invoices grouped by customer, when the user
adds the view to the personal dashboard under the name "Open invoices", then a block named "Open invoices" appears on the
personal dashboard and opening the dashboard re-runs the same filter, grouping and ordering.

**AC-DESKTOP-005 — The active company defaults a new record.** Given a user with two allowed companies, both active,
when the user creates a quotation, then the company of the quotation defaults to the first active company and the price
list, the sales team and the journals proposed are those of that company.

**AC-DESKTOP-006 — Unsaved changes are guarded.** Given a form with one changed field, when the user presses a menu
entry that opens another action, then the client asks to save or discard, and choosing discard reopens the record with
its stored values.

**AC-DESKTOP-007 — Batch action over a whole filter.** Given a list of 500 matching records with a page size of 80, when
the user selects the page and then chooses to select every matching record, then an action run from the action menu
applies to all 500, and the client passes the filter rather than the 500 keys once the active-keys limit is exceeded.

**AC-DESKTOP-008 — A favourite marked as default.** Given a user who saves the current facets as a favourite and marks
it as the default of the action, when the user opens that action from a menu the next day, then the favourite's facets,
groupings and ordering are applied instead of the action's own default filters, and another user opening the same action
sees the action's default filters.

**AC-DESKTOP-009 — The command palette finds a deep menu entry.** Given the menu entry Invoicing, Configuration,
Accounting, Chart of Accounts, when the user opens the palette, types `/` and then the word `chart`, then the entry is
offered with its full path, and choosing it opens the chart of accounts with the navigation stack rooted at that entry.

**AC-DESKTOP-010 — The shortcut overlay.** Given a form in edit, when the user holds the shortcut modifier, then a badge
appears on Save, Discard and every other control that declares a letter, and releasing the modifier removes them.

**AC-DESKTOP-011 — A pushed notification.** Given a user whose screen is a list, when another user assigns a record to
them, then a notification appears without the list being reloaded, and the activity counter of the systray increases by
one.

**AC-DESKTOP-012 — Grouped list paging.** Given a list grouped by customer where one group holds 200 records and the
per-group page size is 80, when the user opens that group, then 80 rows are loaded, the group header shows the count 200
and the aggregates of all 200, and paging inside the group loads the next 80.

## Reconciliation notes

The two drafts merged into this document disagreed with each other and, in places, with the menu tree the system
actually declares. Each point was settled against the source of the system and against the generated catalogues
[`../references/actions-and-menus.md`](../references/actions-and-menus.md) and
[`../references/views.md`](../references/views.md); the resolutions are recorded here.

1. **Applications that do not exist.** One draft carried twelve extra sections whose headings were derived from the
   technical identifier of a menu rather than from an application: an "administration" section identical to the settings
   section, an "email" section that is the technical mail branch of the settings tree, a French statement section, six
   event sections, an application-management section, and three sections derived from the sales, purchasing and contact
   configuration branches. Every one of them repeats entries that already belong to a real application. They were
   removed, and the five event configuration entries and the event list entry they carried were moved into the events
   application, where the menu tree puts them.
2. **Duplicated rows.** The inventory table repeated its whole configuration block (sixteen rows) and the employee table
   repeated two learning rows. The repetitions were removed.
3. **Missing entries.** Compared with the menu tree, the sales application was missing twelve configuration entries
   (product attributes, categories, combination choices, product tags, units and packagings, order tags, the two
   activity entries and the four online payment entries), the invoicing application was missing nineteen entries (the
   journal entry and analytic item lists, the journal item review list, the chart of accounts, taxes, tax groups,
   journals, the multi-ledger, fiscal positions, currencies, cash roundings, payment terms, international commercial
   terms, product categories, the three analytic configuration entries and the two management reports), and the
   timesheet application was missing three reporting entries. All were added.
4. **Counts.** The stated totals (858 entries, 765 listed) matched neither the menu tree nor the tables. The menu tree
   declares 893 entries, 645 of which open an action; the tables now list 630 of them and the introduction states
   exactly which fifteen are left out and why.
5. **Mis-expanded abbreviations.** One draft expanded the abbreviation in the Egyptian electronic invoicing
   configuration branch as "Estimated Time of Arrival"; it stands for the Egyptian Tax Authority, and the branch is now
   named after it.
6. **Labels that carry abbreviations.** Two menu labels are reproduced by the system with abbreviations in them. The
   animated image entries of the technical messaging branch are displayed as "GIF favorite", and the page editor entry
   of the site menu is displayed as "HTML / CSS Editor". The tables use the abbreviation-free wording ("Favourite
   animated images", "Page source editor"); the displayed labels are reproduced here so that a rebuild can show the same
   text. The confirmation button of a transfer is displayed as "Mark as Todo" and the table states both forms.
7. **Acceptance criteria.** One draft ended with five unnumbered scenarios in plain blocks. They are now numbered
   scenarios with stable identifiers, and seven further scenarios were added to cover the guard on unsaved changes,
   batch actions over a whole filter, default favourites, the command palette, the shortcut overlay, pushed
   notifications and grouped paging.
8. **Command palette, keyboard shortcuts and notifications.** Neither draft specified them beyond a single line in the
   frame table. They are now specified in full in part 3, because they are the only way several operations can be
   reached and because the shortcut letters are observable behaviour.
